# Kategoria Studies

### Tabela LessonsOnStudies

- kod DDL

```sql
CREATE TABLE LessonsOnStudies (
    LessonID int  NOT NULL,
    TeacherID int  NOT NULL,
    Date datetime  NOT NULL,
    Cost smallmoney  NOT NULL DEFAULT 1000,
    ConferrenceID int  NOT NULL,
    TranslatorID int NULL,
    LanguageID int  NOT NULL,
    CONSTRAINT NonNegativeCost NOT NULL CHECK (Cost>0),
    CONSTRAINT LessonsOnStudies_pk PRIMARY KEY  (LessonID)
);
```

- Opis

| Nazwa atrybutu | Typ        | Opis/Uwagi                                                           |
| -------------- | ---------- | -------------------------------------------------------------------- |
| LessonID       | int        | Klucz główny. Unikalny identyfikator lekcji.                         |
| TeacherID      | int        | Identyfikator nauczyciela prowadzącego lekcję.                       |
| Date           | datetime   | Data i godzina przeprowadzenia lekcji.                               |
| Cost           | smallmoney | Koszt lekcji. UWAGA, wartość musi być większa niż 0. Domyślnie 1000. |
| ConferrenceID  | int        | Identyfikator zjazdu.                                                |
| TranslatorID   | int        | Identyfikator translatora, może być NULL.                            |
| LanguageID     | int        | Identyfikator języka, w którym prowadzona jest lekcja.               |

### Tabela Conferrence

- kod DDL

```sql
CREATE TABLE Conferrence (
    ConferrenceID int  NOT NULL,
    StudiesID int  NOT NULL,
    Date datetime  NOT NULL,
    CONSTRAINT Subjects_pk PRIMARY KEY  (ConferrenceID)
);
```

- Opis

| Nazwa atrybutu | Typ      | Opis/Uwagi                                               |
| -------------- | -------- | -------------------------------------------------------- |
| ConferrenceID  | int      | Klucz główny. Unikalny identyfikator zjazdu.             |
| StudiesID      | int      | Identyfikator programu studiów, do którego należy zjazd. |
| Date           | datetime | Data zjazdu.                                             |

### Tabela Studies

- kod DDL

```sql
CREATE TABLE Studies (
    StudiesID int  NOT NULL,
    Name varchar(20)  NOT NULL,
    Cost smallmoney  NOT NULL,
    Date datetime  NOT NULL,
    Orderable bit  NOT NULL,
    CONSTRAINT Studies_pk PRIMARY KEY  (StudiesID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                                     |
| -------------- | ----------- | -------------------------------------------------------------- |
| StudiesID      | int         | Klucz główny. Unikalny identyfikator programu studiów.         |
| Name           | varchar(20) | Nazwa programu studiów.                                        |
| Cost           | smallmoney  | Koszt programu studiów.                                        |
| Date           | datetime    | Data rozpoczęcia programu studiów.                             |
| Orderable      | bit         | Informacja o możliwości zakupu (1 - dostępny, 0 - niedostępny) |

### Tabela OnlineSynchronizeStudy

- kod DDL

```sql
CREATE TABLE OnlineSynchronizeStudy (
    LessonID int  NOT NULL,
    Link varchar(50)  NOT NULL,
    CONSTRAINT OnlineSynchronizeStudy_pk PRIMARY KEY  (LessonID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                          |
| -------------- | ----------- | --------------------------------------------------- |
| LessonID       | int         | Klucz główny. Unikalny identyfikator lekcji online. |
| Link           | varchar(50) | Link do zajęć online.                               |

### Tabela OnlineAsynchronizeStudy

- kod DDL

```sql
CREATE TABLE OnlineAsynchronizeStudy (
    LessonID int  NOT NULL,
    Link varchar(50)  NOT NULL,
    CONSTRAINT OnlineAsynchronizeStudy_pk PRIMARY KEY  (LessonID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                          |
| -------------- | ----------- | --------------------------------------------------- |
| LessonID       | int         | Klucz główny. Unikalny identyfikator lekcji online. |
| Link           | varchar(50) | Link do materiałów.                                 |

### Tabela StationaryStudy

- kod DDL

```sql
CREATE TABLE StationaryStudy (
    LessonID int  NOT NULL,
    Room varchar(10)  NOT NULL,
    Limit int  NOT NULL,
    CONSTRAINT StationaryStudy_pk PRIMARY KEY  (LessonID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                   |
| -------------- | ----------- | -------------------------------------------- |
| LessonID       | int         | Klucz główny. Unikalny identyfikator lekcji. |
| Room           | varchar(10) | Numer sali.                                  |
| Limit          | int         | Maksymalna liczba uczestników.               |

### Tabela Internships

- kod DDL

```sql
CREATE TABLE Internships (
    InternshipID int  NOT NULL,
    StudiesID int  NOT NULL,
    CONSTRAINT Internships_pk PRIMARY KEY  (InternshipID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                     |
| -------------- | --- | ---------------------------------------------- |
| InternshipID   | int | Klucz główny. Unikalny identyfikator praktyki. |
| StudiesID      | int | Identyfikator powiązanych studiów.             |

### Tabela InternshipAbsence

- kod DDL

```sql
CREATE TABLE InternshipAbsence (
    InternshipID int  NOT NULL,
    StudentID int  NOT NULL,
    Date datetime  NOT NULL,
    CONSTRAINT InternshipAbsence_pk PRIMARY KEY  (InternshipID,StudentID)
);
```

- Opis

| Nazwa atrybutu | Typ      | Opis/Uwagi                                                                 |
| -------------- | -------- | -------------------------------------------------------------------------- |
| InternshipID   | int      | Część klucza głównego. Identyfikator praktyki.                             |
| StudentID      | int      | Część klucza głównego. Identyfikator studenta biorącego udział w praktyce. |
| Date           | datetime | Data nieobecności.                                                         |

### Tabela StudyLessonPresence

- kod DDL

```sql
CREATE TABLE StudyLessonPresence (
    LessonID int  NOT NULL,
    StudentID int  NOT NULL,
    Presence bit  NOT NULL,
    MakeUpDate datetime  NULL,
    CONSTRAINT StudyLessonPresence_pk PRIMARY KEY  (StudentID,LessonID)
);
```

- Opis

| Nazwa atrybutu | Typ      | Opis/Uwagi                                                               |
| -------------- | -------- | ------------------------------------------------------------------------ |
| LessonID       | int      | Część klucza głównego. Identyfikator lekcji.                             |
| StudentID      | int      | Część klucza głównego. Identyfikator studenta biorącego udział w lekcji. |
| Presence       | bit      | Informacja o obecności (1 - obecny, 0 - nieobecny).                      |
| MakeUpDate     | datetime | Data                                                                     |

### Tabela Exams

- kod DDL

```sql
CREATE TABLE Exams (
    ConferrenceID int  NOT NULL,
    StudentID int  NOT NULL,
    Grade int  NULL,
    CONSTRAINT Exams_pk PRIMARY KEY  (ConferrenceID,StudentID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                                               |
| -------------- | --- | ------------------------------------------------------------------------ |
| ConferrenceID  | int | Część klucza głównego. Identyfikator zjazdu.                             |
| StudentID      | int | Część klucza głównego. Identyfikator studenta biorącego udział w lekcji. |
| Grade          | int | Ocena z egzaminu.                                                        |
