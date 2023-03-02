pour cette exemple il faudra que vous supprimiez tous les tablespaces sauf celui par défaut:

Présentation de pgbackrest:

En tant que root:

Installation de pgbackrest:

```
yum install -y pgbackrest
```

Création du dossier de pgbackrest:

```
mkdir /etc/pgbackrest
mv /etc/pgbackrest.conf /etc/pgbackrest
chown -R postgres:postgres /etc/pgbackrest
```

Modifier le fichier `/etc/pgbackrest/pgbackrest.conf` et décommenter les deux lignes. Modifier la version `10` en `14`.

Vous avez une partie global qui permet de renseigner des paramètres globaux de pgbackrest, cela évite d'avoir des commandes à rallonge.

Le `main` entre `[]` est une stanza, c'est un terme pour désigner un commun de l'instance. Ici vous pourriez la modifier comme vous le souhaitez mais on va rester sur main.

`pg1-path` permet de signifier le chemin où est situé le `PGDATA` de l'instance à sauvegarder.

Vous avez aussi d'autres paramètres comme pg1-host, pg1-port etc... Je vous invite à regarder la documentation de pgbackrest pour ça.

Pour cet exemple nous allons rajouter `log-level-console=info` pour afficher les informations des commandes et `repo1-retention-full = 2` pour ne garder qu'une seule sauvegarde FULL. Ces deux paramètres sont des paramètres globaux.

En tant que postgres:

Nous avons maintenant configuré simplement notre pgbackrest, regarder le contenu de `/var/lib/pgbackrest` et les dossiers enfants.

Nous allons maintenant signifier à pgbackrest de créer la stanza.

```
pgbackrest --stanza=main stanza-create
```

Regarder le contenu de `var/lib/pgbackrest` et ses enfants, que constatez-vous ?

Configurer l'`achive_command` et `archive_mode` de votre instance comme suit:

```
archive_commande = '/usr/bin/pgbackrest --stanza=main archive-push %p'
archive_mode = on
wal_level = replica
```

Redémarrer l'instance.

Lancer un backup FULL avec la commande backup de pgbackrest:

```
pgbackrest --stanza=main backup --type=full
```

Ajouter une base de données avec comme nom `pgbackrest`.

Lancer un backup différentiel avec la commande backup de pgbackrest:

```
pgbackrest --stanza=main backup --type=diff
```

Restauration:

Il faut arrêter postgresql.

A l'aide de `pgbackrest info` récupérer l'heure à laquelle, mon backup s'est déroulé à `2023-03-02 11:26:08` qui me servira pour la commande suivante.

Puis on peut restaurer:

```
pgbackrest --stanza=main --delta --type=time --target="2023-03-02 11:26:09" --target-action=promote restore 
```

On redémarre l'instance.

Lister vos bases de données.

Création d'un secondaire:

Pour créer un secondaire il faudra :

Créer un PGDATA pour le secondaire:

```
mkdir ~/14/standby
```

Création du slot de réplicaiton:

```

```



```
pgbackrest --stanza=main --delta --type=standby --recovery-option=primary_conninfo=port=5432 --recovery-option=primary_slot_name=secondaire --recovery-option=recovery_target_timeline=latest --pg1-path=/var/lib/pgsql/14/standby restore
```

