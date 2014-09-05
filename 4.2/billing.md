@@ -0,0 +1,251 @@
# Laravel Cashier

- [Introduction](#introduction)
- [Configuration](#configuration)
- [Souscrire A Un Plan](#subscribing-to-a-plan)
- [Sans Carte De Crédit A l'Avance](#no-card-up-front)
- [Changement De Souscriptions](#swapping-subscriptions)
- [Souscriptions Multiple](#subscription-quantity)
- [Annulation De Souscriptions](#cancelling-a-subscription)
- [Reprise De Souscriptions](#resuming-a-subscription)
- [Vérification Du Statut D'Une Souscriptions](#checking-subscription-status)
- [Manipuler Des Paiements Perdu](#handling-failed-payments)
- [Manipuler d'Autres Webbooks de Stripe](#handling-other-stripe-webhooks)
- [Factures](#invoices)

<a name="introduction"></a>
## Introduction

Laravel Cashier fournit une interface fluide pour les souscription à des services de facturation [Stripe's](https://stripe.com). Il gère la quasi-totalité du code de souscription à une facturation "passe-partout" que vous redoutez d'écrire. En plus de la gestion de base de la souscription, Cashier peut gérer des coupons, le changement de souscription, la souscription multiple, l'annulation des périodes "de grâce", et même générer des factures en fichiers PDF.

<a name="configuration"></a>
## Configuration

#### Composer

Premièrement, ajoutez le package Cashier à votre fichier `composer.json` : 

  "laravel/cashier": "~2.0"

#### Service Provider

Ensuite, enregistrez `Laravel\Cashier\CashierServiceProvider` dans le fichier de configuration de votre application.

#### Migration

Avant d'utiliser Cashier, nous aurons besoin d'ajouter plusieurs colonnes à votre base de données. Ne vous inquiétez pas, vous pouvez utiliser la commande `cashier:table` d'Artisan afin de créer une migration pour ajouter la colonne nécessaire. Par exemple, utilisez `php artisan cashier:table users` pour ajouter la colonne nécessaire dans la table des utilisateurs. Une fois la migration créée, il suffit de lancer la commande `migrate`.

#### Installation Du Modèle

Ensuite, ajoutez BillableTrait et les mutateurs de dates appropriées pour la définition de votre modèle:

  use Laravel\Cashier\BillableTrait;
  use Laravel\Cashier\BillableInterface;

  class User extends Eloquent implements BillableInterface {

    use BillableTrait;

    protected $dates = ['trial_ends_at', 'subscription_ends_at'];

  }

#### Clé Stripe

Enfin, configurer votre clé Stripe dans l'un de vos fichiers bootstrap:

  User::setStripeKey('stripe-key');

<a name="subscribing-to-a-plan"></a>
## S'Abonner A Un Plan

Une fois que vous avez un exemple de modèle, vous pouvez facilement souscrire l'utilisateur à un plan de Stripe :

  $user = User::find(1);

  $user->subscription('monthly')->create($creditCardToken);

Si vous souhaitez appliquer un coupon lors de la création de la souscription, vous pouvez utiliser la méthode `withCoupon`:

  $user->subscription('monthly')
       ->withCoupon('code')
       ->create($creditCardToken);

La méthode `subscription` créera automatiquement la souscription à Stripe, et mettra à jour votre base de données avec l'ID Stripe du client et les autres informations de facturation pertinentes. Si votre plan a un essai de configuré dans Stripe, la date de fin d'essai sera aussi automatiquement mise sur l'enregistrement de l'utilisateur.

Si votre plan a une période d'essai qui ** n'est pas ** configuré dans Stripe, vous devez définir la date de fin d'essai manuellement après la souscription:

  $user->trial_ends_at = Carbon::now()->addDays(14);

  $user->save();

### Spécifier Des Détails Supplémentaires Pour l'Utilisateur

Si vous souhaitez spécifier des détails supplémentaires sur les clients, vous pouvez le faire en les passant dans un second argument de la méthode `create`:

  $user->subscription('monthly')->create($creditCardToken, [
    'email' => $email, 'description' => 'Our First Customer'
  ]);

Pour en apprendre plus sur les champs additionnels supporté par Stripe, consultez la [documentation sur la création d'un client](https://stripe.com/docs/api#create_customer) de Stripe.

<a name="no-card-up-front"></a>
## Sans Carte De Crédit A l'Avance

Si votre application dispose d'un essai gratuit avoir besoin de carte de crédit, réglez la propriété `cardUpFront` de votre modèle à `false` :

  protected $cardUpFront = false;

A la création du compte, assurez-vous de fixer la date de fin de l'essai sur le modèle :

  $user->trial_ends_at = Carbon::now()->addDays(14);

  $user->save();

<a name="swapping-subscriptions"></a>
## Echange De Souscriptions

Pour permuter un utilisateur vers une nouvelle souscription, utilisez la méthode `swap` : 

  $user->subscription('premium')->swap();

Si l'utilisateur est en periode d'essai, l'essai peut-être maintenu normalement. En outre, si une "quantité" existe pour une souscription, cette quantité peut aussi être maintenu. 

<a name="subscription-quantity"></a>
## Quantités De Souscriptions

Quelques fois, les souscriptions sont affectées par une "quantité". Par Exemple, votre application peut charger 10€ par mois par utilisateur sur un compte. Pour incrémenter ou décrémenter facilement votre quantité souscrite, utiliser les méthodes `increment` et `decrement` :  

  $user = User::find(1);

  $user->subscription()->increment();

  // Ajouter cinq à la quantité actuel d'une souscription...
  $user->subscription()->increment(5);

  $user->subscription->decrement();

  // Soustraire cinq à la quantité actuel d'une souscription ...
  $user->subscription()->decrement(5);

<a name="cancelling-a-subscription"></a>
## Annulation De Souscriptions

Annuler une souscription est une promenade dans le parc :

  $user->subscription()->cancel();

Quand une souscription est annulée, Cashier définit automatiquement la colonne `subscription_ends_at`dans votre base de données. Cette colonne est utilisé pour savoir quand la méthode `souscribed` devrait commencer à retourner `false`. Par exemple, si un client annule une souscription le 1er Mars, mais la souscription n'est pas prévu pour finir avant le 5 Mars, la méthode `subscribed` devrait continuer à retouner `true` jusqu'au 5 Mars.

<a name="resuming-a-subscription"></a>
## Reprise De Souscriptions

Si un utilisateur a annulé sa souscription et souhaiterais la reprendre, utilisez la méthode `resume` : 

  $user->subscription('monthly')->resume($creditCardToken);

Si l'utilisateur annule une souscription et la reprend avant qu'elle ne soit totalement expirée, il ne sera pas facturé immédiatement. La souscription sera simplement réactivée et il sera facturé sur le cycle de facturation d'origine.

<a name="checking-subscription-status"></a>
## Vérification Du Statut D'Une Souscriptions

Pour vérifier que l'utilisateur a souscrit à votre application, utilisez la command `subscribed` :

  if ($user->subscribed())
  {
    //
  }

La méthode `subscribed`fait un bon candidat pour un filtre de route :

  Route::filter('subscribed', function()
  {
    if (Auth::user() && ! Auth::user()->subscribed())
    {
      return Redirect::to('billing');
    }
  });

Vous pouvez aussi déterminer si l'utilisateur est toujours dans la période d'essai en utilisant la méthode `onTrial` : 

  if ($user->onTrial())
  {
    //
  }

Pour déterminer si l'utilisateur a été un abonné actif, mais qu'il a annulé sa souscription, vous pouvez utilisez la méthode `cancelled` :

  if ($user->cancelled())
  {
    //
  }

Vous pouvez aussi déterminer si un utilisateur a annulé une souscription, mais est toujour dans une "période de grâce" sans que sa souscription soit totalement expirée. Par exemple, si un utilisateur a annulé sa souscription le 5 Mars qui devait terminer le 10 Mars, l'utilisateur est en "période de grâce" jusqu'au 10 Mars. Notez que la méthode `subscribed` retourne encore `true` pendant cette période.

  if ($user->onGracePeriod())
  {
    //
  }

La méthode `everSubscribed` peut-être utilisée pour déterminer si un utilisateur a déjà souscrit à un plan dans votre application :

  if ($user->everSubscribed())
  {
    //
  }

La méthode `onPlan`peut-être utilisée pour déterminer si l'utilisateur a souscrit au plan donné en fonction de son ID : 

  if ($user->onPlan('monthly'))
  {
    //
  }

<a name="handling-failed-payments"></a>
## Manipuler Des Paiements Perdu

Que faire si une carte de crédit d'un client expire ? Pas de panique - Cashier inclus un controlleur Webhook qui peut facilement annuler la souscription du client pour vous. il suffit de pointer une route vers le controlleur :

  Route::post('stripe/webhook', 'Laravel\Cashier\WebhookController@handleWebhook');

That's it! Failed payments will be captured and handled by the controller. The controller will cancel the customer's subscription after three failed payment attempts. The `stripe/webhook` URI in this example is just for example. You will need to configure the URI in your Stripe settings.

C'est tout ! Les paiements qui échouent seront capturé et manipulé par le controlleur. Le controlleur annulera la souscription du client après trois tentatives de paiement. L'URI `stripe/webhook` dans cet exemple est juste pour l'exemple. Vous devez configurer l'URI dans votre configuration de Stripe.

<a name="handling-other-stripe-webhooks"></a>
## Manipuler d'Autres Webbooks de Stripe

Si vous avez des evenement de Webhook de Stripe supplémentaire que vous souhaitez gérer, étendez simplement le controller Webhook. Vos nom de méthodes doivent correspondre à la convention prévue pour Cashier, en particulier, les méthodes doivent être préfixées avec `handle`et le nom du Webhook Stripe que vous souhaitez gérer. Par exemple, si vous souhaitez gérer le Webhook `invoice.payment_succeeded`, vous devrez ajouter la méthode `handleInvoicePaymentSucceeded` à votre controlleur.

  class WebhookController extends Laravel\Cashier\WebhookController {

    public function handleInvoicePaymentSucceeded($payload)
    {
      // Handle The Event
    }

  }

> **Note : ** En plus de la mise à jour des informations de souscription dans votre base de données, le controlleur Webhook permet également d'annuler la souscription via une API Stripe.

<a name="invoices"></a>
## Factures

Vous pouvez facilement récupérer un tableau des factures d'un utilisateur à l'aide de la méthode `invoices` :

  $invoices = $user->invoices();

Quand vous listez les factures pour un client, vous pouvez utiliser ces méthodes d'aide pour afficher les informations de la facture correspondante : 

  {{ $invoice->id }}

  {{ $invoice->dateString() }}

  {{ $invoice->dollars() }}

Utilisez la méthode `downloadInvoice` pour générer un téléchargement du PDF de la facture. Oui, c'est si simple :

  return $user->downloadInvoice($invoice->id, [
    'vendor'  => 'Your Company',
    'product' => 'Your Product',
  ]);

