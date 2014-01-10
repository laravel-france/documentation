# Release Notes

- [Laravel 4.1](#laravel-4.1)

<a name="laravel-4.1"></a>
## Laravel 4.1

### Liste complète des changements

La liste complète des changements peut être obtenue par la commande `php artisan changes` d'une installation de Laravel 4.1, ou en [regardant le fichier des changements sur Github](https://github.com/laravel/framework/blob/4.1/src/Illuminate/Foundation/changes.json). Ces notes couvrent seulement les améliorations majeures et les changements de cette version.

### Nouveau composant SSH

Un tout nouveau composant `SSH` a été inclus dans cette version. Cette fonctionnalité permet de se connecter en SSH à des serveurs distants et de lancer des commandes. Pour en savoir plus, consultez [la documentation du composant SSH](/4.1/ssh).

La nouvelle commande `php artisan tail` utilise le composant SSH. Pour plus d'informations, regardez la  [documentation de la commande `tail`](http:///4.1/ssh#tailing-remote-logs).

### Boris dans le Tinker

La commande `php artisan tinker` utilise maintenant [Boris REPL](https://github.com/d11wtq/boris) si votre système le supporte. les extensions PHP `readline` et `pcntl` doivent être installées pour cette fonctionnalité. Si vous n'avez pas ces extensions, le shell de la version 4.0 sera utilisé.

### Amélioration d'Eloquent

Une nouvelle relation `hasManyThrough` a été ajoutée dans Eloquent. Pour en savoir plus, retrouvez des informations dans la [documentation d'Eloquent](/4.1/eloquent#has-many-through).

Une nouvelle méthode `whereHas` a été ajoutée également pour  [retrouver des modèles en se basant sur des contraintes de relation](/4.1/eloquent#querying-relations).

### Connexion lecture/écriture en base de données

Une gestion automatique de connexions séparées pour la lecture et l'écriture en base de données est maintenant disponible dans la couche de base de données, que ce soit pour le Query Builder ou Eloquent. Pour plus d'informations, consultez [la documentation](/4.1/database#read-write-connections).

### Priorité de queues

Vous pouvez maintenant gérer des priorités de queue en passant une liste séparée par une virgule à la commande `queue:listen`.

### Gestion des tâches échouées

Le solution de queue inclus maintenant une gestion automatique des tâches en échec en utilisant la nouvelle option `--tries` sur la commande `queue:listen`. Plus d'informations sur la gestion des tâches en échec dans la [documentation des queues](/4.1/queues#failed-jobs).

### Tags de Cache

Les "sections" de Cache ont été remplacés par des "tags". Les tags de Cache permettent d'assigner plusieurs "tags" à un élément de cache, et de vider tous les éléments d'un même tag. Plus d'informations sur l'utilisation des tags dans la [documentation du cache](/4.1/cache#cache-tags).

### Rappel de mot de passe fléxible

Le moteur de rappel de mot de passe a été changé pour fournir une plus grande flexibilité au développeur lors de la validation du mot de passe, du flashage des messages d'erreurs, etc. Pour plus d'informations sur l'utilisation de cet outil, [consultez la documentation](/4.1/security#password-reminders-and-reset).

### Moteur de routage amélioré

Laravel 4.1 fournit une couche de routage complètement réécrite. L'API est la même; cependant, l'enregistrement d'une route est 100% plus rapide que dans la version 4.0. Le moteur a été grandement simplifié, et la dépendance à Symfony Routing a été minimisée à la compilation des expressions de routes.

### Amélioration du moteur de session

Avec cette version, nous avons introduit un tout nouveau moteur de session. De la même manière que pour le routage, le nouveau moteur de session est plus léger et rapide. Nous n'utilisons plus la gestion de sessions de Symfony (et donc de PHP), nous utilisons une solution personnalisée plus facile à améliorer et maintenir.

### Doctrine DBAL

Si vous utilisez la méthode `renameColumn` dans vos migrations, vous devez ajouter la dépendance `doctrine/dbal` dans votre fichier `composer.json`. Ce package n'est plus inclus par défaut avec Laravel.
