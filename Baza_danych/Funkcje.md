## Funkcje

### Szczegóły koszyka dla danego studenta (GetCartDetailsForStudent)

```sql
CREATE FUNCTION GetCartDetailsForStudent (@StudentID INT)
RETURNS TABLE
AS
RETURN
(
    SELECT
        c.CartID,
        c.Paid,
        c.PaidDate,
        c.DueDate,
        c.Access,
        cc.CountCourses AS CourseID,
        cs.CountStudies AS StudiesID,
        cw.CountWebinars AS WebinarID,
		cls.CountLessons AS SingleLessonID,
		dbo.GetCartValue(c.CartID) as CartValue,
		ISNULL(ISNULL(ISNULL(cc.Name, cs.Name), cw.Name), cls.Name) AS NameOfEvent,
		ISNULL(ISNULL(ISNULL(cc.Date, cs.Date), cw.Date), cls.Date) AS DateOfEvent
    FROM
        Cart c
    LEFT JOIN
        (SELECT CartID, CoursesID AS CountCourses, Name, dbo.GetEarliestModuleDate(CoursesID) AS Date FROM CartCourses cc
		INNER JOIN Courses c ON cc.CoursesID = c.CourseID
		GROUP BY CartID, CoursesID, Name) cc
        ON c.CartID = cc.CartID
    LEFT JOIN
        (SELECT CartID, s.StudiesID AS CountStudies, Name, Date FROM CartStudies cs
		INNER JOIN Studies s ON cs.StudiesID = s.StudiesID
		GROUP BY CartID, s.StudiesID, Name, Date) cs
        ON c.CartID = cs.CartID
    LEFT JOIN
        (SELECT CartID, w.WebinarID AS CountWebinars, Name, Date FROM CartWebinars cw
		INNER JOIN Webinars w ON w.WebinarID = cw.WebinarID
		GROUP BY CartID, w.WebinarID, Name, Date) cw
        ON c.CartID = cw.CartID
	LEFT JOIN
		(SELECT CartID, csl.LessonID AS CountLessons, CONCAT( 'Single lesson in ', s.Name ) as Name, los.Date FROM CartSingleLessons csl
		INNER JOIN LessonsOnStudies los ON csl.LessonID = los.LessonID
		INNER JOIN Conferrence cf ON los.ConferrenceID = cf.ConferrenceID
		INNER JOIN Studies s ON cf.StudiesID = s.StudiesID
		GROUP BY CartID, csl.LessonID, s.Name, los.Date) cls
		ON c.CartID = cls.CartID
    WHERE
        c.StudentID = @StudentID
);
```

### Sprawdzanie czy student zdał egzaminy (CheckIfPassedExams)

```sql
CREATE FUNCTION CheckIfPassedExams(
    @StudentID INT,
    @StudiesID INT
)
RETURNS bit
AS
BEGIN

	IF EXISTS (
		SELECT * FROM Studies s
		INNER JOIN Conferrence c ON s.StudiesID = c.StudiesID AND s.StudiesID = @StudiesID
		LEFT OUTER JOIN Exams e ON c.ConferrenceID = e.ConferrenceID AND e.StudentID = @StudentID
		WHERE ISNULL(e.Grade, 0) <= 2
	) BEGIN
		RETURN 0;
	END;
	RETURN 1;
END;
```

### Sprawdzanie czy student zdaje kurs (CheckPassingForCourse)

```sql
CREATE FUNCTION CheckPassingForCourse(
    @StudentID INT,
    @CourseID INT
)
RETURNS bit
AS
BEGIN
    -- Sprawdzenie, czy student ma dostęp do kursu
    IF NOT EXISTS (
        SELECT 1
        FROM CartCourses AS cc
        JOIN Cart AS c ON cc.CartID = c.CartID
        WHERE c.StudentID = @StudentID AND cc.CoursesID = @CourseID AND c.Access = 1
    )
    BEGIN
        RETURN 0; -- Brak dostępu oznacza brak możliwości zaliczenia
    END;

    -- Sprawdzenie, czy frekwencja >= 80%
    IF [dbo].[GetCourseAttendanceForStudent](@StudentID, @CourseID) >= 0.80
    BEGIN
        RETURN 1; -- Zaliczenie
    END;

    RETURN 0; -- Brak zaliczenia
END;
```

### Sprawdzanie czy student zdaje studia (CheckPassingForStudies)

```sql
CREATE FUNCTION CheckPassingForStudies(
    @StudentID INT,
    @StudiesID INT
)
RETURNS bit
AS
BEGIN
    -- Sprawdzenie, czy student ma dostęp do kursu
    IF NOT EXISTS (
        SELECT 1
        FROM CartStudies AS cs
        JOIN Cart AS c ON cs.CartID = c.CartID
        WHERE c.StudentID = @StudentID AND cs.StudiesID = @StudiesID AND c.Access = 1
    )
    BEGIN
        RETURN 0; -- Brak dostępu oznacza brak możliwości zaliczenia
    END;

    -- Sprawdzenie, czy zdane praktyki i egzaminy oraz czy frekwencja dla każdego zjazdu >= 80%
    IF ([dbo].[CheckStudentInternships](@StudentID, @StudiesID) = 1)
	AND ([dbo].[CheckIfPassedExams](@StudentID, @StudiesID) = 1)
	AND NOT EXISTS(
		SELECT *
		FROM Conferrence
		WHERE StudiesID = @StudiesID AND [dbo].[GetConferrenceAttendanceForStudent](@StudentID, ConferrenceID) < 0.8
	)
    BEGIN
        RETURN 1; -- Zaliczenie
    END;

    RETURN 0; -- Brak zaliczenia
END;
```

### Sprawdzanie staży studenta (CheckStudentInternships)

```sql
CREATE FUNCTION CheckStudentInternships(@StudentID int, @StudiesID int)
RETURNS bit
AS
BEGIN
    DECLARE @Result bit = 1
-- Sprawdzamy, czy student o danym ID ma nieobecność na jakichkolwiek zakończonych praktykach
    IF EXISTS (
		SELECT 1 FROM InternshipAbsence ia
		INNER JOIN Internships i ON ia.InternshipID = i.InternshipID AND i.StudiesID = @StudiesID
		WHERE StudentID = @StudentID AND DATEADD(Day, 14, DATE) < GETDATE())
    BEGIN
        SET @Result = 0
    END

    RETURN @Result
END
```

### Sprawdzanie jakimi językami włada tłumacz (CheckTranslatorLanguage)

```sql

CREATE FUNCTION CheckTranslatorLanguage
(@TranslatorID int null, @LanguageID int null)
RETURNS bit AS
BEGIN
    IF @TranslatorID IS NOT NULL AND NOT EXISTS (SELECT * FROM Translators WHERE TranslatorID = @TranslatorID)
    BEGIN
        RETURN CAST(0 AS bit)
    END

    IF @LanguageID IS NOT NULL AND NOT EXISTS (SELECT * FROM Languages WHERE LanguageID = @LanguageID)
    BEGIN
        RETURN CAST(0 AS bit)
    END

    IF @TranslatorID IS NULL AND @LanguageID IS NOT NULL
    BEGIN
        RETURN CAST(0 AS bit)
    END

    IF @TranslatorID IS NOT NULL AND @LanguageID IS NULL
    BEGIN
        RETURN CAST(0 AS bit)
    END

    IF @TranslatorID IS NOT NULL AND @LanguageID IS NOT NULL AND NOT EXISTS (
	SELECT * FROM TranslatorLanguages WHERE TranslatorID = @TranslatorID AND LanguageID = @LanguageID)
    BEGIN
        RETURN CAST(0 AS bit)
    END

    RETURN CAST(1 AS bit)
END
```

### Sprawdzanie wartości danego koszyka (GetCartValue)

```sql
CREATE FUNCTION GetCartValue(@CartID int)
RETURNS money
AS
BEGIN
    DECLARE @StudiesSum money
    DECLARE @StudyMeetingsSum money
    DECLARE @CoursesSum money
    DECLARE @WebinarsSum money

    SELECT @StudiesSum = ISNULL(SUM(s.Cost), 0)
    FROM Studies AS s
    JOIN CartStudies AS cs ON s.StudiesID = cs.StudiesID
    WHERE cs.CartID = @CartID

    SELECT @StudyMeetingsSum = ISNULL(SUM(ls.Cost), 0)
    FROM LessonsOnStudies AS ls
	JOIN CartSingleLessons as cl ON ls.LessonID = cl.LessonID
    WHERE cl.CartID = @CartID

    SELECT @CoursesSum = ISNULL(SUM(c.Price), 0)
    FROM Courses AS c
    JOIN CartCourses AS cc ON cc.CoursesID = c.CourseID
    WHERE cc.CartID = @CartID

    SELECT @WebinarsSum = ISNULL(SUM(w.Price), 0)
    FROM Webinars AS w
    JOIN CartWebinars AS cw ON w.WebinarID = cw.WebinarID
    WHERE cw.CartID = @CartID

    RETURN @StudiesSum + @CoursesSum + @WebinarsSum + @StudyMeetingsSum
END
```

### Zliczanie frekwencji na danym przedmiocie (GetConferrenceAttendanceForStudent)

```sql
CREATE FUNCTION GetConferrenceAttendanceForStudent(@StudentID int, @ConferrenceID int)
RETURNS FLOAT
AS
BEGIN
    -- Sprawdzenie, czy student istnieje czy kurs istnieje
    IF NOT EXISTS (
        SELECT 1
        FROM Students
        WHERE StudentID = @StudentID
    ) OR NOT EXISTS (
        SELECT 1
        FROM Conferrence
        WHERE ConferrenceID = @ConferrenceID
    )
    BEGIN
        RETURN 0;
    END;

    RETURN ISNULL((
		SELECT AVG(CAST(Presence AS float))
		FROM LessonsOnStudies los
		INNER JOIN StudyLessonPresence slp ON los.LessonID = slp.LessonID AND los.ConferrenceID = @ConferrenceID
		WHERE StudentID = @StudentID
	), 0)
END;
```

### Zliczanie frekwencji na danym kursie (GetCourseAttendanceForStudent)

```sql
CREATE FUNCTION GetCourseAttendanceForStudent(
    @StudentID INT,
    @CourseID INT
)
RETURNS FLOAT
AS
BEGIN

    -- Sprawdzenie, czy student istnieje, czy kurs istnieje, i czy student ma dostęp
    IF NOT EXISTS (
        SELECT 1
        FROM Students
        WHERE StudentID = @StudentID
    ) OR NOT EXISTS (
        SELECT 1
        FROM Courses
        WHERE CourseID = @CourseID
    )
    BEGIN
        RETURN 0;
    END;

	RETURN ISNULL((
		SELECT AVG(CAST(Presence AS float))
		FROM ModulesPresence mp
		JOIN CourseModules cm ON mp.ModuleID = cm.ModuleID AND CourseID = @CourseID
		WHERE mp.StudentID = @StudentID
	), 0)
END;
```

### Sprawdzanie daty najbliższego modułu w kursie (GetEarliestModuleDate)

```sql
CREATE FUNCTION GetEarliestModuleDate(@CourseID INT)
RETURNS DATETIME
AS
BEGIN
    DECLARE @EarliestDate DATETIME;

    SELECT @EarliestDate = MIN(Date)
    FROM CourseModules
    WHERE CourseID = @CourseID;

    RETURN @EarliestDate;
END;
```

### Sprawdź maksymalną pojemność kursu (GetMaxCourseCapacity)

```sql
CREATE FUNCTION GetMaxCourseCapacity(@CourseID int)
RETURNS int
AS
BEGIN
    DECLARE @MaxCapacity int;

    SELECT @MaxCapacity = MIN(sm.Limit)
    FROM StationaryModules sm
    INNER JOIN CourseModules cm ON sm.ModuleID = cm.ModuleID
    WHERE cm.CourseID = @CourseID;

    RETURN @MaxCapacity -- if there are no stationary meetings there is no limit so function returns NULL
END;
```

### Sprawdź maksymalną pojemność studiów (GetMaxStudyCapacity)

```sql
CREATE FUNCTION GetMaxStudyCapacity(@StudiesID int)
RETURNS int
AS
BEGIN
    DECLARE @MaxCapacity int;

    SELECT @MaxCapacity = MIN(Limit)
    FROM StationaryStudy ss
    INNER JOIN LessonsOnStudies ls ON ss.LessonID = ls.LessonID
    INNER JOIN Conferrence c ON ls.ConferrenceID = c.ConferrenceID
    WHERE c.StudiesID = @StudiesID;

    RETURN @MaxCapacity -- if there are no stationary meetings there is no limit so function returns NULL
END;
```

### Sprawdzanie ile należy dopłacić do koszyka (GetRemainingPaymentForCart)

```sql
CREATE FUNCTION GetRemainingPaymentForCart(@CartID int)
RETURNS money
AS
BEGIN


    RETURN dbo.GetCartValue(@CartID) - (
	SELECT ISNULL(Paid, 0)
	FROM Cart
	WHERE CartID = @CartID
	);
END
```

### Sprawdzanie ilości wolnych miejsc na kursie (HowManyCourseVacancies)

```sql
CREATE FUNCTION HowManyCourseVacancies(@CourseID int)
RETURNS int
AS
BEGIN
    DECLARE @MaximumCapacity int;
    SELECT @MaximumCapacity = dbo.GetMaxCourseCapacity(@CourseID);

    IF @MaximumCapacity IS NULL
    BEGIN
        RETURN @MaximumCapacity
    END

    DECLARE @CurrentCapacity int;
    SELECT @CurrentCapacity = COUNT(*)
    FROM Students
    WHERE StudentID IN (
        SELECT s.StudentID
        FROM Students s
        JOIN Cart c WITH (UPDLOCK) ON s.StudentID = c.StudentID  -- Lock cart entries
        JOIN CartCourses cs WITH (UPDLOCK) ON c.CartID = cs.CartID  -- Lock course assignments
        WHERE cs.CoursesID = @CourseID
    );

    RETURN @MaximumCapacity - @CurrentCapacity
END
```

### Sprawdzanie ilości wolnych miejsc na lekcji (HowManyLessonVacancies)

```sql
CREATE FUNCTION HowManyLessonVacancies (@LessonID int)
RETURNS int
AS
BEGIN
    IF NOT EXISTS(
        SELECT *
        FROM StationaryStudy WITH (UPDLOCK) -- Lock lesson metadata
        WHERE LessonID = @LessonID
    )
    BEGIN
        RETURN NULL;
    END

    DECLARE @Limit int;
    SELECT @Limit = MIN(Limit)
    FROM StationaryStudy WITH (UPDLOCK) -- Maintain lock
    WHERE LessonID = @LessonID;

    DECLARE @StudiesID int;
    SELECT @StudiesID = MIN(StudiesID)
    FROM LessonsOnStudies ls WITH (UPDLOCK) -- Lock lesson-study association
    JOIN Conferrence c WITH (UPDLOCK) ON ls.ConferrenceID = c.ConferrenceID -- Lock conference
    WHERE ls.LessonID = @LessonID;

    DECLARE @CurrentCapacity int;
    SELECT @CurrentCapacity = COUNT(*)
    FROM Students
    WHERE StudentID IN (
        SELECT s.StudentID
        FROM Students s
        JOIN Cart AS c WITH (UPDLOCK) ON s.StudentID = c.StudentID -- Lock cart
        JOIN CartStudies cs WITH (UPDLOCK) ON c.CartID = cs.CartID -- Lock study assignments
        WHERE cs.StudiesID = @StudiesID
    );

    SELECT @CurrentCapacity += ISNULL(COUNT(*), 0)
    FROM CartSingleLessons cl WITH (UPDLOCK) -- Lock single-lesson entries
    JOIN Cart c WITH (UPDLOCK) ON cl.CartID = c.CartID -- Lock cart
    WHERE cl.LessonID = @LessonID AND c.Access = 1;

    RETURN @Limit - @CurrentCapacity;
END;
```

### Sprawdzanie ilości wolnych miejsc na studiach (HowManyStudyVacancies)

```sql
CREATE FUNCTION HowManyStudyVacancies(@StudiesID int)
RETURNS int
AS
BEGIN
    DECLARE @MaximumCapacity int;
    SELECT @MaximumCapacity = dbo.GetMaxStudyCapacity(@StudiesID);

    IF @MaximumCapacity IS NULL
    BEGIN
        RETURN @MaximumCapacity
    END

    DECLARE @CurrentCapacity int;
    SELECT @CurrentCapacity = COUNT(*)
        FROM Students
        WHERE StudentID IN (
            SELECT s.StudentID
            FROM Students s
            JOIN Cart AS c WITH (UPDLOCK) ON s.StudentID = c.StudentID
            JOIN CartStudies cs WITH (UPDLOCK) ON c.CartID = cs.CartID
            WHERE cs.StudiesID = @StudiesID
        );

    RETURN @MaximumCapacity - @CurrentCapacity
END

```
