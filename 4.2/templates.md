# Templates

- [Layouts de contrôleur](#controller-layouts)
- [Le moteur de template Blade](#blade-templating)
- [Structures de contrôle Blade](#other-blade-control-structures)

<a name="controller-layouts"></a>
## Layouts de contrôleur

Une méthode pour utiliser les templates dans Laravel est d'utiliser les layouts de contrôleur. En spécifiant la propriété `layout` sur un contrôleur, la vue spécifiée sera créée pour vous et sera utilisée en tant que réponse aux actions.

#### Définition d'un layout sur un contrôleur

	class UserController extends BaseController {

		/**
		 * The layout that should be used for responses.
		 */
		protected $layout = 'layouts.master';

		/**
		 * Show the user profile.
		 */
		public function showProfile()
		{
			$this->layout->content = View::make('user.profile');
		}

	}

<a name="blade-templating"></a>
## Le moteur de template Blade

Blade est un moteur de template simple et puissant fourni par Laravel. A la différence des layouts de contrôleurs, Blade est conduit par _l'héritage de template_ et _les sections_. Les templates Blade doivent avoir comme extension `.blade.php`.

#### Définition d'un layout Blade

	<!-- app/views/layouts/master.blade.php -->

	<html>
		<body>
			@section('sidebar')
				This is the master sidebar.
			@show

			<div class="container">
				@yield('content')
			</div>
		</body>
	</html>

#### Utilisation d'un layout Blade

	@extends('layouts.master')

	@section('sidebar')
		@parent

		<p>This is appended to the master sidebar.</p>
	@stop

	@section('content')
		<p>This is my body content.</p>
	@stop

Notez que les vues qui `extend` un layout Blade surchargent simplement les sections du layout. Le contenu du layout peut être inclus dans une vue enfant en utilisant la directive `@parent` dans une section, vous permettant d'ajouter dans le contenu du layout votre propre contenu, pour par exemple ajouter des liens dans la sidebar ou dans le footer.

Parfois, quand vous n'êtes pas sûr qu'une section a été définie, vous pouvez passer une valeur par défaut à la directive `@yield`. Vous pouvez passer la valeur par défaut comme second argument :

    @yield('section', 'Default Content');

<a name="other-blade-control-structures"></a>
## Structures de contrôle Blade

#### Affichage de données

    Hello, {{{ $name }}}.

    The current UNIX timestamp is {{{ time() }}}.

#### Affichage de données si elles existent

Parfois, vous pourrez vouloir afficher une variable, sans être sûr cependant qu'elle soit définie. Le code équivalent serait :

    {{{ isset($name) ? $name : 'Default' }}}

Cependant, plutôt que d'écrire une condition ternaire, Blade permet d'utiliser un raccourci :

    {{{ $name or 'Default' }}}

#### Affichage de textes bruts avec accolades

Si vous avez besoin d'afficher un texte brut entre accolades et sans traitement par Blade, vous devez préfixer les accolades de votre texte avec un symbole `@` :

    @{{ Ceci ne sera pas traité par Blade }}

Bien sûr, toutes les données utilisateurs doivent être échappées ou purifiées. Pour échapper la sortie, utilisez trois accollades :

	Hello, {{{ $name }}}.

Si vous ne souhaitez pas que vos données soient échappées, il vous faut utiliser les doubles crochets :

        Hello, {{ $name }}.

> **Note:** Soyez vraiment prudent lors de l'affichage de contenu soumit par les utilisateurs de votre application. Utilisez toujours les triples accollades pour échapper toutes les entitées HTML dans le contenu.

#### Déclaration If

    @if (count($records) === 1)
        I have one record!
    @elseif (count($records) > 1)
        I have multiple records!
	@else
		I don't have any records!
	@endif

	@unless (Auth::check())
		You are not signed in.
	@endunless

#### Boucles

	@for ($i = 0; $i < 10; $i++)
		The current value is {{ $i }}
	@endfor

	@foreach ($users as $user)
		<p>This is user {{ $user->id }}</p>
	@endforeach

	@while (true)
		<p>I'm looping forever.</p>
	@endwhile

#### Inclusion d'une sous-vue

	@include('view.name')

Vous pouvez aussi passer un tableau de données à la vue incluse :

    @include('view.name', array('some'=>'data'))

#### Sections de remplacement

Par défaut, les sections sont ajoutées à n'importe quel contenu précédent qui existe dans la section. Pour remplacer une section entièrement, vous pouvez utiliser la déclaration `overwrite`:

    @extends('list.item.container')

    @section('list.item.content')
        <p>This is an item of type {{ $item->type }}</p>
    @overwrite

#### Affichage d'une ligne de langue

	@lang('language.line')

	@choice('language.line', 1);

#### Commentaires

	{{-- This comment will not be in the rendered HTML --}}
