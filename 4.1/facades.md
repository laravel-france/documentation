# Facades

- [Introduction](#introduction)
- [Explication](#explanation)
- [Cas pratique](#practical-usage)
- [Création de Façades](#creating-facades)
- [Mockage de Façades](#mocking-facades)
- [Référence Facade => Classe](#facade-to-class-reference)

<a name="introduction"></a>
## Introduction

Les Facades fournissent une interface "static" vers des classes qui sont accessibles dans le [conteneur IoC](/4.1/ioc). Laravel est livré avec plusieurs Facades et vous en avez probablement utilisés sans même le savoir !

Occasionnellement, vous pourriez souhaiter créer vos propres façades pour vos applications et vos packages, donc voyons le concept, le développement et l'utilisation de ces classes.

> **Note:** Avant de s'attaquer aux Facades, il est fortement recommandé d'être familiarisé avec le [conteneur IoC](/4.1/ioc) de Laravel.

<a name="explanation"></a>
## Explication

Dans le contexte d'une application Laravel, une façade est une classe qui fournit un accès à un objet depuis le conteneur. Le mécanisme qui fait marcher tout cela se trouve dans la classe `Facade`. Les façades de Laravel et n'importe quelle façade que vous souhaitez créer, devront hériter de la classe `Facade`.

Vos classes Facade doivent uniquement contenir une méthode, `getFacadeAccessor`. C'est la mission de la méthode `getFacadeAccessor` de définir ce qui doit être résolu depuis le conteneur. La classe de base `Facade` fait appel à la méthode magique `__callStatic` pour transmettre les appels de votre façade vers l'objet résolu.

Donc, quand vous faites un appel à une façade comme `Cache::get`, Laravel résout la classe de management de Cache depuis l'IoC container et appelle la méthode `get` de la classe. En terme technique, Les façades Laravel apportent une syntaxe agréable pour utiliser l'IoC container de Laravel en tant que service locator.

<a name="practical-usage"></a>
## Cas pratique

Dans l'exemple ci-dessous, un appel est fait au système de cache de Laravel. En jetant un oeil à ce code, quelqu'un pourrait dire que la méthode static `get` est appelée sur le classe `Cache`.

    $value = Cache::get('key');

Cependant, si vous regardons cette classe `Illuminate\Support\Facades\Cache`, vous verrez qu'il n'y a pas de méthode statique `get` :

    class Cache extends Facade {

        /**
         * Get the registered name of the component.
         *
         * @return string
         */
        protected static function getFacadeAccessor() { return 'cache'; }

    }

La classe `Cache` hérite de la classe de base `Facade`, et définit une méthode `getFacadeAccessor()`. Souvenez-vous, le boulot de cette méthode est de retourner le nom d'une liaison IoC.

Lorsqu'un utilisateur fait un appel à une méthode static sur la classe `Cache`, Laravel résout le liaison `cache` depuis le conteneur et exécute la méthode désirée (dans ce cas, `get`) sur cet objet.

Donc, notre appel `Cache::get` pourrait être réécrit comme cela :

    $value = $app->make('cache')->get('key');

<a name="creating-facades"></a>
## Création de Façades

Créer une façade pour votre application ou package est simple. Vous avez besoin de seulement 3 choses.

- Une liaison IoC.
- Une classe Facade.
- Un Alias de Facade dans la configuration.

Regardons un exemple. Ici nous avons une classe qui est définie en tant que `\PaymentGateway\Payment`.

    namespace PaymentGateway;

    class Payment {

        public function process()
        {
            //
        }

    }
    
Cette classe peut se trouver dans le dossier `app/models`, ou dans n'importe quel dossier dont Composer à connaissance pour le chargement automatique.

Nous devons être capables de résoudre cette classe depuis le conteneur IoC. Alors, ajoutons une liaison :

    App::bind('payment', function()
    {
        return new \PaymentGateway\Payment;
    });

Un bon endroit pour enregistrer cette liaison peut être de créer un [fournisseur de service](/4.1/ioc#service-providers) nommé `PaymentServiceProvider`. La liaison sera ajoutée dans la méthode `register()`. Vous pouvez configurer Laravel pour charger vos fournisseurs de services dans le fichier de configuration `app/config/app.php`.

Ensuite, nous pouvons créer notre propre classe façade :

    use Illuminate\Support\Facades\Facade;

    class Payment extends Facade {

        protected static function getFacadeAccessor() { return 'payment'; }

    }

Finalement, si nous le souhaitons, nous pouvons ajouter un alias pour notre façade dans le tableau `aliases` du fichier de configuration `app/config/app.php`. Maintenant, nous pouvons appeler la méthode `process` sur une instance de notre classe `Payment` :

    Payment::process();

### Une note sur le chargement automatique des alias

Les classes dans le tableau `aliases` ne sont disponibles que dans quelques instances car [PHP n'essayera pas de charger automatiquement des classes indéfinies type-hinted](https://bugs.php.net/bug.php?id=39003). Si `\ServiceWrapper\ApiTimeoutException` est un alias de `ApiTimeoutException`, un `catch(ApiTimeoutException $e)` hors de l'espace de nom `\ServiceWrapper` ne captera jamais d'exception, même si une est lancée. Un problème similaire se retrouve aussi dans Models. La seule solution consiste à renoncer aux alias et d'importer avec `use` les classes que vous voulez atteindre au début de chaque fichier qui les requiert.

<a name="mocking-facades"></a>
## Mockage de Façades

Les tests unitaires sont un aspect important de pourquoi les façades marchent comme cela. En fait, la testabilité est la raison pour laquelle les Façades existent. Regardez la section [Mockage de Façades](/4.1/testing#mocking-facades) de la documentation.

<a name="facade-to-class-reference"></a>
## Référence Façade => Classe

Vous trouverez ci-dessous toutes les façades et les classes utilisées derrière. C'est un outil pratique pour creuser dans la documentation pour un façade donnée. La clé de [liaison IoC](/4.1/ioc) est également précisée lorsqu'elle existe.

Facade  |  Class  |  IoC Binding
------------- | ------------- | -------------
App  |  [Illuminate\Foundation\Application](http://laravel.com/api/4.1/Illuminate/Foundation/Application.html)  | `app`
Artisan  |  [Illuminate\Console\Application](http://laravel.com/api/4.1/Illuminate/Console/Application.html)  |  `artisan`
Auth  |  [Illuminate\Auth\AuthManager](http://laravel.com/api/4.1/Illuminate/Auth/AuthManager.html)  |  `auth`
Auth (Instance)  |  [Illuminate\Auth\Guard](http://laravel.com/api/4.1/Illuminate/Auth/Guard.html)  |
Blade  |  [Illuminate\View\Compilers\BladeCompiler](http://laravel.com/api/4.1/Illuminate/View/Compilers/BladeCompiler.html)  |  `blade.compiler`
Cache  |  [Illuminate\Cache\Repository](http://laravel.com/api/4.1/Illuminate/Cache/Repository.html)  |  `cache`
Config  |  [Illuminate\Config\Repository](http://laravel.com/api/4.1/Illuminate/Config/Repository.html)  |  `config`
Cookie  |  [Illuminate\Cookie\CookieJar](http://laravel.com/api/4.1/Illuminate/Cookie/CookieJar.html)  |  `cookie`
Crypt  |  [Illuminate\Encryption\Encrypter](http://laravel.com/api/4.1/Illuminate/Encryption/Encrypter.html)  |  `encrypter`
DB  |  [Illuminate\Database\DatabaseManager](http://laravel.com/api/4.1/Illuminate/Database/DatabaseManager.html)  |  `db`
DB (Instance)  |  [Illuminate\Database\Connection](http://laravel.com/api/4.1/Illuminate/Database/Connection.html)  |
Event  |  [Illuminate\Events\Dispatcher](http://laravel.com/api/4.1/Illuminate/Events/Dispatcher.html)  |  `events`
File  |  [Illuminate\Filesystem\Filesystem](http://laravel.com/api/4.1/Illuminate/Filesystem/Filesystem.html)  |  `files`
Form  |  [Illuminate\Html\FormBuilder](http://laravel.com/api/4.1/Illuminate/Html/FormBuilder.html)  |  `form`
Hash  |  [Illuminate\Hashing\HasherInterface](http://laravel.com/api/4.1/Illuminate/Hashing/HasherInterface.html)  |  `hash`
HTML  |  [Illuminate\Html\HtmlBuilder](http://laravel.com/api/4.1/Illuminate/Html/HtmlBuilder.html)  |  `html`
Input  |  [Illuminate\Http\Request](http://laravel.com/api/4.1/Illuminate/Http/Request.html)  |  `request`
Lang  |  [Illuminate\Translation\Translator](http://laravel.com/api/4.1/Illuminate/Translation/Translator.html)  |  `translator`
Log  |  [Illuminate\Log\Writer](http://laravel.com/api/4.1/Illuminate/Log/Writer.html)  |  `log`
Mail  |  [Illuminate\Mail\Mailer](http://laravel.com/api/4.1/Illuminate/Mail/Mailer.html)  |  `mailer`
Paginator  |  [Illuminate\Pagination\Environment](http://laravel.com/api/4.1/Illuminate/Pagination/Environment.html)  |  `paginator`
Paginator (Instance)  |  [Illuminate\Pagination\Paginator](http://laravel.com/api/4.1/Illuminate/Pagination/Paginator.html)  |
Password  |  [Illuminate\Auth\Reminders\PasswordBroker](http://laravel.com/api/4.1/Illuminate/Auth/Reminders/PasswordBroker.html)  |  `auth.reminder`
Queue  |  [Illuminate\Queue\QueueManager](http://laravel.com/api/4.1/Illuminate/Queue/QueueManager.html)  |  `queue`
Queue (Instance) |  [Illuminate\Queue\QueueInterface](http://laravel.com/api/4.1/Illuminate/Queue/QueueInterface.html)  |
Queue (Base Class) |  [Illuminate\Queue\Queue](http://laravel.com/api/4.1/Illuminate/Queue/Queue.html)  |
Redirect  |  [Illuminate\Routing\Redirector](http://laravel.com/api/4.1/Illuminate/Routing/Redirector.html)  |  `redirect`
Redis  |  [Illuminate\Redis\Database](http://laravel.com/api/4.1/Illuminate/Redis/Database.html)  |  `redis`
Request  |  [Illuminate\Http\Request](http://laravel.com/api/4.1/Illuminate/Http/Request.html)  |  `request`
Response  |  [Illuminate\Support\Facades\Response](http://laravel.com/api/4.1/Illuminate/Support/Facades/Response.html)  |
Route  |  [Illuminate\Routing\Router](http://laravel.com/api/4.1/Illuminate/Routing/Router.html)  |  `router`
Schema  |  [Illuminate\Database\Schema\Blueprint](http://laravel.com/api/4.1/Illuminate/Database/Schema/Blueprint.html)  |
Session  |  [Illuminate\Session\SessionManager](http://laravel.com/api/4.1/Illuminate/Session/SessionManager.html)  |  `session`
Session (Instance)  |  [Illuminate\Session\Store](http://laravel.com/api/4.1/Illuminate/Session/Store.html)  |
SSH  |  [Illuminate\Remote\RemoteManager](http://laravel.com/api/4.1/Illuminate/Remote/RemoteManager.html)  |  `remote`
SSH (Instance)  |  [Illuminate\Remote\Connection](http://laravel.com/api/4.1/Illuminate/Remote/Connection.html)  |
URL  |  [Illuminate\Routing\UrlGenerator](http://laravel.com/api/4.1/Illuminate/Routing/UrlGenerator.html)  |  `url`
Validator  |  [Illuminate\Validation\Factory](http://laravel.com/api/4.1/Illuminate/Validation/Factory.html)  |  `validator`
Validator (Instance)  |  [Illuminate\Validation\Validator](http://laravel.com/api/4.1/Illuminate/Validation/Validator.html)
View  |  [Illuminate\View\Environment](http://laravel.com/api/4.1/Illuminate/View/Environment.html)  |  `view`
View (Instance)  |  [Illuminate\View\View](http://laravel.com/api/4.1/Illuminate/View/View.html)  |
