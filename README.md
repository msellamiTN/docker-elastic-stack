# Pile Elastic (ELK) sur Docker

[! [Version Elastic Stack] (https://img.shields.io/badge/Elastic%20Stack-7.10.2-00bfb3?style=flat&logo=elastic-stack)] (https://www.elastic.co/ blog / catégorie / communiqués)
 

Exécutez la dernière version de [Elastic stack] [elk-stack] avec Docker et Docker Compose.

Il vous donne la possibilité d'analyser n'importe quel ensemble de données en utilisant les capacités de recherche / agrégation d'Elasticsearch et
la puissance de visualisation de Kibana.

*: information_source: Les images Docker supportant cette pile incluent [X-Pack] [xpack] avec [fonctionnalités payantes] [fonctionnalités payantes]
activé par défaut (voir [Comment désactiver les fonctionnalités payantes] (# comment désactiver les fonctionnalités payantes) pour les désactiver). **Le procès
license] [trial-license] est valable 30 jours **. Une fois cette licence expirée, vous pouvez continuer à utiliser les fonctionnalités gratuites
de manière transparente, sans perdre de données. *

Basé sur les images Docker officielles d'Elastic:

* [Elasticsearch] (https://github.com/elastic/elasticsearch/tree/master/distribution/docker)
* [Logstash] (https://github.com/elastic/logstash/tree/master/docker)
* [Kibana] (https://github.com/elastic/kibana/tree/master/src/dev/build/tasks/os_packages/docker_generator)

Autres variantes de piles disponibles:

* [`tls`] (https://github.com/deviantony/docker-elk/tree/tls): cryptage TLS activé dans Elasticsearch.
* [`searchguard`] (https://github.com/deviantony/docker-elk/tree/searchguard): prise en charge de Search Guard

---

## Philosophie

Nous visons à fournir l'entrée la plus simple possible dans la pile Elastic pour quiconque a envie d'expérimenter avec
ce puissant combo de technologies. La configuration par défaut de ce projet est volontairement minimale et sans opinion. Il
ne repose sur aucune dépendance externe ou automatisation personnalisée pour faire fonctionner les choses.

Au lieu de cela, nous croyons en une bonne documentation afin que vous puissiez utiliser ce référentiel comme modèle, le peaufiner et en faire _votre
posséder_. [sherifabdlnaby / elastdocker] [elastdocker] est un exemple parmi d'autres de projet qui s'appuie sur cette idée.

---

## Contenu

1. [Exigences] (# exigences)
   * [Host setup] (# host-setup)
   * [SELinux] (# selinux)
   * [Docker for Desktop] (# docker-for-desktop)
     * [Windows] (# fenêtres)
     * [macOS] (# macos)
1. [Utilisation] (# utilisation)
   * [Sélection de version] (# sélection de version)
   * [Amener la pile] (# remonter la pile)
   * [Nettoyage] (# nettoyage)
   * [Configuration initiale] (# configuration initiale)
     * [Configuration de l'authentification utilisateur] (# setup-up-user-authentication)
     * [Injecting data] (# injecting-data)
     * [Création du modèle d'index Kibana par défaut] (# default-kibana-index-pattern-creation)
1. [Configuration] (# configuration)
   * [Comment configurer Elasticsearch] (# how-to-configure-elasticsearch)
   * [Comment configurer Kibana] (# how-to-configure-kibana)
   * [Comment configurer Logstash] (# how-to-configure-logstash)
   * [Comment désactiver les fonctionnalités payantes] (# comment désactiver les fonctionnalités payantes)
   * [Comment faire évoluer le cluster Elasticsearch] (# how-to-scale-out-the-elasticsearch-cluster)
   * [Comment réinitialiser un mot de passe par programme] (# comment-réinitialiser-un-mot de passe-par programme)
1. [Extensibilité] (# extensibilité)
   * [Comment ajouter des plugins] (# how-to-add-plugins)
   * [Comment activer les extensions fournies] (# how-to-enable-the-provided-extensions)
1. [Réglage JVM] (# jvm-tuning)
   * [Comment spécifier la quantité de mémoire utilisée par un service] (# comment-spécifier-la-quantité-de-mémoire-utilisée-par-un-service)
   * [Comment activer une connexion JMX distante à un service] (# how-to-enable-a-remote-jmx-connection-to-a-service)
1. [Aller plus loin] (# aller-plus loin)
   * [Plugins et intégrations] (# plugins-et-intégrations)
   * [Mode essaim] (# mode essaim)

## Exigences

### Configuration de l'hôte

* [Docker Engine] (https://docs.docker.com/install/) version ** 17.05 ** ou plus récente
* [Docker Compose] (https://docs.docker.com/compose/install/) version ** 1.20.0 ** ou plus récente
* 1,5 Go de RAM

*: information_source: Surtout sous Linux, assurez-vous que votre utilisateur dispose des [permissions requises] [linux-postinstall] pour
interagir avec le démon Docker. *

Par défaut, la pile expose les ports suivants:

* 5044: entrée Logstash Beats
* 5000: entrée TCP Logstash
* 9600: API de surveillance Logstash
* 9200: HTTP Elasticsearch
* 9300: Transport TCP Elasticsearch
* 5601: Kibana

**: avertissement: les [bootstrap checks] [booststap-checks] d'Elasticsearch ont été volontairement désactivés pour faciliter la configuration du
Pile élastique dans les environnements de développement. Pour les configurations de production, nous recommandons aux utilisateurs de configurer leur hôte selon
les instructions de la documentation Elasticsearch: [Configuration système importante] [es-sys-config]. **

### SELinux

Sur les distributions sur lesquelles SELinux est activé prêt à l'emploi, vous devrez soit recontacter les fichiers, soit définir SELinux
en mode Permissive pour que docker-elk démarre correctement. Par exemple sur Redhat et CentOS, ce qui suit sera
appliquer le bon contexte:

`` `console
$ chcon -R system_u: object_r: admin_home_t: s0 docker-elk /
''

### Docker pour ordinateur de bureau

#### Les fenêtres

Assurez-vous que la fonction [Shared Drives] [win-shareddrives] est activée pour le lecteur `C:`.

#### macOS

La configuration par défaut de Docker pour Mac permet de monter des fichiers depuis `/ Users /`, `/ Volumes /`, `/ private /`, et `/ tmp`
exclusivement. Assurez-vous que le référentiel est cloné dans l'un de ces emplacements ou suivez les instructions du
[documentation] [mac-mounts] pour ajouter plus d'emplacements.

## Utilisation

### Sélection de la version

Ce référentiel essaie de rester aligné sur la dernière version de la pile Elastic. La branche `main` suit le courant
version majeure (7.x).

Pour utiliser une version différente des composants Elastic de base, changez simplement le numéro de version dans le fichier `.env`. Si
vous mettez à niveau une pile existante, veuillez lire attentivement la note de la section suivante.

**: avertissement: faites toujours attention aux [instructions de mise à niveau officielles] [mise à niveau] pour chaque composant individuel avant
effectuer une mise à niveau de la pile. **

Les anciennes versions majeures sont également prises en charge sur des branches distinctes:

* [`release-6.x`] (https://github.com/deviantony/docker-elk/tree/release-6.x): série 6.x
* [`release-5.x`] (https://github.com/deviantony/docker-elk/tree/release-5.x): série 5.x (fin de vie)

### Monter la pile

Clonez ce référentiel sur l'hôte Docker qui exécutera la pile, puis démarrez les services localement à l'aide de Docker Compose:

`` `console
$ docker-compose up
''

Vous pouvez également exécuter tous les services en arrière-plan (mode détaché) en ajoutant l'indicateur `-d` à la commande ci-dessus.

**: avertissement: vous devez reconstruire les images de la pile avec `docker-compose build` chaque fois que vous changez de branche ou mettez à jour le
version d'une pile déjà existante. **

Si vous démarrez la pile pour la toute première fois, veuillez lire attentivement la section ci-dessous.

### Nettoyer

Les données Elasticsearch sont conservées par défaut dans un volume.

Pour arrêter complètement la pile et supprimer toutes les données persistantes, utilisez la commande Docker Compose suivante:

`` `console
$ docker-compose down -v
''

## La configuration initiale

### Configuration de l'authentification utilisateur

*: information_source: Reportez-vous à [Comment désactiver les fonctionnalités payantes] (# comment désactiver les fonctionnalités payantes) pour désactiver l'authentification. *

La pile est préconfigurée avec l'utilisateur d'amorçage ** privilégié ** suivant:

* utilisateur: * elastic *
* mot de passe: * changeme *

Bien que tous les composants de la pile fonctionnent prêts à l'emploi avec cet utilisateur, nous vous recommandons vivement d'utiliser la fonction non privilégiée [built-in
utilisateurs] [utilisateurs intégrés] à la place pour une sécurité accrue.

1. Initialisez les mots de passe pour les utilisateurs intégrés

    `` `console
    $ docker-compose exec -T elasticsearch bin/elasticsearch-setup-passwords auto --batch
    ''

    Les mots de passe des 6 utilisateurs intégrés seront générés aléatoirement. Prenez-en note.

1. Désinitialisez le mot de passe d'amorçage (_optionnel_)

    Supprimez la variable d'environnement `ELASTIC_PASSWORD` du service` elasticsearch` dans le fichier Compose
    (`docker-compose.yml`). Il n'est utilisé que pour initialiser le magasin de clés lors du démarrage initial d'Elasticsearch.

1. Remplacez les noms d'utilisateur et les mots de passe dans les fichiers de configuration

    Utilisez l'utilisateur `kibana_system` (` kibana` pour les versions <7.8.0) dans le fichier de configuration de Kibana
    (`kibana / config / kibana.yml`) et l'utilisateur` logstash_system` dans le fichier de configuration Logstash
    (`logstash / config / logstash.yml`) à la place de l'utilisateur` élastique` existant.

    Remplacez le mot de passe de l'utilisateur `elastic` dans le fichier de pipeline Logstash (` logstash / pipeline / logstash.conf`).

    *: information_source: n'utilisez pas l'utilisateur `logstash_system` dans le fichier Logstash ** pipeline **, il n'a pas
    autorisations suffisantes pour créer des index. Suivez les instructions de [Configuration de la sécurité dans Logstash] [ls-security]
    pour créer un utilisateur avec des rôles appropriés. *

    Voir également la section [Configuration] (# configuration) ci-dessous.

1. Redémarrez Kibana et Logstash pour appliquer les modifications

    `` `console
    $ docker-compose restart kibana logstash
    ''

    *: information_source: En savoir plus sur la sécurité de la pile Elastic à [Tutoriel: Premiers pas avec
    sécurité] [tutoriel sec]. *

### Injection de données

Donnez à Kibana environ une minute pour s'initialiser, puis accédez à l'interface utilisateur Web de Kibana en ouvrant <http: // localhost: 5601> dans un site Web
navigateur et utilisez les informations d'identification suivantes pour vous connecter:

* utilisateur: * elastic *
* mot de passe: * \ <votre mot de passe élastique généré> *

Maintenant que la pile est en cours d'exécution, vous pouvez continuer et injecter des entrées de journal. La configuration Logstash expédiée permet
vous pour envoyer du contenu via TCP:

`` `console
# Utilisation de BSD netcat (Debian, Ubuntu, système MacOS, ...)
$ cat /path/to/logfile.log | nc -q0 localhost 5000
''

`` `console
# Utilisation de GNU netcat (CentOS, Fedora, MacOS Homebrew, ...)
$ cat /path/to/logfile.log | nc -c localhost 5000
''

Vous pouvez également charger les exemples de données fournis par votre installation Kibana.

### Création de modèle d'index Kibana par défaut

Lorsque Kibana est lancé pour la première fois, il n'est configuré avec aucun modèle d'index.

#### Via l'interface utilisateur Web de Kibana

*: information_source: vous devez injecter des données dans Logstash avant de pouvoir configurer un modèle d'index Logstash via
l'interface utilisateur Web de Kibana. *

Accédez à la vue _Discover_ de Kibana dans la barre latérale gauche. Vous serez invité à créer un modèle d'index. Entrer
`logstash- *` pour correspondre aux indices Logstash puis, sur la page suivante, sélectionnez `@ timestamp` comme champ de filtre temporel. Finalement,
cliquez sur _Create index pattern_ et revenez à la vue _Discover_ pour inspecter vos entrées de journal.

Reportez-vous à [Connect Kibana with Elasticsearch] [connect-kibana] et [Création d'un modèle d'index] [index-pattern] pour plus de détails
des instructions sur la configuration du modèle d'index.

#### Sur la ligne de commande

Créez un modèle d'index via l'API Kibana:

`` `console
$ curl -XPOST -D- 'http://localhost:5601/api/saved_objects/index-pattern' \
    -H 'Content-Type: application/json' \
    -H 'kbn-version: 7.10.2' \
    -u elastic:<your generated elastic password> \
    -d '{"attributes":{"title":"logstash-*","timeFieldName":"@timestamp"}}'
''

Le modèle créé sera automatiquement marqué comme modèle d'index par défaut dès que l'interface utilisateur de Kibana est ouverte pour le
première fois.

## Configuration

*: information_source: la configuration n'est pas rechargée dynamiquement, vous devrez redémarrer les composants individuels après
tout changement de configuration. *

### Comment configurer Elasticsearch

La configuration Elasticsearch est stockée dans [`elasticsearch/config/elasticsearch.yml`] [config-es].

Vous pouvez également spécifier les options que vous souhaitez remplacer en définissant des variables d'environnement dans le fichier de composition:

`` `yml
elasticsearch:

  environnement:
    network.host: _non_loopback_
    cluster.name: mon-cluster
''

Veuillez consulter la page de documentation suivante pour plus de détails sur la configuration d'Elasticsearch dans Docker
conteneurs: [Installer Elasticsearch avec Docker] [es-docker].

### Comment configurer Kibana

La configuration par défaut de Kibana est stockée dans [`kibana/config/kibana.yml`] [config-kbn].

Il est également possible de mapper tout le répertoire `config` au lieu d'un seul fichier.

Veuillez consulter la page de documentation suivante pour plus de détails sur la configuration de Kibana dans Docker
conteneurs: [Installer Kibana avec Docker] [kbn-docker].

### Comment configurer Logstash

La configuration de Logstash est stockée dans [`logstash/config/logstash.yml`] [config-ls].

Il est également possible de mapper tout le répertoire `config` au lieu d'un seul fichier, mais vous devez être conscient que
Logstash attendra un fichier [`log4j2.properties`] [log4j-props] pour sa propre journalisation.

Veuillez consulter la page de documentation suivante pour plus de détails sur la configuration de Logstash dans Docker
conteneurs: [Configuration de Logstash pour Docker] [ls-docker].

### Comment désactiver les fonctionnalités payantes

Faites passer la valeur de l'option `xpack.license.self_generated.type` d'Elasticsearch de` trial` à `basic` (voir [Licence
settings] [licence d'essai]).

### Comment faire évoluer le cluster Elasticsearch

Suivez les instructions du Wiki: [Scaling out Elasticsearch] (https://github.com/deviantony/docker-elk/wiki/Elasticsearch-cluster)

### Comment réinitialiser un mot de passe par programmation

Si, pour une raison quelconque, vous ne parvenez pas à utiliser Kibana pour modifier le mot de passe de vos utilisateurs (y compris [intégré
users] [builtin-users]), vous pouvez utiliser l'API Elasticsearch à la place et obtenir le même résultat.

Dans l'exemple ci-dessous, nous réinitialisons le mot de passe de l'utilisateur `Elastic` (remarquez" / user / Elastic "dans l'URL):

`` `console
$curl -XPOST -D- 'http://localhost:9200/_security/user/elastic/_password' \
    -H 'Content-Type: application/json' \
    -u elastic:<your current elastic password> \
    -d '{"password" : "<your new password>"}'
''

## Extensibilité

### Comment ajouter des plugins

Pour ajouter des plugins à n'importe quel composant ELK, vous devez:

1. Ajoutez une instruction `RUN` au` Dockerfile` correspondant (par exemple, `RUN logstash-plugin install logstash-filter-json`)
1. Ajoutez la configuration de code de plugin associée à la configuration du service (par exemple, entrée / sortie Logstash)
1. Reconstruisez les images en utilisant la commande `docker-compose build`

### Comment activer les extensions fournies

Quelques extensions sont disponibles dans le répertoire [`extensions`] (extensions). Ces extensions fournissent des fonctionnalités qui
ne font pas partie de la pile Elastic standard, mais peuvent être utilisées pour l'enrichir avec des intégrations supplémentaires.

La documentation de ces extensions est fournie dans chaque sous-répertoire individuel, sur une base par extension. Certains
d'entre eux nécessitent des modifications manuelles de la configuration ELK par défaut.

## Réglage JVM

### Comment spécifier la quantité de mémoire utilisée par un service

Par défaut, Elasticsearch et Logstash commencent avec [1/4 de l'hôte total
memory] (https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/parallel.html#default_heap_size) alloué à
la taille du tas JVM.

Les scripts de démarrage pour Elasticsearch et Logstash peuvent ajouter des options JVM supplémentaires à partir de la valeur d'un environnement
variable, permettant à l'utilisateur d'ajuster la quantité de mémoire qui peut être utilisée par chaque composant:

| Service | Variable d'environnement |
| --------------- | ---------------------- |
| Elasticsearch | ES_JAVA_OPTS |
| Logstash | LS_JAVA_OPTS |

Pour s'adapter aux environnements où la mémoire est rare (Docker pour Mac n'a que 2 Go disponibles par défaut), la taille du tas
l'allocation est limitée par défaut à 256 Mo par service dans le fichier `docker-compose.yml`. Si vous souhaitez remplacer le
configuration JVM par défaut, modifiez la ou les variables d'environnement correspondantes dans le fichier `docker-compose.yml`.

Par exemple, pour augmenter la taille maximale du segment JVM pour Logstash:

`` `yml
logstash:

  environnement:
    LS_JAVA_OPTS: -Xmx1g -Xms1g
''

### Comment activer une connexion JMX distante à un service

En ce qui concerne la mémoire Java Heap (voir ci-dessus), vous pouvez spécifier les options JVM pour activer JMX et mapper le port JMX sur le Docker
hôte.

Mettez à jour la variable d'environnement `{ES, LS} _JAVA_OPTS` avec le contenu suivant (j'ai mappé le service JMX sur le port
18080, vous pouvez changer cela). N'oubliez pas de mettre à jour l'option `-Djava.rmi.server.hostname` avec l'adresse IP de votre
Hôte Docker (remplace ** DOCKER_HOST_IP **):

`` `yml
logstash:

  environnement:
    LS_JAVA_OPTS: -Dcom.sun.management.jmxremote -Dcom.sun.management.jmxremote.ssl = false -Dcom.sun.management.jmxremote.authenticate = false -Dcom.sun.management.jmxremote.port = 18080 -Dcom.sun .management.jmxremote.rmi.port = 18080 -Djava.rmi.server.hostname = DOCKER_HOST_IP -Dcom.sun.management.jmxremote.local.only = false
''

## Aller plus loin

### Plugins et intégrations

Consultez les pages Wiki suivantes:

* [Applications externes] (https://github.com/deviantony/docker-elk/wiki/External-applications)
* [Intégrations populaires] (https://github.com/deviantony/docker-elk/wiki/Popular-integrations)

### Mode essaim

Le support expérimental de Docker [Swarm mode] [swarm-mode] est fourni sous la forme d'un fichier `docker-stack.yml`, qui peut
être déployé dans un cluster Swarm existant à l'aide de la commande suivante:

`` `console
$ docker stack deploy -c docker-stack.yml elk
''

Si tous les composants sont déployés sans aucune erreur, la commande suivante affichera 3 services en cours d'exécution:

`` `console
$ docker stack services elk
''

*: information_source: pour mettre à l'échelle Elasticsearch en mode Swarm, configurez les hôtes d'origine avec le nom DNS `tasks.elasticsearch`
au lieu de `elasticsearch`. *

[elk-stack]: https://www.elastic.co/what-is/elk-stack
[xpack]: https://www.elastic.co/what-is/open-x-pack
[fonctionnalités payantes]: https://www.elastic.co/subscriptions
[licence d'essai]: https://www.elastic.co/guide/en/elasticsearch/reference/current/license-settings.html

[elastdocker]: https://github.com/sherifabdlnaby/elastdocker

[linux-postinstall]: https://docs.docker.com/install/linux/linux-postinstall/

[booststap-checks]: https://www.elastic.co/guide/en/elasticsearch/reference/current/bootstrap-checks.html
[es-sys-config]: https://www.elastic.co/guide/en/elasticsearch/reference/current/system-config.html

[win-shareddrives]: https://docs.docker.com/docker-for-windows/#shared-drives
[mac-mounts]: https://docs.docker.com/docker-for-mac/osxfs/

[utilisateurs intégrés]: https://www.elastic.co/guide/en/elasticsearch/reference/current/built-in-users.html
[ls-security]: https://www.elastic.co/guide/en/logstash/current/ls-security.html
[tutoriel sec]: https://www.elastic.co/guide/en/elasticsearch/reference/current/security-getting-started.html

[connect-kibana]: https://www.elastic.co/guide/en/kibana/current/connect-to-elasticsearch.html
[index-pattern]: https://www.elastic.co/guide/en/kibana/current/index-patterns.html

[config-es]: ./elasticsearch/config/elasticsearch.yml
[config-kbn]: ./kibana/config/kibana.yml
[config-ls]: ./logstash/config/logstash.yml

[es-docker]: https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html
[kbn-docker]: https://www.elastic.co/guide/en/kibana/current/docker.html
[ls-docker]: https://www.elastic.co/guide/en/logstash/current/docker-config.html

[log4j-props]: https://github.com/elastic/logstash/tree/7.6/docker/data/logstash/config
[esuser]: https://github.com/elastic/elasticsearch/blob/7.6/distribution/docker/src/docker/Dockerfile#L23-L24

[mise à niveau]: https://www.elastic.co/guide/en/elasticsearch/reference/current/setup-upgrade.html

[mode essaim]: https://docs.docker.com/engine/swarm/
