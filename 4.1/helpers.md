# Fonctions Helper

- [Tableaux](#arrays)
- [Chemins](#paths)
- [Strings](#strings)
- [URLs](#urls)
- [Divers](#miscellaneous)

## Avertissement

Ces fonctions d'aide ne sont pas accessibles directement dans les vues.

<a name="arrays"></a>
## Tableaux

### array_add

La fonction `array_add` ajoute une paire clé/valeur donnée au tableau si la clé donnée n'existe pas dans la tableau.

    $array = array('foo' => 'bar');

    $array = array_add($array, 'key', 'value');

### array_divide

La fonction `array_divide` retourne deux tableaux, un qui contient les clés, et un autre qui contient les valeurs du tableau original.

    $array = array('foo' => 'bar');

    list($keys, $values) = array_divide($array);

### array_dot

La fonction `array_dot` aplatit un tableau multi-dimensionnel en un tableau à un seul niveau qui utilise la notion "point" pour indiquer la profondeur.

    $array = array('foo' => array('bar' => 'baz'));

    $array = array_dot($array);

    // array('foo.bar' => 'baz');

### array_except

La méthode `array_except` supprime la paire clé/valeur donnée du tableau .

    $array = array_except($array, array('keys', 'to', 'remove'));

### array_fetch

La méthode `array_fetch` retourne un tableau aplati qui contient les éléments imbriqués sélectionnés.

    $array = array(
        array('developer' => array('name' => 'Taylor')),
        array('developer' => array('name' => 'Dayle')),
    );

    $array = array_fetch($array, 'developer.name');

### array_first

La méthode `array_first` retourne le premier élément d'un tableau qui passe un test de vérité.

    $array = array(100, 200, 300);

    $value = array_first($array, function($key, $value)
    {
        return $value >= 150;
    });

Une valeur par défaut peut être passée en troisième paramètre :

    $value = array_first($array, $callback, $default);

### array_last
	
La méthode `array_last` retourne le dernier élément d'un tableau  qui passe un test de vérité.

	$array = array(350, 400, 500, 300, 200, 100);

	$value = array_last($array, function($key, $value)
	{
		return $value > 350;
	});
	
	// 500

Une valeur par défaut peut être passée en troisième paramètre :

	$value = array_last($array, $callback, $default);

### array_flatten

La méthode `array_flatten` va aplatir un tableau multi-dimensionnel en un tableau à une seule dimension.

    $array = array('name' => 'Joe', 'languages' => array('PHP', 'Ruby'));

    $array = array_flatten($array);

    // array('Joe', 'PHP', 'Ruby');

### array_forget

La méthode `array_forget` va supprimer la paire clé/valeur d'un tableau multi-dimensionnel en utilisant une notation en "point".

    $array = array('names' => array('joe' => array('programmer')));

    $array = array_forget($array, 'names.joe');

### array_get

La méthode `array_get` va récupérer une valeur donnée d'un tableau multidimensionnel en utilisant une notation en "point".

    $array = array('names' => array('joe' => array('programmer')));

    $value = array_get($array, 'names.joe');

> **Note:** Si vous souhaitez la même chose pour les objets, utilisez `object_get`.

### array_only

La méthode `array_only` va retourner seulement la paire clé/valeur spécifiée du tableau.

    $array = array('name' => 'Joe', 'age' => 27, 'votes' => 1);

    $array = array_only($array, array('name', 'votes'));

### array_pluck

La méthode `array_pluck` va prendre une liste d'une paire clé/valeur du tableau.

    $array = array(array('name' => 'Taylor'), array('name' => 'Dayle'));

    $array = array_pluck($array, 'name');

    // array('Taylor', 'Dayle');

### array_pull

La méthode `array_pull` retournera une paire clé/valeur depuis le tableau, et la supprimera.

    $array = array('name' => 'Taylor', 'age' => 27);

    $name = array_pull($array, 'name');

### array_set

La méthode `array_set` va définir une valeur dans un tableau multi-dimensionnel en utilisant la notation en "point".

    $array = array('names' => array('programmer' => 'Joe'));

    array_set($array, 'names.editor', 'Taylor');

### array_sort

La méthode `array_sort` trie le tableau par le résultat de la fonction anonyme donnée.

    $array = array(
        array('name' => 'Jill'),
        array('name' => 'Barry'),
    );

    $array = array_values(array_sort($array, function($value)
    {
        return $value['name'];
    }));

### array_where

Filtre un tableau avec la fonction anonyme donnée.

	$array = array(100, '200', 300, '400', 500);

	$array = array_where($array, function($key, $value)
	{
		return is_string($value);
	});
	
	// Array ( [1] => 200 [3] => 400 )

### head

Retourne le premier élément d'un tableau. Utile pour chaîner des méthodes en PHP 5.3.x.

    $first = head($this->returnsArray('foo'));

### last

Retourne le dernier élément d'un tableau. Utile pour chaîner des méthodes.

    $last = last($this->returnsArray('foo'));

<a name="paths"></a>
## Chemins

### app_path

Retourne le chemin d'accès complet vers le dossier `app`.

    echo app_path();

### base_path

Retourne le chemin d'accès complet vers le dossier racine de l'installation.

    echo base_path();

### public_path

Retourne le chemin d'accès complet vers le dossier `public`.

    echo public_path();

### storage_path

Retourne le chemin d'accès complet vers le dossier `app/storage`.

    echo storage_path();

<a name="strings"></a>
## Strings

### camel_case

Convertit une chaîne en `camelCase`.

    $camel = camel_case('foo_bar');

    // fooBar

### class_basename

Obtient le nom d'une classe donnée, sans aucun namespace.

    $class = class_basename('Foo\Bar\Baz');

    // Baz

### e

Exécute `htmlentities` sur la chaîne donnée, avec support d'UTF-8.

    $entities = e('<html>foo</html>');

### ends_with

Détermine si une chaîne se termine par une autre chaîne.

    $value = ends_with('This is my name', 'name');

### snake_case

Convertit une chaîne en `snake_case`.

    $snake = snake_case('fooBar');

    // foo_bar
	
### str_limit

Limite le nombre de caractère d'une chaîne.

	str_limit($value, $limit = 100, $end = '...')
	
Exemple:

	$value = str_limit('The PHP framework for web artisans.', 7);

	// The PHP...

### starts_with

Détermine si une chaîne commence par une autre chaîne.

    $value = starts_with('This is my name', 'This');

### str_contains

Détermine si une chaîne contient une autre chaîne.

    $value = str_contains('This is my name', 'my');

### str_finish

Ajoute une chaîne à une autre chaîne. Supprime toutes les instances de la chaîne ajoutée en doublon.

    $string = str_finish('this/string', '/');

    // this/string/

### str_is

Détermine si une chaîne donnée correspond à un patron donné. Un astérisque `*` peut être utilisé pour indiquer le joker.

    $value = str_is('foo*', 'foobar');

### str_plural

Convertit une chaîne vers son pluriel (Anglais uniquement).

    $plural = str_plural('car');

### str_random

Génère une chaîne aléatoire de la longueur donnée.

    $string = str_random(40);

### str_singular

Convertit une chaîne en son équivalent singulier (Anglais uniquement).

    $singular = str_singular('cars');

### studly_case

Convertit une chaîne en `StudlyCase`.

    $value = studly_case('foo_bar');

    // FooBar

### trans

Traduit une ligne de langue. Alias de `Lang::get`.

    $value = trans('validation.required'):

### trans_choice

Traduit une ligne de langue avec comptage. Alias de `Lang::choice`.

    $value = trans_choice('foo.bar', $count);

<a name="urls"></a>
## URLs

### action

Génère une URL vers une action d'un contrôleur donné.

    $url = action('HomeController@getIndex', $params);

### asset

Génère une URL vers un asset.

    $url = asset('img/photo.jpg');

### link_to

Génère un lien HTML vers une URL donnée.

    echo link_to('foo/bar', $title, $attributes = array(), $secure = null);

### link_to_asset

Génère un lien HTML vers un asset donné.

    echo link_to_asset('foo/bar.zip', $title, $attributes = array(), $secure = null);

### link_to_route

Génère un lien HTML vers une route donnée.

    echo link_to_route('route.name', $title, $parameters = array(), $attributes = array());

### link_to_action

Génère un lien HTML vers une action de contrôleur donnée.

    echo link_to_action('HomeController@getIndex', $title, $parameters = array(), $attributes = array());

### route

Génère une URL pour une route nommée.

    $url = route('routeName', $params);

### secure_asset

Génère un lien HTML vers un asset donné en utilisant HTTPS.

    echo secure_asset('foo/bar.zip', $title, $attributes = array());

### secure_url

Génère une URL complète vers un chemin en utilisant HTTPS.

    echo secure_url('foo/bar', $parameters = array());

### url

Génère une URL complète vers un chemin.

    echo url('foo/bar', $parameters = array(), $secure = null);

<a name="miscellaneous"></a>
## Divers

### csrf_token

Obtient la valeur du jeton CSRF courant.

    $token = csrf_token();

### dd

Dump la variable donnée et stop l'exécution du script.

    dd($value);

### value

Si la valeur donnée est une fonction anonyme, alors retourne ce qu'elle retourne lors de son exécution. Sinon, retourne la valeur donnée.

    $value = value(function() { return 'bar'; });

### with

Retourne un objet donné. Utile pour faire du chaînage de méthodes sur des constructeurs avec PHP 5.3.x.

    $value = with(new Foo)->doWork();
