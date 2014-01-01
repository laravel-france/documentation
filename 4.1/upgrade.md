# Guide de mise à jour

- [Mise à jour de la 4.0 à 4.1](#upgrade-4.1)

<a name="upgrade-4.1"></a>
## Mise à jour de la 4.0 à 4.1

### Mise à jour de la dépendance dans composer

Pour mettre à jour Laravel en version 4.1, changez la version de `laravel/framework` en `4.1.*` dans votre fichier `composer.json`.

### Remplacement de fichiers

Remplacez votre fichier `public/index.php` avec [cette copie fraiche du dépôt](https://github.com/laravel/laravel/blob/master/public/index.php).

Remplacez votre fichier `artisan` avec [cette copie fraiche du dépôt](https://github.com/laravel/laravel/blob/master/artisan).

### Ajout de fichiers de configurations & d'options

Mettez à jour les tableaux `aliases` et `providers` dans le fichier de configuration `app/config/app.php`. Les valeurs mises à jour sont disponibles dans [ce fichier](https://github.com/laravel/laravel/blob/master/app/config/app.php). Assurez vous d'ajouter à nouveau vos Service Providers et Alias personnalisés ensuite.

Ajouter le nouveau fichier `app/config/remote.php` [depuis le dépôt](https://github.com/laravel/laravel/blob/master/app/config/remote.php).

Ajoutez la nouvelle option de configuration `expire_on_close` dans le fichier `app/config/session.php`. la valeur par défaut est `false`.

Ajouter la section de configuration `failed` dans votre fichier `app/config/queue.php`. Voici les valeurs par défaut pour ce fichier :

	'failed' => array(
		'database' => 'mysql', 'table' => 'failed_jobs',
	),

**(Optionnel)** Mettre à jour l'option de configuration `pagination` dans le fichier `app/config/view.php` pour `pagination::slider-3`.

### Controller Updates

Si `app/controllers/BaseController.php` a un déclaration `use` en haut, changez `use Illuminate\Routing\Controllers\Controller;` pour `use Illuminate\Routing\Controller;`.

### Mise à jour du rappel de mot de passe

Le rappel de mot de passe a été revu pour une flexibilité plus grande. Vous pouvez regarder à quoi ressemble le nouveau contrôleur en executant la commande `php artisan auth:reminders-controller`. Vous pouvez aussi regarder [la documentation mise à jour](/4.1/security#password-reminders-and-reset) et mettre à jour votre application en conséquence.

Mettez à jour votre fichier `app/lang/en/reminders.php` pour qu'il corresponde [au dernier fichier](https://github.com/laravel/laravel/blob/master/app/lang/en/reminders.php).

### Mise à jour de la détection d'environnement

Pour des raisons de sécurité, les noms de domaines ne doivent pas être utilisé pour detecter l'environnement de l'application. Ces valeurs sont facilement imitables et permettent d'exécuter une requête sur un environnement non désiré. Vous devez convertir la detection pour qu'elle utilise le nom de la machine (commande `hostname` sur Mac & GNU/Linux).

### Fichier de log simple

Laravel génère maintenant un fichier de log unique : `app/storage/logs/laravel.log`. Cependant, cela reste configurable dans le fichier `app/start/global.php`.

### Suppression de la redirection si l'url se termine par un slash

Dans le fichier `bootstrap/start.php`, supprimez l'appel à `$app->redirectIfTrailingSlash()`. Cette méthode n'est plus utilisée car la règle est gérée directement par le fichier .htaccess fournit avec le framework.

Ensuite, remplacez le fichier `.htaccess` avec [cette nouvelle version](https://github.com/laravel/laravel/blob/master/public/.htaccess)

### Accès à la route actuelle

La route actuelle est accessible via `Route::current()` à la place de `Route::getCurrentRoute()`.

### Composer Update

Une fois que tout les étapes au dessus sont réalisées, vous pouvez lancer la commande `composer update`  pour mettre à jour les fichiers du framework ! Si vous avez des erreurs de chargement de classes, essayez de lancer composer update avec l'option `--no-scripts` comme cela : `composer update --no-scripts`.
