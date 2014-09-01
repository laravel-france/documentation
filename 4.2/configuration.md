# Configuration

- [Introduction](#introduction)
- [Configuration des environnements](#environment-configuration)
- [Configuration des fournisseurs](#provider-configuration)
- [Protection des infos de configurations sensibles](#protecting-sensitive-configuration)
- [Mode de maintenance](#maintenance-mode)

<a name="introduction"></a>
## Introduction

Tous les fichiers de configuration du framework Laravel sont situés dans le dossier `app/config`. Chaque option dans les fichiers est documentée, alors n'hésitez pas à regarder ces fichiers et à vous familiariser avec les options disponibles.

Parfois vous pourriez avoir besoin d'accéder aux valeurs de configuration durant l'exécution de l'application. Vous pouvez le faire en utilisant la classe `Config` :

#### Accède à une valeur de configuration

	Config::get('app.timezone');

Remarquez que la syntaxe de style "point" peut être utilisée pour accéder aux valeurs des fichiers de configuration. Si vous souhaitez définir une valeur pendant l'exécution :

#### Définit une valeur de configuration

	Config::set('database.default', 'sqlite');

Les valeurs de configurations qui sont définies lors de l'éxécution sont définies uniquement pour la requête en cours, et ne seront pas persistées pour les requêtes suivantes.

<a name="environment-configuration"></a>
## Configuration des environnements

Il est souvent utile d'avoir différentes valeurs de configuration basées sur l'environnement sur lequel l'application tourne. Par exemple, vous pourriez vouloir utiliser un driver de cache différent pour votre environnement de développement et votre serveur de production. Ceci est facile à accomplir grâce aux fichiers de configuration basés sur l'environnement.

Créez simplement dans un dossier dans votre dossier `config` qui correspond au nom de votre environnement, comme par exemple `local`. Ensuite, créez les fichiers de configuration que vous souhaitez surcharger et spécifiez les options que vous souhaitez changer. Par exemple, pour surcharger le driver de cache de votre environnement "local", vous devrez créer un fichier `cache.php` dans `app/config/local` avec le contenu suivant :

	<?php
	return array(
		'driver' => 'file',
	);

> **Note:** N'utilisez pas le nom d'environnement 'testing' en tant que nom d'environnement, ce nom est réservé pour les tests unitaires.

Notez que vous n'avez pas à spécifier _toutes_ les options qui se trouvent dans le fichier de base, mais seulement celles que vous souhaitez réécrire. Les fichiers de configuration des environnements sont en cascade vis à vis du fichier de base.

Ensuite, nous devons indiquer au framework comment déterminer sur quel environnement il tourne actuellement. Par défaut, l'environnement sera toujours `production`. Cependant, vous pouvez configurer d'autres environnements dans le fichier `bootstrap/start.php` depuis la racine de votre installation. Dans ce fichier, vous trouverez un appel à `$app->detectEnvironment`. Le tableau passé à cette méthode est utilisé pour déterminer l'environnement courant. Vous pouvez ajouter d'autres environnements et noms de machines dans le tableau au besoin.

	<?php

	$env = $app->detectEnvironment(array(
		'local' => array('your-machine-name'),
	));

Dans cet exemple, 'local' est le nom de l'environnement, et 'your-machine-name' est le nom d'hôte de votre serveur web. Dans Linux et Mac OS, vous pouvez déterminer votre nom d'hôte en ouvrant un terminal et en tapant la commande `hostname`.

Vous pouvez également passer une fonction anonyme à la méthode `detectEnvironment`, vous permettant d'implémenter votre propre détection d'environnement :

Si vous avez besoin d'une détection d'environnement plus flexible, vous pouvez passer une fonction anonyme à la méthode `detectEnvironment`, vous permettant de faire la détection comme vous le souhaitez :


	$env = $app->detectEnvironment(function()
	{
		return $_SERVER['MY_LARAVEL_ENV'];
	});

Vous pouvez accéder à l'environnement courant de l'application par la méthode `environment` :

#### Accède à l'environnement courant de l'application

	$environment = App::environment();

Vous pouvez également passé des arguments à la méthode `environment ` method to check if the environment matches a given value:

  if (App::environment('local'))
  {
    // L'environnement est local
  }

  if (App::environment('local', 'staging'))
  {
    // L'environnement est local ou staging...
  }

<a name="provider-configuration"></a>
### Configuration des fournisseurs

Lorsque vous utilisez des fichiers de configurations spécifiques à vos environnements, vous pourriez vouloir ajouter des [service providers](/4.1/ioc#service-providers) à votre fichier de configuration `app` principal. Cependant, si vous le faites, vous suchargerez la clé "providers" principale et écraserez tous les services providers enregistrés. Pour forcer l'ajout plutôt que le remplacement, utilisez la fonction `append_config` dans le fichier de configuration de votre environnement :

	'providers' => append_config(array(
		'LocalOnlyServiceProvider',
	))

<a name="protecting-sensitive-configuration"></a>
## Protection des infos de configurations sensibles

Pour de "vrais" applications, il est sage de garder vos infos sensibles en dehors des fichiers de configurations. des choses tels que votre mot de passe de base de données, des clés d'API, et des clés de chiffrages doivent se trouver en dehors des fichiers de configuration quand cela est possible. Mais où les placer ? Heuresement, Laravel fournit une solution simple pour protéger ces informations en utilisant un fichier caché.

Premièrement, [configurez votre applications](/4.1/configuration#environment-configuration) pour reconnaitre votre machine en tant qu'environnement `local`. Ensuitez, créez un fichier `.env.local.php` à la racine de votre projet. la racine est là où votre `composer.json` se trouve. Le fichier `.env.local.php` doit retourner un tableau de type clé/valeur, tout comme un fichier de configuration :

	<?php

	return array(

		'TEST_STRIPE_KEY' => 'super-secret-sauce',

	);

Toutes les données retournées par ce fichier se trouverons dans les superglobales PHP `$_ENV` et `$_SERVER`. Vous pouvez utilisez ces superglobales dans vos fichiers de configuration :

	'key' => $_ENV['TEST_STRIPE_KEY']

N'oubliez pas d'ajouter `.env.local.php` à votre `.gitignore`. Cela permettra aux autres développeurs de créer le leur, et de ne pas partager ces informations dans votre système de contrôle de version.

Sur votre serveur de production, créez un fichier `.env.php` à la racine de votre projet avec vos valeurs de production. Ce fichier ne doit également jamais être se trouver dans votre système de contrôle de version.

> **Note:** Vous pouvez créer un fichier pour chaque environnemnt supporté par votre application. par exemple, un environnnement `recette` chargera le fichier `.env.recette.php` si il existe.

<a name="maintenance-mode"></a>
## Mode de maintenance

Lorsque votre application est en mode de maintenance, une vue personnalisée sera afficher sur toutes les routes de votre applications. C'est une manière simple de "désactiver" votre application pendant sa mise à jour ou durant des tâches de maintenance. Un appel à la méthode `App::down` est déjà présent dans votre fichier `app/start/global.php`. La réponse de cette méthode sera envoyée aux utilisateurs lorsque l'application est en mode de maintenance.

Pour activer le mode de maintenance, exécutez la commande Artisan `down` :

	php artisan down

Et pour désactiver ce mode, utilisez la commande `up` :

	php artisan up

Pour afficher une vue lorsque votre application est en mode maintenance, vous pouvez ajouter quelque chose comme cela dans votre fichier app/start/global.php :

	App::down(function()
	{
    		return Response::view('maintenance', array(), 503);
	});

Si la fonction anonyme passée à la méthode `down` retourne `NULL`, le mode maintenant sera ignoré pour cette requête.

### Maintenance Mode & Queues

Quand vous êtes en mode maintenance, aucune [tâche de fond](/4.1/queues) ne sera gérée. Les tâches seront à nouveaux traitées normalement lorsque l'application ne sera plus en mode maintenance.
