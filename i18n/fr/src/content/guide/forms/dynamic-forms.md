<!-- source: be680110bde2d94a8487eb388a3402c94f87c60a -->
# Créer des formulaires dynamiques

De nombreux formulaires, comme les questionnaires, peuvent se ressembler beaucoup dans leur format et leur objectif.
Pour générer plus rapidement et plus facilement différentes versions d'un tel formulaire, vous pouvez créer un _template de formulaire dynamique_ basé sur des métadonnées qui décrivent le modèle objet métier.
Utilisez ensuite ce template pour générer automatiquement de nouveaux formulaires en fonction des changements du modèle de données.

Cette technique est particulièrement utile quand vous avez un type de formulaire dont le contenu doit changer fréquemment pour répondre à des exigences métier et réglementaires en évolution rapide.
Un cas d'usage typique est le questionnaire.
Vous pouvez avoir besoin de recueillir les saisies des utilisateurs dans différents contextes.
Le format et le style des formulaires que voit un utilisateur doivent rester constants, tandis que les questions que vous devez réellement poser varient selon le contexte.

Dans ce tutoriel, vous allez créer un formulaire dynamique qui présente un questionnaire simple.
Vous créez une application en ligne pour des héros en recherche d'emploi.
L'agence modifie constamment le processus de candidature, mais grâce au formulaire dynamique,
vous pouvez créer les nouveaux formulaires à la volée sans modifier le code de l'application.

Le tutoriel vous guide à travers les étapes suivantes :

1. Activer les reactive forms dans un projet.
1. Établir un modèle de données pour représenter les contrôles de formulaire.
1. Remplir le modèle avec des données d'exemple.
1. Développer un composant pour créer dynamiquement des contrôles de formulaire.

Le formulaire que vous créez utilise la validation des saisies et des styles pour améliorer l'expérience utilisateur.
Il comporte un bouton Submit qui n'est activé que lorsque toutes les saisies de l'utilisateur sont valides, et signale les saisies invalides par un code couleur et des messages d'erreur.

Cette version de base peut évoluer pour prendre en charge une plus grande variété de questions, un rendu plus soigné et une meilleure expérience utilisateur.

## Activer les reactive forms dans votre projet {#enable-reactive-forms-for-your-project}

Les formulaires dynamiques reposent sur les reactive forms.

Pour donner à l'application accès aux directives des reactive forms, importez `ReactiveFormsModule` depuis le package `@angular/forms` dans les composants qui en ont besoin.

<docs-code-multifile>
    <docs-code header="dynamic-form.component.ts" path="adev/src/content/examples/dynamic-form/src/app/dynamic-form.component.ts"/>
    <docs-code header="dynamic-form-question.component.ts" path="adev/src/content/examples/dynamic-form/src/app/dynamic-form-question.component.ts"/>
</docs-code-multifile>

## Créer un modèle objet de formulaire {#create-a-form-object-model}

Un formulaire dynamique nécessite un modèle objet capable de décrire tous les scénarios requis par les fonctionnalités du formulaire.
L'exemple de formulaire de candidature pour héros est un ensemble de questions : chaque contrôle du formulaire doit poser une question et accepter une réponse.

Le modèle de données de ce type de formulaire doit représenter une question.
L'exemple inclut le `DynamicFormQuestionComponent`, qui définit une question comme l'objet fondamental du modèle.

La classe `QuestionBase` suivante est une classe de base pour un ensemble de contrôles capables de représenter la question et sa réponse dans le formulaire.

<docs-code header="question-base.ts" path="adev/src/content/examples/dynamic-form/src/app/question-base.ts"/>

### Définir les classes de contrôles {#define-control-classes}

À partir de cette base, l'exemple dérive deux nouvelles classes, `TextboxQuestion` et `DropdownQuestion`, qui représentent différents types de contrôles.
Quand vous créerez le template du formulaire à l'étape suivante, vous instancierez ces types de questions spécifiques afin d'afficher dynamiquement les contrôles appropriés.

Le type de contrôle `TextboxQuestion` est représenté dans un template de formulaire par un élément `<input>`. Il présente une question et permet aux utilisateurs de saisir une réponse. L'attribut `type` de l'élément est défini à partir du champ `type` indiqué dans l'argument `options` (par exemple `text`, `email`, `url`).

<docs-code header="question-textbox.ts" path="adev/src/content/examples/dynamic-form/src/app/question-textbox.ts"/>

Le type de contrôle `DropdownQuestion` présente une liste de choix dans une liste déroulante.

 <docs-code header="question-dropdown.ts" path="adev/src/content/examples/dynamic-form/src/app/question-dropdown.ts"/>

### Composer les groupes de formulaire {#compose-form-groups}

Un formulaire dynamique utilise un service pour créer des ensembles groupés de contrôles de saisie, à partir du model de formulaire.
Le `QuestionControlService` suivant rassemble un ensemble d'instances `FormGroup` qui consomment les métadonnées du modèle de questions.
Vous pouvez définir des valeurs par défaut et des règles de validation.

<docs-code header="question-control.service.ts" path="adev/src/content/examples/dynamic-form/src/app/question-control.service.ts"/>

## Composer le contenu du formulaire dynamique {#compose-dynamic-form-contents}

Le formulaire dynamique lui-même est représenté par un composant conteneur, que vous ajouterez à une étape ultérieure.
Chaque question est représentée dans le template du composant de formulaire par une balise `<app-question>`, qui correspond à une instance de `DynamicFormQuestionComponent`.

Le `DynamicFormQuestionComponent` se charge d'afficher les détails d'une question individuelle à partir des valeurs de l'objet question lié aux données.
Le formulaire s'appuie sur une [directive `[formGroup]`](api/forms/FormGroupDirective 'API reference') pour relier le HTML du template aux objets de contrôle sous-jacents.
Le `DynamicFormQuestionComponent` crée des groupes de formulaire et les remplit avec les contrôles définis dans le modèle de questions, en précisant les règles d'affichage et de validation.

<docs-code-multifile>
  <docs-code header="dynamic-form-question.component.html" path="adev/src/content/examples/dynamic-form/src/app/dynamic-form-question.component.html"/>
  <docs-code header="dynamic-form-question.component.ts" path="adev/src/content/examples/dynamic-form/src/app/dynamic-form-question.component.ts"/>
</docs-code-multifile>

Le rôle du `DynamicFormQuestionComponent` est de présenter les types de questions définis dans votre modèle.
Vous n'avez que deux types de questions pour l'instant, mais vous pouvez en imaginer bien d'autres.
Le bloc `@switch` du template détermine le type de question à afficher.
Le switch utilise des directives ayant les selectors [`formControlName`](api/forms/FormControlName 'FormControlName directive API reference') et [`formGroup`](api/forms/FormGroupDirective 'FormGroupDirective API reference').
Ces deux directives sont définies dans `ReactiveFormsModule`.

### Fournir les données {#supply-data}

Un autre service est nécessaire pour fournir l'ensemble précis de questions à partir duquel construire un formulaire donné.
Pour cet exercice, vous créez le `QuestionService` pour fournir ce tableau de questions à partir des données d'exemple codées en dur.
Dans une application réelle, le service pourrait récupérer les données depuis un système backend.
L'essentiel, cependant, est que vous contrôlez entièrement les questions de candidature des héros via les objets renvoyés par `QuestionService`.
Pour maintenir le questionnaire au fil de l'évolution des exigences, il vous suffit d'ajouter, de modifier et de supprimer des objets du tableau `questions`.

Le `QuestionService` fournit un ensemble de questions sous la forme d'un tableau lié à l'`input()` questions.

<docs-code header="question.service.ts" path="adev/src/content/examples/dynamic-form/src/app/question.service.ts"/>

## Créer un template de formulaire dynamique {#create-a-dynamic-form-template}

Le composant `DynamicFormComponent` est le point d'entrée et le conteneur principal du formulaire, représenté par `<app-dynamic-form>` dans un template.

Le composant `DynamicFormComponent` présente une liste de questions en liant chacune d'elles à un élément `<app-question>` qui correspond au `DynamicFormQuestionComponent`.

<docs-code-multifile>
    <docs-code header="dynamic-form.component.html" path="adev/src/content/examples/dynamic-form/src/app/dynamic-form.component.html"/>
    <docs-code header="dynamic-form.component.ts" path="adev/src/content/examples/dynamic-form/src/app/dynamic-form.component.ts"/>
</docs-code-multifile>

### Afficher le formulaire {#display-the-form}

Pour afficher une instance du formulaire dynamique, le template shell `AppComponent` transmet le tableau `questions` renvoyé par le `QuestionService` au composant conteneur du formulaire, `<app-dynamic-form>`.

<docs-code header="app.component.ts" path="adev/src/content/examples/dynamic-form/src/app/app.component.ts"/>

Cette séparation entre modèle et données vous permet de réutiliser les composants pour tout type d'enquête, tant qu'il est compatible avec le modèle objet _question_.

### Garantir des données valides {#ensuring-valid-data}

Le template du formulaire utilise un data binding dynamique des métadonnées pour afficher le formulaire sans faire d'hypothèse codée en dur sur des questions spécifiques.
Il ajoute dynamiquement à la fois les métadonnées des contrôles et les critères de validation.

Pour garantir des saisies valides, le bouton _Save_ est désactivé tant que le formulaire n'est pas dans un état valide.
Quand le formulaire est valide, cliquez sur _Save_ et l'application affiche les valeurs actuelles du formulaire au format JSON.

La figure suivante montre le formulaire final.

<img alt="Formulaire dynamique" src="assets/images/guide/dynamic-form/dynamic-form.png">

## Prochaines étapes {#next-steps}

<docs-pill-row>
  <docs-pill title="Valider les saisies d'un formulaire" href="guide/forms/reactive-forms#validating-form-input" />
  <docs-pill title="Guide de validation des formulaires" href="guide/forms/form-validation" />
</docs-pill-row>
