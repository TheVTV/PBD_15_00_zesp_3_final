## Triggery

### Dawanie dostępu studentom, którzy opłacili zamówienie (trg_SetAccessAfterPayment)

```sql
CREATE TRIGGER trg_SetAccessAfterPayment
ON Cart
AFTER INSERT, UPDATE
AS
BEGIN
  UPDATE Cart
  SET Access = 1
  WHERE CartID IN (
    SELECT CartID
    FROM inserted
    WHERE Paid = dbo.GetCartValue(CartID)
  ) AND Access = 0;
END;
```

### Aktualizacja obecności (trg_UpdatePresence)

```sql
CREATE TRIGGER trg_UpdatePresence
ON StudyLessonPresence
AFTER INSERT, UPDATE
AS
BEGIN
    SET NOCOUNT ON;

    -- Aktualizacja wartości Presence na 1, gdy MakeUpDate nie jest NULL
    UPDATE SLP
    SET Presence = 1
    FROM StudyLessonPresence SLP
    INNER JOIN inserted i
        ON SLP.StudentID = i.StudentID
        AND SLP.LessonID = i.LessonID
    WHERE i.MakeUpDate IS NOT NULL;
END;
```
