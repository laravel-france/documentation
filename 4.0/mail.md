# Mail

- [Configuration](#configuration)
- [Utilisation](#basic-usage)
- [Incrustation de pièces jointes](#embedding-inline-attachments)
- [Mise en queue de mail](#queueing-mail)
- [Mail & Local Development](#mail-and-local-development)

<a name="configuration"></a>
## Configuration

Laravel fournit une API simple et claire de la célèbre librairie [SwiftMailer](http://swiftmailer.org). La configuration de l'envoi de message s'effectue dans le fichier `app/config/mail.php`. Vous pouvez configurer le host SMTP, son port, les informations d'authentification et aussi la mise en oeuvre d'une adresse destinée à l'envoi "En tant que". Vous pouvez utiliser n'importe quel type de serveur SMTP. Si vous souhaitez utiliser la fonction PHP `mail` pour envoyer des e-mails, alors changez le `driver` pour `mail` dans le fichier de configuration. Un driver `sendmail` est également disponible.

<a name="basic-usage"></a>
## Utilisation

La méthode `Mail::send` doit être utilisée pour l'envoi de message :


	Mail::send('emails.welcome', $data, function($m)
	{
		$m->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

Le premier argument attendu par la méthode `send` est le nom de la vue à utiliser pour constituer le texte du message. Le second argument attendu est le tableau `$data` à transmettre à la vue et le troisième argument est une closure permettant d'indiquer des options relatives au message.

> **Remarque:** Une variable `$message` est toujours passée aux vues de messages, ce qui permet l'incrustation de pièces jointes. Il est donc préférable de ne pas passer de variable `$message` à une vue d'attachement de pièce jointe.

Vous pouvez aussi définir une vue plein texte en plus de la vue HTML :

	Mail::send(array('html.view', 'text.view'), $data, $callback);

Ou aussi indiquer un seul type de vue en utilisant les clés `html` ou `text` :

	Mail::send(array('text' => 'view'), $data, $callback);

Vous pouvez définir d'autres options comme des correspondants ou des fichiers joints :

	Mail::send('emails.welcome', $data, function($m)
	{
		$m->from('us@example.com', 'Laravel');

		$m->to('foo@example.com')->cc('bar@example.com');

		$m->attach($pathToFile);
	});

Lors de l'ajout de fichiers joints, vous devez définir le type MIME et/ou un nom de fichier :

	$m->attach($pathToFile, array('as' => $display, 'mime' => $mime));

> **Remarque:** L'instance de message passée à la closure lors d'un appel à la méthode `Mail::send` étend la classe de message SwiftMailer laquelle fournit des méthodes destinées à la construction du message.

<a name="embedding-inline-attachments"></a>
## Incrustation de pièces jointes

Incruster des images dans les messages est généralement laborieux ; cependant, Laravel fournit une manière pratique d'attacher des images aux messages et d'obtenir le Content-ID approprié.

**Incruster une image dans une vue de message**

	<body>
		Here is an image:

		<img src="<?php echo $message->embed($pathToFile); ?>">
	</body>

**Incruster dans une vue de message, une image présente en mémoire**

	<body>
		Here is an image from raw data:

		<img src="<?php echo $message->embedData($data, $name); ?>">
	</body>

Notez que la variable `$message` est toujours passée aux vues de message par la classe `Mail`.

<a name="queueing-mail"></a>
## Mise en queue d'e-mail

Etant donné que l'envoi d'e-mail peut augmenter drastiquement le temps de réponse de votre application, plusieurs développeurs choisissent de mettre en queue les emails pour un envoi de tâche de fond. Laravel rend cela simple en utilisant son [API de queue](/4.0/queues). Pour mettre en queue un e-mail, utilisez simplement la méthode `queue` sur la classe `Mail` :

**Mise en queue d'un e-mail**

	Mail::queue('emails.welcome', $data, function($m)
	{
		$m->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

Vous pouvez également spécifier un délai en secondes avant l'envoi de l'e-mail avec la méthode `later` :

	Mail::later(5, 'emails.welcome', $data, function($m)
	{
		$m->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

Si vous souhaitez spécifier une queue spécifique, ou "tube", sur laquelle votre message doit être placé, vous pouvez utilisez les méthodes `queueOn` et `laterOn` :

	Mail::queueOn('queue-name', 'emails.welcome', $data, function($m)
	{
		$m->to('foo@example.com', 'John Smith')->subject('Welcome!');
	});

<a name="mail-and-local-development"></a>
## E-mail & développement local

Lors du développement d'une application qui envoie des e-mails, il est généralement préférable de désactiver l'envoi des e-mails depuis votre environnement local, ou de développement. Pour ce faire, vous pouvez soit utiliser la méthode `Mail::pretend`, ou définir l'option `pretend` dans le fichier de configuration `app/config/mail.php` à `true`. Quand l'e-mail est en mode `pretend`, les messages seront écris dans le fichier de log de votre application plutôt que d'être envoyés au destinataire.

**Active le mode simulation d'envoi d'e-mail**

	Mail::pretend();
