<!-- source: 6e3394c4cc9184d50a078c658d6161b4d79b7e20 -->
# Reactive forms

Les reactive forms proposent une approche pilotée par le modèle pour gérer les saisies de formulaire dont les valeurs évoluent au fil du temps.
Ce guide vous montre comment créer et mettre à jour un contrôle de formulaire simple, utiliser plusieurs contrôles dans un groupe, valider les valeurs d'un formulaire et créer des formulaires dynamiques dans lesquels vous pouvez ajouter ou retirer des contrôles à l'exécution.

## Présentation des reactive forms {#overview-of-reactive-forms}

Les reactive forms adoptent une approche explicite et immuable pour gérer l'état d'un formulaire à un instant donné.
Chaque modification de l'état du formulaire renvoie un nouvel état, ce qui préserve l'intégrité du modèle entre les modifications.
Les reactive forms reposent sur des flux observables, dans lesquels les saisies et les valeurs du formulaire sont fournies sous forme de flux accessibles de manière synchrone.

Les reactive forms facilitent aussi les tests, car vous avez l'assurance que vos données sont cohérentes et prévisibles quand vous les demandez.
Tous les consommateurs de ces flux peuvent manipuler les données en toute sécurité.

Les reactive forms se distinguent des [template-driven forms](guide/forms/template-driven-forms) sur plusieurs points.
Les reactive forms offrent un accès synchrone au modèle de données, l'immuabilité grâce aux opérateurs d'observables et le suivi des modifications via des flux observables.

Les template-driven forms permettent de modifier directement les données dans votre template, mais sont moins explicites que les reactive forms, car ils s'appuient sur des directives intégrées au template ainsi que sur des données mutables pour suivre les modifications de manière asynchrone.
Consultez la [présentation des formulaires](guide/forms) pour une comparaison détaillée des deux paradigmes.

## Ajouter un contrôle de formulaire simple {#adding-a-basic-form-control}

L'utilisation des contrôles de formulaire se fait en trois étapes.

1. Générez un nouveau composant et enregistrez le module des reactive forms. Ce module déclare les directives nécessaires pour utiliser les reactive forms.
1. Instanciez un nouveau `FormControl`.
1. Enregistrez le `FormControl` dans le template.

Vous pouvez ensuite afficher le formulaire en ajoutant le composant au template.

Les exemples suivants montrent comment ajouter un contrôle de formulaire unique.
Dans l'exemple, l'utilisateur saisit son nom dans un champ de saisie ; le formulaire capture cette valeur et affiche la valeur courante de l'élément de contrôle de formulaire.

<docs-workflow>

<docs-step title="Générer un nouveau composant et importer le ReactiveFormsModule {#generate-a-new-component-and-import-the-reactiveformsmodule}">
Utilisez la commande CLI `ng generate component` pour générer un composant dans votre projet, puis importez `ReactiveFormsModule` depuis le package `@angular/forms` et ajoutez-le au tableau `imports` de votre composant.

<docs-code header="name-editor.component.ts (extrait)" path="adev/src/content/examples/reactive-forms/src/app/name-editor/name-editor.component.ts" region="imports" />
</docs-step>

<docs-step title="Déclarer une instance de FormControl {#declare-a-formcontrol-instance}">
Utilisez le constructeur de `FormControl` pour définir sa valeur initiale, ici une chaîne vide. En créant ces contrôles dans la classe de votre composant, vous pouvez immédiatement écouter, mettre à jour et valider l'état de la saisie du formulaire.

<docs-code header="name-editor.component.ts" path="adev/src/content/examples/reactive-forms/src/app/name-editor/name-editor.component.ts" region="create-control"/>
</docs-step>

<docs-step title="Enregistrer le contrôle dans le template {#register-the-control-in-the-template}">
Après avoir créé le contrôle dans la classe du composant, vous devez l'associer à un élément de contrôle de formulaire dans le template. Mettez à jour le template avec le contrôle de formulaire en utilisant le binding `formControl` fourni par `FormControlDirective`, qui est lui aussi inclus dans le `ReactiveFormsModule`.

<docs-code header="name-editor.component.html" path="adev/src/content/examples/reactive-forms/src/app/name-editor/name-editor.component.html" region="control-binding" />

Grâce à la syntaxe de binding du template, le contrôle de formulaire est maintenant enregistré sur l'élément de saisie `name` du template. Le contrôle de formulaire et l'élément du DOM communiquent entre eux : la vue reflète les modifications du modèle, et le modèle reflète les modifications de la vue.
</docs-step>

<docs-step title="Afficher le composant {#display-the-component}">
Le `FormControl` assigné à la propriété `name` s'affiche quand le composant `<app-name-editor>` est ajouté à un template.

<docs-code header="app.component.html (éditeur de nom)" path="adev/src/content/examples/reactive-forms/src/app/app.component.1.html" region="app-name-editor"/>
</docs-step>
</docs-workflow>

### Afficher la valeur d'un contrôle de formulaire {#displaying-a-form-control-value}

Vous pouvez afficher la valeur des manières suivantes :

- Via l'observable `valueChanges`, qui vous permet d'écouter les modifications de la valeur du formulaire dans le template avec `AsyncPipe` ou dans la classe du composant avec la méthode `subscribe()`
- Avec la propriété `value`, qui vous donne un snapshot de la valeur courante

L'exemple suivant montre comment afficher la valeur courante avec l'interpolation dans le template.

<docs-code header="name-editor.component.html (valeur du contrôle)" path="adev/src/content/examples/reactive-forms/src/app/name-editor/name-editor.component.html" region="display-value"/>

La valeur affichée change à mesure que vous modifiez l'élément de contrôle de formulaire.

Les reactive forms donnent accès aux informations sur un contrôle donné via les propriétés et méthodes fournies avec chaque instance.
Ces propriétés et méthodes de la classe sous-jacente [AbstractControl](api/forms/AbstractControl 'API reference') servent à contrôler l'état du formulaire et à déterminer quand afficher des messages lors de la [validation des saisies](#validating-form-input 'Learn more about validating form input').

Découvrez les autres propriétés et méthodes de `FormControl` dans la [référence de l'API](api/forms/FormControl 'Detailed syntax reference').

### Remplacer la valeur d'un contrôle de formulaire {#replacing-a-form-control-value}

Les reactive forms disposent de méthodes pour modifier la valeur d'un contrôle par programmation, ce qui vous permet de mettre à jour la valeur sans interaction de l'utilisateur.
Une instance de contrôle de formulaire fournit une méthode `setValue()` qui met à jour la valeur du contrôle de formulaire et valide la structure de la valeur fournie par rapport à la structure du contrôle.
Par exemple, quand vous récupérez des données de formulaire depuis une API backend ou un service, utilisez la méthode `setValue()` pour mettre à jour le contrôle avec sa nouvelle valeur, en remplaçant entièrement l'ancienne.

L'exemple suivant ajoute à la classe du composant une méthode qui met à jour la valeur du contrôle à _Nancy_ avec la méthode `setValue()`.

<docs-code header="name-editor.component.ts (mise à jour de la valeur)" path="adev/src/content/examples/reactive-forms/src/app/name-editor/name-editor.component.ts" region="update-value"/>

Mettez à jour le template avec un bouton pour simuler une mise à jour du nom.
Quand vous cliquez sur le bouton **Update Name**, la valeur saisie dans l'élément de contrôle de formulaire est reflétée comme sa valeur courante.

<docs-code header="name-editor.component.html (mise à jour de la valeur)" path="adev/src/content/examples/reactive-forms/src/app/name-editor/name-editor.component.html" region="update-value"/>

Le model de formulaire est la source de vérité du contrôle : quand vous cliquez sur le bouton, la valeur de la saisie est donc modifiée dans la classe du composant, ce qui écrase sa valeur courante.

HELPFUL: Dans cet exemple, vous utilisez un seul contrôle.
Quand vous utilisez la méthode `setValue()` avec une instance de [groupe de formulaire](#grouping-form-controls) ou de [tableau de formulaire](#creating-dynamic-forms), la valeur doit correspondre à la structure du groupe ou du tableau.

## Regrouper des contrôles de formulaire {#grouping-form-controls}

Les formulaires contiennent généralement plusieurs contrôles liés entre eux.
Les reactive forms proposent deux façons de regrouper plusieurs contrôles liés dans un même formulaire de saisie.

| Groupes de formulaire | Détails |
| :---------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Groupe de formulaire | Définit un formulaire avec un ensemble fixe de contrôles que vous pouvez gérer ensemble. Les bases des groupes de formulaire sont abordées dans cette section. Vous pouvez aussi [imbriquer des groupes de formulaire](#creating-nested-form-groups 'See more about nesting groups') pour créer des formulaires plus complexes. |
| Tableau de formulaire | Définit un formulaire dynamique, dans lequel vous pouvez ajouter et retirer des contrôles à l'exécution. Vous pouvez aussi imbriquer des tableaux de formulaire pour créer des formulaires plus complexes. Pour en savoir plus sur cette option, consultez [Créer des formulaires dynamiques](#creating-dynamic-forms). |

De la même façon qu'une instance de contrôle de formulaire vous donne le contrôle d'un champ de saisie unique, une instance de groupe de formulaire suit l'état d'un groupe d'instances de contrôle de formulaire \(par exemple, un formulaire\).
Chaque contrôle d'une instance de groupe de formulaire est suivi par son nom lors de la création du groupe de formulaire.
L'exemple suivant montre comment gérer plusieurs instances de contrôle de formulaire dans un même groupe.

Générez un composant `ProfileEditor` et importez les classes `FormGroup` et `FormControl` depuis le package `@angular/forms`.

```shell
ng generate component ProfileEditor
```

<docs-code header="profile-editor.component.ts (imports)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.ts" region="imports"/>

Pour ajouter un groupe de formulaire à ce composant, suivez les étapes suivantes.

1. Créez une instance de `FormGroup`.
1. Associez le model et la vue du `FormGroup`.
1. Enregistrez les données du formulaire.

<docs-workflow>

<docs-step title="Créer une instance de FormGroup {#create-a-formgroup-instance}">
Créez dans la classe du composant une propriété nommée `profileForm` et affectez-lui une nouvelle instance de groupe de formulaire. Pour initialiser le groupe de formulaire, passez au constructeur un objet dont les clés nommées sont associées à leur contrôle.

Pour le formulaire de profil, ajoutez deux instances de contrôle de formulaire nommées `firstName` et `lastName`

<docs-code header="profile-editor.component.ts (groupe de formulaire)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.ts" region="formgroup"/>

Les contrôles de formulaire individuels sont maintenant rassemblés dans un groupe. Une instance de `FormGroup` fournit la valeur de son model sous la forme d'un objet construit à partir des valeurs de chaque contrôle du groupe. Une instance de groupe de formulaire possède les mêmes propriétés (comme `value` et `untouched`) et méthodes (comme `setValue()`) qu'une instance de contrôle de formulaire.
</docs-step>

<docs-step title="Associer le model et la vue du FormGroup {#associate-the-formgroup-model-and-view}">
Un groupe de formulaire suit le statut et les modifications de chacun de ses contrôles : si l'un des contrôles change, le contrôle parent émet lui aussi un nouveau statut ou une nouvelle valeur. Le model du groupe est maintenu à partir de ses membres. Après avoir défini le model, vous devez mettre à jour le template pour refléter le model dans la vue.

<docs-code header="profile-editor.component.html (groupe de formulaire du template)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.html" region="formgroup"/>

De la même façon qu'un groupe de formulaire contient un groupe de contrôles, le `FormGroup` _profileForm_ est lié à l'élément `form` avec la directive `FormGroup`, ce qui crée une couche de communication entre le model et le formulaire contenant les saisies. L'input `formControlName` fourni par la directive `FormControlName` lie chaque saisie individuelle au contrôle de formulaire défini dans le `FormGroup`. Les contrôles de formulaire communiquent avec leurs éléments respectifs. Ils communiquent aussi leurs modifications à l'instance de groupe de formulaire, qui constitue la source de vérité pour la valeur du model.
</docs-step>

<docs-step title="Enregistrer les données du formulaire {#save-form-data}">
Le composant `ProfileEditor` accepte les saisies de l'utilisateur, mais dans un scénario réel, vous voulez capturer la valeur du formulaire et la rendre disponible pour un traitement ultérieur en dehors du composant. La directive `FormGroup` écoute l'événement `submit` émis par l'élément `form` et émet un événement `ngSubmit` que vous pouvez lier à une fonction de callback. Ajoutez un event listener `ngSubmit` à la balise `form` avec la méthode de callback `onSubmit()`.

<docs-code header="profile-editor.component.html (événement submit)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.html" region="ng-submit"/>

La méthode `onSubmit()` du composant `ProfileEditor` capture la valeur courante de `profileForm`. Utilisez `output()` pour garder le formulaire encapsulé et fournir la valeur du formulaire en dehors du composant. L'exemple suivant utilise `console.warn` pour afficher un message dans la console du navigateur.

<docs-code header="profile-editor.component.ts (méthode de soumission)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.ts" region="on-submit"/>

L'événement `submit` est émis par la balise `form` via l'événement DOM natif. Vous déclenchez l'événement en cliquant sur un bouton de type `submit`. Cela permet à l'utilisateur d'appuyer sur la touche **Entrée** pour soumettre le formulaire rempli.

Utilisez un élément `button` pour ajouter un bouton en bas du formulaire afin de déclencher la soumission du formulaire.

<docs-code header="profile-editor.component.html (bouton de soumission)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.html" region="submit-button"/>

Le bouton de l'extrait précédent possède aussi un binding `disabled` qui désactive le bouton quand `profileForm` est invalide. Vous n'effectuez encore aucune validation, donc le bouton est toujours activé. La validation de formulaire de base est abordée dans la section [Valider les saisies du formulaire](#validating-form-input).
</docs-step>

<docs-step title="Afficher le composant {#display-the-component-form-group}">
Pour afficher le composant `ProfileEditor` qui contient le formulaire, ajoutez-le au template d'un composant.

<docs-code header="app.component.html (éditeur de profil)" path="adev/src/content/examples/reactive-forms/src/app/app.component.1.html" region="app-profile-editor"/>

`ProfileEditor` vous permet de gérer les instances de contrôle de formulaire des contrôles `firstName` et `lastName` au sein de l'instance de groupe de formulaire.
</docs-step>
</docs-workflow>

### Créer des groupes de formulaire imbriqués {#creating-nested-form-groups}

Les groupes de formulaire peuvent accepter comme enfants à la fois des instances de contrôle de formulaire individuelles et d'autres instances de groupe de formulaire.
Cela facilite la maintenance des models de formulaire complexes et leur regroupement logique.

Quand vous construisez des formulaires complexes, il est plus simple de gérer les différentes zones d'information dans des sections plus petites.
Utiliser une instance de groupe de formulaire imbriquée vous permet de découper les grands groupes de formulaire en groupes plus petits et plus faciles à gérer.

Pour créer des formulaires plus complexes, suivez les étapes suivantes.

1. Créez un groupe imbriqué.
1. Regroupez le formulaire imbriqué dans le template.

Certains types d'informations se rangent naturellement dans un même groupe.
Un nom et une adresse sont des exemples typiques de tels groupes imbriqués, et sont utilisés dans les exemples suivants.

<docs-workflow>
<docs-step title="Créer un groupe imbriqué {#create-a-nested-group}">
Pour créer un groupe imbriqué dans `profileForm`, ajoutez un élément imbriqué `address` à l'instance de groupe de formulaire.

<docs-code header="profile-editor.component.ts (groupe de formulaire imbriqué)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.ts" region="nested-formgroup"/>

Dans cet exemple, `address group` combine les contrôles existants `firstName` et `lastName` avec les nouveaux contrôles `street`, `city`, `state` et `zip`. Même si l'élément `address` du groupe de formulaire est un enfant de l'élément global `profileForm` du groupe de formulaire, les mêmes règles s'appliquent aux modifications de valeur et de statut. Les modifications de statut et de valeur du groupe de formulaire imbriqué se propagent au groupe de formulaire parent, ce qui maintient la cohérence avec le modèle global.
</docs-step>

<docs-step title="Regrouper le formulaire imbriqué dans le template {#group-the-nested-form-in-the-template}">
Après avoir mis à jour le model dans la classe du composant, mettez à jour le template pour connecter l'instance de groupe de formulaire et ses éléments de saisie. Ajoutez au template de `ProfileEditor` le groupe de formulaire `address` contenant les champs `street`, `city`, `state` et `zip`.

<docs-code header="profile-editor.component.html (groupe de formulaire imbriqué du template)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.html" region="formgroupname"/>

Le formulaire `ProfileEditor` s'affiche comme un seul groupe, mais le model est découpé davantage pour représenter les zones de regroupement logique.

Affichez la valeur de l'instance de groupe de formulaire dans le template du composant avec la propriété `value` et `JsonPipe`.
</docs-step>
</docs-workflow>

### Mettre à jour des parties du modèle de données {#updating-parts-of-the-data-model}

Quand vous mettez à jour la valeur d'une instance de groupe de formulaire qui contient plusieurs contrôles, vous pouvez vouloir ne mettre à jour que certaines parties du modèle.
Cette section explique comment mettre à jour des parties spécifiques du modèle de données d'un contrôle de formulaire.

Il existe deux façons de mettre à jour la valeur du model :

| Méthodes        | Détails |
| :------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `setValue()`   | Définit une nouvelle valeur pour un contrôle individuel. La méthode `setValue()` respecte strictement la structure du groupe de formulaire et remplace entièrement la valeur du contrôle. |
| `patchValue()` | Remplace toutes les propriétés définies dans l'objet qui ont changé dans le model de formulaire. |

Les vérifications strictes de la méthode `setValue()` aident à détecter les erreurs d'imbrication dans les formulaires complexes, tandis que `patchValue()` échoue silencieusement sur ces erreurs.

Dans `ProfileEditorComponent`, utilisez la méthode `updateProfile` avec l'exemple suivant pour mettre à jour le prénom et l'adresse de l'utilisateur.

<docs-code header="profile-editor.component.ts (patch de la valeur)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.ts" region="patch-value"/>

Simulez une mise à jour en ajoutant au template un bouton qui met à jour le profil de l'utilisateur à la demande.

<docs-code header="profile-editor.component.html (mise à jour de la valeur)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.html" region="patch-value"/>

Quand un utilisateur clique sur le bouton, le model `profileForm` est mis à jour avec de nouvelles valeurs pour `firstName` et `street`. Notez que `street` est fourni dans un objet à l'intérieur de la propriété `address`.
C'est nécessaire, car la méthode `patchValue()` applique la mise à jour en suivant la structure du model.
`patchValue()` ne met à jour que les propriétés que le model de formulaire définit.

## Utiliser le service FormBuilder pour générer des contrôles {#using-the-formbuilder-service-to-generate-controls}

Créer manuellement des instances de contrôle de formulaire peut devenir répétitif quand vous gérez plusieurs formulaires.
Le service `FormBuilder` fournit des méthodes pratiques pour générer des contrôles.

Suivez les étapes suivantes pour tirer parti de ce service.

1. Importez la classe `FormBuilder`.
1. Injectez le service `FormBuilder`.
1. Générez le contenu du formulaire.

Les exemples suivants montrent comment refactoriser le composant `ProfileEditor` pour utiliser le service FormBuilder afin de créer les instances de contrôle de formulaire et de groupe de formulaire.

<docs-workflow>
<docs-step title="Importer la classe FormBuilder {#import-the-formbuilder-class}">
Importez la classe `FormBuilder` depuis le package `@angular/forms`.

<docs-code header="profile-editor.component.ts (import)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.2.ts" region="form-builder-imports"/>

</docs-step>

<docs-step title="Injecter le service FormBuilder {#inject-the-formbuilder-service}">
Le service `FormBuilder` est un provider injectable issu du module des reactive forms. Utilisez la fonction `inject()` pour injecter cette dépendance dans votre composant.

<docs-code header="profile-editor.component.ts (initialisation de la propriété)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.2.ts" region="inject-form-builder"/>

</docs-step>
<docs-step title="Générer les contrôles de formulaire {#generate-form-controls}">
Le service `FormBuilder` dispose de trois méthodes : `control()`, `group()` et `array()`. Ce sont des méthodes de fabrique qui génèrent des instances dans les classes de vos composants, notamment des contrôles de formulaire, des groupes de formulaire et des tableaux de formulaire. Utilisez la méthode `group` pour créer les contrôles de `profileForm`.

<docs-code header="profile-editor.component.ts (FormBuilder)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.2.ts" region="form-builder"/>

Dans l'exemple précédent, vous utilisez la méthode `group()` avec le même objet pour définir les propriétés du model. La valeur de chaque nom de contrôle est un tableau contenant la valeur initiale comme premier élément.

TIP: Vous pouvez définir le contrôle avec seulement la valeur initiale, mais si vos contrôles nécessitent une validation synchrone ou asynchrone, ajoutez les validateurs synchrones et asynchrones comme deuxième et troisième éléments du tableau. Comparez l'utilisation du FormBuilder avec la création manuelle des instances.

  <docs-code-multifile>
    <docs-code header="profile-editor.component.ts (instances)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.1.ts" region="formgroup-compare"/>
    <docs-code header="profile-editor.component.ts (FormBuilder)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.2.ts" region="formgroup-compare"/>
  </docs-code-multifile>
</docs-step>

</docs-workflow>

## Valider les saisies du formulaire {#validating-form-input}

La _validation de formulaire_ sert à garantir que les saisies de l'utilisateur sont complètes et correctes.
Cette section explique comment ajouter un validateur unique à un contrôle de formulaire et afficher le statut global du formulaire.
La validation de formulaire est traitée plus en détail dans le guide [Validation de formulaire](guide/forms/form-validation).

Suivez les étapes suivantes pour ajouter une validation de formulaire.

1. Importez une fonction de validation dans le composant de votre formulaire.
1. Ajoutez le validateur au champ du formulaire.
1. Ajoutez la logique de gestion du statut de validation.

La validation la plus courante consiste à rendre un champ obligatoire.
L'exemple suivant montre comment ajouter une validation obligatoire au contrôle `firstName` et afficher le résultat de la validation.

<docs-workflow>
<docs-step title="Importer une fonction de validation {#import-a-validator-function}">
Les reactive forms incluent un ensemble de fonctions de validation pour les cas d'usage courants. Ces fonctions reçoivent le contrôle à valider et renvoient un objet d'erreur ou une valeur nulle selon le résultat de la vérification.

Importez la classe `Validators` depuis le package `@angular/forms`.

<docs-code header="profile-editor.component.ts (import)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.ts" region="validator-imports"/>
</docs-step>

<docs-step title="Rendre un champ obligatoire {#make-a-field-required}">
Dans le composant `ProfileEditor`, ajoutez la méthode statique `Validators.required` comme deuxième élément du tableau du contrôle `firstName`.

<docs-code header="profile-editor.component.ts (validateur required)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.ts" region="required-validator"/>
</docs-step>

<docs-step title="Afficher le statut du formulaire {#display-form-status}">
Quand vous ajoutez un champ obligatoire au contrôle de formulaire, son statut initial est invalide. Ce statut invalide se propage à l'élément de groupe de formulaire parent, ce qui rend son statut invalide. Accédez au statut courant de l'instance de groupe de formulaire via sa propriété `status`.

Affichez le statut courant de `profileForm` avec l'interpolation.

<docs-code header="profile-editor.component.html (affichage du statut)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.html" region="display-status"/>

Le bouton **Submit** est désactivé, car `profileForm` est invalide en raison du contrôle de formulaire `firstName` obligatoire. Une fois la saisie `firstName` remplie, le formulaire devient valide et le bouton **Submit** est activé.

Pour en savoir plus sur la validation de formulaire, consultez le guide [Validation de formulaire](guide/forms/form-validation).
</docs-step>
</docs-workflow>

## Créer des formulaires dynamiques {#creating-dynamic-forms}

`FormArray` est une alternative à `FormGroup` pour gérer un nombre quelconque de contrôles non nommés.
Comme avec les instances de groupe de formulaire, vous pouvez insérer et retirer dynamiquement des contrôles dans les instances de tableau de formulaire, et la valeur ainsi que le statut de validation de l'instance de tableau de formulaire sont calculés à partir de ses contrôles enfants.
En revanche, vous n'avez pas besoin de définir une clé nommée pour chaque contrôle : c'est donc une excellente option si vous ne connaissez pas à l'avance le nombre de valeurs enfants.

Pour définir un formulaire dynamique, suivez les étapes suivantes.

1. Importez la classe `FormArray`.
1. Définissez un contrôle `FormArray`.
1. Accédez au contrôle `FormArray` avec une méthode getter.
1. Affichez le tableau de formulaire dans un template.

L'exemple suivant vous montre comment gérer un tableau d'_alias_ dans `ProfileEditor`.

<docs-workflow>
<docs-step title="Importer la classe `FormArray` {#import-the-formarray-class}">
Importez la classe `FormArray` depuis `@angular/forms` pour l'utiliser comme information de type. Le service `FormBuilder` est prêt à créer une instance de `FormArray`.

<docs-code header="profile-editor.component.ts (import)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.2.ts" region="form-array-imports"/>
</docs-step>

<docs-step title="Définir un contrôle `FormArray` {#define-a-formarray-control}">
Vous pouvez initialiser un tableau de formulaire avec un nombre quelconque de contrôles, de zéro à plusieurs, en les définissant dans un tableau. Ajoutez une propriété `aliases` à l'instance de groupe de formulaire de `profileForm` pour définir le tableau de formulaire.

Utilisez la méthode `FormBuilder.array()` pour définir le tableau, et la méthode `FormBuilder.control()` pour remplir le tableau avec un contrôle initial.

<docs-code header="profile-editor.component.ts (tableau de formulaire aliases)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.ts" region="aliases"/>

Le contrôle aliases de l'instance de groupe de formulaire contient maintenant un seul contrôle, jusqu'à ce que d'autres contrôles soient ajoutés dynamiquement.
</docs-step>

<docs-step title="Accéder au contrôle `FormArray` {#access-the-formarray-control}">
Un getter donne accès aux alias de l'instance de tableau de formulaire, plutôt que de répéter la méthode `profileForm.get()` pour obtenir chaque instance. L'instance de tableau de formulaire représente un nombre indéfini de contrôles dans un tableau. Il est pratique d'accéder à un contrôle via un getter, et cette approche se répète facilement pour d'autres contrôles. <br />

Utilisez la syntaxe getter pour créer une propriété de classe `aliases` qui récupère le contrôle de tableau de formulaire des alias depuis le groupe de formulaire parent.

<docs-code header="profile-editor.component.ts (getter aliases)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.ts" region="aliases-getter"/>

Comme le contrôle renvoyé est de type `AbstractControl`, vous devez fournir un type explicite pour accéder à la syntaxe des méthodes de l'instance de tableau de formulaire. Définissez une méthode pour insérer dynamiquement un contrôle d'alias dans le tableau de formulaire des alias. La méthode `FormArray.push()` insère le contrôle comme nouvel élément du tableau, et vous pouvez aussi passer un tableau de contrôles à FormArray.push() pour enregistrer plusieurs contrôles à la fois.

<docs-code header="profile-editor.component.ts (ajout d'un alias)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.ts" region="add-alias"/>

Dans le template, chaque contrôle s'affiche comme un champ de saisie distinct.

</docs-step>

<docs-step title="Afficher le tableau de formulaire dans le template {#display-the-form-array-in-the-template}">

Pour rattacher les alias de votre model de formulaire, vous devez les ajouter au template. De la même façon que l'input `formGroupName` fourni par `FormGroupName`, `formArrayName` relie l'instance de tableau de formulaire au template avec `FormArrayName`.

Ajoutez le HTML de template suivant après la `<div>` qui ferme l'élément `formGroupName`.

<docs-code header="profile-editor.component.html (template du tableau de formulaire aliases)" path="adev/src/content/examples/reactive-forms/src/app/profile-editor/profile-editor.component.html" region="formarrayname"/>

Le bloc `@for` parcourt chaque instance de contrôle de formulaire fournie par l'instance de tableau de formulaire aliases. Comme les éléments d'un tableau de formulaire ne sont pas nommés, vous affectez l'index à la variable `i` et le passez à chaque contrôle pour le lier à l'input `formControlName`.

Chaque fois qu'une nouvelle instance d'alias est ajoutée, la nouvelle instance de tableau de formulaire reçoit son contrôle en fonction de l'index. Cela vous permet de suivre chaque contrôle individuel lors du calcul du statut et de la valeur du contrôle racine.

NOTE: Dans les applications zoneless, modifier le model d'un reactive form (par exemple en appelant `FormArray.push()`) ne planifie pas automatiquement la détection de changements du composant. Si votre template dépend de modifications structurelles du model, comme `aliases.controls`, assurez-vous que le composant notifie Angular pour lancer la détection de changements, par exemple en reliant un observable des formulaires à `ChangeDetectorRef.markForCheck()` :

```ts
import {ChangeDetectorRef, Component, inject} from '@angular/core';
import {takeUntilDestroyed} from '@angular/core/rxjs-interop';

@Component({/* ... */})
export class ProfileEditor {
  private readonly cdr = inject(ChangeDetectorRef);

  constructor() {
    this.profileForm.valueChanges
      .pipe(takeUntilDestroyed())
      .subscribe(() => this.cdr.markForCheck());
  }
}
```

</docs-step>

### Utiliser `FormArrayDirective` pour les tableaux de formulaire de premier niveau {#using-formarraydirective-for-top-level-form-arrays}

Vous pouvez lier un `FormArray` directement à un élément `<form>` en utilisant la `FormArrayDirective`.  
C'est utile quand le formulaire n'utilise pas de `FormGroup` de premier niveau et que le tableau lui-même représente l'ensemble du model de formulaire.

```angular-ts
import {Component} from '@angular/core';
import {FormArray, FormControl, ReactiveFormsModule} from '@angular/forms';

@Component({
  selector: 'form-array-example',
  imports: [ReactiveFormsModule],
  template: `
    <form [formArray]="form">
      @for (control of form.controls; track $index) {
        <input [formControlName]="$index" />
      }
    </form>
  `,
})
export class FormArrayExampleComponent {
  controls = [new FormControl('fish'), new FormControl('cat'), new FormControl('dog')];

  form = new FormArray(this.controls);
}
```

<docs-step title="Ajouter un alias {#add-an-alias}">

Au départ, le formulaire contient un seul champ `Alias`. Pour ajouter un autre champ, cliquez sur le bouton **Add Alias**. Vous pouvez aussi valider le tableau d'alias rapporté par le model de formulaire, affiché par `Form Value` en bas du template. Au lieu d'une instance de contrôle de formulaire pour chaque alias, vous pouvez composer une autre instance de groupe de formulaire avec des champs supplémentaires. Le processus de définition d'un contrôle pour chaque élément reste le même.
</docs-step>

</docs-workflow>

## Événements unifiés de changement d'état des contrôles {#unified-control-state-change-events}

Tous les contrôles de formulaire exposent un flux unique et unifié d'**événements de changement d'état des contrôles** via l'observable `events` d'`AbstractControl` (`FormControl`, `FormGroup`, `FormArray` et `FormRecord`).
Ce flux unifié vous permet de réagir aux changements d'état de **valeur**, de **statut**, **pristine**, **touched** et **reset**, ainsi qu'aux **actions au niveau du formulaire** comme la **soumission**, ce qui vous permet de gérer toutes les mises à jour avec un seul abonnement au lieu de brancher plusieurs observables.

### Types d'événements {#event-types}

Chaque élément émis par `events` est une instance d'une classe d'événement spécifique :

- **`ValueChangeEvent`** — quand la **valeur** du contrôle change.
- **`StatusChangeEvent`** — quand le **statut de validation** du contrôle passe à l'une des valeurs de `FormControlStatus` (`VALID`, `INVALID`, `PENDING` ou `DISABLED`).
- **`PristineChangeEvent`** — quand l'état **pristine/dirty** du contrôle change.
- **`TouchedChangeEvent`** — quand l'état **touched/untouched** du contrôle change.
- **`FormResetEvent`** — quand un contrôle ou un formulaire est réinitialisé, soit via l'API `reset()`, soit via une action native.
- **`FormSubmittedEvent`** — quand le formulaire est soumis.

Toutes les classes d'événement étendent `ControlEvent` et incluent une référence `source` vers l'`AbstractControl` à l'origine du changement, ce qui est utile dans les grands formulaires.

```ts
import {Component} from '@angular/core';
import {
  FormControl,
  ValueChangeEvent,
  StatusChangeEvent,
  PristineChangeEvent,
  TouchedChangeEvent,
  FormResetEvent,
  FormSubmittedEvent,
  ReactiveFormsModule,
  FormGroup,
} from '@angular/forms';

@Component(/* ... */)
export class UnifiedEventsBasicComponent {
  form = new FormGroup({
    username: new FormControl(''),
  });

  constructor() {
    this.form.events.subscribe((e) => {
      if (e instanceof ValueChangeEvent) {
        console.log('Value changed to: ', e.value);
      }

      if (e instanceof StatusChangeEvent) {
        console.log('Status changed to: ', e.status);
      }

      if (e instanceof PristineChangeEvent) {
        console.log('Pristine status changed to: ', e.pristine);
      }

      if (e instanceof TouchedChangeEvent) {
        console.log('Touched status changed to: ', e.touched);
      }

      if (e instanceof FormResetEvent) {
        console.log('Form was reset');
      }

      if (e instanceof FormSubmittedEvent) {
        console.log('Form was submitted');
      }
    });
  }
}
```

### Filtrer des événements spécifiques {#filtering-specific-events}

Privilégiez les opérateurs RxJS quand vous n'avez besoin que d'un sous-ensemble des types d'événements.

```ts
import {filter} from 'rxjs/operators';
import {StatusChangeEvent} from '@angular/forms';

control.events
  .pipe(filter((e) => e instanceof StatusChangeEvent))
  .subscribe((e) => console.log('Status:', e.status));
```

### Unifier plusieurs abonnements {#unifying-from-multiple-subscriptions}

**Avant**

```ts
import {combineLatest} from 'rxjs';

combineLatest([control.valueChanges, control.statusChanges]).subscribe(([value, status]) => {
  /* ... */
});
```

**Après**

```ts
control.events.subscribe((e) => {
  // Handle ValueChangeEvent, StatusChangeEvent, etc.
});
```

NOTE: Lors d'un changement de valeur, l'émission a lieu juste après la mise à jour de la valeur de ce contrôle. La valeur d'un contrôle parent (par exemple si ce FormControl fait partie d'un FormGroup) est mise à jour plus tard : accéder à la valeur d'un contrôle parent (avec la propriété `value`) depuis le callback de cet événement peut donc renvoyer une valeur qui n'a pas encore été mise à jour. Abonnez-vous plutôt aux `events` du contrôle parent.

## Gérer l'état des contrôles de formulaire {#managing-form-control-state}

Les reactive forms suivent l'état des contrôles via **touched/untouched** et **pristine/dirty**. Angular les met à jour automatiquement lors des interactions avec le DOM, mais vous pouvez aussi les gérer par programmation.

**[`markAsTouched`](api/forms/FormControl#markAsTouched)** — Marque un contrôle ou un formulaire comme touché, à la suite d'événements de focus et de blur qui ne modifient pas la valeur. Se propage par défaut aux contrôles parents.

```ts
// Show validation errors after user leaves a field
onEmailBlur() {
  const email = this.form.get('email');
  email.markAsTouched();
}
```

**[`markAsUntouched`](api/forms/FormControl#markAsUntouched)** — Marque un contrôle ou un formulaire comme non touché. Se répercute sur tous les contrôles enfants et recalcule le statut touched de tous les contrôles parents.

```ts
// Reset form state after successful submission
onSubmitSuccess() {
  this.form.markAsUntouched();
  this.form.markAsPristine();
}
```

**[`markAsDirty`](api/forms/FormControl#markAsDirty)** — Marque un contrôle ou un formulaire comme modifié (dirty), ce qui signifie que la valeur a changé. Se propage par défaut aux contrôles parents.

```ts
// Mark programmatically changed values as modified
autofillAddress() {
  const previousAddress = getAddress();
  this.form.patchValue(previousAddress, { emitEvent: false });
  this.form.markAsDirty();
}
```

**[`markAsPristine`](api/forms/FormControl#markAsPristine)** — Marque un contrôle ou un formulaire comme intact (pristine). Marque tous les contrôles enfants comme intacts et recalcule le statut pristine de tous les contrôles parents.

```ts
// Reset pristine state after saving to track new changes
saveForm() {
  this.api.save(this.form.value).subscribe(() => {
    this.form.markAsPristine();
  });
}
```

**[`markAllAsDirty`](api/forms/FormControl#markAllAsDirty)** — Marque le contrôle ou le formulaire et tous ses contrôles descendants comme modifiés (dirty).

```ts
// Mark imported data as dirty
loadData(data: FormData) {
  this.form.patchValue(data);
  this.form.markAllAsDirty();
}
```

**[`markAllAsTouched`](api/forms/FormControl#markAllAsTouched)** — Marque le contrôle ou le formulaire et tous ses contrôles descendants comme touchés. Utile pour afficher les erreurs de validation sur l'ensemble du formulaire.

```ts
// Show all validation errors before submission
onSubmit() {
  if (this.form.invalid) {
    this.form.markAllAsTouched();
    return;
  }
  this.saveForm();
}
```

## Contrôler l'émission et la propagation des événements {#controlling-event-emission-and-propagation}

Quand vous mettez à jour des contrôles de formulaire par programmation, vous contrôlez précisément la manière dont les modifications se propagent dans la hiérarchie du formulaire et si des événements sont émis.

### Comprendre l'émission des événements {#understanding-event-emission}

Par défaut (`emitEvent: true`), toute modification d'un contrôle émet des événements via les observables `valueChanges` et `statusChanges`. Définir `emitEvent: false` supprime ces émissions, ce qui est utile pour définir des valeurs par programmation sans déclencher de comportement réactif comme la sauvegarde automatique, pour éviter des mises à jour circulaires entre contrôles, ou pour effectuer des mises à jour groupées où les événements ne doivent être émis qu'une seule fois à la fin.

```ts
@Component({/* ... */})
export class BlogPostEditor {
  postForm = new FormGroup({
    title: new FormControl(''),
    content: new FormControl(''),
  });

  constructor() {
    // Auto-save draft every time user types
    this.postForm.valueChanges.subscribe((formValue) => {
      this.autosaveDraft(formValue);
    });
  }

  loadExistingDraft(savedDraft: {title: string; content: string}) {
    // Restore draft without triggering auto-save
    this.postForm.setValue(savedDraft, {emitEvent: false});
  }
}
```

### Comprendre le contrôle de la propagation {#understanding-propagation-control}

Par défaut (`onlySelf: false`), les mises à jour se répercutent sur les contrôles parents, dont les valeurs et le statut de validation sont recalculés. Définir `onlySelf: true` limite la mise à jour au contrôle courant et empêche la notification du parent. C'est utile pour les opérations groupées où vous voulez déclencher manuellement une seule fois la mise à jour du parent.

```ts
updatePostalCodeValidator(country: string) {
  const postal = this.addressForm.get('postalCode');

  const validators = country === 'US'
    ? [Validators.maxLength(5)]
    : [Validators.maxLength(7)];

  postal.setValidators(validators);
  postal.updateValueAndValidity({ onlySelf: true, emitEvent: false });
}
```

HELPFUL: Pour gérer dynamiquement les validateurs à l'exécution, consultez la section [Gérer dynamiquement les validateurs dans les reactive forms](guide/forms/form-validation#managing-validators-dynamically-in-reactive-forms) du guide Validation de formulaire.

## Fonctions utilitaires pour affiner les types de contrôles de formulaire {#utility-functions-for-narrowing-form-control-types}

Angular fournit quatre fonctions utilitaires qui aident à déterminer le type concret d'un `AbstractControl`. Ces fonctions agissent comme des **type guards** et affinent le type du contrôle quand elles renvoient `true`, ce qui vous permet d'accéder en toute sécurité aux propriétés propres au sous-type dans le même bloc.

| Fonction utilitaire | Détails |
| :--------------- | :-------------------------------------------------- |
| `isFormControl`  | Renvoie `true` quand le contrôle est un `FormControl`. |
| `isFormGroup`    | Renvoie `true` quand le contrôle est un `FormGroup` |
| `isFormRecord`   | Renvoie `true` quand le contrôle est un `FormRecord` |
| `isFormArray`    | Renvoie `true` quand le contrôle est un `FormArray` |

Ces helpers sont particulièrement utiles dans les **validateurs personnalisés**, où la signature de la fonction reçoit un `AbstractControl`, alors que la logique est destinée à un type de contrôle précis.

```ts
import {AbstractControl, isFormArray} from '@angular/forms';

export function positiveValues(control: AbstractControl) {
  if (!isFormArray(control)) {
    return null; // Not a FormArray: validator is not applicable.
  }

  // Safe to access FormArray-specific API after narrowing.
  const hasNegative = control.controls.some((c) => c.value < 0);
  return hasNegative ? {positiveValues: true} : null;
}
```

## Résumé de l'API des reactive forms {#reactive-forms-api-summary}

Le tableau suivant liste les classes de base et les services utilisés pour créer et gérer les contrôles de formulaire des reactive forms.
Pour le détail complet de la syntaxe, consultez la documentation de référence de l'API du [package Forms](api#angular_forms 'API reference').

### Classes {#classes}

| Classe             | Détails |
| :---------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AbstractControl` | La classe de base abstraite des classes concrètes de contrôle de formulaire `FormControl`, `FormGroup` et `FormArray`. Elle fournit leurs comportements et propriétés communs. |
| `FormControl`     | Gère la valeur et le statut de validité d'un contrôle de formulaire individuel. Il correspond à un contrôle de formulaire HTML comme `<input>` ou `<select>`. |
| `FormGroup`       | Gère la valeur et l'état de validité d'un groupe d'instances d'`AbstractControl`. Les propriétés du groupe incluent ses contrôles enfants. Le formulaire de premier niveau de votre composant est un `FormGroup`. |
| `FormArray`       | Gère la valeur et l'état de validité d'un tableau indexé numériquement d'instances d'`AbstractControl`. |
| `FormBuilder`     | Un service injectable qui fournit des méthodes de fabrique pour créer des instances de contrôles. |
| `FormRecord`      | Suit la valeur et l'état de validité d'une collection d'instances de `FormControl`, qui ont toutes le même type de valeur. |

### Directives {#directives}

| Directive              | Détails |
| :--------------------- | :----------------------------------------------------------------------------------------- |
| `FormControlDirective` | Synchronise une instance autonome de `FormControl` avec un élément de contrôle de formulaire. |
| `FormControlName`      | Synchronise par son nom un `FormControl` d'une instance existante de `FormGroup` avec un élément de contrôle de formulaire. |
| `FormGroupDirective`   | Synchronise une instance existante de `FormGroup` avec un élément du DOM. |
| `FormGroupName`        | Synchronise une instance imbriquée de `FormGroup` avec un élément du DOM. |
| `FormArrayName`        | Synchronise une instance imbriquée de `FormArray` avec un élément du DOM. |
| `FormArrayDirective`   | Synchronise une instance autonome de `FormArray` avec un élément du DOM. |
