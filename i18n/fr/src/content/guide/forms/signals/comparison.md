<!-- source: 8524ce9d652dde638e91dbaf0869c94134122b84 -->
# Comparaison avec les autres approches de formulaires

Angular propose trois approches pour créer des formulaires : les Signal Forms, les reactive forms et les template-driven forms. Chacune a ses propres patterns pour gérer l'état, la validation et le flux de données. Ce guide vous aide à comprendre les différences et à choisir l'approche adaptée à votre projet.

## Comparaison rapide {#quick-comparison}

| Fonctionnalité   | Signal Forms                       | Reactive forms                        | Template-driven forms   |
| ---------------- | ---------------------------------- | ------------------------------------- | ----------------------- |
| Source de vérité | Model en signal modifiable défini par l'utilisateur | `FormControl`/`FormGroup`             | Model utilisateur dans le composant |
| Sûreté du typage | Inférée depuis le model            | Explicite avec les formulaires typés  | Minimale                |
| Validation       | Schéma avec validateurs basés sur des chemins | Liste de validateurs passée aux contrôles | Basée sur des directives |
| Gestion de l'état | Basée sur les signals             | Basée sur les observables             | Gérée par Angular       |
| Mise en place    | Signal + fonction de schéma        | Arbre de FormControl                  | NgModel dans le template |
| Idéal pour       | Applications basées sur les signals | Formulaires complexes                | Formulaires simples     |
| Courbe d'apprentissage | Moyenne                      | Moyenne à élevée                      | Faible                  |
| Statut           | Stable (v22+)                      | Stable                                | Stable                  |

## Par l'exemple : un formulaire de connexion {#by-example-login-form}

La meilleure façon de comprendre les différences est de voir le même formulaire implémenté avec les trois approches.

<docs-code-multifile>
  <docs-code language="angular-ts" header="Signal Forms" path="adev/src/content/examples/signal-forms/src/comparison/app/signal-forms.ts"/>
  <docs-code header="Reactive forms" path="adev/src/content/examples/signal-forms/src/comparison/app/reactive-forms.ts"/>
  <docs-code header="Template-driven forms" path="adev/src/content/examples/signal-forms/src/comparison/app/template-driven-forms.ts"/>
</docs-code-multifile>

## Comprendre les différences {#understanding-the-differences}

Les trois approches font des choix de conception différents qui influencent la façon dont vous écrivez et maintenez vos formulaires. Ces différences viennent de l'endroit où chaque approche stocke l'état du formulaire et de la façon dont elle gère la validation.

### Où vivent les données de votre formulaire {#where-your-form-data-lives}

La différence la plus fondamentale est l'endroit que chaque approche considère comme la « source de vérité » pour les valeurs du formulaire.

Les Signal Forms stockent les données dans un signal modifiable. Lorsque vous avez besoin des valeurs courantes du formulaire, vous appelez le signal :

```ts
const credentials = this.loginModel(); // { email: '...', password: '...' }
```

Cela conserve les données de votre formulaire dans un conteneur réactif unique qui notifie automatiquement Angular quand les valeurs changent. La structure du formulaire reflète exactement votre modèle de données.

Les reactive forms stockent les données dans des instances de FormControl et de FormGroup. Vous accédez aux valeurs via la hiérarchie du formulaire :

```ts
const credentials = this.loginForm.value; // { email: '...', password: '...' }
```

Cela sépare la gestion de l'état du formulaire du modèle de données de votre composant. La structure du formulaire est explicite mais demande plus de code de mise en place.

Les template-driven forms stockent les données dans les propriétés du composant. Vous accédez directement aux valeurs :

```ts
const credentials = {email: this.email, password: this.password};
```

C'est l'approche la plus directe, mais elle vous oblige à assembler manuellement les valeurs lorsque vous en avez besoin. Angular gère l'état du formulaire via des directives dans le template.

### Comment fonctionne la validation {#how-validation-works}

Chaque approche définit les règles de validation différemment, ce qui influence l'endroit où vit votre logique de validation et la façon dont vous la maintenez.

Les Signal Forms utilisent une fonction de schéma dans laquelle vous associez des validateurs aux chemins des champs :

```ts
loginForm = form(this.loginModel, (fieldPath) => {
  required(fieldPath.email, {message: 'Email is required'});
  email(fieldPath.email, {message: 'Enter a valid email address'});
});
```

Toutes les règles de validation sont regroupées en un seul endroit. La fonction de schéma s'exécute une seule fois à la création du formulaire, et les validateurs s'exécutent automatiquement quand les valeurs des champs changent. Les messages d'erreur font partie de la définition de la validation.

Les reactive forms attachent les validateurs à la création des contrôles :

```ts
loginForm = new FormGroup({
  email: new FormControl('', [Validators.required, Validators.email]),
});
```

Les validateurs sont liés à chaque contrôle de la structure du formulaire. La validation est ainsi répartie dans toute la définition de votre formulaire. Les messages d'erreur se trouvent généralement dans votre template.

Les template-driven forms utilisent des attributs de directive dans le template :

```html
<input [(ngModel)]="email" required email />
```

Les règles de validation se trouvent dans votre template, à côté du HTML. La validation reste ainsi proche de l'interface, mais la logique est répartie entre le template et le composant.

### Sûreté du typage et autocomplétion {#type-safety-and-autocomplete}

L'intégration avec TypeScript diffère sensiblement d'une approche à l'autre, ce qui influence la mesure dans laquelle le compilateur vous aide à éviter les erreurs.

Les Signal Forms infèrent les types à partir de la structure de votre model :

```ts
const loginModel = signal({email: '', password: ''});
const loginForm = form(loginModel);
// TypeScript knows: loginForm.email exists and returns FieldState<string>
```

Vous définissez la forme de vos données une seule fois dans le signal, et TypeScript sait automatiquement quels champs existent et quels sont leurs types. Accéder à `loginForm.username` (qui n'existe pas) produit une erreur de type.

Les reactive forms nécessitent des annotations de type explicites avec les formulaires typés :

```ts
const loginForm = new FormGroup({
  email: new FormControl<string>(''),
  password: new FormControl<string>(''),
});
// TypeScript knows: loginForm.controls.email is FormControl<string>
```

Vous précisez les types de chaque contrôle individuellement. TypeScript valide la structure de votre formulaire, mais vous maintenez les informations de type séparément de votre modèle de données.

Les template-driven forms offrent une sûreté du typage minimale :

```ts
email = '';
password = '';
// TypeScript only knows these are strings, no form-level typing
```

TypeScript comprend les propriétés de votre composant mais ne connaît ni la structure du formulaire ni la validation. Vous perdez la vérification à la compilation pour les opérations sur le formulaire.

## Choisir votre approche {#choose-your-approach}

### Utilisez les Signal Forms si : {#use-signal-forms-if}

- Vous créez de nouvelles applications basées sur les signals (Angular v22+)
- Vous voulez une sûreté du typage inférée depuis la structure de votre model
- La validation par schéma vous attire
- Votre équipe connaît bien les signals

### Utilisez les reactive forms si : {#use-reactive-forms-if}

- Vous avez besoin d'une stabilité éprouvée en production
- Vous créez des formulaires complexes et dynamiques
- Vous préférez les patterns de type observable
- Vous avez besoin d'un contrôle fin sur l'état du formulaire
- Vous travaillez sur une base de code existante utilisant les reactive forms

### Utilisez les template-driven forms si : {#use-template-driven-forms-if}

- Vous créez des formulaires simples (connexion, contact, recherche)
- Vous faites du prototypage rapide
- La logique de votre formulaire est simple
- Vous préférez garder la logique de formulaire dans les templates
- Vous travaillez sur une base de code existante utilisant les template-driven forms

## Prochaines étapes {#next-steps}

Pour en savoir plus sur chaque approche :

- **Signal Forms** : consultez le [guide de vue d'ensemble](guide/forms/signals/overview) pour commencer, ou plongez dans les [models de formulaire](guide/forms/signals/models), la [validation](guide/forms/signals/validation) et la [gestion de l'état des champs](guide/forms/signals/field-state-management)
- **Reactive forms** : consultez le [guide des reactive forms](guide/forms/reactive-forms) dans la documentation Angular
- **Template-driven forms** : consultez le [guide des template-driven forms](guide/forms/template-driven-forms) dans la documentation Angular
