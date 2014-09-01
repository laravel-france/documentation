# Sécurité

- [Configuration](#configuration)
- [Stockage de mot de passe](#storing-passwords)
- [Identifier les utilisateurs](#authenticating-users)
- [Identifier des utilisateurs manuellement](#manually)
- [Protection de routes](#protecting-routes)
- [Identification HTTP Basic](#http-basic-authentication)
- [Réinitialisation du mot de passe](#password-reminders-and-reset)
- [Chiffrage](#encryption)

<a name="configuration"></a>
## Configuration

Laravel cherche à rendre l'authentification très simple. En fait, presque tout est déjà configuré pour vous dès le début. Le fichier de configuration de l'authentification se situe dans `app/config/auth.php`, il contient plusieurs options bien documentées pour personnaliser le comportement de la solution d'authentification.

Par défaut, Laravel inclut un modèle `User` dans votre dossier `app/models` qui peut être utilisé avec le driver par défaut : Eloquent. Souvenez-vous lorsque vous construisez la table pour ce modèle que le champ mot de passe doit faire au minimum 60 caractères.

Si votre application n'utilise pas Eloquent, vous pouvez utiliser le driver d'authentification `database` qui utilise le constructeur de requête Laravel.

<a name="storing-passwords"></a>
## Stockage de mot de passe

La classe Laravel `Hash` fournit un cryptage sécurisé Bcrypt :

#### Cryptage d'un mot de passe en utilisant Bcrypt

    $password = Hash::make('secret');

#### Vérification d'un mot de passe contre son équivalent crypté

    if (Hash::check('secret', $hashedPassword)) {
        // The passwords match...
    }

#### Vérifie si un mot de passe a besoin d'être recrypté

    if (Hash::needsRehash($hashed)) {
        $hashed = Hash::make('secret');
    }

<a name="authenticating-users"></a>
## Identifier les utilisateurs

Pour connecter un utilisateur dans votre application, vous devez utiliser la méthode `Auth::attempt`.

    if (Auth::attempt(array('email' => $email, 'password' => $password)))) {
        return Redirect::intended('dashboard');
    }

Notez que `email` n'est pas requis, il est utilisé simplement en tant qu'exemple. Vous devez utiliser la colonne qui correspond à votre "nom d'utilisateur" dans votre base de données. La fonction `Redirect::intended` redirigera l'utilisateur vers l'URL qu'il tentait d'atteindre avant de se faire attraper par le filtre d'identification. Une URL par défaut peut être donnée à la méthode dans le cas où l'URL qu'il souhaitait atteindre n'est pas déterminée.

Lorsque la méthode `attempt` est appelée, l'[événement](/4.1/events) `auth.attempt` est lancé. Si l'identification est un succès et que l'utilisateur est connecté, l'événement `auth.login` sera également exécuté.

Pour déterminer si un utilisateur est déjà connecté à votre application, vous pouvez utiliser la méthode `check` :

#### Détermine si un utilisateur est identifié

    if (Auth::check()) {
        // The user is logged in...
    }

Si vous souhaitez fournir la fonctionnalité "Se souvenir de moi" dans votre application, vous devez passer `true` en tant que second argument à la méthode `attempt`, cela gardera l'utilisateur connecté indéfiniement (ou jusqu'à ce qu'il se déconnecte) :

#### Identifier un utilisateur et se souvenir de lui

    if (Auth::attempt(array('email' => $email, 'password' => $password), true)) {
        // The user is being remembered...
    }

> **Note:** Si la méthode `attempt` retourne `true`, alors l'utilisateur est connecté à votre application.

#### Détermine si un utilisateur est connecté avec l'option remember

Si vous utiliser l'option "remember" lors de la connexion de l'utilisateur, vous pouvez utiliser la méthode `viaRemember` pour savoir si l'utilisateur a été connecté via le cookie :

    if (Auth::viaRemember())
    {
        //
    }

Vous pouvez ajouter des conditions particulières à la requête d'identification :

    if (Auth::attempt(array('email' => $email, 'password' => $password, 'active' => 1))) {
        // The user is active, not suspended, and exists.
    }

> **Note:** Pour ajouter ajouter une protection contre la fixation de session, l'ID de la session utilisateur est régénérée après l'identification.

Une fois qu'un utilisateur est connecté, vous pouvez accéder à son modèle/enregistrement :

#### Accès à l'utilisateur connecté

    $email = Auth::user()->email;

Pour connecter simplement un utilisateur dans votre application en utilisant son Id, utilisez la méthode `loginUsingId` :

    Auth::loginUsingId(1);

La méthode `validate` vous permet de valider que les identifiants d'un utilisateur sont corrects sans le connecter à l'application :

#### Valide les identifiants d'un utilisateur sans le connecter

    if (Auth::validate($credentials)) {
        //
    }

Vous pouvez également utiliser la méthode `once` pour connecter un utilisateur le temps d'une seule requête. Il n'y aura ni session ni cookie pour cet utilisateur.

#### Connecte un utilisateur pour une seule requête

    if (Auth::once($credentials)) {
        //
    }

#### Déconnecte un utilisateur

    Auth::logout();

<a name="manually"></a>
## Identifier des utilisateurs à la main

Si vous avez besoin d'identifier une instance d'un utilisateur dans votre application, vous pouvez simplement appeler la méthode `login` avec cette instance :

    $user = User::find(1);

    Auth::login($user);

Ceci est l'équivalent de la connexion d'un utilisateur via la commande `attempt`.

<a name="protecting-routes"></a>
## Protection de routes

Les filtres de routes peuvent être utilisés pour autoriser uniquement les utilisateurs connectés à accéder à certaines routes. Laravel fournit un filtre `auth` par défaut, qui se situe dans le fichier `app/filters.php`.

#### Protection d'une route

    Route::get('profile', array('before' => 'auth', function()
    {
        // Only authenticated users may enter...
    }));

### Protection CSRF

Laravel fournit une méthode simple pour protéger votre application contre les attaques de type [CSRF](http://fr.wikipedia.org/wiki/Cross-site_request_forgery).

#### Insertion du jeton CSRF dans votre formulaire

    <input type="hidden" name="_token" value="<?php echo csrf_token(); ?>">

#### Validation du jeton CSRF envoyé

    Route::post('register', array('before' => 'csrf', function()
    {
        return 'You gave a valid CSRF token!';
    }));

<a name="http-basic-authentication"></a>
## Identification HTTP Basic

L'identification HTTP Basic fournit une manière rapide d'identifier des utilisateurs de votre application sans avoir à créer une page de "login". Pour commencer, attachez le filtre `auth.basic` à votre route :

#### Protection d'une route avec HTTP Basic

    Route::get('profile', array('before' => 'auth.basic', function()
    {
        // Only authenticated users may enter...
    }));

Par défaut, le filtre `basic` utilisera la colonne `email` de l'enregistrement de l'utilisateur pour faire l'identification. Si vous souhaitez utiliser une autre colonne, vous pouvez passer le nom de la colonne en tant que premier paramètre de la méthode `basic` :

    return Auth::basic('username');

Vous pouvez également utiliser l'identification HTTP Basic sans conserver l'utilisateur connecté en session après la requête, ce qui est utile pour l'identification dans une API. Pour ce faire, créez un filtre qui retourne la méthode `onceBasic` :

#### Définit un filtre HTTP Basic de connexion stateless

    Route::filter('basic.once', function()
    {
        return Auth::onceBasic();
    });

Si vous utilisez PHP FastCGI, l'authentification HTTP Basic ne fonctionnera pas correctement par défaut. Les lignes suivantes doivent être ajoutées à votre fichier `.htaccess` :

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="password-reminders-and-reset"></a>
## Réinitialisation du mot de passe

### Modèle & Table

La plupart des sites fournissent la possibilité à l'utilisateur de réinitialiser son mot de passe. Plutôt que de vous forcer à réimplémenter cela sur chaque application, Laravel fournit des méthodes pratiques pour envoyer un rappel de mot de passe ou réinitialiser ce dernier. Pour commencer, vérifiez que votre modèle `User` implémente l'interface `Illuminate\Auth\RemindableInterface`. Bien sûr, le modèle par défaut `User` inclus dans le framework l'implémente déjà.

#### Implémentation de l'interface RemindableInterface

    class User extends Eloquent implements RemindableInterface {

        public function getReminderEmail()
        {
            return $this->email;
        }

    }

Ensuite, une table doit être créée pour stocker le jeton de réinitialisation du mot de passe. Pour générer une migration pour cette table, exécutez simplement la commande artisan `auth:reminders-table` :

#### Génération de la migration pour la table de rappel

    php artisan auth:reminders-table

    php artisan migrate

### Contrôleur du rappel de mot de passe

Maintenant nous sommes prêt à générer le contrôleur de rappel de mot de passe. Pour générer automatiquement le contrôleur, vous pouvez utiliser la commande `auth:reminders-controller`, qui va créer le fichier `RemindersController.php` dans le dossier `app/controllers`.

    php artisan auth:reminders-controller

Le contrôleur généré aura déjà la méthode `getRemind` qui s'occupe d'afficher le formulaire de rappel du mot de passe. Tout ce que vous devez faire est créé la [vue](/4.1/responses#views) `password.remind`. Cette vue doit avoir un formulaire basique avec un champ `email`. Ce formulaire du faire un POST vers l'action `RemindersController@postRemind`.

Le formulaire de la vue `password.remind` peut ressembler à cela :

    <form action="{{ action('RemindersController@postRemind') }}" method="POST">
        <input type="email" name="email">
        <input type="submit" value="Send Reminder">
     </form>

En plus de `getRemind`, le contrôleur généré a déjà une méthode `postRemind` qui s'occupe d'envoyer l'email de rappel de mot passe à l'utilisateur. Cette méthode attend un champ `email` dans la variable `POST`. Si l'email de rappel est envoyé à l'utilisateur, un message `status` sera flash en session. S'il échoue, un message `error` sera flashé à la place.

Dans la méthode `postRemind` du contrôleur, vous pouvez modifier le message avant de de l'envoyer à l'utilisateur :

	Password::remind(Input::only('email'), function($message)
	{
		$message->subject('Password Reminder');
	});

Votre utilisateur recevra un email avec un lien qui pointe vers la méthode `getReset` du contrôleur. Le jeton du rappel de mot de passe, qui est utilisé pour identifier une tentative de rappel de mot de passe, sera aussi passé à la méthode de contrôleur. L'action est déjà configuré pour retourner une vue `password.reset` que vous devez construire. Le `jeton` (token) sera passé à la vue, et vous devez placé ce jeton dans un champ caché nommé `token`. En plus de ce `token`, votre formulaire doit contenir les champs : `email`, `password`, et `password_confirmation`. Le formulaire doit faire une requête POST sur `RemindersController@postReset`

Le formulaire de la vue `password.reset` peut ressembler à ça :

    <form action="{{ action('RemindersController@postReset') }}" method="POST">
        <input type="hidden" name="token" value="{{ $token }}">
        <input type="email" name="email">
        <input type="password" name="password">
        <input type="password" name="password_confirmation">
        <input type="submit" value="Reset Password">
    </form>

Finalement, la méthode `postReset` est responsable du changement du mot de passe. Dans cette action du contrôleur, la fonction anonyme passée à la méthode `Password::reset` définie l'attribut `password` sur la classe `User` et appelle la méthode `save`. De plus, cette fonction anonyme présume que votre modèle `User` est un [modèle Eloquent](/4.1/eloquent); cependant, vous êtes libre de modifier cette fonction anonyme selon vos besoins pour votre application.

Si le mot de passe est remis à zéro avec succès, l'utilisateur sera redirigé vers la page d'accueil de votre site. Là encore, vous êtes libre de changer cette URL. Si la remise à zéro échoue, l'utilisateur sera redirigé vers le formulaire, et un message `error` sera flashé en dessin.

### Validation du mot de passe

Par défaut, la méthode `Password::reset` vérifiera que le mot de passe fait plus de six caractères. Vous pouvez personnaliser cette règle en utilisant la méthode `Password::validator`, qui accepte une fonction anonyme. Dans cette fonction anonyme, vous pouvez faire la validation comme vous le souhaitez. Notez que vous n'êtes pas obligé de faire une vérification sur la confirmation du mot de passe, car cela est déjà fait par le framework.

    Password::validator(function($credentials)
    {
        return strlen($credentials['password']) >= 8;
    });

> **Note:** Par défaut, les tokens de remise à zéro des mots de passe expirent après une heure. Vous pouvez changer cette donnée via l'option `reminder.expire` de votre fichier `app/config/auth.php`.

<a name="encryption"></a>
## Chiffrage

Laravel fournit une solution pour du chiffrage fort AES-256 avec l'extension PHP mcrypt:

#### Chiffrage d'une valeur

    $encrypted = Crypt::encrypt('secret');

> **Note:** Veuillez vous assurer d'avoir défini une clé de 32 caractères aléatoires dans l'option `key` du fichier de configuration `app/config/app.php`. Sans cela, le chiffrage ne sera pas assez fort.

#### Déchiffrage d'une valeur

    $decrypted = Crypt::decrypt($encryptedValue);

Vous pouvez également préciser le chiffrement ou le mode utilisé par le chiffreur :

#### Réglage du chiffrement et du mode

    Crypt::setMode('ctr');

    Crypt::setCipher($cipher);
