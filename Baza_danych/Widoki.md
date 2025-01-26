## Widoki

### Zestawienie przychodów każdego szkolenia (FINANCIAL_REPORT)

```sql
CREATE VIEW FINANCIAL_REPORT AS
SELECT *, 'Webinar' AS Type
FROM [FINANCIAL_REPORT_WEBINARS]

UNION

SELECT *, 'Course' AS Type
FROM [FINANCIAL_REPORT_COURSES]

UNION

SELECT *, 'Study' AS Type
FROM [FINANCIAL_REPORT_STUDIES]
```

### Zestawienie przychodów z webinarów (FINANCIAL_REPORT_WEBINARS)

```sql
CREATE VIEW FINANCIAL_REPORT_WEBINARS AS
SELECT w.WebinarID AS ID, w.Name, w.Price *
    (SELECT count(*)
    FROM CartWebinars cw JOIN
    Cart c ON c.CartID = cw.CartID
    WHERE cw.WebinarID = w.WebinarID) AS TotalIncome
FROM Webinars w
```

### Roczne zestawienie przychodów z webinarów (FINANCIAL_REPORT_WEBINARS_BY_YEAR)

```sql
CREATE VIEW FINANCIAL_REPORT_WEBINARS_BY_YEAR AS
SELECT
    w.WebinarID AS ID,
    w.Name,
    YEAR(c.PaidDate) AS IncomeYear,
    SUM(c.Paid) AS TotalIncome
FROM
    Webinars w
    JOIN CartWebinars cw ON cw.WebinarID = w.WebinarID
    JOIN Cart c ON c.CartID = cw.CartID
WHERE
    c.PaidDate IS NOT NULL -- Uwzględniamy tylko zamówienia, które zostały opłacone
GROUP BY
    w.WebinarID,
    w.Name,
    YEAR(c.PaidDate);
```

### Zestawienie przychodów z kursów (FINANCIAL_REPORT_COURSES)

```sql
CREATE VIEW FINANCIAL_REPORT_COURSES AS
SELECT cs.CourseID AS ID, cs.Name, cs.Price *
    (SELECT count(*)
    FROM CartCourses cc JOIN
    Cart c ON c.CartID = cc.CartID
    WHERE cc.CoursesID = cs.CourseID) AS TotalIncome
FROM Courses cs
```

### Roczne zestawienie przychodów z kursów (FINANCIAL_REPORT_COURSES_BY_YEAR)

```sql
CREATE VIEW FINANCIAL_REPORT_COURSES_BY_YEAR AS
SELECT
    cs.CourseID AS ID,
    cs.Name,
    YEAR(c.PaidDate) AS IncomeYear,
    sum(c.Paid) AS TotalIncome
FROM
    Courses cs
    JOIN CartCourses cc ON cc.CoursesID = cs.CourseID
    JOIN Cart c ON c.CartID = cc.CartID
WHERE
    c.PaidDate IS NOT NULL -- Uwzględniamy tylko zamówienia, które zostały opłacone
GROUP BY
    cs.CourseID,
    cs.Name,
    YEAR(c.PaidDate)
```

### Zestawienie przychodów ze studiów (FINANCIAL_REPORT_STUDIES)

```sql
CREATE VIEW FINANCIAL_REPORT_STUDIES AS
SELECT s.StudiesID AS ID, s.Name, s.Cost *
    (SELECT count(*)
    FROM CartStudies cs JOIN
    Cart c ON c.CartID = cs.CartID
    WHERE cs.StudiesID = s.StudiesID) AS TotalIncome
FROM Studies s
```

### Roczne zestawienie przychodów ze studiów (FINANCIAL_REPORT_STUDIES_BY_YEAR)

```sql
CREATE VIEW FINANCIAL_REPORT_STUDIES_BY_YEAR AS
SELECT
    s.StudiesID AS ID,
    s.Name,
    YEAR(c.PaidDate) AS IncomeYear,
    SUM(c.Paid) AS TotalIncome
FROM
    Studies s
    JOIN CartStudies cs ON cs.StudiesID = s.StudiesID
    JOIN Cart c ON c.CartID = cs.CartID
WHERE
    c.PaidDate IS NOT NULL -- Uwzględniamy tylko zamówienia, które zostały opłacone
GROUP BY
    s.StudiesID,
    s.Name,
    YEAR(c.PaidDate)
```

### Zestawienie przychodów z lekcji studyjnych (FINANCIAL_REPORT_STUDIES_LESSON)

```sql
CREATE VIEW FINANCIAL_REPORT_STUDIES_LESSON AS
SELECT s.LessonID AS ID, 'SingleLesson' as Name, s.Cost *
    (SELECT count(*)
    FROM CartSingleLessons cs JOIN
    Cart c ON c.CartID = cs.CartID
    WHERE cs.LessonID = s.LessonID) AS TotalIncome
FROM LessonsOnStudies s
```

### Zestawienie obecności studentów na wszystkich szkoleniach (PRESENCE_ALL)

```sql
CREATE VIEW PRESENCE_ALL AS
SELECT *, 'Course Module' AS EventType
FROM [PRESENCE_COURSE_MODULES]

UNION ALL

SELECT *, 'Study Lesson' AS EventType
FROM [PRESENCE_STUDY_LESSONS]
```

### Zestawienie obecności na spotkaniach studyjnych (PRESENCE_STUDY_LESSONS)

```sql
CREATE VIEW PRESENCE_STUDY_LESSONS AS
SELECT slp.LessonID AS ID, los.Date, s.FirstName,
s.LastName, (CASE WHEN Presence = 1 THEN 'Present' ELSE 'Absent' END) AS Presence
FROM LessonsOnStudies AS los INNER JOIN
StudyLessonPresence AS slp ON los.LessonID = slp.LessonID INNER JOIN
Students AS s ON slp.StudentID = s.StudentID
WHERE (los.Date < GETDATE())
```

### Zestawienie obecności na modułach (PRESENCE_COURSE_MODULES)

```sql
CREATE VIEW PRESENCE_COURSE_MODULES AS
SELECT cm.ModuleID AS ID, cm.Date, s.FirstName,
s.LastName, (CASE WHEN Presence = 1 THEN 'Present' ELSE 'Absent' END) AS Presence
FROM CourseModules AS cm INNER JOIN
ModulesPresence AS mp ON cm.ModuleID = mp.ModuleID INNER JOIN
Students AS s ON mp.StudentID = s.StudentID
WHERE (cm.Date < GETDATE())
```

### Zestawienie wydarzeń w nadchodzącym miesiącu (UPCOMING_EVENTS_MONTH)

- Komentarz
  Jest to zestawienie, które pokazuje wszystkie zaplanowane wydarzenia na miesiąc do przodu. Dodadkowo zawiera imię i nazwisko nauczyciela prowadzącego i typ spotkania (Online synch., asynch., stacjonarne)

```sql
CREATE VIEW UPCOMING_EVENTS_MONTH AS
SELECT *, 'Webinar' as EventType
FROM [UPCOMING_WEBINARS_MONTH]

UNION ALL
SELECT *, 'Study Lesson' as EventType
FROM [UPCOMING_STUDY_LESSONS_MONTH]

UNION ALL
SELECT *, 'Course Module' as EventType
FROM [UPCOMING_COURSE_MODULES_MONTH]
```

### Zestawienie spotkań studyjnych w następnym miesiącu (UPCOMING_STUDY_LESSONS_MONTH)

```sql
CREATE VIEW UPCOMING_STUDY_LESSONS_MONTH AS
SELECT
    l.Date,
    s.ConferrenceID as EventID,
    e.FirstName + ' ' + e.LastName AS Teacher,
	CASE
        WHEN SL.LessonID IS NOT NULL THEN 'Stationary'
        WHEN OAS.LessonID IS NOT NULL THEN 'Online Asynchronous'
        WHEN OS.LessonID IS NOT NULL THEN 'Online Synchronous'
        ELSE 'Unknown'
    END AS MeetingType
FROM LessonsOnStudies l
JOIN Conferrence s ON l.ConferrenceID = s.ConferrenceID
JOIN Teacher t ON l.TeacherID = t.TeacherID
JOIN Employees e ON t.EmployeeID = e.EmployeeID
LEFT JOIN StationaryStudy sl ON l.LessonID = sl.LessonID
LEFT JOIN OnlineAsynchronizeStudy oas ON l.LessonID = oas.LessonID
LEFT JOIN OnlineSynchronizeStudy os ON l.LessonID = os.LessonID
WHERE l.Date BETWEEN GETDATE() AND DATEADD(MONTH, 1, GETDATE())
```

### Zestawienie modułów w następnym miesiącu (UPCOMING_COURSE_MODULES_MONTH)

```sql
CREATE VIEW UPCOMING_COURSE_MODULES_MONTH AS
SELECT
    cm.Date,
    cm.ModuleID as EventID,
    e.FirstName + ' ' + e.LastName AS Teacher,
	CASE
        WHEN SM.ModuleID IS NOT NULL THEN 'Stationary'
        WHEN OAM.ModuleID IS NOT NULL THEN 'Online Asynchronous'
        WHEN OSM.ModuleID IS NOT NULL THEN 'Online Synchronous'
        ELSE 'Unknown'
    END AS MeetingType
FROM CourseModules cm
JOIN Courses c ON cm.CourseID = c.CourseID
JOIN Teacher t ON cm.TeacherID = t.TeacherID
JOIN Employees e ON t.EmployeeID = e.EmployeeID
LEFT JOIN StationaryModules sm ON cm.ModuleID = sm.ModuleID
LEFT JOIN OnlineAsynchronizeModules oam ON cm.ModuleID = oam.ModuleID
LEFT JOIN OnlineSynchronizeModules osm ON cm.ModuleID = osm.ModuleID
WHERE cm.Date BETWEEN GETDATE() AND DATEADD(MONTH, 1, GETDATE())
```

### Zestawienie webinarów w następnym miesiącu (UPCOMING_WEBINARS_MONTH)

```sql
CREATE VIEW UPCOMING_WEBINARS_MONTH AS
SELECT
    w.Date,
    w.WebinarID as EventID,
    e.FirstName + ' ' + e.LastName AS Teacher,
	'Online' AS MeetingType
FROM Webinars w
JOIN Teacher t ON w.TeacherID = t.TeacherID
JOIN Employees e ON t.EmployeeID = e.EmployeeID
WHERE w.Date BETWEEN GETDATE() AND DATEADD(MONTH, 1, GETDATE());
```

### Zestawienie dłużników (DEBTORS)

```sql
CREATE VIEW DEBTORS AS
SELECT c.CartID, fp.Name, fp.[Full Price], CASE WHEN c.Paid IS NULL THEN 0 ELSE c.Paid END AS Paid,
s.FirstName + ' ' + s.LastName AS [Full Name], s.Email, s.PhoneNumber
FROM

(SELECT cc.CartID, c.Price AS [Full Price], c.Name FROM CartCourses AS cc INNER JOIN
Courses AS c ON cc.CoursesID = c.CourseID

UNION

SELECT cw.CartID, CASE WHEN w.Price IS NULL THEN 0 ELSE w.Price END AS [Full Price], w.Name FROM CartWebinars AS cw INNER JOIN
Webinars AS w ON cw.WebinarID = w.WebinarID

UNION

SELECT cs.CartID, s.Cost AS [Full Price], s.Name FROM CartStudies AS cs INNER JOIN
Studies AS s ON cs.StudiesID = s.StudiesID

UNION

SELECT csl.CartID, los.Cost AS [Full Price], st.Name + ' - ' + su.ConferrenceID AS ConferrenceID FROM CartSingleLessons AS csl INNER JOIN
LessonsOnStudies AS los ON csl.LessonID = los.LessonID INNER JOIN
Conferrence AS su ON los.ConferrenceID = su.ConferrenceID INNER JOIN
Studies AS st ON su.StudiesID = st.StudiesID) AS fp JOIN
Cart AS c ON fp.CartID = c.CartID INNER JOIN
Students AS s ON c.StudentID = s.StudentID
WHERE (fp.[Full Price] > c.Paid OR c.Paid IS NULL) AND c.Access = 1;
```

### Zestawienie liczby osób zapisanych na przyszłe wydarzenia (PEOPLE_SIGNED_FOR_FUTURE_EVENTS)

```sql
CREATE VIEW PEOPLE_SIGNED_FOR_FUTURE_EVENTS AS
SELECT *, 'Webinar' as eventType
FROM [PEOPLE_SIGNED_FOR_FUTURE_WEBINARS]

UNION ALL

SELECT *, 'Study Lesson' as eventType
FROM [PEOPLE_SIGNED_FOR_FUTURE_STUDY_LESSONS]

UNION ALL

SELECT *, 'Course Module' as eventType
FROM [PEOPLE_SIGNED_FOR_FUTURE_COURSE_MODULES]
```

### Zestawienie liczby osób zapisanych na przyszłe spotkania studyjne (PEOPLE_SIGNED_FOR_FUTURE_STUDY_LESSONS)

```sql
CREATE VIEW PEOPLE_SIGNED_FOR_FUTURE_STUDY_LESSONS AS
SELECT
    l.Date,
    s.ConferrenceID as eventID,
    e.FirstName + ' ' + e.LastName AS Teacher,
	CASE
        WHEN SL.LessonID IS NOT NULL THEN 'Stationary'
        WHEN OAS.LessonID IS NOT NULL THEN 'Online Asynchronous'
        WHEN OS.LessonID IS NOT NULL THEN 'Online Synchronous'
        ELSE 'Unknown'
    END AS Meeting_Type,
	COUNT(*) AS Signed_Users
FROM LessonsOnStudies l
JOIN Conferrence s ON l.ConferrenceID = s.ConferrenceID
JOIN Teacher t ON l.TeacherID = t.TeacherID
JOIN Employees e ON t.EmployeeID = e.EmployeeID
LEFT JOIN StationaryStudy sl ON l.LessonID = sl.LessonID
LEFT JOIN OnlineAsynchronizeStudy oas ON l.LessonID = oas.LessonID
LEFT JOIN OnlineSynchronizeStudy os ON l.LessonID = os.LessonID
INNER JOIN CartSingleLessons AS csl ON l.LessonID = csl.LessonID
INNER JOIN Cart AS c ON csl.CartID = c.CartID
WHERE l.Date > GETDATE() AND c.Access = 1
GROUP BY CASE WHEN SL.LessonID IS NOT NULL THEN 'Stationary' WHEN OAS.LessonID IS NOT NULL THEN 'Online Asynchronous' WHEN OS.LessonID IS NOT NULL THEN 'Online Synchronous' ELSE 'Unknown' END,
                  e.FirstName + ' ' + e.LastName, s.ConferrenceID, l.Date
```

### Zestawienie liczby osób zapisanych na przyszłe moduły (PEOPLE_SIGNED_FOR_FUTURE_COURSE_MODULES)

```sql
CREATE VIEW PEOPLE_SIGNED_FOR_FUTURE_COURSE_MODULES AS
SELECT
    cm.Date,
    cm.ModuleID AS eventID,
    e.FirstName + ' ' + e.LastName AS Teacher,
	CASE
        WHEN SM.ModuleID IS NOT NULL THEN 'Stationary'
        WHEN OAM.ModuleID IS NOT NULL THEN 'Online Asynchronous'
        WHEN OSM.ModuleID IS NOT NULL THEN 'Online Synchronous'
        ELSE 'Unknown'
    END AS Meeting_Type,
	COUNT(*) AS Signed_Users
FROM CourseModules cm
JOIN Courses c ON cm.CourseID = c.CourseID
JOIN Teacher t ON cm.TeacherID = t.TeacherID
JOIN Employees e ON t.EmployeeID = e.EmployeeID
LEFT JOIN StationaryModules sm ON cm.ModuleID = sm.ModuleID
LEFT JOIN OnlineAsynchronizeModules oam ON cm.ModuleID = oam.ModuleID
LEFT JOIN OnlineSynchronizeModules osm ON cm.ModuleID = osm.ModuleID
INNER JOIN CartCourses AS cc ON c.CourseID = cc.CoursesID
INNER JOIN Cart AS ca ON cc.CartID = ca.CartID
WHERE cm.Date > GETDATE() AND ca.Access = 1
GROUP BY cm.Date, cm.ModuleID, e.FirstName + ' ' + e.LastName, CASE WHEN SM.ModuleID IS NOT NULL THEN 'Stationary' WHEN OAM.ModuleID IS NOT NULL
                  THEN 'Online Asynchronous' WHEN OSM.ModuleID IS NOT NULL THEN 'Online Synchronous' ELSE 'Unknown' END
```

### Zestawienie liczby osób zapisanych na przyszłe webinary (PEOPLE_SIGNED_FOR_FUTURE_WEBINARS)

```sql
CREATE VIEW PEOPLE_SIGNED_FOR_FUTURE_WEBINARS AS
SELECT
    w.Date,
    w.WebinarID as eventID,
    e.FirstName + ' ' + e.LastName AS Teacher,
	'Online' AS Meeting_Type,
	COUNT(*) AS Signed_Users
FROM Webinars w
JOIN Teacher t ON w.TeacherID = t.TeacherID
JOIN Employees e ON t.EmployeeID = e.EmployeeID
INNER JOIN CartWebinars AS cw ON w.WebinarID = cw.WebinarID
INNER JOIN Cart AS c ON cw.CartID = c.CartID
WHERE w.Date > GETDATE() AND c.Access = 1
GROUP BY w.Date, w.WebinarID, e.FirstName + ' ' + e.LastName
```

### Zestawienie obecności na poszczególnych wydarzeniach (ATTENDANCE)

```sql
CREATE VIEW ATTENDANCE AS
SELECT *, 'Study Lesson' as type FROM dbo.ATTENDANCE_ON_STUDY_LESSONS

UNION ALL

SELECT *, 'Course Module' as type FROM dbo.ATTENDANCE_ON_COURSE_MODULES
```

### Zestawienie obecności na poszczególnych spotkaniach studyjnych (ATTENDANCE_ON_STUDY_LESSONS)

```sql
CREATE VIEW ATTENDANCE_ON_STUDY_LESSONS AS
SELECT slp.LessonID AS 'ID', los.Date, ROUND(100*(CAST(SUM(CAST(slp.Presence AS Int)) AS FLOAT) / COUNT(CAST(slp.Presence AS INT))),2) AS [% Frequence]
FROM LessonsOnStudies AS los INNER JOIN
StudyLessonPresence AS slp ON los.LessonID = slp.LessonID INNER JOIN
Students AS s ON slp.StudentID = s.StudentID
WHERE (los.Date < GETDATE())
GROUP BY slp.LessonID, los.Date
```

### Zestawienie obecności na poszczególnych modułach (ATTENDANCE_ON_COURSE_MODULES)

```sql
CREATE VIEW ATTENDANCE_ON_COURSE_MODULES AS
SELECT cm.ModuleID AS 'ID', cm.Date, ROUND(100*CAST(SUM(CAST(mp.Presence AS Int)) AS FLOAT) / COUNT(CAST(mp.Presence AS Int)),2) AS [% Frequence]
FROM CourseModules AS cm INNER JOIN
ModulesPresence AS mp ON cm.ModuleID = mp.ModuleID INNER JOIN
Students AS s ON mp.StudentID = s.StudentID
WHERE (cm.Date < GETDATE())
GROUP BY cm.ModuleID, cm.Date;
```
