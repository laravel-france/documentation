# Laravel Homestead

- [Introduction](#introduction)
- [Logiciel Inclus](#included-software)
- [Installation & Configuration](#installation-and-setup)
- [Utilisation Quotidienne](#daily-usage)
- [Ports](#ports)

<a name="introduction"></a>
## Introduction

Laravel s'efforce de rendre l'expérience du développement PHP agréable, cela comprend également votre environement de développement. [Vagrant](http://vagrantup.com) fournit un moyen simple et élégant de manager et de mettre à disposition des machines virtuelles.

Laravel Homestead est une "box" Vagrant officiel et pré-emballé qui vous offre un merveilleux environnement de développement sans avoir à installer PHP, HHVM, un server Web et tout autre logiciel sur votre machine local. Plus besoin de s'inquiéter de gâcher votre système d'exploitation ! Les "box" Vagrant sont complètement jetables. Si quelque chose tourne mal, vous pouvez détuire et recréer votre "box" en quelques minutes !

Homestead fonctionne sur n'importe quel ordinateur Windows, Mac et Linux, et inclus un serveur web Nginx, PHP 5.6, MySQL, Postgres, Redis, Memcached et tous les autres goodies dont vous aurez besoin pour développer.

> **Note:** Si vous utilisez Windows, vous devrez peut-être activer la virtualisation matérielle (VT-x). Généralement, elle peut-être activée via votre BIOS.

Homestead est actuellement construit et testé avec Vagrant 1.6.

<a name="included-software"></a>
## Logiciel Inclus

- Ubuntu 14.04
- PHP 5.6
- HHVM
- Nginx
- MySQL
- Postgres
- Node (avec Bower, Grunt, et Gulp)
- Redis
- Memcached
- Beanstalkd
- [Laravel Envoy](/docs/ssh#envoy-task-runner)
- Extension Fabric + HipChat

<a name="installation-and-setup"></a>
## Installation & Configuration

### Installation de VirtualBox & Vagrant

Avant de lancer l'installation de votre environement Homestead, vous devez installer [VirtualBox](https://www.virtualbox.org/wiki/Downloads) et [Vagrant](http://www.vagrantup.com/downloads.html). Ces deux installations possèdent une interface d'installation facile pour les systèmes d'exploitation populaires.

### Ajouter la box Vagrant

Une fois VirtualBox et Vagrant installés, vous devez ajouter la box `laravel/homestead` à votre installation vagrant en utilisant la commande suivante dans votre console. Cela devrait prendre quelques minutes pour télécharger la boîte, suivant votre vitesse de connexion :

	vagrant box add laravel/homestead

### Cloner le repository Homestead

Après avoir ajouté la box à votre installation Vagrant, vous devrez cloner ou télécharger son repository. Considérons que vous clonez le repository dans le dossier centrale `homestead` où vous conservez tous vos projets Laravel, tout comme la box Homestead servira d'hôte à l'ensemble de vos projets Laravel (et autres PHP).

	git clone https://github.com/laravel/homestead.git Homestead

### Configurer Votre Clé SSH

Ensuite, éditez le fichier `Homestead.yaml` présent dans le repository. Dans ce fichier, vous pouvez configurer le chemin de votre clé SSH public, ainsi que les dossiers que vous souhaitez partager entre votre machine prncipale et la machine virtuelle Homestead.

Vous n'avez pas de clé SSH ? Sur Mac ou Linux, vous pouvez la générer avec la command suivante :

	ssh-keygen -t rsa -C "your@email.com"

Sur Windows, vous pouvez installer [Git](http://git-scm.com) et utiliser le shell `Git Bash` inclus avec Git pour émettre la commande ci-dessus. Sinon, vous pouvez utiliser [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) et [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Une fois la clé SSH créée, spécifiez son chemin dans la propriété `autorize` de votre fichier `Homestead.yaml`.

### Configurer Vos Dossiers Partagés

La propriété `folders` de votre `Homestead.yaml` listes l'ensemble des dossiers que vous souhaitez partager avec votre environnement Homestead. Tant que les fichiers seront modifiés dans ces dossiers, ils seront synchronisés entre votre machine local et votre environnement Homestead. Vous pouvez configurer autant de dossier que nécessaires !

### Configurer Vos Sites Nginx

Vous n'êtes pas familier avec Nginx ? Pas de problème. La propriété `sites` permet de mapper facilement un "domaine" vers un dossier de votre environement Homestead. Un exemple de configuration est inclus dans le fichier `Homestead.yaml`. De plus, vous pouvez ajouter autant de sites dans votre environement Homestead que nécessaire. Homestead peut servir d'environement de virtualisation simple pour tous les projets Laravel sur lesquels vous travaillez ! 

Vous pouvez faire n'importe quel site Homestead en utilisant [HHVM](http://hhvm.com) en règlant l'option `hhvm`à `true` :

	sites:
	    - map: homestead.app
	      to: /home/vagrant/Code/Laravel/public
	      hhvm: true

### Alias Bash

Pour ajouter un alias Bash à votre box Homestead, ajoutez-le simplement au fichier `aliases` à la racine de votre dossier Homestead.

### Démarrer La Box Vagrant

Une fois que vous avez édité le fichier `homestead.yaml` comme vous le souhaitez, lancez la commande `vagrant up` depuis le dossier Homestead dans votre terminal. Vagrant devrait démarrer la machine virtuel et confugurer vos dossiers partagés et Nginx automatiquement !

N'oubliez pas d'ajouter les "domaines" de vos sites Nginx dans le fichier `hosts` de votre machine ! Le fichier `hosts` redirige les requêtes de vos domaines local dans votre environement Homestead. Sur Mac et Linux, ce fichier se situe dans `/etc/hosts`. Sur Windows, il est présent dans `C:\Windows\System32\drivers\etc\hosts`. Les lignes à ajouter devraient ressembler à :

	127.0.0.1  homestead.app

Une fois que vous avez ajouté le domaine à votre fichier `hosts`, vous pouvez accéder au site via votre navigateur web sur le port 8000 !

	http://homestead.app:8000

Pour savoir comment vous connecter à vos bases de données, lisez la suite!

<a name="daily-usage"></a>
## Utilisation Quotidienne

### Connexion Via SSH

Pour vous connecter à votre environement Homestead via SSH, vous devez vous connecter à `127.0.0.1` sur le port 2222 en utilisant la clé SSH que vous avez spécifiée dans votre fichier `Homestead.yaml`. Vous pouvez utiliser simplement la commande `vagrant ssh` depuis votre dossier Homestead.

Si vous voulez encore plus de confort, il peut-être utile d'ajouter  l'alias suivant à votre `~/.bash_aliases` ou `~/.bash_profile`

	alias vm='ssh vagrant@127.0.0.1 -p 2222'

### Connection A Vos Bases De Données

Une base de données `Homestead` est configurée pour utiliser MySQL et Posgres en dehors de la box. Pour plus de confort, la configuration d'une base de données `local` de Laravel est configuré pour utiliser cette base par défaut.

Pour vous connecter à votre base de données MySQL ou Postgres de votre machine principale via Navicat ou Sequel Pro, vous devez vous connecter à `127.0.0.1` et le port 33060 (MySQL) ou 54320 (Postgres). Le nom d'utilisateur et mot de passe pour les deux bases de données est `homestead`/` secret`.


> ** Note: ** Vous ne devez utiliser ces ports non standards que lors de la connexion aux bases de données à partir de votre machine principale. Vous devez utiliser la valeur par défaut les ports 3306 et 5432 dans votre fichier de configuration de base de données Laravel lorsque Laravel fonctionne _dans_ la machine virtuelle.

### Ajouter Des Sites Supplémentaires

Une fois que votre environnement Homestead est provisionné et en cours d'exécution, vous pouvez ajouter des sites supplémentaires Nginx pour vos applications Laravel. Vous pouvez exécuter autant d'installations Laravel que vous souhaitez sur un environnement de Homestead unique. Il y a deux façons de le faire : d'abord, vous pouvez simplement ajouter des sites à votre fichier `Homestead.yaml` puis exécuter `vagrant provision`.

Sinon, vous pouvez utiliser le script `serve` qui est disponible sur votre environnement Homestead. Pour utiliser le script `serve`, SSH dans votre environnement Homestead et exécutez la commande suivante:

	serve domain.app /home/vagrant/Code/path/to/public/directory

> ** Note: ** Après l'exécution de la commande `serve`, n'oubliez pas d'ajouter le nouveau site dans le fichier `hosts` sur votre machine principale !

<a name="ports"></a>
## Ports

Les ports suivants sont transférés à votre environnement de Homestead :

- **SSH:** 2222 -> transferé vers 22
- **HTTP:** 8000 -> transferé vers 80
- **MySQL:** 33060 -> transferé vers 3306
- **Postgres:** 54320 -> transferé vers 5432
