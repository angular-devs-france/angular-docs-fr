<!-- source: b847642eb19eee0f690d62ba966bc2dee8a51080 -->
# Créer un template-driven form

Ce tutoriel vous montre comment créer un template-driven form. Les éléments de contrôle du formulaire sont liés à des propriétés de données dotées d'une validation des saisies. La validation des saisies aide à préserver l'intégrité des données, et le style améliore l'expérience utilisateur.

Les template-driven forms utilisent le [two-way binding](guide/templates/two-way-binding) pour mettre à jour le modèle de données du composant quand des modifications sont faites dans le template, et inversement.

<docs-callout helpful title="Template-driven forms ou reactive forms">
Angular prend en charge deux approches de conception pour les formulaires interactifs. Les template-driven forms vous permettent d'utiliser des directives spécifiques aux formulaires dans votre template Angular. Les reactive forms proposent une approche pilotée par le modèle pour construire des formulaires.

Les template-driven forms sont un excellent choix pour les formulaires petits ou simples, tandis que les reactive forms passent mieux à l'échelle et conviennent aux formulaires complexes. Pour une comparaison des deux approches, consultez [Choisir une approche](guide/forms#choosing-an-approach)
</docs-callout>

Vous pouvez créer presque n'importe quel type de formulaire avec un template Angular : formulaires de connexion, formulaires de contact et à peu près tous les formulaires métier.
Vous pouvez disposer les contrôles de façon créative et les lier aux données de votre modèle objet.
Vous pouvez définir des règles de validation et afficher les erreurs de validation, autoriser sous condition la saisie dans certains contrôles, déclencher un retour visuel intégré, et bien plus encore.

## Objectifs {#objectives}

Ce tutoriel vous apprend à :

- Créer un formulaire Angular avec un composant et un template
- Utiliser `ngModel` pour créer des two-way bindings qui lisent et écrivent les valeurs des contrôles de saisie
- Fournir un retour visuel grâce à des classes CSS spéciales qui suivent l'état des contrôles
- Afficher les erreurs de validation aux utilisateurs et autoriser sous condition la saisie dans les contrôles de formulaire selon l'état du formulaire
- Partager des informations entre éléments HTML à l'aide de [variables de référence de template](guide/templates/variables#template-reference-variables)

## Créer un template-driven form {#build-a-template-driven-form}

Les template-driven forms s'appuient sur des directives définies dans le `FormsModule`.

| Directives     | Détails                                                                                                                                                                                                                                                                         |
| :------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `NgModel`      | Réconcilie les changements de valeur de l'élément de formulaire associé avec les changements du modèle de données, ce qui vous permet de répondre aux saisies de l'utilisateur avec de la validation et de la gestion d'erreurs.                                                     |
| `NgForm`       | Crée une instance `FormGroup` de premier niveau et la lie à un élément `<form>` pour suivre la valeur agrégée du formulaire et son état de validation. Dès que vous importez `FormsModule`, cette directive est active par défaut sur toutes les balises `<form>`. Vous n'avez pas besoin d'ajouter de selector particulier. |
| `NgModelGroup` | Crée une instance `FormGroup` et la lie à un élément du DOM.                                                                                                                                                                                                                      |

### Aperçu des étapes {#step-overview}

Au cours de ce tutoriel, vous liez un exemple de formulaire à des données et gérez les saisies de l'utilisateur en suivant les étapes ci-dessous.

1. Créez le formulaire de base.
   - Définissez un exemple de modèle de données
   - Incluez l'infrastructure nécessaire, comme le `FormsModule`
1. Liez les contrôles de formulaire aux propriétés de données à l'aide de la directive `ngModel` et de la syntaxe du two-way binding.
   - Examinez comment `ngModel` signale l'état des contrôles à l'aide de classes CSS
   - Nommez les contrôles pour les rendre accessibles à `ngModel`
1. Suivez la validité des saisies et l'état des contrôles avec `ngModel`.
   - Ajoutez du CSS personnalisé pour fournir un retour visuel sur l'état
   - Affichez et masquez les messages d'erreur de validation
1. Répondez à un événement natif de clic sur un bouton HTML en ajoutant des données au modèle.
1. Gérez la soumission du formulaire à l'aide de la propriété d'output [`ngSubmit`](api/forms/NgForm#ngSubmit) du formulaire.
   - Désactivez le bouton **Submit** tant que le formulaire n'est pas valide
   - Après la soumission, remplacez le formulaire terminé par un autre contenu sur la page

## Créer le formulaire {#build-the-form}

<!-- TODO: link to preview -->
<!-- <docs-code live/> -->

1. L'application d'exemple fournie crée la classe `Actor`, qui définit le modèle de données reflété dans le formulaire.

   <docs-code header="actor.ts" language="typescript" path="adev/src/content/examples/forms/src/app/actor.ts"/>

1. La mise en page et les détails du formulaire sont définis dans la classe `ActorFormComponent`.

   <docs-code header="actor-form.component.ts (v1)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.ts" region="v1"/>

   La valeur « app-actor-form » du `selector` du composant signifie que vous pouvez placer ce formulaire dans un template parent avec la balise `<app-actor-form>`.

1. Le code suivant crée une nouvelle instance d'acteur, afin que le formulaire initial puisse afficher un exemple d'acteur.

   <docs-code language="typescript" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.ts" language="typescript" region="Marilyn"/>

   Cette démo utilise des données factices pour `model` et `skills`.
   Dans une vraie application, vous injecteriez un service de données pour obtenir et enregistrer de vraies données, ou vous exposeriez ces propriétés sous forme d'inputs et d'outputs.

1. Le composant active la fonctionnalité Forms en important le module `FormsModule`.

   <docs-code language="typescript" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.ts" language="typescript" region="imports"/>

1. Le formulaire est affiché dans la mise en page de l'application définie par le template du composant racine.

   <docs-code header="app.component.html" language="html" path="adev/src/content/examples/forms/src/app/app.component.html"/>

   Le template initial définit la mise en page d'un formulaire avec deux groupes de formulaire et un bouton de soumission.
   Les groupes de formulaire correspondent à deux propriétés du modèle de données Actor : name et studio.
   Chaque groupe comporte un libellé et une zone de saisie.
   - L'élément de contrôle `<input>` **Name** possède l'attribut HTML5 `required`
   - L'élément de contrôle `<input>` **Studio** ne l'a pas, car `studio` est facultatif

   Le bouton **Submit** porte quelques classes de style.
   À ce stade, la mise en page du formulaire est entièrement en HTML5 pur, sans binding ni directive.

1. L'exemple de formulaire utilise quelques classes de style de [Twitter Bootstrap](https://getbootstrap.com/css) : `container`, `form-group`, `form-control` et `btn`.
   Pour utiliser ces styles, la feuille de style de l'application importe la library.

   <docs-code header="styles.css" path="adev/src/content/examples/forms/src/styles.1.css"/>

1. Le formulaire exige que la compétence d'un acteur soit choisie dans une liste prédéfinie de `skills` gérée en interne dans `ActorFormComponent`.
   La boucle Angular `@for` parcourt les valeurs de données pour remplir l'élément `<select>`.

   <docs-code header="actor-form.component.html (skills)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="skills"/>

Si vous lancez l'application maintenant, vous voyez la liste des compétences dans le contrôle de sélection.
Les éléments de saisie ne sont pas encore liés à des valeurs de données ni à des événements : ils sont donc encore vides et n'ont aucun comportement.

## Lier les contrôles de saisie aux propriétés de données {#bind-input-controls-to-data-properties}

L'étape suivante consiste à lier les contrôles de saisie aux propriétés `Actor` correspondantes avec le two-way binding, afin qu'ils répondent aux saisies de l'utilisateur en mettant à jour le modèle de données, et qu'ils répondent aussi aux changements programmatiques des données en mettant à jour l'affichage.

La directive `ngModel` déclarée dans le `FormsModule` vous permet de lier les contrôles de votre template-driven form aux propriétés de votre modèle de données.
Quand vous incluez la directive avec la syntaxe du two-way binding, `[(ngModel)]`, Angular peut suivre la valeur du contrôle et les interactions de l'utilisateur avec celui-ci, et garder la vue synchronisée avec le model.

1. Modifiez le fichier de template `actor-form.component.html`.
1. Trouvez la balise `<input>` à côté du libellé **Name**.
1. Ajoutez la directive `ngModel` avec la syntaxe du two-way binding `[(ngModel)]="..."`.

<docs-code header="actor-form.component.html (extrait)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="ngModelName-1"/>

HELPFUL: Cet exemple comporte une interpolation de diagnostic temporaire après chaque balise input, `{{model.name}}`, pour afficher la valeur actuelle de la propriété correspondante. Le commentaire vous rappelle de supprimer les lignes de diagnostic quand vous avez fini d'observer le two-way binding en action.

### Accéder à l'état global du formulaire {#access-the-overall-form-status}

Quand vous avez importé le `FormsModule` dans votre composant, Angular a automatiquement créé et attaché une directive [NgForm](api/forms/NgForm) à la balise `<form>` du template (car `NgForm` a le selector `form`, qui correspond aux éléments `<form>`).

Pour accéder au `NgForm` et à l'état global du formulaire, déclarez une [variable de référence de template](guide/templates/variables#template-reference-variables).

1. Modifiez le fichier de template `actor-form.component.html`.
1. Ajoutez à la balise `<form>` une variable de référence de template, `#actorForm`, et définissez sa valeur comme suit.

   <docs-code header="actor-form.component.html (extrait)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="template-variable"/>

   La variable de template `actorForm` est désormais une référence à l'instance de la directive `NgForm` qui gouverne le formulaire dans son ensemble.

1. Lancez l'application.
1. Commencez à taper dans la zone de saisie **Name**.

   À mesure que vous ajoutez et supprimez des caractères, vous les voyez apparaître et disparaître du modèle de données.

La ligne de diagnostic qui affiche les valeurs interpolées montre que les valeurs circulent réellement de la zone de saisie vers le model, et inversement.

### Nommer les éléments de contrôle {#naming-control-elements}

Quand vous utilisez `[(ngModel)]` sur un élément, vous devez définir un attribut `name` pour cet élément.
Angular utilise le nom attribué pour enregistrer l'élément auprès de la directive `NgForm` attachée à l'élément `<form>` parent.

L'exemple a ajouté un attribut `name` à l'élément `<input>` et l'a défini à « name », ce qui est logique pour le nom de l'acteur.
N'importe quelle valeur unique convient, mais un nom descriptif est utile.

1. Ajoutez des bindings `[(ngModel)]` et des attributs `name` similaires à **Studio** et **Skill**.
1. Vous pouvez maintenant supprimer les messages de diagnostic qui affichent les valeurs interpolées.
1. Pour vérifier que le two-way binding fonctionne pour tout le modèle d'acteur, ajoutez en haut du template du composant un nouveau binding de texte avec le pipe [`json`](api/common/JsonPipe), qui sérialise les données en chaîne de caractères.

Après ces modifications, le template du formulaire devrait ressembler à ceci :

<docs-code header="actor-form.component.html (extrait)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="ngModel-2"/>

Vous remarquerez que :

- Chaque élément `<input>` possède une propriété `id`.
  Elle est utilisée par l'attribut `for` de l'élément `<label>` pour associer le libellé à son contrôle de saisie.
  Il s'agit d'une [fonctionnalité HTML standard](https://developer.mozilla.org/docs/Web/HTML/Element/label).

- Chaque élément `<input>` possède aussi la propriété `name` requise, qu'Angular utilise pour enregistrer le contrôle auprès du formulaire.

Une fois les effets observés, vous pouvez supprimer le binding de texte `{{ model | json }}`.

## Suivre l'état du formulaire {#track-form-states}

Angular applique la classe `ng-submitted` aux éléments `form` une fois le formulaire soumis. Cette classe peut servir à modifier le style du formulaire après sa soumission.

## Suivre l'état des contrôles {#track-control-states}

Ajouter la directive `NgModel` à un contrôle lui ajoute des noms de classes qui décrivent son état.
Ces classes peuvent servir à modifier le style d'un contrôle selon son état.

Le tableau suivant décrit les noms de classes qu'Angular applique selon l'état du contrôle.

| États                            | Classe si vrai | Classe si faux |
| :------------------------------- | :------------ | :------------- |
| Le contrôle a été visité.        | `ng-touched`  | `ng-untouched` |
| La valeur du contrôle a changé.  | `ng-dirty`    | `ng-pristine`  |
| La valeur du contrôle est valide. | `ng-valid`    | `ng-invalid`   |

Angular applique aussi la classe `ng-submitted` aux éléments `form` lors de la soumission, mais pas aux contrôles à l'intérieur de l'élément `form`.

Vous utilisez ces classes CSS pour définir les styles de votre contrôle selon son état.

### Observer l'état des contrôles {#observe-control-states}

Pour voir comment le framework ajoute et retire les classes, ouvrez les outils de développement du navigateur et inspectez l'élément `<input>` qui représente le nom de l'acteur.

1. À l'aide des outils de développement de votre navigateur, trouvez l'élément `<input>` qui correspond à la zone de saisie **Name**.
   Vous pouvez voir que l'élément possède plusieurs classes CSS en plus de « form-control ».

1. Au premier affichage, les classes indiquent que sa valeur est valide, que la valeur n'a pas été modifiée depuis l'initialisation ou la réinitialisation, et que le contrôle n'a pas été visité depuis l'initialisation ou la réinitialisation.

   ```html
   <input class="form-control ng-untouched ng-pristine ng-valid" />
   ```

1. Effectuez les actions suivantes sur la zone `<input>` **Name** et observez les classes qui apparaissent.
   - Regardez sans toucher.
     Les classes indiquent qu'il est non touché, intact (pristine) et valide.

   - Cliquez dans la zone du nom, puis cliquez en dehors.
     Le contrôle a maintenant été visité, et l'élément porte la classe `ng-touched` au lieu de la classe `ng-untouched`.

   - Ajoutez des barres obliques à la fin du nom.
     Il est maintenant touché et modifié (dirty).

   - Effacez le nom.
     La valeur devient alors invalide, donc la classe `ng-invalid` remplace la classe `ng-valid`.

### Créer un retour visuel pour les états {#create-visual-feedback-for-states}

La paire `ng-valid`/`ng-invalid` est particulièrement intéressante, car vous voulez envoyer un
signal visuel fort quand les valeurs sont invalides.
Vous voulez aussi signaler les champs obligatoires.

Vous pouvez signaler à la fois les champs obligatoires et les données invalides avec une barre colorée
à gauche de la zone de saisie.

Pour modifier l'apparence de cette façon, suivez les étapes ci-dessous.

1. Ajoutez des définitions pour les classes CSS `ng-*`.
1. Ajoutez ces définitions de classes dans un nouveau fichier `forms.css`.
1. Ajoutez le nouveau fichier au projet, au même niveau que `index.html` :

   <docs-code header="forms.css" language="css" path="adev/src/content/examples/forms/src/assets/forms.css"/>

1. Dans le fichier `index.html`, modifiez la balise `<head>` pour inclure la nouvelle feuille de style.

   <docs-code header="index.html (styles)" path="adev/src/content/examples/forms/src/index.html" region="styles"/>

### Afficher et masquer les messages d'erreur de validation {#show-and-hide-validation-error-messages}

La zone de saisie **Name** est obligatoire, et la vider fait passer la barre au rouge.
Cela indique que quelque chose ne va pas, mais l'utilisateur ne sait pas quoi ni comment y remédier.
Vous pouvez fournir un message utile en vérifiant l'état du contrôle et en y réagissant.

La liste déroulante **Skill** est aussi obligatoire, mais elle n'a pas besoin de ce type de gestion d'erreurs, car la liste de sélection limite déjà le choix à des valeurs valides.

Pour définir et afficher un message d'erreur au moment opportun, suivez les étapes ci-dessous.

<docs-workflow>
<docs-step title="Ajouter une référence locale à l'input {#add-a-local-reference-to-the-input}">
Ajoutez à la balise `input` une variable de référence de template qui vous permet d'accéder au contrôle Angular de la zone de saisie depuis le template. Dans l'exemple, la variable est `#name="ngModel"`.

La variable de référence de template (`#name`) est définie à `"ngModel"`, car c'est la valeur de la propriété [`NgModel.exportAs`](api/core/Directive#exportAs). Cette propriété indique à Angular comment relier une variable de référence à une directive.
</docs-step>

<docs-step title="Ajouter le message d'erreur {#add-the-error-message}">
Ajoutez un `<div>` qui contient un message d'erreur adapté.
</docs-step>

<docs-step title="Rendre le message d'erreur conditionnel {#make-the-error-message-conditional}">
Affichez ou masquez le message d'erreur en liant des propriétés du contrôle `name` à la propriété `hidden` de l'élément `<div>` du message.
</docs-step>

<docs-code header="actor-form.component.html (hidden-error-msg)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="hidden-error-msg"/>

<docs-step title="Ajouter un message d'erreur conditionnel au nom {#add-a-conditional-error-message-to-name}">
Ajoutez un message d'erreur conditionnel à la zone de saisie `name`, comme dans l'exemple suivant.

<docs-code header="actor-form.component.html (extrait)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="name-with-error-msg"/>
</docs-step>
</docs-workflow>

<docs-callout title="Illustration de l'état « pristine »">

Dans cet exemple, vous masquez le message quand le contrôle est soit valide, soit _intact (pristine)_.
Intact signifie que l'utilisateur n'a pas modifié la valeur depuis son affichage dans ce formulaire.
Si vous ignorez l'état `pristine`, vous ne masqueriez le message que lorsque la valeur est valide.
Si vous arrivez dans ce composant avec un nouvel acteur vierge ou un acteur invalide, vous verrez le message d'erreur immédiatement, avant même d'avoir fait quoi que ce soit.

Vous voudrez peut-être que le message ne s'affiche que lorsque l'utilisateur effectue une modification invalide.
Masquer le message tant que le contrôle est dans l'état `pristine` permet d'atteindre cet objectif.
Vous verrez l'importance de ce choix quand vous ajouterez un nouvel acteur au formulaire à l'étape suivante.

</docs-callout>

## Ajouter un nouvel acteur {#add-a-new-actor}

Cet exercice montre comment répondre à un événement natif de clic sur un bouton HTML en ajoutant des données au modèle.
Pour permettre aux utilisateurs du formulaire d'ajouter un nouvel acteur, vous allez ajouter un bouton **New Actor** qui répond à un événement de clic.

1. Dans le template, placez un élément `<button>` « New Actor » en bas du formulaire.
1. Dans le fichier du composant, ajoutez la méthode de création d'acteur au modèle de données des acteurs.

   <docs-code header="actor-form.component.ts (méthode New Actor)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.ts" region="new-actor"/>

1. Liez l'événement de clic du bouton à une méthode de création d'acteur, `newActor()`.

   <docs-code header="actor-form.component.html (bouton New Actor)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="new-actor-button-no-reset"/>

1. Relancez l'application et cliquez sur le bouton **New Actor**.

   Le formulaire se vide, et les barres _obligatoire_ à gauche de la zone de saisie sont rouges, ce qui indique que les propriétés `name` et `skill` sont invalides.
   Remarquez que les messages d'erreur sont masqués.
   C'est parce que le formulaire est intact : vous n'avez encore rien modifié.

1. Saisissez un nom et cliquez à nouveau sur **New Actor**.

   L'application affiche maintenant un message d'erreur `Name is required`, car la zone de saisie n'est plus intacte.
   Le formulaire se souvient que vous avez saisi un nom avant de cliquer sur **New Actor**.

1. Pour rétablir l'état intact des contrôles du formulaire, effacez tous les indicateurs de façon impérative en appelant la méthode `reset()` du formulaire après avoir appelé la méthode `newActor()`.

   <docs-code header="actor-form.component.html (réinitialiser le formulaire)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="new-actor-button-form-reset"/>

   Désormais, cliquer sur **New Actor** réinitialise à la fois le formulaire et les indicateurs de ses contrôles.

## Soumettre le formulaire avec `ngSubmit` {#submit-the-form-with-ngsubmit}

L'utilisateur doit pouvoir soumettre ce formulaire après l'avoir rempli.
Le bouton **Submit** en bas du formulaire ne fait rien par lui-même, mais il déclenche un événement de soumission du formulaire du fait de son type (`type="submit"`).

Pour répondre à cet événement, suivez les étapes ci-dessous.

<docs-workflow>

<docs-step title="Écouter ngOnSubmit {#listen-to-ngonsubmit}">
Liez la propriété d'événement [`ngSubmit`](api/forms/NgForm#ngSubmit) du formulaire à la méthode `onSubmit()` du composant actor-form.

<docs-code header="actor-form.component.html (ngSubmit)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="ngSubmit"/>
</docs-step>

<docs-step title="Lier la propriété disabled {#bind-the-disabled-property}">
Utilisez la variable de référence de template `#actorForm` pour accéder au formulaire qui contient le bouton **Submit** et créer un event binding.

Vous allez lier la propriété du formulaire qui indique sa validité globale à la propriété `disabled` du bouton **Submit**.

<docs-code header="actor-form.component.html (submit-button)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="submit-button"/>
</docs-step>

<docs-step title="Lancer l'application {#run-the-application}">
Remarquez que le bouton est activé, même s'il ne fait encore rien d'utile.
</docs-step>

<docs-step title="Supprimer la valeur de Name {#delete-the-name-value}">
Cela enfreint la règle « obligatoire » : le message d'erreur s'affiche, et remarquez que le bouton **Submit** est aussi désactivé.

Vous n'avez pas eu à relier explicitement l'état activé du bouton à la validité du formulaire.
Le `FormsModule` l'a fait automatiquement quand vous avez défini une variable de référence de template sur l'élément de formulaire enrichi, puis fait référence à cette variable dans le contrôle du bouton.
</docs-step>
</docs-workflow>

### Répondre à la soumission du formulaire {#respond-to-form-submission}

Pour montrer une réponse à la soumission du formulaire, vous pouvez masquer la zone de saisie des données et afficher autre chose à sa place.

<docs-workflow>
<docs-step title="Envelopper le formulaire {#wrap-the-form}">
Enveloppez tout le formulaire dans un `<div>` et liez sa propriété `hidden` à la propriété `ActorFormComponent.submitted`.

<docs-code header="actor-form.component.html (extrait)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="edit-div"/>

Le formulaire principal est visible dès le départ, car la propriété `submitted` vaut false jusqu'à ce que vous soumettiez le formulaire, comme le montre ce fragment de `ActorFormComponent` :

<docs-code header="actor-form.component.ts (submitted)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.ts" region="submitted"/>

Quand vous cliquez sur le bouton **Submit**, l'indicateur `submitted` passe à true et le formulaire disparaît.
</docs-step>

<docs-step title="Ajouter l'état soumis {#add-the-submitted-state}">
Pour afficher autre chose pendant que le formulaire est à l'état soumis, ajoutez le HTML suivant sous le nouveau `<div>` d'enveloppe.

<docs-code header="actor-form.component.html (extrait)" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="submitted"/>

Ce `<div>`, qui affiche un acteur en lecture seule avec des bindings d'interpolation, n'apparaît que lorsque le composant est à l'état soumis.

L'affichage alternatif comprend un bouton _Edit_ dont l'événement de clic est lié à une expression qui efface l'indicateur `submitted`.
</docs-step>

<docs-step title="Tester le bouton Edit {#test-the-edit-button}">
Cliquez sur le bouton *Edit* pour revenir à l'affichage du formulaire modifiable.
</docs-step>
</docs-workflow>

## Résumé {#summary}

Le formulaire Angular présenté sur cette page tire parti des fonctionnalités suivantes
du framework pour prendre en charge la modification des données, la validation, et plus encore.

- Un template de formulaire HTML Angular
- Une classe de composant de formulaire avec un décorateur `@Component`
- La gestion de la soumission du formulaire par un binding sur la propriété d'événement `NgForm.ngSubmit`
- Des variables de référence de template comme `#actorForm` et `#name`
- La syntaxe `[(ngModel)]` pour le two-way binding
- L'utilisation d'attributs `name` pour la validation et le suivi des changements des éléments de formulaire
- La propriété `valid` de la variable de référence sur les contrôles de saisie, qui indique si un contrôle est valide ou doit afficher des messages d'erreur
- Le contrôle de l'état activé du bouton **Submit** par un binding sur la validité de `NgForm`
- Des classes CSS personnalisées qui fournissent aux utilisateurs un retour visuel sur les contrôles invalides

Voici le code de la version finale de l'application :

<docs-code-multifile>
    <docs-code header="actor-form.component.ts" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.ts" region="final"/>
    <docs-code header="actor-form.component.html" path="adev/src/content/examples/forms/src/app/actor-form/actor-form.component.html" region="final"/>
    <docs-code header="actor.ts" path="adev/src/content/examples/forms/src/app/actor.ts"/>
    <docs-code header="app.component.html" path="adev/src/content/examples/forms/src/app/app.component.html"/>
    <docs-code header="app.component.ts" path="adev/src/content/examples/forms/src/app/app.component.ts"/>
    <docs-code header="main.ts" path="adev/src/content/examples/forms/src/main.ts"/>
    <docs-code header="forms.css" path="adev/src/content/examples/forms/src/assets/forms.css"/>
</docs-code-multifile>
