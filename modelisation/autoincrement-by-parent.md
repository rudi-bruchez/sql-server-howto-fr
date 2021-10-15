# Gérer plusieurs auto incréments

Comment gérer plusieurs auto-incréments sur une seule table, lorsqu'on doit sous-numéroter une colonne par rapport à une première colonne ?

Exemple classique, une table qui contient un numéro de document, par exemple de facture, par numéro d'agence :

```sql
CREATE TABLE facture (
    NumeroAgence TINYINT NOT NULL,
    NumeroFacture INT NOT NULL,
    CONSTRAINT pk_facture PRIMARY KEY (NumeroFacture, NumeroAgence)
);
```
