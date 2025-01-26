# Kategoria Courses

### Tabela Courses

- kod DDL

```sql
CREATE TABLE Courses (
    CourseID int  NOT NULL,
    Name varchar(50)  NOT NULL,
    Price int  NOT NULL,
    Description varchar(100)  NULL,
    Orderable bit  NOT NULL,
    CONSTRAINT Courses_pk PRIMARY KEY  (CourseID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                                     |
| -------------- | --- | -------------------------------------------------------------- |
| CourseID       | int | Klucz główny. Unikalny identyfikator kursu.                    |
| Name           | int | Nazwa kursu.                                                   |
| Price          | int | Cena kursu.                                                    |
| Description    | int | Opcjonalny opis kursu.                                         |
| Orderable      | bit | Informacja o możliwości zakupu (1 - dostępny, 0 - niedostępny) |

### Tabela CourseModules

- kod DDL

```sql
CREATE TABLE CourseModules (
    ModuleID int  NOT NULL,
    CourseID int  NOT NULL,
    Name varchar(50)  NOT NULL,
    Date datetime  NOT NULL,
    TeacherID int  NOT NULL,
    TranslatorID int  NULL,
    LanguageID int  NOT NULL,
    CONSTRAINT CourseModules_pk PRIMARY KEY  (ModuleID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                            |
| -------------- | ----------- | ----------------------------------------------------- |
| ModuleID       | int         | Klucz główny. Unikalny identyfikator modułu kursu.    |
| CourseID       | int         | Identyfikator kursu, do którego należy moduł.         |
| Name           | varchar(50) | Nazwa modułu kursu.                                   |
| Date           | datetime    | Data przeprowadzenia modułu kursu.                    |
| TeacherID      | int         | Identyfikator nauczyciela prowadzącego moduł.         |
| TranslatorID   | int         | Identyfikator tłumacza. Może być NULL.                |
| LanguageID     | int         | Identyfikator języka, w którym prowadzony jest moduł. |

### Tabela ModulesPresence

- kod DDL

```sql
CREATE TABLE ModulesPresence (
    ModuleID int  NOT NULL,
    StudentID int  NOT NULL,
    Presence bit  NOT NULL,
    CONSTRAINT ModulesPresence_pk PRIMARY KEY  (ModuleID,StudentID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                                       |
| -------------- | --- | ---------------------------------------------------------------- |
| ModuleID       | int | Część klucza głównego. Identyfikator modułu.                     |
| StudentID      | int | Część klucza głównego. Identyfikator studenta.                   |
| Presence       | bit | Informacja o obecności na zajęciach (0 - nieobecny, 1 - obecny). |

### Tabela StationaryModules

- kod DDL

```sql
CREATE TABLE StationaryModules (
    ModuleID int  NOT NULL,
    Room varchar(10)  NOT NULL,
    Limit int  NOT NULL,
    CONSTRAINT StationaryModules_pk PRIMARY KEY  (ModuleID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                          |
| -------------- | ----------- | ----------------------------------- |
| ModuleID       | int         | Klucz główny. Identyfikator modułu. |
| Room           | varchar(10) | Numer pokoju.                       |
| Limit          | int         | Maksymalna liczba uczestników.      |

### Tabela OnlineAsynchronizeModules

- kod DDL

```sql
CREATE TABLE OnlineAsynchronizeModules (
    ModuleID int  NOT NULL,
    Link varchar(50)  NOT NULL,
    CONSTRAINT OnlineAsynchronizeModules_pk PRIMARY KEY  (ModuleID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                          |
| -------------- | ----------- | ----------------------------------- |
| ModuleID       | int         | Klucz główny. Identyfikator modułu. |
| Link           | varchar(50) | Link do zasobów modułu online.      |

### Tabela OnlineSynchronizeModules

- kod DDL

```sql
CREATE TABLE OnlineSynchronizeModules (
    ModuleID int  NOT NULL,
    Link varchar(50)  NOT NULL,
    CONSTRAINT OnlineSynchronizeModules_pk PRIMARY KEY  (ModuleID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                          |
| -------------- | ----------- | ----------------------------------- |
| ModuleID       | int         | Klucz główny. Identyfikator modułu. |
| Link           | varchar(50) | Link do zasobów modułu.             |
