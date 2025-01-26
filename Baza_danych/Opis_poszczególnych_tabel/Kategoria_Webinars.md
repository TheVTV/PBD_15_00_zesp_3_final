# Kategoria Webinars

### Tabela Webinars

- kod DDL

```sql
CREATE TABLE Webinars (
    WebinarID int  NOT NULL,
    Name varchar(50)  NOT NULL,
    Price smallmoney  NULL,
    VideoLink varchar(50)  NOT NULL,
    Date datetime  NOT NULL,
    TeacherID int  NOT NULL,
    TranslatorID int  NULL,
    LanguageID int  NOT NULL,
    Description varchar(100)  NULL,
    Duration time(7)  NOT NULL,
    CONSTRAINT Webinars_pk PRIMARY KEY  (WebinarID)
);
```

- Opis

| Nazwa atrybutu | Typ          | Opis/Uwagi                                              |
| -------------- | ------------ | ------------------------------------------------------- |
| WebinarID      | int          | Klucz główny. Identyfikator webinaru.                   |
| Name           | varchar(50)  | Nazwa webinaru.                                         |
| Price          | smallmoney   | Cena webinaru. Może być pusta (NULL).                   |
| VideoLink      | varchar(50)  | Link do nagrania webinaru.                              |
| Date           | datetime     | Data przeprowadzenia webinaru.                          |
| TeacherID      | int          | Identyfikator nauczyciela prowadzącego webinar.         |
| TranslatorID   | int          | Identyfikator tłumacza. Może być NULL.                  |
| LanguageID     | int          | Identyfikator języka, w którym prowadzony jest webinar. |
| Description    | varchar(100) | Opis webinaru. Może być pusty (NULL).                   |
| Duration       | time(7)      | Czas trwania webinaru.                                  |
