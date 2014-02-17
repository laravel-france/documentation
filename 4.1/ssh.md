# SSH

- [Configuration](#configuration)
- [Basic Usage](#basic-usage)
- [Tasks](#tasks)
- [Téléchargement SFTP](#sftp-downloads)
- [SFTP Uploads](#sftp-uploads)
- [Tailing Remote Logs](#tailing-remote-logs)
- [Envoy Task Runner](#envoy-task-runner)

<a name="configuration"></a>
## Configuration

Laravel includes a simple way to SSH into remote servers and run commands, allowing you to easily build Artisan tasks that work on remote servers. The `SSH` facade provides the access point to connecting to your remote servers and running commands.

The configuration file is located at `app/config/remote.php`, and contains all of the options you need to configure your remote connections. The `connections` array contains a list of your servers keyed by name. Simple populate the credentials in the `connections` array and you will be ready to start running remote tasks. Note that the `SSH` can authenticate using either a password or an SSH key.

> **Note:** Besoin d'exécuter plusieurs tâches sur un serveur distant ? Jetez un oeil à [Envoy task runner](#envoy-task-runner)!

<a name="basic-usage"></a>
## Basic Usage

#### Running Commands On The Default Server

To run commands on your `default` remote connection, use the `SSH::run` method:

	SSH::run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Running Commands On A Specific Connection

Alternatively, you may run commands on a specific connection using the `into` method:

	SSH::into('staging')->run(array(
		'cd /var/www',
		'git pull origin master',
	));

#### Catching Output From Commands

You may catch the "live" output of your remote commands by passing a Closure into the `run` method:

	SSH::run($commands, function($line)
	{
		echo $line.PHP_EOL;
	});

## Tasks
<a name="tasks"></a>

If you need to define a group of commands that should always be run together, you may use the `define` method to define a `task`:

	SSH::into('staging')->define('deploy', array(
		'cd /var/www',
		'git pull origin master',
		'php artisan migrate',
	));

Once the task has been defined, you may use the `task` method to run it:

	SSH::into('staging')->task('deploy', function($line)
	{
		echo $line.PHP_EOL;
	});

<a name="sftp-downlaods"></a>
+## Téléchargement SFTP

La classe `SSH` inclue un moyen simple de récuperer un fichier ou une chaine :

    SSH::into('staging')->get($remotePath, $localPath);

    $contents = SSH::into('staging')->getString($remotePath);

<a name="sftp-uploads"></a>
## SFTP Uploads

The `SSH` class also includes a simple way to upload files, or even strings, to the server using the `put` and `putString` methods:

	SSH::into('staging')->put($localFile, $remotePath);

	SSH::into('staging')->putString($remotePath, 'Foo');

<a name="tailing-remote-logs"></a>
## Tailing Remote Logs

Laravel includes a helpful command for tailing the `laravel.log` files on any of your remote connections. Simple use the `tail` Artisan command and specify the name of the remote connection you would like to tail:

	php artisan tail staging

	php artisan tail staging --path=/path/to/log.file

<a name="envoy-task-runner"></a>
## Envoy Task Runner

- [Installation](#envoy-installation)
- [Exécution de tâches](#envoy-running-tasks)
- [Serveurs multiples](#envoy-multiple-servers)
- [Exécutions parallèles](#envoy-parallel-execution)
- [Macro](#envoy-task-macros)
- [Notifications](#envoy-notifications)
- [Mise à jour](#envoy-updating-envoy)

Laravel Envoy fournit une syntaxe propre et minimale pour définir les tâches que vous exécutez sur vos serveurs distants. Utilisant une syntaxe comme celle de [Blade](/4.1/templates#blade-templating), vous pouvez facilement mettre en place vos tâches de déploiements, commandes artisan et autre.

> **Note:** Envoy nécessite PHP >= 5.4, et fonctione uniquement sur Mac/Linux.

<a name="envoy-installation"></a>
### Installation

Premièrement, téléchargez [l'archive Phar](https://github.com/laravel/envoy/raw/master/envoy.phar) d'Envoy et placez la dans `/usr/local/bin` en tant que `envoy` pour un accès facile. Avant de lancer vos tâches, vous devez donner les droits d'exécution au fichier `envoy`.

Ensuitez, créez un fichier `Envoy.blade.php` à la racine de votre projet. Voici un exemple pour commencer :

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

Comem vous pouvez le constater, un tableau de `@servers` est défini en haut du fichier. Vous pouvez utiliser ces serveurs dans l'option `on` de vos déclarations de tâche. Dans vos déclarations de tâches (`@task`) vous devez placer votre code bash qui sera exécuter sur le serveur.

<a name="envoy-running-tasks"></a>
### Exécution de tâches

Pour lancer une tâche, utilisez la commande `run` d'Envoy:

	envoy run foo

Si besoin, vous pouvez passez des variables au fichier Envoy en utilisant des switchs :

	envoy run deploy --branch=master

Vous pouvez utiliser les options avec la syntaxe de Blade :

	@servers(['web' => '192.168.1.1'])

	@task('deploy', ['on' => 'web'])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-multiple-servers"></a>
### Serveurs multiples

Vous pouvez facilement lancer une taches sur de multiples serveurs. Listez simplement les serveurs dans votre déclaration de tâche :

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2']])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

Par défaut, la tâche sera exécutée sur chaque serveur un par un. Cela signifie que quand elle aura fini sur un serveur, elle passera au suivant.

<a name="envoy-parallel-execution"></a>
### Exécutions parallèles

Si vous souhaitez lancer des tâches sur plusieurs serveurs en parallèle, ajoutez l'option `parallel` à la déclaration de votre tâche :

	@servers(['web-1' => '192.168.1.1', 'web-2' => '192.168.1.2'])

	@task('deploy', ['on' => ['web-1', 'web-2'], 'parallel' => true])
		cd site
		git pull origin {{ $branch }}
		php artisan migrate
	@endtask

<a name="envoy-task-macros"></a>
### Macros

Les macros vous permettent de définir une liste de tâches qui seront exécutés en une seule commande. Par exemple :

	@servers(['web' => '192.168.1.1'])

	@macro('deploy')
		foo
		bar
	@endmacro

	@task('foo')
		echo "HELLO"
	@endtask

	@task('bar')
		echo "WORLD"
	@endtask

La macro `deploy` peut maintenant être utilisé via une simple commande :

	envoy run deploy

<a name="envoy-hipchat-notifications"></a>
### Notifications

#### HipChat

Après l'exécution d'une tâche, vous pouvez envoyer une notification à la salle HipChat de votre équipe avec la directive `@hipchat` :

	@servers(['web' => '192.168.1.1'])

	@task('foo', ['on' => 'web'])
		ls -la
	@endtask

	@after
		@hipchat('token', 'room', 'Envoy')
	@endafter

C'est une manière vraiment simple de notifier à l'équipe qu'une tâche a été exécutée sur un serveur

#### Slack

La syntaxe suivante peut être utilisé pour envoyer une notification vers [Slack](https://slack.com):

	@after
		@slack('team', 'token', 'channel')
	@endafter


<a name="envoy-updating-envoy"></a>
### Mise à jour

Pour mettre à jour Envoy, lancez le avec la commande `self-update`:

	envoy self-update

Si votre installation se trouve dans `/usr/local/bin`, vous devez utiliser `sudo`:

	sudo envoy self-update
