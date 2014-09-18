# Laravel Homestead

- [Introduction](#introduction)
- [Logiciel Inclus](#included-software)
- [Installation & Configuration](#installation-and-setup)
- [Utilisation Quotidienne](#daily-usage)
- [Ports](#ports)

<a name="introduction"></a>
## Introduction

Laravel s'efforce de rendre l'expérience du développement PHP agréable, y compris votre environement de développement. [Vagrant](http://vagrantup.com) fournit un moyen simple et élégant de manager et de mettre à disposition des machines virtuelles.

Laravel Homestead est une "boite" officiel et pré-emballé Vagrant qui vous offre un merveilleux environnement de développement sans avoir à installer PHP, HHVM, un server Web et tout autre logiciel sur votre machine local. Plus besoin de s'inquiéter de gâcher votre système d'opération ! Les "boîtes" Vagrant sont complètement jetable. Si quelque chose tourne mal, vous pouvez détuire et recréer votre "boîte" en quelques minutes !

Homestead fonctionne sur n'importe quel ordinateur Windows, Mac et Linux, et inclus un serveur web Nginx, PHP 5.6, MySQL, Postgres, Redis, Memcached et tous les autres goodies dont vous aurez besoin pour développer.

> **Note:** Si vous utilisez Windows, vous devrez peut-être activé la virtualisation matérielle (VT-x). Généralement, il peut-être activé via votre BIOS.

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

Avant de lancer l'installation de votre environement Homestead, vous devez installer [VirtualBox](https://www.virtualbox.org/wiki/Downloads) et [Vagrant](http://www.vagrantup.com/downloads.html). Ces deux installation possède une interface d'installation facile pour les système d'opération populaire.

### Ajouter la boîte Vagrant

Une fois VirtualBox et Vagrant installés, vous devez ajouter la boîte `laravel/homestead` à votre installation vagrant en utilisant la command suivante dans votre console. Cela devrait prendre quelques minutes pour télécharger la boîte, dependament de votre vitesse de connexion :

	vagrant box add laravel/homestead

### Cloner le repository Homestead

Après avoir ajouté la boîte à votre installation Vagrant, vous devrez cloner ou télécharger son repository. Considérons que vous clonez le repository dans le dossier centrale `homestead` où vous conservez tous vos projet Laravel, comme la boîte Homestead servira d'hôte à l'ensemble de vos projets Laravel (et autres PHP).
	git clone https://github.com/laravel/homestead.git Homestead

### Configurer Votre Clé SSH

Ensuite, éditez le fichier `Homestead.yaml` présent dans le repository. Dans ce fichier, vous pouvez configurer le chemin de votre clé ssh public,  aussi bien que le dossier que vous souhaitez partager entre votre machine prncipale et la machine virtuelle Homestead.

Vous n'avez pas de clé SSH ? Sur Mac ou Linux, vous pouvez la générer avec la command suivante :

	ssh-keygen -t rsa -C "your@email.com"

Sur Windows, vous pouvez installer [Git](http://git-scm.com) et utiliser le shell `Git Bash` inclus avec Git pour émettre la commande ci-dessus. Sinon, vous pouvez utiliser [Putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) et [PuTTYgen](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html).

Une fois la clé SSH crée, spécifiez son chemin dans la propriété àutorize` de votre fichier `Homestead.yaml`.

### Configurer Vos Dossiers Partagés

La propriété `dossiers` de votre `Homestead.yaml` listes l'ensemble des dossiers que vous souhaitez partager avec votre environnement Homestead. Tant que les fichiers contenu dans ces dossiers seront modifié, ils seront synchronisés entre votre machine local et votre environnement Homestead. Vous pouvez configurer autant de dossier que nécessaires ! 

### Configurer Vos Sites Nginx

Vous n'êtes pas familier avec Nginx ? pas de problème. La propriété `sites` permet de mapper facilement un "domaine" vers un dossier de votre environement Homestead. Un exemple de configuration est inclus dans le fichier `Homestead.yaml`. De plus, vous pouvez ajouter autant de sites dans votre environement Homestead que nécessaire. Homestead peut servir d'environement de virtualisation pratique pour chaque projet Laravel sur lequel vous travaillez ! 

Vous pouvez faire n'importe quel site Homestead en utilisant [HHVM](http://hhvm.com) en règlant l'option `hhvm`à `true` :

	sites:
	    - map: homestead.app
	      to: /home/vagrant/Code/Laravel/public
	      hhvm: true

### Alias Bash

Pour ajouter un alias Bash à votre boîte Homestead, ajoutez-le simplement au fichier `aliases` à la racine de votre dossier Homestead.

### Démarrer La Boîte Vagrant

Une fois que vous avez éditer le fichier `homestead.yaml` comme vous le souhaitez, lancez la commande `vagrant up` depuis le dossier Homestead dans votre terminal. Vagrant devrait démarrer la machine virtuel et confugurer vos dossier partager et Nginx automatiquement !

N'oubliez pas d'ajouter les "domaines" de vos sites Nginx dans le fichier `hosts` de votre machine ! le fichier `hosts` redirige vos requêtes de vos domaines local dans votre environement Homestead. Sur Mac et Linux, ce fichier ce situe dans `/etc/hosts`. Sur Windows, il est présent dans `C:\Windows\System32\drivers\etc\hosts`. Les lignes à ajouter devraent ressembler à :

	127.0.0.1  homestead.app

Une fois que vous avez ajouté le domain à votre fichier `hosts`, vous pouvez accéder au site via votre navigateur web sur le port 8000 !

	http://homestead.app:8000

Pour savoir comment vous connecter à vos bases de données, lisez la suite!

<a name="daily-usage"></a>
## Usage Quotidien

### Connection Via SSH

Pour vous connecter à votre environement Homestead via SSH, vous devrez vous connecter à `127.0.0.1`sur le port 2222 en utilisant la clé SSH que vous avez spécifié dans votre fichier `Homestead.yaml`. Vous pouvez utiliser simplement la commande `vagrant ssh`depuis votre dossier Homestead.

Si vous voulez encore plus de confort, il peut-être utile d'ajouter  l'alias suivant à votre `~/.bash_aliases` ou `~/.bash_profile`

	alias vm='ssh vagrant@127.0.0.1 -p 2222'

### Connection A Vos Bases De Données

Une base de données `Homestead`est configuré pour utiliser MySQL et Posgres en dehors de la boîte. Pour plus de confort, une configuration d'une base de données `local`de Laravel est configuré pour utiliser cette base par défaut.

Pour vous connecter à votre base de données MySQL ou Postgres de votre machine principale via Navicat ou Sequel Pro, vous devez vous connecter à `127.0.0.1` et le port 33060 (MySQL) ou 54320 (Postgres). Le nom d'utilisateur et mot de passe pour les deux bases de données est `homestead` /` secret`.


> ** Note: ** Vous devez utiliser ces ports non standards que lors de la connexion aux bases de données à partir de votre machine principale. Vous allez utiliser la valeur par défaut les ports 3306 et 5432 dans votre fichier de configuration de base de données Laravel depuis Laravel fonctionne _dans_ la machine virtuelle.

### Ajouter Des Sites Supplémentaires

Une fois que votre environnement Homestead est provisionné et en cours d'exécution, vous pouvez ajouter des sites supplémentaires Nginx pour vos applications Laravel. Vous pouvez exécuter le plus grand nombre d'installations Laravel que vous souhaitez sur un environnement de Homestead unique. Il ya deux façons de le faire: d'abord, vous pouvez simplement ajouter des sites à votre fichier `Homestead.yaml` puis exécutez `vagrant provision`.

Alternativement, vous pouvez utiliser le script `serve` qui est disponible sur votre environnement Homestead. Pour utiliser le script `serve`, SSH dans votre environnement Homestead et exécutez la commande suivante:

	serve domain.app /home/vagrant/Code/path/to/public/directory

> ** Note: ** Après l'exécution de la commande `serve`, n'oubliez pas d'ajouter le nouveau site dans le fichier `hosts` sur votre machine principale!

<a name="ports"></a>
## Ports

Les ports suivants sont transmis à votre environnement de Homestead :

- **SSH:** 2222 -> transferé vers 22
- **HTTP:** 8000 -> transferé vers 80
- **MySQL:** 33060 -> transferé vers 3306
- **Postgres:** 54320 -> transferé vers 5432
