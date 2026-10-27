<!-- source: 760d47033b6d701a6762bd096f1f6b6f29cc493a -->
# Vérification de types des templates

## Présentation de la vérification de types des templates {#overview-of-template-type-checking}

Tout comme TypeScript détecte les erreurs de type dans votre code, Angular vérifie les expressions et les bindings dans les templates de votre application et peut signaler toutes les erreurs de type qu'il trouve.
Par défaut, Angular vérifie les templates en mode strict.
Si nécessaire, vous pouvez le désactiver temporairement en définissant l'option `strictTemplates` à `false` dans les [options du compilateur Angular](reference/configs/angular-compiler-options), ce qui revient à un mode de base moins exhaustif.

### Mode de base {#basic-mode}

Dans le mode de vérification de types de base, avec l'option `strictTemplates` définie à `false`, Angular ne valide que les expressions de premier niveau d'un template.

Si vous écrivez `<map [city]="user.address.city">`, le compilateur vérifie les points suivants :

- `user` est une propriété de la classe du composant
- `user` est un objet doté d'une propriété address
- `user.address` est un objet doté d'une propriété city

Le compilateur ne vérifie pas que la valeur de `user.address.city` est assignable à l'input city du composant `<map>`.

Le compilateur présente aussi d'importantes limitations dans ce mode :

- Surtout, il ne vérifie pas les vues intégrées, comme `@if`, `@for`, `*ngIf`, `*ngFor` ou les autres vues intégrées `<ng-template>`.
- Il ne détermine pas les types des `#refs`, les résultats des pipes, ni le type de `$event` dans les event bindings.

Dans de nombreux cas, ces éléments finissent avec le type `any`, ce qui peut empêcher la vérification des parties suivantes de l'expression.

### Mode strict {#strict-mode}

Le mode strict est le mode par défaut ; il est activé lorsque l'option `strictTemplates` vaut `true` ou n'est pas définie.

En mode strict, Angular effectue les opérations suivantes :

- Vérifie les vues intégrées \(comme celles de `@if`, `@for`, `*ngIf` ou `*ngFor`\)
- Infère le type de retour correct des pipes
- Infère le type correct des références locales vers des directives et des pipes
- Vérifie que les bindings de composant ou de directive sont assignables à leurs `input()`
- Respecte l'option `strictNullChecks` de TypeScript lors de la validation des bindings d'inputs
- Infère le type correct des composants et des directives, y compris les génériques
- Infère les types de contexte des templates lorsqu'ils sont configurés \(par exemple, pour permettre une vérification de types correcte de `NgFor`\)
- Infère le type correct de `$event` dans les event bindings de composant ou de directive, du DOM et des animations
- Infère le type correct des références locales vers des éléments du DOM, en fonction du nom de la balise \(par exemple, le type que `document.createElement` renverrait pour cette balise\)

## Vérification de `*ngFor` {#checking-of-ngfor}

Les deux modes de vérification de types traitent les vues intégrées différemment.
Considérez l'exemple suivant.

```ts {header:"User interface"}
interface User {
  name: string;
  address: {
    city: string;
    state: string;
  };
}
```

```html
<div *ngFor="let user of users">
  <h2>{{config.title}}</h2>
  <span>City: {{user.address.city}}</span>
</div>
```

Les éléments `<h2>` et `<span>` se trouvent dans la vue intégrée de `*ngFor`.
En mode de base, Angular ne vérifie aucun des deux.
En mode strict, Angular sait que `user` dans le `<span>` est de type `User`, et que `address` est un objet doté d'une propriété `city` de type `string`.

## Résoudre les erreurs de template {#troubleshooting-template-errors}

En mode strict, vous pouvez rencontrer des erreurs de template qui n'apparaissaient pas en mode de base.
Ces erreurs correspondent souvent à de véritables incompatibilités de types dans les templates, que l'outillage précédent ne détectait pas.
Dans ce cas, le message d'erreur devrait indiquer clairement à quel endroit du template le problème se produit.

Il peut aussi y avoir des faux positifs lorsque les typings d'une library Angular sont incomplets ou incorrects, ou lorsque les typings ne correspondent pas tout à fait aux attentes, comme dans les cas suivants.

- Lorsque les typings d'une library sont faux ou incomplets \(par exemple, s'il manque `null | undefined` parce que la library n'a pas été écrite en tenant compte de `strictNullChecks`\)
- Lorsque les types des inputs d'une library sont trop restrictifs et que la library n'a pas ajouté les métadonnées appropriées pour qu'Angular le comprenne.
  Cela se produit généralement avec disabled ou d'autres inputs booléens courants utilisés comme attributs, par exemple `<input disabled>`.

- Lorsque vous utilisez `$event.target` pour les événements du DOM \(en raison de la propagation possible des événements, `$event.target` n'a pas, dans les typings du DOM, le type auquel vous pourriez vous attendre\)

En cas de faux positif de ce type, plusieurs options s'offrent à vous :

- Utilisez la fonction de conversion de type `$any()` dans certains contextes pour désactiver la vérification de types d'une partie de l'expression
- Désactivez entièrement les vérifications strictes en définissant `strictTemplates: false` dans le fichier de configuration TypeScript de l'application, `tsconfig.json`
- Désactivez individuellement certaines opérations de vérification de types, tout en conservant la rigueur sur les autres aspects, en définissant une _option de rigueur_ à `false`
- Si vous voulez utiliser `strictTemplates` et `strictNullChecks` ensemble, désactivez la vérification stricte des types null spécifiquement pour les bindings d'inputs avec `strictNullInputTypes`

Chacune des options suivantes prend la valeur de `strictTemplates` \(`true` lorsque `strictTemplates` vaut `true`, et inversement\) :

| Option de rigueur | Effet |
| :-------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `strictInputTypes` | Indique si l'assignabilité d'une expression de binding au champ `@Input()` est vérifiée. Affecte aussi l'inférence des types génériques des directives. |
| `strictNullInputTypes` | Indique si `strictNullChecks` est respecté lors de la vérification des bindings `@Input()` \(selon `strictInputTypes`\). Le désactiver peut être utile lorsque vous utilisez une library qui n'a pas été compilée en tenant compte de `strictNullChecks`. |
| `strictAttributeTypes` | Indique s'il faut vérifier les bindings `@Input()` réalisés avec des attributs texte. Par exemple, `<input matInput disabled="true">` \(qui définit la propriété `disabled` à la chaîne `'true'`\) par rapport à `<input matInput [disabled]="true">` \(qui définit la propriété `disabled` au booléen `true`\). |
| `strictSafeNavigationTypes` | Indique si le type de retour des opérations de navigation sécurisée est inféré \(par exemple, `user?.name` sera correctement inféré en fonction du type de `user`\). Si l'option est désactivée, `user?.name` sera de type `any`. |
| `strictDomLocalRefTypes` | Indique si les références locales vers des éléments du DOM auront le type correct. Si l'option est désactivée, `ref` sera de type `any` pour `<input #ref>`. |
| `strictOutputEventTypes` | Indique si `$event` aura le type correct pour les event bindings vers un `@Output()` de composant ou de directive, ou vers des événements d'animation. Si l'option est désactivée, il sera de type `any`. |
| `strictDomEventTypes` | Indique si `$event` aura le type correct pour les event bindings vers des événements du DOM. Si l'option est désactivée, il sera de type `any`. |
| `strictContextGenerics` | Indique si les paramètres de type des composants génériques seront inférés correctement \(y compris les contraintes génériques\). Si l'option est désactivée, tous les paramètres de type seront `any`. |
| `strictLiteralTypes` | Indique si le type des objets et tableaux littéraux déclarés dans le template sera inféré. Si l'option est désactivée, le type de ces littéraux sera `any`. |

Autres options (indépendantes de `strictTemplates`) :

| Option de rigueur | Effet |
| :--------------------------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `strictInputAccessModifiers` | Indique si les modificateurs d'accès comme `private`/`protected`/`readonly` sont respectés lors de l'assignation d'une expression de binding à un `@Input()` ou à un `input()`. Si l'option est désactivée, les modificateurs d'accès de l'input sont ignorés ; seul le type est vérifié. Remarque : cette vérification ne s'applique qu'aux inputs, pas aux outputs. |
| `strictUnclaimedEventNames` | Indique s'il faut signaler les event bindings dont le nom ne correspond ni à un output d'une directive appliquée à l'élément, ni à un événement natif connu du DOM (ceux décrits par la `GlobalEventHandlersEventMap` de TypeScript, comparés sans tenir compte de la casse), ce qui indique généralement un nom d'output mal orthographié. Seuls les noms en camelCase composés d'un seul identifiant sont vérifiés ; les noms séparés par des tirets \(par exemple, `my-event`\), utilisés par convention pour les événements personnalisés émis par les descendants, sont exemptés, de même que les éléments sans composant correspondant lorsque `CUSTOM_ELEMENTS_SCHEMA` est utilisé. Cette option vaut `false` par défaut, même avec `strictTemplates` défini à `true`. |

Si vous rencontrez encore des problèmes après avoir essayé ces options, le dernier recours consiste à revenir au mode de base en désactivant `strictTemplates`.

Une erreur de vérification de types que vous ne parvenez à résoudre avec aucune des méthodes recommandées peut provenir d'un bug dans le vérificateur de types des templates lui-même.
Si vous obtenez des erreurs qui vous obligent à revenir au mode de base, il s'agit probablement d'un tel bug.
Dans ce cas, [ouvrez une issue](https://github.com/angular/angular/issues) pour que l'équipe puisse le corriger.

## Inputs et vérification de types {#inputs-and-type-checking}

Le vérificateur de types des templates vérifie si le type d'une expression de binding est compatible avec celui de l'input correspondant de la directive.
Par exemple, considérez le composant suivant :

```angular-ts
export interface User {
  name: string;
}

@Component({
  selector: 'user-detail',
  template: '{{ user().name }}',
})
export class UserDetailComponent {
  user = input.required<User>();
}
```

Le template de `AppComponent` utilise ce composant comme suit :

```angular-ts
@Component({
  selector: 'app-root',
  template: '<user-detail [user]="selectedUser"></user-detail>',
})
export class AppComponent {
  selectedUser: User | null = null;
}
```

Ici, pendant la vérification de types du template de `AppComponent`, le binding `[user]="selectedUser"` correspond à l'input `UserDetailComponent.user`.
Angular assigne donc la propriété `selectedUser` à `UserDetailComponent.user`, ce qui produirait une erreur si leurs types étaient incompatibles.
TypeScript vérifie l'assignation selon son système de types, en respectant les options comme `strictNullChecks` telles qu'elles sont configurées dans l'application.

Évitez les erreurs de type à l'exécution en fournissant au vérificateur de types des templates des exigences de type plus précises dans le template.
Rendez les exigences de type des inputs de vos propres directives aussi précises que possible en fournissant des fonctions de template guard dans la définition de la directive.
Consultez [Améliorer la vérification de types des templates pour les directives personnalisées](/guide/directives/structural-directives#improving-template-type-checking-for-custom-directives) dans ce guide.

### Vérifications strictes des valeurs null {#strict-null-checks}

Lorsque vous activez `strictTemplates` et l'option TypeScript `strictNullChecks`, des erreurs de vérification de types peuvent survenir dans certaines situations difficiles à éviter.
Par exemple :

- Une valeur nullable liée à une directive provenant d'une library qui n'avait pas activé `strictNullChecks`.

  Pour une library compilée sans `strictNullChecks`, ses fichiers de déclaration n'indiquent pas si un champ peut être `null` ou non.
  Lorsque la library gère correctement `null`, c'est problématique, car le compilateur vérifie une valeur nullable par rapport à des fichiers de déclaration qui omettent le type `null`.
  Le compilateur produit donc une erreur de vérification de types, car il respecte `strictNullChecks`.

- L'utilisation du pipe `async` avec un observable dont vous savez qu'il émettra de façon synchrone.

  Le pipe `async` suppose actuellement que l'observable auquel il s'abonne peut être asynchrone, ce qui signifie qu'il est possible qu'aucune valeur ne soit encore disponible.
  Dans ce cas, il doit tout de même renvoyer quelque chose, à savoir `null`.
  Autrement dit, le type de retour du pipe `async` inclut `null`, ce qui peut entraîner des erreurs dans les situations où l'on sait que l'observable émet de façon synchrone une valeur non nullable.

Il existe deux contournements possibles aux problèmes précédents :

- Dans le template, ajoutez l'opérateur d'assertion non-null `!` à la fin d'une expression nullable, par exemple

```html
<user-detail [user]="user!"></user-detail>
```

Dans cet exemple, le compilateur ignore les incompatibilités de types liées à la nullabilité, comme dans du code TypeScript.
Dans le cas du pipe `async`, notez que l'expression doit être placée entre parenthèses, par exemple

```html
<user-detail [user]="(user$ | async)!"></user-detail>
```

- Désactivez complètement les vérifications strictes des valeurs null dans les templates Angular.

  Lorsque `strictTemplates` est activé, il reste possible de désactiver certains aspects de la vérification de types.
  Définir l'option `strictNullInputTypes` à `false` désactive les vérifications strictes des valeurs null dans les templates Angular.
  Cette option s'applique à tous les composants qui font partie de l'application.

### Conseils pour les auteurs de libraries {#advice-for-library-authors}

En tant qu'auteur de library, vous pouvez prendre plusieurs mesures pour offrir une expérience optimale à vos utilisateurs.
Tout d'abord, activer `strictNullChecks` et inclure `null` dans le type d'un input, lorsque c'est approprié, indique à vos utilisateurs s'ils peuvent fournir une valeur nullable ou non.
De plus, il est possible de fournir des indications de type propres au vérificateur de types des templates.
Consultez [Améliorer la vérification de types des templates pour les directives personnalisées](/guide/directives/structural-directives#improving-template-type-checking-for-custom-directives) et [Coercition dans le setter d'input](#input-setter-coercion).

## Coercition dans le setter d'input {#input-setter-coercion}

Il est parfois souhaitable que la propriété `input()` d'une directive ou d'un composant modifie la valeur qui lui est liée, généralement à l'aide d'une fonction `transform` pour l'input.
Par exemple, considérez ce composant de bouton personnalisé :

Considérez la directive suivante :

```angular-ts
@Component({
  selector: 'submit-button',
  template: `
    <div class="wrapper">
      <button [disabled]="disabled()">Submit</button>
    </div>
  `,
})
class SubmitButton {
  disabled = input.required({transform: booleanAttribute});
}
```

Ici, l'input `disabled` du composant est transmis au `<button>` dans le template.
Tout fonctionne comme prévu, tant qu'une valeur `boolean` est liée à l'input.
Mais supposons qu'un utilisateur utilise cet input dans le template comme un attribut :

```html
<submit-button disabled></submit-button>
```

Cela a le même effet que le binding suivant :

```html
<submit-button [disabled]="''"></submit-button>
```

À l'exécution, l'input sera défini à la chaîne vide, qui n'est pas une valeur `boolean`.
Les libraries de composants Angular qui traitent ce problème « convertissent » souvent la valeur dans le bon type dans le setter :

```ts

set disabled(value: boolean) {
  this._disabled = (value === '') || value;
}

```

L'idéal serait de changer ici le type de `value`, de `boolean` en `boolean|''`, pour correspondre à l'ensemble des valeurs réellement acceptées par le setter.
Avant la version 4.3, TypeScript exige que le getter et le setter aient le même type : si le getter doit renvoyer un `boolean`, le setter est donc contraint au type le plus restrictif.

Si l'utilisateur a activé la vérification de types la plus stricte d'Angular pour les templates, cela pose un problème : la chaîne vide \(`''`\) n'est en réalité pas assignable au champ `disabled`, ce qui produit une erreur de type lorsque la forme attribut est utilisée.

Pour contourner ce problème, Angular permet de vérifier un type plus large et plus permissif pour `@Input()` que celui déclaré pour le champ de l'input lui-même.
Activez ce comportement en ajoutant une propriété statique avec le préfixe `ngAcceptInputType_` à la classe du composant :

```ts
class SubmitButton {
  private _disabled: boolean;

  @Input()
  get disabled(): boolean {
    return this._disabled;
  }

  set disabled(value: boolean) {
    this._disabled = value === '' || value;
  }

  static ngAcceptInputType_disabled: boolean | '';
}
```

Depuis TypeScript 4.3, le setter peut être déclaré pour accepter le type `boolean|''`, ce qui rend obsolète le champ de coercition du setter d'input.
Les champs de coercition des setters d'input sont donc dépréciés.

Ce champ n'a pas besoin d'avoir de valeur.
Son existence indique au vérificateur de types d'Angular que l'input `disabled` doit être considéré comme acceptant les bindings correspondant au type `boolean|''`.
Le suffixe doit être le nom du _champ_ `@Input`.

Veillez à ce que, si une surcharge `ngAcceptInputType_` est présente pour un input donné, le setter puisse gérer toutes les valeurs du type surchargé.

## Désactiver la vérification de types avec `$any()` {#disabling-type-checking-using-any}

Désactivez la vérification d'une expression de binding en entourant l'expression d'un appel à la pseudo-fonction de conversion `$any()`.
Le compilateur la traite comme une conversion vers le type `any`, comme en TypeScript lorsqu'une conversion `<any>` ou `as any` est utilisée.

Dans l'exemple suivant, la conversion de `person` vers le type `any` supprime l'erreur `Property address does not exist`.

```angular-ts
@Component({
  selector: 'my-component',
  template: '{{$any(person).address.street}}',
})
class MyComponent {
  person?: Person;
}
```
