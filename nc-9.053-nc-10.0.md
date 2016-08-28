Dans le courant du mois de juin 2016, au lieu de mettre à jour mon instance ownCloud de la version 8 à la version 9, comme un certain nombre de personne, j'étais passé à NextCloud. Cette migration s'était déroulée de manière similaire à une mise à jour habituelle. Les mises à jour mineures suivantes n'ont pas dérogé à ce principe. Et tout récemment, l'équipe de NextCloud a officiellement sorti la version 10 dans le canal stable.

C'est un peu irrationnel de ma part, peut-être, mais avant de faire cette mise à jour, je voulais m'assurer que les applications que j'utilise le plus étaient encore compatibles. Pour ce qui est de [*Calendar*](https://apps.nextcloud.com/app/calendar) et [*Contacts*](https://apps.owncloud.com/content/show.php/Calendar?content=168707), le doute n'était pas bien grand, par contre je n'étais pas certain de [*Tasks*](https://apps.owncloud.com/content/show.php/Tasks?content=164356), que j'utilise beaucoup, ou [*Notes*](https://github.com/owncloud/notes).

Or, si l'on consulte [apps.nextcloud](https://apps.nextcloud.com), on n'en apprend pas encore beaucoup. Et sur [apps.owncloud](https://apps.owncloud.com/), il est assez rare d'avoir une information sur la compatibilité avec NextCloud... Alors, comment faire ?

J'ai utilisé une image *Docker* : [wonderfall/nextcloud](https://hub.docker.com/r/wonderfall/nextcloud/). Il y a un `docker-compose.yaml` quasi tout prêt, ainsi que la méthode pour compléter le `config.php`, afin de pouvoir ajouter manuellement des applications.

Je précise que je ne maîtrise pas *Docker*, aussi il y a certainement d'autres méthodes. Je me suis placé dans un répertoire créé pour la man&oelig;uvre, bien entendu sur une machine en local, et pas sur mon serveur. J'y ai créé le fichier `docker-compose.yaml` sur l'exemple donné :

```
nextcloud:
  image: wonderfall/nextcloud
  links:
    - db_nextcloud:db_nextcloud
  environment:
    - UID=1000
    - GID=1000
  volumes:
    - /mnt/nextcloud/data:/data
    - /mnt/nextcloud/config:/config
    - /mnt/nextcloud/apps:/apps2

db_nextcloud:
  image: mariadb:10
  volumes:
    - /mnt/nextcloud/db:/var/lib/mysql
  environment:
    - MYSQL_ROOT_PASSWORD=supersecretpassword
    - MYSQL_DATABASE=nextcloud
    - MYSQL_USER=nextcloud
    - MYSQL_PASSWORD=supersecretpassword
```

Il est bien entendu conseillé de changer les `supersecretpassword`, mais comme j'allais rester en local...

Ensuite, on peut lancer le tout avec `docker-compose up -d`. Une fois les images téléchargées, montées et liées, il fallait trouver comment y accéder et comment finir l'installation. On trouve le nom des images montées avec `docker ps -a` et ensuite on peut connaître les IP des images avec `docker inspect [nom de l'image]`. Ce qui est utile pour ouvrir la bonne adresse dans son navigateur, ainsi que pour renseigner correctement le script d'installation, à savoir donner la bonne IP pour joindre le serveur MariaDB.

Une fois cela fait, il reste à modifier le `config.php` pour pouvoir utiliser le répertoire des applications, en ajoutant les lignes suivantes :

```
  "apps_paths" => array (
      0 => array (
              "path"     => "/nextcloud/apps",
              "url"      => "/apps",
              "writable" => false,
      ),
      1 => array (
              "path"     => "/apps2",
              "url"      => "/apps2",
              "writable" => true,
      ),
  ),
```

Ainsi, j'ai pu m'assurer que toutes les applications que j'utilisent sont en effet compatibles. Pour la plupart, je n'ai eu qu'à les activer : *Calendar*, *Contacts*, *Documents*, *Tasks*, mais j'ai dû télécharger [*Announcement Center*](https://github.com/nextcloud/announcementcenter) et *Notes*.

Fort de cette information, j'ai mis à jour mon instance de la version 9.0.53 à la version 10.0, [manuellement, avec la même procédure que d'habitude](https://docs.nextcloud.com/server/10/admin_manual/maintenance/manual_upgrade.html), sans problème.
