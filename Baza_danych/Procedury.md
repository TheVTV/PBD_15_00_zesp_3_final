## Procedury

### Pomyślna opłata części koszyka (AcceptedPartialPaymentForTheCart)

```sql
CREATE PROCEDURE AcceptedPartialPaymentForTheCart
    @CartID INT,
	@AmountPaid SMALLMONEY
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;

    BEGIN TRY
        BEGIN TRANSACTION;

		IF NOT EXISTS(SELECT CartID FROM Cart WHERE CartID=@CartID)
		BEGIN
            THROW 51000, 'Cart does not exist.', 1;
        END

        -- Create payment record
        INSERT INTO Payments (Value, CartID, IsSuccessful, CreatedAt)
        VALUES (@AmountPaid, @CartID, 1, GETDATE());

        -- Execute additional payment logic
        EXEC dbo.PayPartialForCart @CartID = @CartID, @AmountPaid = @AmountPaid;

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END;
```

### Pomyślna opłata koszyka (AcceptedPaymentForTheCart)

```sql
CREATE PROCEDURE AcceptedPaymentForTheCart
    @CartID INT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;

    DECLARE @PaymentValue SMALLMONEY;

    BEGIN TRY
        BEGIN TRANSACTION;

        -- Verify cart exists and lock it
        SELECT @PaymentValue = dbo.GetRemainingPaymentForCart(@CartID)  -- Assuming Paid column stores cart total
        FROM Cart WITH (UPDLOCK)
        WHERE CartID = @CartID;

		IF NOT EXISTS(SELECT CartID FROM Cart WHERE CartID=@CartID)
		BEGIN
            THROW 51000, 'Cart does not exist.', 1;
        END

        -- Create payment record
        INSERT INTO Payments (Value, CartID, IsSuccessful, CreatedAt)
        VALUES (@PaymentValue, @CartID, 1, GETDATE());

        -- Execute additional payment logic
        EXEC dbo.PayForWholeCart @CartID = @CartID;

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END;
```

### Dodaj kurs do koszyka (AddCourseToCart)

```sql
CREATE PROCEDURE AddCourseToCart
    @StudentID INT,
    @CourseID INT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON; -- Auto-rollback on errors

    DECLARE @CartID INT,
            @CourseStartDate DATETIME,
            @Vacancies INT;

    BEGIN TRANSACTION;

    -- Check course existence and get start date with lock
    SELECT @CourseStartDate = DATEADD(DAY, -3, dbo.GetEarliestModuleDate(@CourseID))
    FROM Courses WITH (UPDLOCK)
    WHERE CourseID = @CourseID;

    IF @CourseStartDate IS NULL
    BEGIN
        ROLLBACK;
        THROW 51000, 'Invalid CourseID provided.', 1;
    END;

    -- Check course vacancies
    SET @Vacancies = dbo.HowManyCourseVacancies(@CourseID);

    IF @Vacancies <= 0
    BEGIN
        ROLLBACK;
        THROW 51001, 'No available vacancies for this course.', 1;
    END;

    -- Insert new cart (CartID is auto-generated)
    INSERT INTO Cart (StudentID, Paid, PaidDate, DueDate, Access)
    VALUES (@StudentID, 0, NULL, @CourseStartDate, 0);

    -- Get auto-generated CartID
    SET @CartID = SCOPE_IDENTITY();

    -- Add course to cart
    INSERT INTO CartCourses (CartID, CoursesID)
    VALUES (@CartID, @CourseID);

    COMMIT TRANSACTION;
END;
```

### Dodaj nieobecność na stażu (AddInternshipAbsence)

```sql
CREATE PROCEDURE AddInternshipAbsence
    @InternshipID INT,
    @StudentIDList NVARCHAR(MAX), -- Lista ID studentów w formacie np. '1,2,3'
    @Date DATETIME
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        -- Rozdzielenie listy StudentID na tabelę
        DECLARE @StudentIDs TABLE (StudentID INT);
        DECLARE @XML XML = CAST('<i>' + REPLACE(@StudentIDList, ',', '</i><i>') + '</i>' AS XML);

        INSERT INTO @StudentIDs (StudentID)
        SELECT T.c.value('.', 'INT')
        FROM @XML.nodes('/i') T(c);

        -- Sprawdzenie, czy staż istnieje
        IF NOT EXISTS (
            SELECT 1
            FROM [dbo].[Internships]
            WHERE [InternshipID] = @InternshipID
        )
        BEGIN
            THROW 60001, 'Nie znaleziono stażu o podanym ID.', 1;
        END

        -- Iteracja przez studentów i dodawanie indywidualnie
        DECLARE @StudentID INT;

        DECLARE student_cursor CURSOR FOR
        SELECT StudentID FROM @StudentIDs;

        OPEN student_cursor;

        FETCH NEXT FROM student_cursor INTO @StudentID;

        WHILE @@FETCH_STATUS = 0
        BEGIN
            -- Sprawdzenie dostępu studenta do kursu
            IF NOT EXISTS (
                SELECT 1
                FROM [dbo].[Cart] c
                INNER JOIN CartStudies cs on c.CartID = cs.CartID
                INNER JOIN Studies s on s.StudiesID = cs.StudiesID
                INNER JOIN Internships i on i.StudiesID = s.StudiesID
                WHERE StudentID = @StudentID AND InternshipID = @InternshipID AND Access = 1
            )
            BEGIN
                -- Rzucenie błędu, jeśli student nie jest przypisany do kursu
                DECLARE @ErrorMessage NVARCHAR(200);
                SET @ErrorMessage = 'StudentID ' + CAST(@StudentID AS NVARCHAR) + ' nie jest przypisany do tego kursu.';
                THROW 60002, @ErrorMessage, 1;
            END

            -- Dodanie nieobecności, jeśli nie istnieje
            IF NOT EXISTS (
                SELECT 1
                FROM [dbo].[InternshipAbsence]
                WHERE InternshipID = @InternshipID AND StudentID = @StudentID AND Date = @Date
            )
            BEGIN
                INSERT INTO [dbo].[InternshipAbsence] (InternshipID, StudentID, Date)
                VALUES (@InternshipID, @StudentID, @Date);
            END

            FETCH NEXT FROM student_cursor INTO @StudentID;
        END

        CLOSE student_cursor;
        DEALLOCATE student_cursor;

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
		IF @@TRANCOUNT > 0
		BEGIN
			ROLLBACK TRANSACTION
		END;
        THROW;
    END CATCH
END;
```

### Dodaj obecność na zajęciach (AddLessonPresence)

```sql
CREATE PROCEDURE AddLessonPresence
    @LessonID INT,
    @StudentIDList NVARCHAR(MAX) -- Lista ID studentów w formacie np. '1,2,3'
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        -- Rozdzielenie listy StudentID na tabelę
        DECLARE @StudentIDs TABLE (StudentID INT);
        DECLARE @XML XML = CAST('<i>' + REPLACE(@StudentIDList, ',', '</i><i>') + '</i>' AS XML);

        INSERT INTO @StudentIDs (StudentID)
        SELECT T.c.value('.', 'INT')
        FROM @XML.nodes('/i') T(c);

        -- Sprawdzenie, czy lekcja istnieje
        IF NOT EXISTS (
            SELECT 1
            FROM LessonsOnStudies
            WHERE LessonID = @LessonID
        )
        BEGIN
            THROW 60001, 'Nie znaleziono lekcji o podanym ID.', 1;
        END

		-- Iteracja przez studentów i dodawanie indywidualnie
        DECLARE @StudentID INT;

        DECLARE student_cursor CURSOR FOR
        SELECT StudentID FROM @StudentIDs;

        OPEN student_cursor;

        FETCH NEXT FROM student_cursor INTO @StudentID;

        WHILE @@FETCH_STATUS = 0
        BEGIN
			-- Sprawdzenie dostępu studenta do kursu
            IF NOT EXISTS (
                SELECT 1
                FROM [dbo].[Cart] c
                INNER JOIN CartStudies cs on c.CartID = cs.CartID
                INNER JOIN Studies s on s.StudiesID = cs.StudiesID
                INNER JOIN Conferrence i on i.StudiesID = s.StudiesID
				INNER JOIN LessonsOnStudies l on l.ConferrenceID = i.ConferrenceID
                WHERE StudentID = @StudentID AND LessonID = @LessonID AND Access = 1

				UNION

				SELECT 1
                FROM [dbo].[Cart] c
                INNER JOIN CartSingleLessons cs on c.CartID = cs.CartID
                INNER JOIN LessonsOnStudies s on s.LessonID = cs.LessonID
                WHERE StudentID = @StudentID AND s.LessonID = @LessonID AND Access = 1
            )
            BEGIN
                -- Rzucenie błędu, jeśli student nie jest przypisany do kursu
                DECLARE @ErrorMessage NVARCHAR(200);
                SET @ErrorMessage = 'StudentID ' + CAST(@StudentID AS NVARCHAR) + ' nie jest przypisany do tego kursu.';
                THROW 60002, @ErrorMessage, 1;
            END

			-- Dodanie obecności, jeśli nie istnieje
            IF NOT EXISTS (
                SELECT 1
                FROM [dbo].StudyLessonPresence
                WHERE StudentID = @StudentID AND LessonID = @LessonID
            )
            BEGIN
                INSERT INTO [dbo].StudyLessonPresence (LessonID, StudentID, Presence)
                VALUES (@LessonID, @StudentID, 1);
            END
			ELSE
			BEGIN
				DECLARE @ErrorMessage2 NVARCHAR(200);
                SET @ErrorMessage2 = 'StudentID ' + CAST(@StudentID AS NVARCHAR) + ' ma już obecność.';
                THROW 60002, @ErrorMessage2, 1;
			END

			FETCH NEXT FROM student_cursor INTO @StudentID;
        END

		CLOSE student_cursor;
        DEALLOCATE student_cursor;

			-- Ustaw obecność (Presence = 0) dla studentów, którzy mają kurs w koszyku i Access = 1, ale nie są na liście
        INSERT INTO StudyLessonPresence (LessonID, StudentID, Presence)
        SELECT DISTINCT @LessonID, st.StudentID, 0
                FROM [dbo].Students st
				INNER JOIN Cart c on st.StudentID = c.StudentID
                INNER JOIN CartStudies cs on c.CartID = cs.CartID
                INNER JOIN Studies s on s.StudiesID = cs.StudiesID
                INNER JOIN Conferrence i on i.StudiesID = s.StudiesID
				INNER JOIN LessonsOnStudies l on l.ConferrenceID = i.ConferrenceID
                WHERE st.StudentID NOT IN (SELECT StudentID FROM @StudentIDs) AND LessonID = @LessonID AND Access = 1
				AND NOT EXISTS (
              SELECT 1
              FROM  StudyLessonPresence slp
              WHERE slp.LessonID = @LessonID AND slp.StudentID = st.StudentID
          );



		INSERT INTO StudyLessonPresence (LessonID, StudentID, Presence)
        SELECT DISTINCT @LessonID, st.StudentID, 0
				FROM Students st
                INNER JOIN Cart c on c.StudentID = st.StudentID
                INNER JOIN CartSingleLessons cs on c.CartID = cs.CartID
                INNER JOIN LessonsOnStudies s on s.LessonID = cs.LessonID
				WHERE st.StudentID NOT IN (SELECT StudentID FROM @StudentIDs) AND s.LessonID = @LessonID AND Access = 1
          AND NOT EXISTS (
              SELECT 1
              FROM StudyLessonPresence slp
              WHERE slp.LessonID = @LessonID AND slp.StudentID = st.StudentID
          );

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
		IF @@TRANCOUNT > 0
		BEGIN
			ROLLBACK TRANSACTION;
		END;
        -- Rzucenie błędu z informacją


        THROW;
    END CATCH
END;
```

### Dodaj lekcję do koszyka (AddLessonToCart)

```sql
CREATE PROCEDURE AddLessonToCart
    @StudentID INT,
    @LessonID INT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON; -- Auto-rollback on errors

    DECLARE @CartID INT,
            @LessonDate DATETIME,
            @Vacancies INT;

    BEGIN TRANSACTION;

    -- Check lesson existence with lock
    SELECT @LessonDate = Date
    FROM LessonsOnStudies WITH (UPDLOCK)
    WHERE LessonID = @LessonID;

    IF @LessonDate IS NULL
    BEGIN
        ROLLBACK;
        THROW 51000, 'Invalid LessonID provided.', 1;
    END;

    -- Check vacancies using the lesson vacancy function
    SET @Vacancies = dbo.HowManyLessonVacancies(@LessonID);

    IF @Vacancies <= 0
    BEGIN
        ROLLBACK;
        THROW 51001, 'No available vacancies for this lesson.', 1;
    END;

    -- Insert new cart (CartID is auto-generated)
    INSERT INTO Cart (StudentID, Paid, PaidDate, DueDate, Access)
    VALUES (@StudentID, 0, NULL, @LessonDate, 0);

    -- Get the auto-generated CartID
    SET @CartID = SCOPE_IDENTITY();

    -- Add lesson to cart
    INSERT INTO CartSingleLessons (CartID, LessonID)
    VALUES (@CartID, @LessonID);

    COMMIT TRANSACTION;
END;
```

### Dodaj obecność na module (AddModulePresence)

```sql
CREATE PROCEDURE AddModulePresence
    @ModuleID INT,
    @StudentIDList NVARCHAR(MAX) -- Lista ID studentów w formacie np. '1,2,3'
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        -- Rozdzielenie listy StudentID na tabelę
        DECLARE @StudentIDs TABLE (StudentID INT);
        DECLARE @XML XML = CAST('<i>' + REPLACE(@StudentIDList, ',', '</i><i>') + '</i>' AS XML);

        INSERT INTO @StudentIDs (StudentID)
        SELECT T.c.value('.', 'INT')
        FROM @XML.nodes('/i') T(c);

        -- Pobierz CourseID dla danego ModuleID
        DECLARE @CourseID INT;
        SELECT @CourseID = CourseID
        FROM CourseModules
        WHERE ModuleID = @ModuleID;

        IF @CourseID IS NULL
        BEGIN
            THROW 50001, 'Nie znaleziono kursu dla podanego ModuleID.', 1;
        END


		-- Iteracja przez studentów i dodawanie indywidualnie
        DECLARE @StudentID INT;

        DECLARE student_cursor CURSOR FOR
        SELECT StudentID FROM @StudentIDs;

        OPEN student_cursor;

        FETCH NEXT FROM student_cursor INTO @StudentID;

        WHILE @@FETCH_STATUS = 0
        BEGIN
			-- Sprawdzenie dostępu studenta do kursu
            IF NOT EXISTS (
                SELECT 1
                FROM [dbo].[Cart] c
                INNER JOIN CartCourses cs on c.CartID = cs.CartID
                INNER JOIN Courses s on s.CourseID = cs.CoursesID
                INNER JOIN CourseModules i on i.CourseID = s.CourseID
                WHERE StudentID = @StudentID AND ModuleID = @ModuleID AND Access = 1
            )
            BEGIN
                -- Rzucenie błędu, jeśli student nie jest przypisany do kursu
                DECLARE @ErrorMessage NVARCHAR(200);
                SET @ErrorMessage = 'StudentID ' + CAST(@StudentID AS NVARCHAR) + ' nie jest przypisany do tego kursu.';
                THROW 60002, @ErrorMessage, 1;
            END

			-- Dodanie nieobecności, jeśli nie istnieje
            IF NOT EXISTS (
                SELECT 1
                FROM [dbo].ModulesPresence
                WHERE ModuleID = @ModuleID AND StudentID = @StudentID
            )
            BEGIN
                INSERT INTO [dbo].ModulesPresence (ModuleID, StudentID, Presence)
                VALUES (@ModuleID, @StudentID, 1);
            END
			ELSE
			BEGIN
				DECLARE @ErrorMessage2 NVARCHAR(200);
                SET @ErrorMessage2 = 'StudentID ' + CAST(@StudentID AS NVARCHAR) + ' ma już obecność.';
                THROW 60002, @ErrorMessage2, 1;
			END

            FETCH NEXT FROM student_cursor INTO @StudentID;
        END

		CLOSE student_cursor;
        DEALLOCATE student_cursor;

        -- Ustaw obecność (Presence = 0) dla studentów, którzy mają kurs w koszyku i Access = 1, ale nie są na liście
        INSERT INTO ModulesPresence (ModuleID, StudentID, Presence)
        SELECT DISTINCT @ModuleID, s.StudentID, 0
        FROM Students s
        INNER JOIN Cart c ON c.StudentID = s.StudentID
        INNER JOIN CartCourses cc ON cc.CartID = c.CartID AND cc.CoursesID = @CourseID
        WHERE s.StudentID NOT IN (SELECT StudentID FROM @StudentIDs)
          AND c.Access = 1
          AND NOT EXISTS (
              SELECT 1
              FROM ModulesPresence mp
              WHERE mp.ModuleID = @ModuleID AND mp.StudentID = s.StudentID
          );

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0
		BEGIN
			ROLLBACK TRANSACTION;
		END;
        -- Rzucenie błędu z informacją
        THROW;
    END CATCH
END;
```

### Dodaj studia do koszyka (AddStudiesToCart)

```sql
CREATE PROCEDURE AddStudiesToCart
    @StudentID INT,
    @StudiesID INT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON; -- Auto-rollback on errors

    DECLARE @Vacancies INT,
            @CartID INT,
            @StudiesStartDate DATETIME;

    BEGIN TRANSACTION;

    -- Check study existence with lock
    SELECT @StudiesStartDate = DATEADD(DAY, -3, Date)
    FROM Studies WITH (UPDLOCK)
    WHERE StudiesID = @StudiesID;

    IF @StudiesStartDate IS NULL
    BEGIN
        ROLLBACK;
        THROW 51000, 'Invalid StudiesID provided.', 1;
    END;

    -- Check vacancies (uses UPDLOCK in function)
    SET @Vacancies = dbo.HowManyStudyVacancies(@StudiesID);

    IF @Vacancies <= 0
    BEGIN
        ROLLBACK;
        THROW 51001, 'No available vacancies for these studies.', 1;
    END;

    -- Insert new cart (CartID is auto-generated)
    INSERT INTO Cart (StudentID, Paid, PaidDate, DueDate, Access)
    VALUES (@StudentID, 0, NULL, @StudiesStartDate, 0);

    -- Get the auto-generated CartID
    SET @CartID = SCOPE_IDENTITY();

    -- Link studies to cart
    INSERT INTO CartStudies (CartID, StudiesID)
    VALUES (@CartID, @StudiesID);

    COMMIT TRANSACTION;
END;
```

### Dodaj nowego tłumacza wraz z jego władanymi językami (AddTranslatorWithLanguages)

```sql
CREATE PROCEDURE AddTranslatorWithLanguages
    @EmployeeID INT,
    @LanguageIDs NVARCHAR(MAX) -- Lista języków w formacie: '1,2,3'
AS
BEGIN
    BEGIN TRY
        BEGIN TRANSACTION;

        -- Znalezienie pierwszego wolnego TranslatorID
        DECLARE @TranslatorID INT;
        SELECT @TranslatorID = ISNULL(MAX(TranslatorID), 0) + 1 FROM Translators;

        -- Dodanie nowego tłumacza do tabeli Translators
        INSERT INTO Translators (TranslatorID, EmployeeID)
        VALUES (@TranslatorID, @EmployeeID);

        -- Iteracja po ID języków i dodanie wpisów do TranslatorLanguages
        DECLARE @LanguageID INT;
        DECLARE @LanguageTable TABLE (ID INT);

        -- Rozdzielenie wartości z listy języków (np. '1,2,3') i zapisanie w tabeli tymczasowej
        INSERT INTO @LanguageTable (ID)
        SELECT value
        FROM STRING_SPLIT(@LanguageIDs, ',');

        -- Iteracja po językach i dodanie do TranslatorLanguages
        DECLARE cur CURSOR FOR
        SELECT ID FROM @LanguageTable;

        OPEN cur;
        FETCH NEXT FROM cur INTO @LanguageID;

        WHILE @@FETCH_STATUS = 0
        BEGIN
            INSERT INTO TranslatorLanguages (TranslatorID, LanguageID)
            VALUES (@TranslatorID, @LanguageID);

            FETCH NEXT FROM cur INTO @LanguageID;
        END;

        CLOSE cur;
        DEALLOCATE cur;

        -- Zatwierdzenie transakcji
        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        -- W przypadku błędu cofnięcie transakcji
        ROLLBACK TRANSACTION;

        -- Rzucenie komunikatu błędu
        THROW;
    END CATCH
END;
```

### Dodaj webinar do koszyka (AddWebinarToCart)

```sql
CREATE PROCEDURE AddWebinarToCart
    @StudentID INT,
    @WebinarID INT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON; -- Auto-rollback on errors

    DECLARE @CartID INT,
            @WebinarPrice SMALLMONEY,
            @Vacancies INT;

    BEGIN TRANSACTION;

    -- Check webinar existence and get price with lock
    SELECT @WebinarPrice = Price
    FROM Webinars WITH (UPDLOCK)
    WHERE WebinarID = @WebinarID;

    IF @WebinarPrice IS NULL
    BEGIN
        ROLLBACK;
        THROW 51000, 'Invalid WebinarID provided.', 1;
    END;

    -- Insert new cart (CartID is auto-generated)
    INSERT INTO Cart (StudentID, Paid, PaidDate, DueDate, Access)
    VALUES (
        @StudentID,
        0,
        NULL,  -- Changed from 0 to NULL for proper datetime handling
        DATEADD(DAY, 7, GETDATE()),
        CASE WHEN @WebinarPrice = 0 THEN 1 ELSE 0 END
    );

    -- Get auto-generated CartID
    SET @CartID = SCOPE_IDENTITY();

    -- Add webinar to cart
    INSERT INTO CartWebinars (CartID, WebinarID)
    VALUES (@CartID, @WebinarID);

    COMMIT TRANSACTION;
END;
```

### Niepomyślna opłata części koszyka (FailedPartialPaymentForTheCart)

```sql
CREATE PROCEDURE FailedPartialPaymentForTheCart
    @CartID INT,
	@AmountPaid SMALLMONEY
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;

    BEGIN TRY
        BEGIN TRANSACTION;

		IF NOT EXISTS(SELECT CartID FROM Cart WHERE CartID=@CartID)
		BEGIN
            THROW 51000, 'Cart does not exist.', 1;
        END

        -- Create payment record
        INSERT INTO Payments (Value, CartID, IsSuccessful, CreatedAt)
        VALUES (@AmountPaid, @CartID, 0, GETDATE());


        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END;
```

### Niepomyślna opłata koszyka (FailedPaymentForTheCart)

```sql
CREATE PROCEDURE FailedPaymentForTheCart
    @CartID INT
AS
BEGIN
    SET NOCOUNT ON;
    SET XACT_ABORT ON;

    DECLARE @PaymentValue SMALLMONEY;

    BEGIN TRY
        BEGIN TRANSACTION;

        -- Verify cart exists and lock it
        SELECT @PaymentValue = dbo.GetRemainingPaymentForCart(@CartID)  -- Assuming Paid column stores cart total
        FROM Cart WITH (UPDLOCK)
        WHERE CartID = @CartID;

		IF NOT EXISTS(SELECT CartID FROM Cart WHERE CartID=@CartID)
		BEGIN
            THROW 51000, 'Cart does not exist.', 1;
        END

        -- Create payment record
        INSERT INTO Payments (Value, CartID, IsSuccessful, CreatedAt)
        VALUES (@PaymentValue, @CartID, 0, GETDATE());

        COMMIT TRANSACTION;
    END TRY
    BEGIN CATCH
        IF @@TRANCOUNT > 0 ROLLBACK TRANSACTION;
        THROW;
    END CATCH
END;
```

### Zapłać za cały koszyk (PayForWholeCart)

```sql
CREATE PROCEDURE PayForWholeCart
    @CartID INT
AS
BEGIN
    DECLARE @TotalValue SMALLMONEY;

    -- Pobierz pełną wartość koszyka
    SELECT @TotalValue = dbo.GetRemainingPaymentForCart(@CartID);

    -- Aktualizuj tabelę Cart: oznacz jako opłacone
    UPDATE Cart
    SET Paid = Paid + @TotalValue,
        PaidDate = GETDATE()
    WHERE CartID = @CartID;
END;
```

### Zapłać częściowo za koszyk (PayPartialForCart)

```sql
CREATE PROCEDURE PayPartialForCart
    @CartID INT,
    @AmountPaid SMALLMONEY
AS
BEGIN
    DECLARE @RemainingPayment SMALLMONEY;

    -- Pobierz pozostałą kwotę do zapłaty
    SELECT @RemainingPayment = dbo.GetRemainingPaymentForCart(@CartID);

    -- Sprawdź, czy podana kwota nie przekracza pozostałej
    IF @AmountPaid > @RemainingPayment
    BEGIN
        SELECT @AmountPaid = @RemainingPayment
    END

    -- Dodaj do Paid nową płatność
    UPDATE Cart
    SET Paid = ISNULL(Paid, 0) + @AmountPaid,
        PaidDate = CASE
                     WHEN ISNULL(Paid, 0) + @AmountPaid = dbo.GetCartValue(@CartID) THEN GETDATE()
                     ELSE PaidDate
                   END
    WHERE CartID = @CartID;
END;
```
