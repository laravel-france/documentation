# La pagination

- [Configuration](#configuration)
- [Utilisation](#usage)
- [Ajout de paramètres sur les liens de pagination](#appending-to-pagination-links)
- [Conversion en JSON](#converting-to-json)

<a name="configuration"></a>
## Configuration

Dans d'autres frameworks, mettre en place une pagination peut être difficile. Laravel facilite cette tâche. Il suffit de valoriser l'option `pagination` du fichier `app/config/view.php` avec la vue à utiliser pour créer des liens de pagination. Par défaut, Laravel inclut deux vues.

La vue `pagination::slider` a pour fonction de produire une restitution élaborée d'une série de liens sur la page courante tandis que la vue `pagination::simple` affiche les boutons "précédent" et "suivant". **Ces deux vues sont compatibles avec Twitter Bootstrap.**

<a name="usage"></a>
## Utilisation

Il existe plusieurs manières de paginer des éléments. La plus simple est d'utiliser la méthode `paginate` du requêteur ou d'un modèle Eloquent.

#### Paginer le résultat d'une requête de base de données

	$users = DB::table('users')->paginate(15);

Vous pouvez aussi paginer un modèle [Eloquent](/4.1/eloquent) :

#### Paginer un modèle Eloquent

    $allUsers = User::paginate(15);

	$someUsers = User::where('votes', '>', 100)->paginate(15);

L'argument transmis à la méthode `paginate` est le nombre d'éléments à afficher par page. Vous pouvez placer les éléments à afficher dans une vue et créer les liens de pagination à l'aide de la méthode `links` :

	<div class="container">
		<?php foreach ($users as $user): ?>
			<?php echo $user->name; ?>
		<?php endforeach; ?>
	</div>

	<?php echo $users->links(); ?>

Créer un système de pagination est aussi simple que cela ! Notez qu'il n'est pas nécessaire d'indiquer l'identité de la page courante au framework. Laravel le détermine automatiquement.

Si vous souhaitez spécifier une vue personnalisée pour la pagination, vous pouvez passer une vue à la méthode `links` :

    <?php echo $users->links('view.name'); ?>

Vous pouvez également accéder à des informations additionelles sur la pagination en utilisant les méthodes suivantes :

- `getCurrentPage`
- `getLastPage`
- `getPerPage`
- `getTotal`
- `getFrom`
- `getTo`
- `count`

Si vous souhaitez créer une pagination manuellement en fournissant un tableau d'éléments, utilisez la méthode `Paginator::make` :

#### Créer manuellement une pagination

	$paginator = Paginator::make($items, $totalItems, $perPage);

#### Personnaliser l'URI de la pagination

Vous pouvez aussi personnaliser l'URI utilisée par la pagination via la méthode `setBaseUrl` :

    $users = User::paginate();

    $users->setBaseUrl('custom/url');

L'exemple ci-dessus créera des URLs de la forme suivante : http://example.com/custom/url?page=2

<a name="appending-to-pagination-links"></a>
## Ajout de paramètres sur les liens de pagination

Vous pouvez ajouter des paramètres sur les liens de pagination en utilisant la méthode `appends` sur le  Paginator:

	<?php echo $users->appends(array('sort' => 'votes'))->links(); ?>

Cela génèrera une URL qui ressemblera à cela :

	http://example.com/something?page=2&sort=votes

Si vous souhaitez ajouter un hashtag aux URLs du Paginator, vous pouvez utiliser la méthode `fragment`:

    <?php echo $users->fragment('foo')->links(); ?>

L'appel à cette méthode permet de générer des URLs qui ressemblent à cela  :

    http://example.com/something?page=2#foo


<a name="converting-to-json"></a>
## Conversion en JSON

La classe `Paginator` implémente le contrat `Illuminate\Support\Contracts\JsonableInterface` et fournit la méthode `toJson`. Vous pouvez aussi convertir une instance de `Paginator` en JSON en la retournant depuis une route. La version JSON retournera quelques informations "meta" tel que `total`, `current_page`, `last_page`, `from`, et `to`. Les données de l'instance se trouvent dans la clé JSON `data`.
