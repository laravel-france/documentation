# Vues & Réponses

- [Réponses basiques](#basic-responses)
- [Redirections](#redirects)
- [Vues](#views)
- [Compositeurs de vues](#view-composers)
- [Réponses spéciales](#special-responses)
- [Macros de résponse](#response-macros)

<a name="basic-responses"></a>
## Réponses basiques

#### Retourne une chaîne de caractères depuis une route

	Route::get('/', function()
	{
		return 'Hello World';
	});

#### Création d'une réponse personnalisée

Une instance de `Response` hérite de la classe `Symfony\Component\HttpFoundation\Response`, qui fournit une multitude de méthodes pour construire une réponse HTTP.

	$response = Response::make($contents, $statusCode);

	$response->header('Content-Type', $value);

	return $response;

Si vous souhaitez accéder aux méthodes de la classe `Response` mais que vous souhaitez retourner une vue à l'utilisateur, vous pouvez utiliser la méthode `Response::view` :

  return Response::view('hello')->header('Content-Type', $type);

#### Attachement d'un cookie à une réponse

	$cookie = Cookie::make('name', 'value');

	return Response::make($content)->withCookie($cookie);

<a name="redirects"></a>
## Redirections

#### Retourne une redirection

	return Redirect::to('user/login');

#### Retourne une redirection avec des données flash

    return Redirect::to('user/login')->with('message', 'Login Failed');

> **Note** : Depuis que la méthode `with` envoie les données à la session, vous pouvez récupérer les données en utilisant la méthode `Session::get`.

#### Retourne une redirection vers une route nommée

	return Redirect::route('login');

#### Retourne une redirection vers une route nommée avec des paramètres

	return Redirect::route('profile', array(1));

#### Retourne une redirection vers une route nommée avec des paramètres nommés

	return Redirect::route('profile', array('user' => 1));

#### Retourne une redirection vers une action d'un contrôleur

	return Redirect::action('HomeController@index');

#### Retourne une redirection vers une action d'un contrôleur avec des paramètres

	return Redirect::action('UserController@profile', array(1));

#### Retourne une redirection vers une action d'un contrôleur avec des paramètres nommés

	return Redirect::action('UserController@profile', array('user' => 1));

<a name="views"></a>
## Vues

Les vues contiennent habituellement les fichiers HTML de votre application et fournissent une manière simple de séparer vos contrôleurs et la logique métier de la partie présentation. Les vues sont stockées dans le dossier `app/views`.

Une vue peut ressembler à ceci :

	<!-- View stored in app/views/greeting.php -->

	<html>
		<body>
			<h1>Hello, <?php echo $name; ?></h1>
		</body>
	</html>

Cette vue peut être retournée en tant que réponse comme ceci :

	Route::get('/', function()
	{
		return View::make('greeting', array('name' => 'Taylor'));
	});

Le second argument passé ici à `View::make` est un tableau de données qui doivent être passées à la vue.

#### Passage de données à une vue

    // Using conventional approach
	$view = View::make('greeting')->with('name', 'Steve');

    // Using Magic Methods
    $view = View::make('greeting')->withName('steve');

Dans l'exemple ci-dessus, `$name` sera accessible dans la vue, et aura comme valeur `Steve`.

Si vous le désirez, vous pouvez passer un tableau de données comme second paramètre à la méthode `make` :

	$view = View::make('greetings', $data);

Vous pouvez également partager des données avec toutes les vues :

    View::share('name', 'Steve');

#### Passage d'une sous-vue à une vue

Vous pouvez également passer une vue à une autre vue. Par exemple, nous pouvons passer une sous-vue qui se trouve dans le fichier `app/views/child/view.php`, à une autre vue de la manière suivante :

	$view = View::make('greeting')->nest('child', 'child.view');

	$view = View::make('greeting')->nest('child', 'child.view', $data);

La sous-vue peut alors être rendue depuis la vue parent :

	<html>
		<body>
			<h1>Hello!</h1>
			<?php echo $child; ?>
		</body>
	</html>

<a name="view-composers"></a>
## Compositeurs de vues

Les compositeurs de vues sont des fonctions anonymes ou des méthodes de classes qui sont appelées lorsqu'une vue est créée. Si vous avez des données que vous souhaitez lier à une vue chaque fois qu'elle est créée dans votre application, alors un compositeur de vue peut organiser ce code en un seul endroit. Par conséquent, les compositeurs de vues peuvent fonctionner comme des "VueModèle" ( design pattern MVVM ) ou des "Présentation" (design pattern MVP).

#### Définition d'un compositeur de vue

	View::composer('profile', function($view)
	{
		$view->with('count', User::count());
	});

Maintenant, chaque fois que la vue `profile` est créée, la donnée `count` lui sera liée.

Vous pouvez également définir un même comportement pour plusieurs vues :

    View::composer(array('profile','dashboard'), function($view)
    {
        $view->with('count', User::count());
    });

Si vous préférez utiliser une classe en tant que compositeur de vue de type classe, qui fournit l'avantage de pouvoir utiliser le [conteneur IoC](/4.1/ioc), vous devez faire comme ceci :

	View::composer('profile', 'ProfileComposer');

Une classe compositeur de vue doit être définie comme ceci :

	class ProfileComposer {

		public function compose($view)
		{
			$view->with('count', User::count());
		}
	}

#### Définition de plusieurs compositeurs

Vous pouvez utiliser la méthode `composers` pour enregistrer un groupe de compositeurs en une fois :

    View::composers(array(
        'AdminComposer' => array('admin.index', 'admin.profile'),
        'UserComposer' => 'user',
     ));

 > **Note:** Il n'y a pas de convention sur l'endroit où les compositeurs de vues doivent être stockés. Vous êtes libres de les mettre où vous le souhaitez, tant qu'ils peuvent être chargés automatiquement par l'une des directives de votre fichier `composer.json`.

### Créateurs de vues

Un **créateur** de vue fonctionne exactement comme les composeurs de vues ; cependant, ils sont lancés immédiatement quand la vue est instanciée. Pour enregistrer un créateur de vue, utilisez simplement la méthode `creator` :

    View::creator('profile', function($view)
    {
        $view->with('count', User::count());
    });

<a name="special-responses"></a>
## Réponses spéciales

#### Création d'une réponse JSON

	return Response::json(array('name' => 'Steve', 'state' => 'CA'));

#### Création d'une réponse JSONP

	return Response::json(array('name' => 'Steve', 'state' => 'CA'))->setCallback(Input::get('callback'));

#### Création d'un téléchargement de fichier

	return Response::download($pathToFile);

	return Response::download($pathToFile, $status, $headers);

> **Note:** HttpFoundation de Symfony, qui s'occupe du téléchargement, a besoin que le fichier téléchargé ait un nom de fichier en ASCII.

<a name="response-macros"></a>
## Macros de résponse

Si vous voulez définir des réponses personnalisées que vous pouvez réutiliser sur plusieurs routes et contrôleurs, vous pouvez les créer avec la méthode `Response::macro` :

    Response::macro('caps', function($value)
    {
        return Response::make(strtoupper($value));
    });

La méthode `macro` accepte un nom en tant que premier argument, et une fonction anonyme en second argument. La fonction anonyme de la macro sera exécutée lors de l'appel de la macro sur la classe `Response` :

    return Response::caps('foo');

Vous pouvez définir vos macros dans un de vos fichiers `app/start`. Ou alors, écrire vos macros dans un fichier séparé que vous incluez dans l'un de vos fichiers `start`.