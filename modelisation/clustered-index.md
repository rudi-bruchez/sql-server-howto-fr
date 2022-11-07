# Choisir un index clustered

Il existe deux grands types d’index dans SQL Server : les index clustered et nonclustered.

Leur structure est la même : un arbre de recherche en B-Tree.

- Un index nonclustered copie les données de la clé de l’index dans une structure physique séparée de la table. C’est l’équivalent d’un index à la fin d’un livre. Il ne se mélange pas avec le contenu du livre à proprement parler.
- Un index clustered structure la table. Physiquement, les lignes de la table sont stockées au dernier niveau de l’arbre. C’est l’équivalent d’un dictionnaire, où le livre tout entier est ordonné sur une clé de recherche triée.

## Quand choisir un index clustered ?

L’index clustered est très structurant pour la structure de la table. Une table clustered a les propriétés suivantes :

- Elle doit être maintenue à tout moment dans l’ordre de la clé de l’index clustered.
- La référence à une ligne est la clé de l’index clustered. Cela veut dire que tous les index nonclustered doivent contenir la clé de l’index clustered.

Le choix de l’index clustered est donc important. Principes :

- On ne peut, bien sûr, avoir qu’un seul index clustered sur une table, puisque la table est structurée dans l’index.
- Il est vraiment préférable de placer l’index clustered sur une colonne auto-incrémentale ou en horodatage, qu’on appelle en augmentation monotone. Cela évite la fragmentation de la table et les ralentissements à l’insertion. Le GUID (`UNIQUEIDENTIFIER`) est donc un mauvais choix.
- La clé de l’index clustered étant contenue dans tous les index nonclustered de la table (c’est la référence de la ligne), il vaut mieux que la clé de l’index clustered soit petite pour ne pas augmenter la taille de tous les index (un `INT`, un `BIGINT`, mais pas un `UNIQUEIDENTIFIER`).
- La clé de l’index clustered doit identifier uniquement une ligne. Il est possible de créer une index clustered non unique à l’aide de l’instruction `CREATE CLUSTERED INDEX`. Mais cela oblige SQL Server à ajouter un entier (`INT`) caché supplémentaire pour unifier les valeurs doublonnées. Il faut donc toujours créer un index clustered avec l’instruction `CREATE UNIQUE CLUSTERED INDEX`.

# Choix de l'index clustered pour les performances

La meilleure solution est en général de laisser l’index clustered sur la clé primaire : la clé primaire est unique, et les recherches sur la clé primaire sont nombreuses, dans les clauses de jointure. L’index clustered optimise les jointure en évitant les lookups (recherche des lignes correspondant à la référence d’une clé d’index).

<p align="right">
<i><small>[<a href="https://www.pachadata.com/contact/">Besoin de services avec SQL Server ? Contactez-moi</a>]</small></i>
</p>
