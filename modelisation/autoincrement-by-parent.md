# Gérer plusieurs auto incréments

Comment gérer plusieurs auto-incréments sur une seule table, lorsqu'on doit sous-numéroter une colonne par rapport à une première colonne ?

Exemple classique, une table qui contient un numéro de document, par exemple de facture, par numéro d'agence :

```sql
CREATE TABLE dbo.facture (
    NumeroAgence TINYINT NOT NULL,
    NumeroFacture INT NOT NULL,
    CONSTRAINT pk_facture PRIMARY KEY (NumeroFacture, NumeroAgence)
);
```

Exemple de procédure

```sql
CREATE PROCEDURE [dbo].[AjouteFacture]
    @NumeroAgence tinyint
AS BEGIN
    SET NOCOUNT ON

    DECLARE @new_id int;

    SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;

    BEGIN TRY

        BEGIN TRANSACTION;

        SELECT @new_id = ISNULL(max(NumeroFacture),0) + 1 
        FROM facture
        WHERE NumeroAgence = @NumeroAgence ;

        INSERT INTO facture (NumeroAgence, NumeroFacture)
        VALUES (@NumeroAgence, @new_id) ;

        COMMIT TRAN

        SELECT @new_id AS id

    END TRY
    BEGIN CATCH
        ROLLBACK TRAN;

        SELECT NULL AS id;

        ;THROW;

    END CATCH

    SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

    RETURN @new_id
END
GO
```


Exemple de déclencheur

```sql
CREATE TRIGGER atr_i_facture_autoincrement
   ON dbo.facture
   AFTER INSERT
AS BEGIN
   IF @@ROWCOUNT = 0 RETURN;

   UPDATE dbo.facture
   SET NumeroFacture = (SELECT COALESCE(MAX(NumeroFacture),0) + 1 FROM facture)
   FROM INSERTED i
   JOIN dbo.facture f
   WHERE i.NumeroFacture IS NULL
   AND i.NumeroAgence = f.NumeroAgence;

END;
```
