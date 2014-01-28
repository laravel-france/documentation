# Démarage rapide avec Laravel

- [Installation](#installation)
- [Routage](#routing)
- [Création d'une vue](#creating-a-view)
- [Création d'une migration](#creating-a-migration)
- [L'ORM Eloquent](#eloquent-orm)
- [Affichage de données dans la vue](#displaying-data)

<a name="installation"></a>
## Installation

Le framework Laravel utilise [Composer](http://getcomposer.org) pour l'installation et la gestion des dépendances. Si vous ne l'avez pas encore, commencez par [installer Composer](http://getcomposer.org/doc/00-intro.md).

Maintenant vous pouvez installer Laravel avec votre terminal grâce à la ligne de commande suivante :

    composer create-project laravel/laravel your-project-name --prefer-dist

Cette commande téléchargera et installera une nouvelle copie de Laravel dans un nouveau répertoire `your-project-name` dans votre répertoire actuel.

Si vous préférez cette méthode, vous pouvez également télécharger une copie [sur le dépôt Github](https://github.com/laravel/laravel/archive/master.zip). Ensuite, [installez composer](http://getcomposer.org), lancez la commande `composer install` depuis la racine du projet. Cette commande téléchargera et installera les dépendances du framework.

<a name="permissions"></a>
### Permissions

Après installation, vous avez besoin de donner les droits en écriture pour le serveur web sur les dossiers dans `app/storage. Voir la documentation [Installation](/4.1/installation#configuration) pour plus de détails sur la configuration.

<a name="directories"></a>
### Structure des répertoires

Après avoir installé le framework, jetez un coup d'oeil sur du projet pour vous familiariser avec la structure des dossiers. Le dossier `app` contient des dossiers tels que `views`, `controllers` et `models`. La majorité du code de votre application va résider dans ce dossier. Vous pouvez également explorer le dossier `app/config` et les options de configuration qui s'offrent à vous.

<a name="routing"></a>
## Routage

Pour commencer, créons notre première route. Avec Laravel, la route la plus simple est la route vers une fonction anonyme. Ouvrez le fichier `app/routes.php` et ajoutez la route suivante à la fin du fichier :

    Route::get('users', function()
    {
        return 'Users!';
    });

Maintenant, visitez la route `/users` dans votre navigateur, vous devriez voir `Users!` affiché en tant que réponse. Bien ! Vous venez de créer votre première route.

Les routes peuvent également être attachées à un contrôleur. Par exemple :

    Route::get('users', 'UserController@getIndex');

Cette route informe le framework que la requête vers la route `/users` doit appeler la méthode `getIndex` de la classe `UserController`. Pour plus d'informations à propos du routage vers un contrôleur, lisez la [documentation sur les contrôleurs](/4.1/controllers).

<a name="creating-a-view"></a>
## Création d'une vue

Ensuite, nous allons créer une simple vue pour afficher nos données utilisateur. Les vues résident dans le dossier `app/views` et contiennent le code HTML de notre application. Nous allons placer deux nouvelles vues dans ce dossier : `layout.blade.php` et `users.blade.php`. Premièrement, créons notre fichier `layout.blade.php` :

    <html>
        <body>
            <h1>Laravel Quickstart</h1>

            @yield('content')
        </body>
    </html>

Ensuite, créons notre vue `users.blade.php` :

    @extends('layout')

    @section('content')
        Users!
    @stop

Certaines parties de cette syntaxe doivent vous sembler étranges. C'est parce que nous utilisons le système de templating de Laravel : Blade. Blade est très rapide, car c'est simplement une poignée d'expressions régulières exécutées sur votre template pour le compiler en du PHP pur. Blade fournit des fonctionnalités puissantes comme l'héritage de template, ainsi qu'une syntaxe embellie pour les structures de contrôle de PHP telles que `if` et `for`. Lisez la [documentation de Blade](/4.1/templates) pour plus de détails.

Maintenant que nous avons nos vues, retournons les depuis notre route `/users`. Plutôt que de retourner `Users!` depuis la route, retournons la vue :

    Route::get('users', function()
    {
        return View::make('users');
    });

Magnifique ! Maintenant vous avez mis en place une vue simple qui hérite d'un layout. Ensuite, travaillons sur la base de données.

<a name="creating-a-migration"></a>
## Création d'une migration

Pour créer une table qui garde nos données, nous allons utiliser le système de migration de Laravel. Les migrations vous laissent définir des modifications sur votre base de données de manière expressive, et de les partager facilement avec le reste de votre équipe.

Premièrement, configurons une connexion à la base de données. Vous pouvez configurer vos connexions à la base de données depuis le fichier `app/config/database.php`. Par défault, Laravel est configuré pour utiliser MySQL et vous aurez besoin de fournir les informations de connexion au fichier de configuration de bases de données. Si vous le souhaitez, vous pouvez changer l'option `driver` pour `sqlite` et vous utiliserez la base de données SQLite incluse dans le répertoire `app/database`.

Ensuite, pour créer une migration, nous allons utiliser [Artisan](/4.1/artisan). Depuis la racine de votre projet, exécutez la ligne suivante dans votre terminal :

    php artisan migrate:make create_users_table

Ensuitez, trouvez le fichier de migration généré dans le dossier `app/database/migrations`. Ce fichier contient une classe avec deux méthodes : `up` et `down`. Dans la méthode `up`, vous devez faire les changements désirés sur votre base de données, et dans la méthode `down`, vous faites ce qu'il faut pour supprimer vos modifications.

Définissons une migration qui ressemble à cela :

    public function up()
    {
        Schema::create('users', function($table)
        {
            $table->increments('id');
            $table->string('email')->unique();
            $table->string('name');
            $table->timestamps();
        });
    }

    public function down()
    {
        Schema::drop('users');
    }

Maintenant, nous pouvons exécuter nos migrations depuis notre terminal en utilisant la commande `migrate`. Exécutez simplement cette commande depuis la racine de votre projet :

    php artisan migrate

Si vous souhaitez annuler une migration, vous pouvez utiliser la commande `migrate:rollback`. Maintenant que nous avons notre table en base de données, récupérons quelques données !

<a name="eloquent-orm"></a>
## L'ORM Eloquent

Laravel est founi avec un superbe ORM : Eloquent. Si vous avez utilisé le framework Ruby on Rails, vous allez trouver Eloquent familier, car il suit le style de l'ORM ActiveRecord pour les interactions avec la base de données.

Premièrement, définissons un modèle. Un modèle Eloquent peut être utilisé pour requêter une table associée dans la base de données, mais aussi pour représenter une ligne dans la table. Ne vous inquiétez pas. Cela aura du sens très rapidement ! Les modèles sont généralement placés dans le dossier `app/models`. Définissons un modèle `User.php` dans ce dossier comme ceci :

    class User extends Eloquent {}

Notez que nous n'avons pas à indiquer à Eloquent quelle table utiliser. Eloquent a une variété de conventions, l'une d'entre elles est d'utiliser le pluriel du nom du modèle en tant que nom de table. Pratique !

En utilisant votre outil d'administration de base de données préféré, insérez quelques lignes dans votre table `users`, et nous utiliserons Eloquent pour les récupérer et les passer à notre vue.

Maintenant, modifiez la route `/users` pour qu'elle ressemble à ceci :

    Route::get('users', function()
    {
        $users = User::all();

        return View::make('users')->with('users', $users);
    });

Analysons dans le détail cette route. Premièrement, la méthode `all` sur le modèle `User` va retourner toutes les lignes de la table `users`. Ensuitez, nous passons nos enregistrements à la vue avec la méthode `with`. La méthode `with` accepte une clé et une valeur, et est utilisée pour rendre des données disponibles dans la vue.

Génial. Maintenant nous sommes prêts à afficher les utilisateurs dans notre vue !

<a name="displaying-data"></a>
## Affichage de données dans la vue

Maintenant que nous avons rendu les `users` disponibles pour notre vue, nous pouvons les afficher comme ceci :

    @extends('layout')

    @section('content')
        @foreach($users as $user)
            <p>{{ $user->name }}</p>
        @endforeach
    @stop

Vous devez vous demander où se trouve la directive `echo`. Lorsque vous utilisez Blade, vous pouvez afficher des données en les entourant par des doubles accolades. Un vrai jeu d'enfant. Maintenant, vous devez être capable d'aller sur la route `/users` et de voir vos utilisateurs affichés dans la réponse.

Ce n'est que le début. Dans ce tutoriel, vous avez vu les bases de Laravel, mais il y a encore tellement de choses excitantes à apprendre ! Continuez à lire la documentaiton et attaquez plus en profondeur les fonctionnalités puissantes offertes par [Eloquent](/4.1/eloquent) et [Blade](/4.1/templates). Ou, peut-être vous serez plus intéréssé par les [queues](/4.1/queues) et les [tests unitaires](/4.1/testing). Et ensuite, peut-être que vous souhaiterez rendre plus souple votre architecture avec le [conteneur IoC](/4.1/ioc). Le choix est vôtre !
