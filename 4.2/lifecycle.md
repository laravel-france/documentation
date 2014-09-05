# Cycle de vie d'une requête

- [Présentation](#overview)
- [Cycle de vie d'une requête](#request-lifecycle)
- [Les fichiers de démarrage](#start-files)
- [Les gestionnaires d'événements](#application-events)

<a name="overview"></a>
## Présentation

Lorsque vous utilisez un outil dans la "vrai vie", vous vous sentez plus en confiance lorsque vous savez comment cet outil marche. Ce n'est pas différent pour le développement d'applications. Lorsque vous comprenez comment vos outils marchent, vous vous sentirez plus à l'aise et en confiance en les utilisant. Le but de ce document est de vous donner un aperçu haut niveau de comment le framework Laravel fonctionne. En assimilant la manière dont fonctionne le framework, tout semble moins "magique" et vous vous sentirez plus en confiance lors de la construction de vos applications. En plus de cet aperçu du cycle de vie d'une requête, nous parlerons les fichiers de démarrage et des événements d'application.

Si vous ne comprenez pas tous les termes, n'abandonnez pas. Tentez à minima d'avoir une idée basique de ce qui se passe, et vos connaissances vont grandir à mesure que vous avancerez dans la documentation.

<a name="request-lifecycle"></a>
## Request Lifecycle

Toutes les requêtes vers votre application sont dirigées vers le script `public/index.php`. Lorsque vous utilisez Apache, le fichier `.htaccess` fourni avec Laravel se charge de tout rediriger vers `index.php`. C'est là que Laravel commence le processus de gestion de gestion des requêtes et de retour de réponse au client. Avoir une idée globale du processus de démarrage de Laravel sera utile, alors nous allons regarder cela maintenant !

De loin, le concept le plus important à comprendre lorsque vous souhaitez apprendre le mécanisme de démarrage de Laravel est le concept des **Service Providers**. Vous pouvez trouver une liste de services providers dans le fichier de configuration `app/config/app.php`, au niveau du tableau `providers`. Ces providers servent de mécanisme de démarrage primaire pour Laravel. Mais avant de creuser ce sujet, retournons au fichier `index.php`. Une fois que la requête entre dans votre fichier `index.php`, le fichier `bootstrap/start.php` sera chargé. Ce fichier crée l'objet `Application`, qui sert d'[IoC container](/4.1/ioc).

Une fois l'objet `Application` créé, quelques chemins du projet sont définis et la [détection d'environnement](/4.1/configuration#environment-configuration) est réalisée. Ensuite, un mécanisme de démarrage interne de Laravel est appelé. Ces fichiers se trouvent dans le code source du framework Laravel, et définissent quelques autres paramètres basés sur vos fichiers de configuration. tel que la timezone, le report d'erreurs, etc. Et, en plus de ces paramétrages, il s'occupe d'une chose très importante : enregistrer tous les services providers configurés dans votre application.

Les services providers les plus simples n'ont qu'une méthode : `register`. Cette méthode `register` est appelée lorsque le service provider est enregistré par la méthode `register` de l'objet application. Dans cette méthode, le service provider enregistre des choses dans [l'IoC container](/4.1/ioc). Souvent, chaque service provider lie une ou plusieurs [fonction anonyme](http://us3.php.net/manual/en/functions.anonymous.php) dans le conteneur, qui permet l'accès à ces services liés dans votre application. Donc par exemple, `QueueServiceProvider` enregistre une fonction qui résout les différents classes liées aux [queues](/4.1/queues). Bien sur, les services providers peuvent être utilisés pour n'importe quel tâche de démarrage, pas seulement pour enregistrer des choses dans l'IoC container. Un service provider peut enregistrer un écouteur d'événement, un composeur de vue, une commande Artisan, ou plus.

Une fois que tous les services providers ont été enregistrés,les fichiers de `app/start` seront chargés. Et finalement, votre fichier `app/routes.php` sera chargé. Une fois que le fichier `routes.php` a été chargé. l'objet Request est envoyé à l'application pour être dispatché à une route.

Résumons:

1. Une requête entre dans `public/index.php`.
2. Le fichier `bootstrap/start.php` crée Application et détecte l'environment.
3. Le fichier interne `framework/start.php` configure les paramètres et et charge les services providers.
4. Les fichiers de `app/start` sont chargés.
5. Le fichier `app/routes.php` est chargé.
6. L'objet est envoyé à Application, qui tourne un objet Response.
7. L'objet Response est envoyé au client.

Maintenant que vous avez une bonne idée de comment une requête vers une application Laravel est gérée, regardons plus en détails ces fichiers de démarrage !

<a name="start-files"></a>
## Les fichiers de démarrage

Les fichiers de démarrage sont situés dans le répertoire `app/start`. Par défaut, les fichiers `global.php`, `local.php` et `artisan.php` sont inclus dans votre application. Pour plus de détails sur le fichier `artisan.php`, consultez la rubrique [Artisan CLI](/4.1/commands#registering-commands).

Par défaut, le fichier `global.php` contient quelques éléments de base tels que l'enregistrement du [gestionnaire d'erreurs](/4.1/errors) et l'inclusion de votre fichier `app/filters.php`. Vous pouvez compléter ce fichier en fonction de vos besoins sans limite particulière. Ce fichier est automatiquement inclus à _chaque_ requête de votre application indépendamment de l'environnement. Pour plus d'informations sur les environnements, consultez la rubrique [Configuration](/4.1/configuration).

Bien sûr, si vous avez d'autres environnements que l'environnement `local`, vous devez créer un fichier de démarrage par environnement supplémentaire. A l'exécution d'un des environnements, le fichier de démarrage associé est automatiquement inclus dans votre application. Donc par exemple si vous avez un environnement `development` configuré dans votre fichier `bootstrap/start.php`, vous pouvez créer un fichier `app/start/development.php`, qui sera inclus lorsqu'une requête entrera dans votre application sur cet environnement.

### Que mettre dans ces fichiers de démarrage

Les fichiers de démarrage servent à mettre n'importe quel code de démarrage. Par exemple, vous pouvez enregistrer un compteur de vue. configurer vos préférences de journalisation, définir quelques configurations PHP. C'est vraiment comme vous le souhaitez. Bien sûr, place tout votre code démarrage dans ces fichiers peut être mauvais. Pour de grosses applications, ou si vous avez l'impression que c'est une mauvaise pratique, pensez à déplacer du code dans des [service providers](/4.1/ioc#service-providers).

<a name="application-events"></a>
## Les gestionnaires d'événements

Vous pouvez ajouter des opérations précédant et suivant l'exécution de la requête en enregistrant les gestionnaires des événements `before`, `after`, `finish`, et `shutdown`.

#### Enregistrer des gestionnaires d'événements

	App::before(function($request)
	{
		//
	});

	App::after(function($request, $response)
	{
		//
	});

Les listeners de ces événements seront exécutées `avant` (before) et `après` (after) chaque requête de votre application. Ces événements peuvent être utile pour du filtrage global ou de la modification globale de réponse pour votre application. Vous pouvez les enregistrer dans vos fichiers `start` ou dans un [service provider](/4.1/ioc#service-providers).

Vous pouvez également enregistrer un événement `matched`, qui est lancé lorsqu'une requête est liée à une route, mais que la route n'a pas été exécutée :

	Route::matched(function($route, $request)
	{
		//
	});


L'événement `finish` est appelé une fois que la réponse est envoyée vers le client. C'est l'endroit idéal pour placer des opérations de dernière minute dont votre application aurait besoin. L'événement `shutdown` est appelé tout de suite après que tous les événements `finish` sont traités, et est la dernière opportunité de faire des choses avant que votre script ne soit terminé. La plupart du temps, vous n'aurez pas besoin d'utiliser ces événements.
