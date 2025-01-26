# Kategoria People

### Tabela Students

- kod DDL

```sql
CREATE TABLE Students (
    StudentID int  NOT NULL,
    FirstName varchar(20)  NOT NULL,
    LastName varchar(20)  NOT NULL,
    Email varchar(50)  NOT NULL CHECK (Email LIKE '%_@__%.__%'),
    PhoneNumber varchar(15)  NULL CHECK (ISNUMERIC(PhoneNumber) = 1),
    CONSTRAINT Students_pk PRIMARY KEY  (StudentID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                                                       |
| -------------- | ----------- | -------------------------------------------------------------------------------- |
| StudentID      | int         | Klucz główny. Unikalny identyfikator studenta.                                   |
| FirstName      | varchar(20) | Imię studenta.                                                                   |
| LastName       | varchar(20) | Nazwisko studenta.                                                               |
| Email          | varchar(50) | Adres e-mail studenta. UWAGA, musi spełniać format adresu e-mail (`%_@__%.__%`). |
| PhoneNumber    | varchar(15) | Numer telefonu studenta (opcjonalny). UWAGA, musi składać się wyłącznie z cyfr.  |

### Tabela Employees

- kod DDL

```sql
CREATE TABLE Employees (
    EmployeeID int  NOT NULL,
    FirstName varchar(20)  NOT NULL,
    LastName varchar(20)  NOT NULL,
    Email varchar(50)  NOT NULL CHECK (Email LIKE '%_@__%.__%'),
    PhoneNumber varchar(15)  NULL CHECK (ISNUMERIC(PhoneNumber) = 1),
    CONSTRAINT Employees_pk PRIMARY KEY  (EmployeeID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                                                         |
| -------------- | ----------- | ---------------------------------------------------------------------------------- |
| EmployeeID     | int         | Klucz główny. Unikalny identyfikator pracownika.                                   |
| FirstName      | varchar(20) | Imię pracownika.                                                                   |
| LastName       | varchar(20) | Nazwisko pracownika.                                                               |
| Email          | varchar(50) | Adres e-mail pracownika. UWAGA, musi spełniać format adresu e-mail (`%_@__%.__%`). |
| PhoneNumber    | varchar(15) | Numer telefonu pracownika (opcjonalny). UWAGA, musi składać się wyłącznie z cyfr.  |

### Tabela Administrators

- kod DDL

```sql
CREATE TABLE Administrators (
    AdministratorID int  NOT NULL,
    EmployeeID int  NOT NULL,
    CONSTRAINT Administrators_pk PRIMARY KEY  (AdministratorID)
);
```

- Opis

| Nazwa atrybutu  | Typ | Opis/Uwagi                                                    |
| --------------- | --- | ------------------------------------------------------------- |
| AdministratorID | int | Klucz główny. Unikalny identyfikator administratora.          |
| EmployeeID      | int | Identyfikator pracownika przypisanego do roli administratora. |

### Tabela Principals

- kod DDL

```sql
CREATE TABLE Principals (
    PrincipalID int  NOT NULL,
    EmployeeID int  NOT NULL,
    CONSTRAINT Principals_pk PRIMARY KEY  (PrincipalID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                               |
| -------------- | --- | -------------------------------------------------------- |
| PrincipalID    | int | Klucz główny. Unikalny identyfikator dyrektora.          |
| EmployeeID     | int | Identyfikator pracownika przypisanego do roli dyrektora. |

### Tabela SchoolAdministrators

- kod DDL

```sql
CREATE TABLE SchoolAdministrators (
    SchoolAdministratorID int  NOT NULL,
    EmployeeID int  NOT NULL,
    CONSTRAINT SchoolAdministrators_pk PRIMARY KEY  (SchoolAdministratorID)
);
```

- Opis

| Nazwa atrybutu        | Typ | Opis/Uwagi                                                           |
| --------------------- | --- | -------------------------------------------------------------------- |
| SchoolAdministratorID | int | Klucz główny. Unikalny identyfikator administratora szkoły.          |
| EmployeeID            | int | Identyfikator pracownika przypisanego do roli administratora szkoły. |

### Tabela Translators

- kod DDL

```sql
CREATE TABLE Translators (
    TranslatorID int  NOT NULL,
    EmployeeID int  NOT NULL,
    CONSTRAINT Translators_pk PRIMARY KEY  (TranslatorID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                              |
| -------------- | --- | ------------------------------------------------------- |
| TranslatorID   | int | Klucz główny. Unikalny identyfikator tłumacza.          |
| EmployeeID     | int | Identyfikator pracownika przypisanego do roli tłumacza. |

### Tabela Teacher

- kod DDL

```sql
CREATE TABLE Teacher (
    TeacherID int  NOT NULL,
    EmployeeID int  NOT NULL,
    CONSTRAINT Teacher_pk PRIMARY KEY  (TeacherID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                                 |
| -------------- | --- | ---------------------------------------------------------- |
| TeacherID      | int | Klucz główny. Unikalny identyfikator nauczyciela.          |
| EmployeeID     | int | Identyfikator pracownika przypisanego do roli nauczyciela. |

### Tabela Languages

- kod DDL

```sql
CREATE TABLE Languages (
    LanguageID int  NOT NULL,
    LanguageName varchar(20)  NOT NULL,
    CONSTRAINT Languages_pk PRIMARY KEY  (LanguageID)
);
```

- Opis

| Nazwa atrybutu | Typ         | Opis/Uwagi                                   |
| -------------- | ----------- | -------------------------------------------- |
| LanguageID     | int         | Klucz główny. Unikalny identyfikator języka. |
| LanguageName   | varchar(20) | Nazwa języka.                                |

### Tabela TranslatorLanguages

- kod DDL

```sql
CREATE TABLE TranslatorLanguages (
    TranslatorID int  NOT NULL,
    LanguageID int  NOT NULL,
    CONSTRAINT TranslatorLanguages_pk PRIMARY KEY  (TranslatorID,LanguageID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                                            |
| -------------- | --- | --------------------------------------------------------------------- |
| TranslatorID   | int | Część klucza głównego. Identyfikator tłumacza.                        |
| LanguageID     | int | Część klucza głównego. Identyfikator języka, który tłumacz obsługuje. |
