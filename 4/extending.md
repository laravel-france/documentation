# Etendre le Framework

- [Introduction](#introduction)
- [Managers & Factories](#managers-and-factories)
- [Cache](#cache)
- [Session](#session)
- [Authentication](#authentication)
- [Extension basée sur l'IoC](#ioc-based-extension)
- [Extension de la classe Request](#request-extension)

<a name="introduction"></a>
## Introduction

Laravel vous offre plusieurs possibilités pour étendre le comportement des composants du framework, ou même les remplacer complètement. Par exemple, les solutions de hashage sont définies avec un contrat `HasherInterface`, que vous pouvez implémenter en vous basant sur vos besoins. Vous pouvez également hériter de l'objet `Request`, vous permettant d'ajouter des "helpers" sur ce dernier. Vous pouvez également ajouter de nouveaux drivers pour l'identification, le cache ou les sessions !

Les composants de Laravel peuvent être généralement étendus de deux manières : en liant une nouvelle implémentation dans le conteneur IoC, ou en enregistrant une extension avec une classe `Manager`, qui sont l'implémentation du design pattern "Factory". Dans ce chapitre, nous allons explorer les différentes manières d'étendre le framework et examiner le code nécessaire.

> **Note:** Souvenez-vous, les composants sont généralement étendus soit au chargement de la liaison dans le conteneur IoC, soit via une classe `Manager`. Les classes manager sont une implémentation du design pattern "Factory", et sont utilisées pour créer des solutions avec ces drivers telles que le cache et les sessions.

<a name="managers-and-factories"></a>
## Managers & Factories

Laravel a plusieurs classes `Manager` qui se chargent de la création de composants contenant des drivers. Cela inclut les composants de cache, session, identification et de queue. La classe `Manager` est responsable de la création d'une implémentation d'un driver basé sur le fichier de configuration de l'application. Par exemple, la classe `CacheManager` peut créer une implémentation de cache avec APC, Memcached, Native, et divers autres drivers de cache.

Chacun de ses managers implémente une méthode `extend` qui peut être utilisée pour injecter facilement un nouveau driver dans le manager. Nous verrons chacun de ses managers ci-dessous, avec des exemples montrant comment injecter des drivers personnalisés dans chacun d'entre eux.

> **Note:** Prenez un moment pour regarder les différentes classes `Manager` que Laravel contient, telles que `CacheManager` et `SessionManager`, cela vous aidera à comprendre ce qui se passe sous le capot. Toutes les classes `Manager` héritent de `Illuminate\Support\Manager`, qui fournit des fonctionnalités utiles et communes pour chaque manager.

<a name="cache"></a>
## Cache

Pour étendre la solution de cache, nous allons appeler la méthode `extend` sur `CacheManager`, qui est utilisée pour fournir un driver personnalisé au manager, et qui est commune à tous les managers. Par exemple, pour créer un nouveau driver de cache appelé "mongo", nous ferions comme ceci :

    Cache::extend('mongo', function($app)
    {
        // Return Illuminate\Cache\Repository instance...
    });

Le premier argument passé à la méthode `extend` est le nom du driver. Cela correspond à l'option `driver` du fichier de configuration `app/config/cache.php`. Le second argument est une fonction anonyme qui doit retourner une instance de `Illuminate\Cache\Repository`. La fonction anonyme aura pour argument une instance de `$app`, qui est une instance de `Illuminate\Foundation\Application` et un conteneur IoC.

Pour créer notre propre driver de cache, nous devons premièrement implémenter le contrat `Illuminate\Cache\StoreInterface`. Donc, notre implémentation de cache avec MongoDB devrait ressembler à cela :

    class MongoStore implements Illuminate\Cache\StoreInterface {

        public function get($key) {}
        public function put($key, $value, $minutes) {}
        public function increment($key, $value = 1) {}
        public function decrement($key, $value = 1) {}
        public function forever($key, $value) {}
        public function forget($key) {}
        public function flush() {}

    }

Nous devons juste implémenter chacune de ses méthodes en utilisant une connexion MongoDB. Une fois que cela est implémenté, nous pouvons finir l'enregistrement de notre driver :

    use Illuminate\Cache\Repository;

    Cache::extend('mongo', function($app)
    {
        return new Repository(new MongoStore);
    });

Comme vous pouvez le voir sur l'exemple ci-dessus, nous devons utiliser `Illuminate\Cache\Repository` lorsque nous créons notre driver de cache personnalisé. Il n'y a aucun besoin de créer notre propre dépôt.

Si vous vous demandez où mettre vos drivers de cache personnalisés, pensez à les mettre sur Packagist ! Ou alors, vous pouvez créer un namespace `Extensions` dans votre dossier d'application. Par exemple, pour une application appelée Snappy, vous pourriez mettre votre driver dans  `app/Snappy/Extensions/MongoStore.php`. Cependant, gardez en tête que Laravel n'a pas une structure d'application rigide, et que vous êtes libre d'organiser votre code comme vous le désirez.

> **Note:** Si vous vous demandez où mettre une morceau de code, pensez toujours aux fournisseurs de services. Utiliser un fournisseur de service pour organiser vos extensions est une bonne manière de garder votre code organisé.

<a name="session"></a>
## Session

Etendre Laravel avec un driver de session personnalisé est aussi simple que d'étendre le système de cache. Une fois de plus, nous allons utiliser la méthode `extend` pour enregistrer un driver :

    Session::extend('mongo', function($app)
    {
        // Return implementation of SessionHandlerInterface
    });

Notez que notre driver doit implémenter le contrat `SessionHandlerInterface`. Cette interface est inclue dans PHP 5.4+. Si vous utilisez PHP 5.3, cette interface sera définie pour vous par Laravel pour être compatible. Cette interface contient quelques méthodes à implémenter. Une implémentation pour MongoDB ressemblerait à cela :

    class MongoHandler implements SessionHandlerInterface {

        public function open($savePath, $sessionName) {}
        public function close() {}
        public function read($sessionId) {}
        public function write($sessionId, $data) {}
        public function destroy($sessionId) {}
        public function gc($lifetime) {}

    }   

Regardons rapidement le rôle de chacune de ses méthodes :

- La méthode `open` sera principalement utilisée dans un système de stockage des sessions basé sur des fichiers. Etant donné que Laravel est fourni avec un driver `native` qui utilise le système de stockage des sessions natif de PHP dans des fichiers, vous n'aurez presque jamais rien à mettre dans cette méthode. C'est simplement dû au fait d'un système d'interface pauvre (nous en discuterons plus tard) que PHP nous oblige à implémenter cette méthode.
- La méthode `close`, comme la méthode `open`, peut également rester vide. Pour la plupart des drivers, ce n'est pas requis.
- La méthode `read` doit retourner une version de la session qui correspond à `$sessionId` sous forme d'une chaine de caractères. Pas besoin de faire de la sérialisation, lorsque vous écrivez vos données, Laravel s'en charge pour vous.
- La méthode `write` doit écrire les données `$data` associées à la session `$sessionId` dans un système de stockage, tel que MongoDB, etc.
- La méthode `destroy` doit supprimer les données associées à la session `$sessionId` du système de stockage.
- la méthode `gc` doit détruire toutes les données de session qui sont plus vieilles que le `$lifetime` donné, qui est un timestamp UNIX. Pour des systèmes avec expiration tels que Redis, cette méthode peut rester vide.

Une fois que l'interface `SessionHandlerInterface` a été implémentée, nous sommes prêts à l'implémenter :

    Session::extend('mongo', function($app)
    {
        return new MongoHandler;
    });

Une fois que le driver de session a été enregistré, nous pouvons utiliser le driver `mongo` dans notre fichier de configuration `app/config/session.php`.

> **Note:** Si vous écrivez un driver de session personnalisé, partagez le sur Packagist !

<a name="authentication"></a>
## Identification

L'identification peut être étendue comme le cache et les sessions. Utilisons la méthode `extend` comme nous en avons l'habitude maintenant :

    Auth::extend('riak', function($app)
    {
        // Return implementation of Illuminate\Auth\UserProviderInterface
    });

Les implémentations de `UserProviderInterface` ont pour rôle de récupérer une implémentation de `UserInterface` depuis un système de stockage persistant, tel que MySQL, Riak, etc. Ces deux interfaces permettent au mécanisme d'identification de Laravel de continuer à fonctionner peu importe la manière dont les données des utilisateurs sont stockées, ou quel type de classe est utilisé pour représenter l'utilisateur. Regardons le contrat `UserProviderInterface`:

    interface UserProviderInterface {

        public function retrieveById($identifier);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(UserInterface $user, array $credentials);

    }

La méthode `retrieveById` reçoit généralement un identifiant unique qui représente un utilisateur, généralement un entier qui s'auto-incrémente. L'implémentation de `UserInterface` qui implémente cette ID doit être retrouvée et retournée par cette méthode.

La méthode `retrieveByCredentials` reçoit le tableau d'identification passé à la méthode `Auth::attempt` lors de l'essai de connexion à l'application. La méthode doit alors se charger de "requêter" le système de stockage des utilisateurs pour trouver l'utilisateur qui a ces identifiants. Typiquement, cette méthode va lancer une requête avec une condition "where" sur `$credentails['username']`. **Cette méthode ne doit pas tenter de faire une validation de mot de passe ou une connexion.**

La méthode `validateCredentials` va comparer le `$user` donné avec `$credentials` pour identifier l'utilisateur. Par exemple, cette méthode peut comparer la chaîne `$user->getAuthPassword()` à un `Hash::make` de `$credentials['password']`.

Maintenant que nous avons exploré chacune des méthodes de `UserProviderInterface`, regardons l'interface `UserInterface`. Souvenez vous, le fournisseur doit retourner une implémentation de cette interface depuis les méthodes `retrieveById` et `retrieveByCredentials`:

    interface UserInterface {

        public function getAuthIdentifier();
        public function getAuthPassword();

    }

Cette interface est simple. La méthode `getAuthIdentifier` doit retourner la "clé primaire" de l'utilisateur. Avec un backend MySQL, cela serait la clé primaire auto-incrémentée. La méthode `getAuthPassword` devrait retourner le mot de passe hashé de l'utilisateur. Cette interface permet au système d'identification de fonctionner avec n'importe quelle classe User, et qu'importe l'ORM ou le système de stockage que vous utilisez. Par défaut, Laravel inclut une classe `User` dans le dossier `app/models` qui implémente cette interface, vous pouvez donc utiliser cette classe pour un exemple d'implémentation.

Finalement, une fois que nous avons implémenté `UserProviderInterface`, nous sommes prêts à enregistrer notre extension auprès de la façade `Auth` :

    Auth::extend('riak', function($app)
    {
        return new RiakUserProvider($app['riak.connection']);
    });

Une fois que vous avez enregistré le driver avec la méthode `extend`, vous pouvez passer à votre nouveau driver dans le fichier de configuration `app/config/auth.php`.

<a name="ioc-based-extension"></a>
## Extension par l'IoC

Presque tous les fournisseurs de services inclus dans le framework Laravel lient des objets dans le conteneur IoC. Vous pouvez trouver une liste des services providers dans le fichier `app/config/app.php`. Quand vous aurez le temps, vous devriez jeter un oeil à chacun de ces fichiers. Ainsi, vous comprendrez bien mieux ce que chacun fournit au framework, et également quelles sont les clés utilisées pour les liaisons dans le conteneur IoC.

Par exemple, `PaginationServiceProvider` est enregistré dans le conteneur IoC avec la clé `paginator` qui résout une instance de `Illuminate\Pagination\Environment`. Vous pouvez facilement étendre et surcharger cette classe dans votre application en écrasant cette liaison dans l'IoC. Par exemple, vous pouvez créer une classe qui hérite de la classe de base `Environment`:

    namespace Snappy\Extensions\Pagination;

    class Environment extends \Illuminate\Pagination\Environment {

        //

    }

Une fois que vous avez créé votre classe fille, vous pouvez créer un fournisseur de service `SnappyPaginationProvider` qui surcharge le paginator dans sa méthode `boot` :

    class SnappyPaginationProvider extends PaginationServiceProvider {

        public function boot()
        {
            App::bind('paginator', function()
            {
                return new Snappy\Extensions\Pagination\Environment;
            });

            parent::boot();
        }

    }

Notez que cette classe hérite de `PaginationServiceProvider`, et non de la classe de base `ServiceProvider`. Une fois que vous avez étendu le fournisseur de service, changez l'entrée `PaginationServiceProvider` dans votre fichier de configuration `app/config/app.php` par le nom de votre propre fournisseur de service.

C'est la méthode générale pour étendre n'importe quelle classe du coeur de Laravel qui est définie dans l'IoC. La quasi totalité des classes au coeur du framework sont liées au conteneur IoC de la même manière, et peuvent être surchargées. Une fois de plus, lisez le code source des fournisseurs de services inclus dans le framework pour vous familiariser avec les classes qui sont liées dans le conteneur, et à quelles clés elles sont liées. C'est une bonne manière d'apprendre comment Laravel est construit.

<a name="request-extension"></a>
## Extension de la classe Request

Comme c'est une pièce fondamentale du framework et qu'elle est instanciée très tôt dans le cycle d'une requête, l'extension de la classe `Request` fonctionne un peu différemment que dans l'exemple précédent.

Premièrement, héritez de la classe comme d'habitude :

    <?php namespace QuickBill\Extensions;

    class Request extends \Illuminate\Http\Request {

        // Custom, helpful methods here...

    }

Une fois que cela est fait, ouvrez le fichier `bootstrap/start.php`. Ce fichier est l'un des premiers à être inclu dans chaque requête vers votre application. Notez que la première action effectuée est la création de l'instance de la variable `$app` :

    $app = new \Illuminate\Foundation\Application;

Lorsqu'une instance d'application est créée, cela va créer une nouvelle instance de `Illuminate\Http\Request` et le lier au conteneur IoC en utilisant la clé `request`. Nous avons donc besoin de spécifier une classe personnalisée qui doit être utilisée comme le type de requête par défaut. Et heureusement, la méthode `requestClass` de la classe Application sert justement à cela ! Nous pouvons donc ajouter cette ligne tout en haut de notre fichier `bootstrap/start.php` :

    use Illuminate\Foundation\Application;

    Application::requestClass('QuickBill\Extensions\Request');

Une fois que vous avez spécifié la classe de requête personnalisée, Laravel va utiliser cette classe chaque fois qu'il crée une instance d'une requête, vous permettant aisément d'avoir votre propre classe pour représenter la requête, même dans les tests unitaires !
