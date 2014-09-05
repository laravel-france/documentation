# Cache

- [Configuration](#configuration)
- [Utilisation](#cache-usage)
- [Incrémenter et Décrémenter](#increments-and-decrements)
- [Tags de cache](#cache-tags)
- [Cache de base en données](#database-cache)

<a name="configuration"></a>
## Configuration

Laravel fournit une API unique pour différents gestionnaires de cache. La configuration du cache est située dans le fichier `app/config/cache.php`. Dans ce fichier, vous devez indiquer le driver à utiliser par défaut dans votre application. Laravel supporte les célèbres gestionnaires de cache [Memcached](http://memcached.org) et [Redis](http://redis.io).

De plus, le fichier de configuration du cache fournit diverses options. Consultez ces options, elles sont documentées directement dans le fichier de configuration. Par défaut, Laravel est configuré pour utiliser le gestionnaire de cache `file` qui enregistre les objets sérialisés dans des fichiers. Pour les applications de grande envergure, il est recommandé d'utiliser un cache mémoire comme Memcached ou APC.

<a name="cache-usage"></a>
## Utilisation

#### Stocker une variable dans le cache

    Cache::put('key', 'value', $minutes);

#### Utilisation d'un objet Carbon pour définir une date d'expiration

    $expiresAt = Carbon::now()->addMinutes(10);

    Cache::put('key', 'value', $expiresAt);

#### Stocker un élément dans le cache s'il n'existe pas

    Cache::add('key', 'value', $minutes);

La méthode `add` retourne `true` si l'élément est **ajouté** au cache. Sinon, elle retournera `false`.

#### Lire une variable dans le cache

    $value = Cache::get('key');

#### Lire une variable ou retourner une valeur par défaut

    $value = Cache::get('key', 'default');

    $value = Cache::get('key', function() { return 'default'; });

#### Stocker une variable dans le cache de manière permanente

    Cache::forever('key', 'value');

Lors de la lecture d'une variable dans le cache, si vous souhaitez enregistrer une valeur par défaut dans le cas où la variable n'existe pas, utilisez la méthode `Cache::remember` :

    $value = Cache::remember('users', $minutes, function()
    {
        return DB::table('users')->get();
    });

Vous pouvez aussi combiner les méthodes `remember` et `forever` :

    $value = Cache::rememberForever('users', function()
    {
        return DB::table('users')->get();
    });

Notez que les variables mises en cache étant sérialisées, n'importe quel type de variable peut être mis en cache.

#### Supprimer une variable du cache

    Cache::forget('key');

<a name="increments-and-decrements"></a>
## Incrémenter et Décrémenter

Tous les drivers sauf `file` et `database` supportent les opérations `increment` et `decrement` :

#### Incrémentage d'une valeur

    Cache::increment('key');

    Cache::increment('key', $amount);

#### Décrémentage d'une valeur

    Cache::decrement('key');

    Cache::decrement('key', $amount);

<a name="cache-tags"></a>
 ## Tags de Cache

> **Note:** Les tags de cache ne sont pas supportés par les drivers `file` et `database`. De plus, si vous utilisez plusieurs tags avec des caches stockés indéfiniement, les performances seront meilleures avec un driver tel que `memcached`, qui purge automatiquement les données racis.


Les sections de cache vous permettent de grouper des éléments de même nature dans le cache, et également de vider la section d'un coup. Pour accéder à un tag, utilisez la méthode `tags` :

#### Accès à un tag de cache

Vous pouvez stocker un caché taggué en passant une liste triée de tag en argument, ou en tant que tableau avec une liste triée de nom de tag :

    Cache::tags('people', 'authors')->put('John', $john, $minutes);

    Cache::tags(array('people', 'artists'))->put('Anne', $anne, $minutes);

Vous pouvez utiliser n'importe quelle méthode de cache en combinaison avec tags, tel que `remember`, `forever`, et `rememberForever`. Vous pouvez également accéder aux éléments du tag, et utilisez les autres méthodes tel que `increment` et `decrement`:

#### Accès à un élément d'un tag

Pour accéder à un élément taggué, passez la même liste ordonnée de tags utilisée pour la sauvegarde.

    $anne = Cache::tags('people', 'artists')->get('Anne');

    $john = $cache::tags(array('people', 'authors'))->get('John');

Vous pouvez vider tous les items d'un tag avec un nom, ou une liste de nom. Par exemple, la ligne ci dessous supprimera tous les éléments avec 'people', 'authors', ou les deux. Donc Anne et John seront supprimés.

  Cache::tags('people', 'authors')->flush();

Et la ligne ci dessous supprimera uniquement les éléments taggués avec 'authors', donc uniquement John.

    Cache::tags('authors')->flush();

<a name="database-cache"></a>
## Cache de base de données

Pour utiliser le driver de cache `database`, vous devez créer une table destinée à stocker les variables de cache. Voici un exemple de création d'une telle table :

    Schema::create('cache', function($t)
    {
        $t->string('key')->unique();
        $t->text('value');
        $t->integer('expiration');
    });
