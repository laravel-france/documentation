# Installation

- [Installation de Composer](#install-composer)
- [Installation de Laravel](#install-laravel)
- [Prérequis](#server-requirements)
- [Configuration](#configuration)
- [Permissions](#permissions)
- [Des URLs propres](#pretty-urls)

<a name="install-composer"></a>
## Installation de Composer

Laravel utilise [Composer](http://getcomposer.org) pour gérer ses dépendances. Premièrement, téléchargez une copie de `composer.phar`. Une fois que vous avez l'archive PHAR, vous pouvez soit la laisser dans le dossier local de votre projet, soit la déplacer vers `usr/local/bin` pour l'utiliser de manière globale sur votre système. Sur Windows, vous pouvez utiliser l'installeur de Composer [pour Windows](https://getcomposer.org/Composer-Setup.exe).

<a name="install-laravel"></a>
## Installation de Laravel

### Via Laravel Installer

D'abord, téléchargez [l'archive PHAR de Laravel installer](http://laravel.com/laravel.phar). Pour des raisons pratique, nous vous conseillons de renommer le fichier en `laravel` et de le déplacer dans `/usr/local/bin`. Une fois installé, la simple commande `laravel new` vous créera une installation fraiche de Laravel dans le dossier spécifié. Par exemple, `laravel new blog` créera un dossier `blog` qui contiendra une installation fraiche de Laravel avec toutes les dépendances installées. Cette méthode est plus rapide que celle via Composer

### Via Composer Create-Project

Vous pouvez également installer Laravel en exécutant la commande `create-project` de composer dans votre terminal :

    composer create-project laravel/laravel --prefer-dist

### Via un téléchargement

Une fois que Composer est installé, téléchargez la [dernière version](https://github.com/laravel/laravel/archive/master.zip) du framework, et extrayez son contenu dans un dossier sur votre serveur. Ensuite, à la racine de votre application Laravel, lancez la commande `php composer.phar install` pour installer toutes les dépendances du framework. Ce process requis que git soit installé sur le serveur pour terminer l'installation.

Si vous souhaitez mettre à jour le framework Laravel, vous pouvez exécuter la commande `php composer.phar update`.

<a name="server-requirements"></a>
## Prérequis

Le framework Laravel a quelques prérequis système :

- PHP >= 5.3.7
- L'extension PHP MCrypt

 > Quant à PHP 5.5, certaines distributions d'OS peuvent requérir l'installation manuelle de l'extension PHP JSON. Pour Ubuntu, la commande est `apt-get install php5-json`.

<a name="configuration"></a>
## Configuration

Laravel n'a presque pas besoin de configuration pour fonctionner. En fait, vous êtes libre de commencer à développer ! Cependant, vous devriez au minimum jeter un oeil au fichier `app/config/app.php` et à sa documentation. Il contient plusieurs options comme `timezone` et `locale` que vous pourriez vouloir changer pour votre application.

<a name="permissions"></a>
### Permissions

Laravel peut avoir besoin que le serveur web ait un accès en écriture sur les dossiers à l'intérieur de `app/storage`.

<a name="paths"></a>
### Chemins

Plusieurs chemins des dossiers du Framework sont configurables. Pour changer leurs positions, regardez le fichier `bootstrap/paths.php`.

<a name="pretty-urls"></a>
## Des URLs propres

Le framework est fourni avec un fichier `public/.htaccess` qui est utilisé pour autoriser les URLs sans `index.php`. Si vous utilisez Apache pour servir votre application Laravel, veuillez vous assurer que le module `mod_rewrite` est actif.

Si le fichier `.htaccess` fourni avec Laravel ne fonctionne pas, essayez celui ci :

    Options +FollowSymLinks
    RewriteEngine On

    RewriteCond %{REQUEST_FILENAME} !-d
    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [L]
