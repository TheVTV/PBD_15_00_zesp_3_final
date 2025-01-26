# Kategoria Cart

### Tabela Cart

- kod DDL

```sql
CREATE TABLE Cart (
    CartID int IDENTITY(1,1) NOT NULL,
    StudentID int  NOT NULL,
    Paid smallmoney  NOT NULL,
    PaidDate datetime  NOT NULL,
    DueDate datetime  NOT NULL,
    Access bit  NOT NULL,
    CONSTRAINT Cart_pk PRIMARY KEY  (CartID)
);
```

- Opis

| Nazwa atrybutu | Typ        | Opis/Uwagi                                                                                 |
| -------------- | ---------- | ------------------------------------------------------------------------------------------ |
| CartID         | int        | Klucz główny. Identyfikator koszyka.                                                       |
| StudentID      | int        | Identyfikator studenta, do którego należy koszyk.                                          |
| Paid           | smallmoney | Kwota zapłacona za koszyk.                                                                 |
| PaidDate       | datetime   | Data dokonania płatności za koszyk.                                                        |
| DueDate        | datetime   | Data do kiedy należy dokonać płatności.                                                    |
| Access         | bit        | Informacja o tym, czy student ma już dostęp do przedmiotów (1 - dostęp, 0 - brak dostępu). |

### Tabela CartCourses

- kod DDL

```sql
CREATE TABLE CartCourses (
    CartID int  NOT NULL,
    CoursesID int  NOT NULL,
    CONSTRAINT CartCourses_pk PRIMARY KEY  (CoursesID,CartID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                    |
| -------------- | --- | --------------------------------------------- |
| CartID         | int | Część klucza głównego. Identyfikator koszyka. |
| CoursesID      | int | Część klucza głównego. Identyfikator kursu.   |

### Tabela CartWebinars

- kod DDL

```sql
CREATE TABLE CartWebinars (
    CartID int  NOT NULL,
    WebinarID int  NOT NULL,
    CONSTRAINT CartWebinars_pk PRIMARY KEY  (CartID,WebinarID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                     |
| -------------- | --- | ---------------------------------------------- |
| CartID         | int | Część klucza głównego. Identyfikator koszyka.  |
| WebinarID      | int | Część klucza głównego. Identyfikator webinaru. |

### Tabela CartStudies

- kod DDL

```sql
CREATE TABLE CartStudies (
    CartID int  NOT NULL,
    StudiesID int  NOT NULL,
    CONSTRAINT CartStudies_pk PRIMARY KEY  (CartID,StudiesID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                    |
| -------------- | --- | --------------------------------------------- |
| CartID         | int | Część klucza głównego. Identyfikator koszyka. |
| StudiesID      | int | Część klucza głównego. Identyfikator studiów. |

### Tabela CartSingleLessons

- kod DDL

```sql
CREATE TABLE CartSingleLessons (
    CartID int  NOT NULL,
    LessonID int  NOT NULL,
    CONSTRAINT CartSingleLessons_pk PRIMARY KEY  (CartID,LessonID)
);
```

- Opis

| Nazwa atrybutu | Typ | Opis/Uwagi                                    |
| -------------- | --- | --------------------------------------------- |
| CartID         | int | Część klucza głównego. Identyfikator koszyka. |
| LessonID       | int | Część klucza głównego. Identyfikator lekcji.  |

---

### Tabela Payments

- kod DDL

```sql
CREATE TABLE dbo.Payments (
    PaymentID int  NOT NULL IDENTITY(1, 1),
    Value smallmoney  NOT NULL,
    CartID int  NOT NULL,
    IsSuccessful bit  NOT NULL,
    CreatedAt datetime  NOT NULL,
    CONSTRAINT CHK_Payments_Value CHECK (( ( [ Value ] > ( 0 ) ) )),
    CONSTRAINT Payments_pk PRIMARY KEY (PaymentID)
)
```

- Opis

| Nazwa atrybutu | Typ        | Opis/Uwagi                             |
| -------------- | ---------- | -------------------------------------- |
| PaymentID      | int        | Klucz główny. Identyfikator płatności. |
| Value          | smallmoney | Wartość płatności.                     |
| CartID         | int        | Identyfikator koszyka.                 |
| IsSuccessful   | bit        | Informacja o pomyślności transakcji.   |
| CreatedAt      | datetime   | Data utworzenia płatności.             |
