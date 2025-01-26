## Indeksy

### Tabela Students

```sql
CREATE UNIQUE INDEX idx_students_student_id ON Students(StudentID);
CREATE INDEX idx_students_lastname ON Students(LastName);
```

### Tabela Cart

```sql
CREATE UNIQUE INDEX idx_cart_cart_id ON Cart(CartID);
CREATE INDEX idx_cart_student_id ON Cart(StudentID);
```

### Tabela Courses

```sql
CREATE UNIQUE INDEX idx_courses_course_id ON Courses(CourseID);
CREATE INDEX idx_courses_name ON Courses(Name);
CREATE INDEX idx_courses_price ON Courses(Price);
```

### Tabela Webinars

```sql
CREATE UNIQUE INDEX idx_webinars_webinar_id ON Webinars(WebinarID);
CREATE INDEX idx_webinars_teacher_id ON Webinars(TeacherID);
CREATE INDEX idx_webinars_date ON Webinars(Date);
CREATE INDEX idx_webinars_duration ON Webinars(Duration);
CREATE INDEX idx_webinars_price ON Webinars(Price);
```

### Tabela LessonsOnStudies

```sql
CREATE UNIQUE INDEX idx_lessons_on_studies_lesson_id ON LessonsOnStudies(LessonID);
CREATE INDEX idx_lessons_on_studies_duration ON LessonsOnStudies(Duration);
```

### Tabela CartCourses

```sql
CREATE UNIQUE INDEX idx_cart_courses_cart_id_course_id ON CartCourses(CartID, CoursesID);
```

### Tabela Employees

```sql
CREATE UNIQUE INDEX idx_employees_employee_id ON Employees(EmployeeID);
CREATE INDEX idx_employees_lastname ON Employees(LastName);
```

### Tabela Payments

```sql
CREATE UNIQUE INDEX idx_payments_payment_id ON Payments(PaymentID);
CREATE INDEX idx_payments_cart_id ON Payments(CartID);
CREATE INDEX idx_payments_created_at ON Payments(CreatedAt);
```

### Tabela StudyLessonsPresence

```sql
CREATE INDEX idx_study_lessons_presence_student_id ON StudyLessonPresence(StudentID);
CREATE INDEX idx_study_lessons_presence_lesson_id ON StudyLessonPresence(LessonID);
```

### Tabela Teachers

```sql
CREATE UNIQUE INDEX idx_teachers_teacher_id ON Teacher(TeacherID);
```

### Tabela Languages

```sql
CREATE UNIQUE INDEX idx_languages_language_id ON Languages(LanguageID);
CREATE INDEX idx_languages_name ON Languages(LanguageName);
```

### Tabela Internships

```sql
CREATE INDEX idx_internships_start_date ON Internships(StartDate);
```
