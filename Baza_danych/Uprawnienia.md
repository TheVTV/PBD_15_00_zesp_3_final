## Uprawnienia

### Rola admin

```sql
CREATE ROLE admin
GRANT CONTROL ON DATABASE::[u_swierzy] TO admin;
```

### Rola dyrektor

```sql
CREATE ROLE dyrektor
GRANT SELECT ON PRESENCE_ALL TO dyrektor
GRANT SELECT ON PRESENCE_STUDY_LESSONS TO dyrektor
GRANT SELECT ON PRESENCE_COURSE_MODULES TO dyrektor
GRANT SELECT ON UPCOMING_EVENTS_MONTH TO dyrektor
GRANT SELECT ON UPCOMING_STUDY_LESSONS_MONTH TO dyrektor
GRANT SELECT ON UPCOMING_COURSE_MODULES_MONTH TO dyrektor
GRANT SELECT ON UPCOMING_WEBINARS_MONTH TO dyrektor
GRANT SELECT ON DEBTORS TO dyrektor
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_EVENTS TO dyrektor
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_STUDY_LESSONS TO dyrektor
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_COURSE_MODULES TO dyrektor
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_WEBINARS TO dyrektor
GRANT SELECT ON ATTENDANCE TO dyrektor
GRANT SELECT ON ATTENDANCE_ON_STUDY_LESSONS TO dyrektor
GRANT SELECT ON ATTENDANCE_ON_COURSE_MODULES TO dyrektor
```

### Rola administrator_szkoly

```sql
CREATE ROLE administrator_szkoly
GRANT SELECT ON FINANCIAL_REPORT TO administrator_szkoly
GRANT SELECT ON FINANCIAL_REPORT_WEBINARS TO administrator_szkoly
GRANT SELECT ON FINANCIAL_REPORT_COURSES TO administrator_szkoly
GRANT SELECT ON FINANCIAL_REPORT_STUDIES TO administrator_szkoly
GRANT SELECT ON UPCOMING_EVENTS_MONTH TO administrator_szkoly
GRANT SELECT ON UPCOMING_STUDY_LESSONS_MONTH TO administrator_szkoly
GRANT SELECT ON UPCOMING_COURSE_MODULES_MONTH TO administrator_szkoly
GRANT SELECT ON UPCOMING_WEBINARS_MONTH TO administrator_szkoly
GRANT SELECT ON DEBTORS TO administrator_szkoly
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_EVENTS TO administrator_szkoly
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_STUDY_LESSONS TO administrator_szkoly
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_COURSE_MODULES TO administrator_szkoly
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_WEBINARS TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON Students TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON Employees TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON Administrators TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON Principals TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON SchoolAdministrators TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON Teacher TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON Translators TO administrator_szkoly
GRANT SELECT, INSERT, UPDATE, DELETE ON Languages TO administrator_szkoly
GRANT EXECUTE ON AddTranslatorWithLanguages TO administrator_szkoly
```

### Rola nauczyciel

```sql
CREATE ROLE nauczyciel
GRANT SELECT ON ATTENDANCE TO nauczyciel
GRANT SELECT ON ATTENDANCE_ON_STUDY_LESSONS TO nauczyciel
GRANT SELECT ON ATTENDANCE_ON_COURSE_MODULES TO nauczyciel
GRANT EXECUTE ON AddLessonPresence TO nauczyciel
GRANT EXECUTE ON AddModulePresence TO nauczyciel
GRANT EXECUTE ON AddInternshipAbsence TO nauczyciel
GRANT SELECT, INSERT, UPDATE, DELETE ON Webinars to nauczyciel
```

### Rola kierownik_kursu

```sql
CREATE ROLE kierownik_kursu
GRANT nauczyciel TO kierownik_kursu
GRANT SELECT, INSERT, UPDATE, DELETE ON Courses TO kierownik_kursu
GRANT SELECT, INSERT, UPDATE, DELETE ON CourseModules TO kierownik_kursu
GRANT SELECT, INSERT, UPDATE, DELETE ON StationaryModules TO kierownik_kursu
GRANT SELECT, INSERT, UPDATE, DELETE ON OnlineAsynchronizeModules TO kierownik_kursu
GRANT SELECT, INSERT, UPDATE, DELETE ON OnlineSynchronizeModules TO kierownik_kursu
GRANT EXECUTE ON GetMaxCourseCapacity TO kierownik_kursu
GRANT EXECUTE ON HowManyCourseVacancies TO kierownik_kursu
GRANT EXECUTE ON GetEarliestModuleDate TO kierownik_kursu
GRANT EXECUTE ON GetCourseAttendanceForStudent TO kierownik_kursu
GRANT EXECUTE ON CheckPassingForCourse TO kierownik_kursu
```

### Rola kierownik_przedmiotu

```sql
CREATE ROLE kierownik_przedmiotu
GRANT nauczyciel TO kierownik_przedmiotu
GRANT SELECT, INSERT, UPDATE, DELETE ON LessonsOnStudies TO kierownik_przedmiotu
GRANT SELECT, INSERT, UPDATE, DELETE ON StationaryStudy TO kierownik_przedmiotu
GRANT SELECT, INSERT, UPDATE, DELETE ON OnlineAsynchronizeStudy TO kierownik_przedmiotu
GRANT SELECT, INSERT, UPDATE, DELETE ON OnlineSynchronizeStudy TO kierownik_przedmiotu
GRANT EXECUTE ON HowManyLessonVacancies TO kierownik_przedmiotu
```

### Rola kierownik_studiow

```sql
CREATE ROLE kierownik_studiow
GRANT kierownik_przedmiotu TO kierownik_studiow
GRANT SELECT, INSERT, UPDATE, DELETE ON Studies TO kierownik_studiow
GRANT SELECT, INSERT, UPDATE, DELETE ON Exams TO kierownik_studiow
GRANT SELECT, INSERT, UPDATE, DELETE ON Internships TO kierownik_studiow
GRANT EXECUTE ON GetMaxStudyCapacity TO kierownik_studiow
GRANT EXECUTE ON HowManyStudyVacancies TO kierownik_studiow
GRANT EXECUTE ON CheckPassingForStudies TO kierownik_studiow
GRANT EXECUTE ON CheckStudentInternships TO kierownik_studiow
```

### Rola kierownik_praktyk

```sql
CREATE ROLE kierownik_praktyk
GRANT nauczyciel TO kierownik_praktyk
GRANT SELECT, INSERT, UPDATE, DELETE ON InternshipAbsence TO kierownik_praktyk
GRANT SELECT, INSERT, UPDATE, DELETE ON Internships TO kierownik_praktyk
GRANT EXECUTE ON CheckStudentInternships TO kierownik_praktyk
```

### Rola tlumacz

```sql
CREATE ROLE tlumacz
GRANT SELECT ON UPCOMING_EVENTS_MONTH TO tlumacz
GRANT SELECT ON Webinars TO tlumacz
GRANT SELECT ON LessonsOnStudies TO tlumacz
GRANT SELECT ON CourseModules TO tlumacz
```

### Rola student

```sql
CREATE ROLE student
GRANT SELECT ON PEOPLE_SIGNED_FOR_FUTURE_EVENTS TO student
GRANT SELECT ON UPCOMING_EVENTS_MONTH TO student
GRANT EXECUTE ON CheckStudentInternships TO student
GRANT EXECUTE ON HowManyCourseVacancies TO student
GRANT EXECUTE ON GetMaxCourseCapacity TO student
GRANT EXECUTE ON HowManyStudyVacancies TO student
GRANT EXECUTE ON GetMaxStudyCapacity TO student
```
