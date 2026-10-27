<!-- source: 8c12a9a5c29634d603e558875b54218c70200ba6 -->
# Guide de style Angular

## Introduction {#introduction}

Ce guide couvre un ensemble de conventions de style pour le code des applications Angular. Ces recommandations
ne sont pas nécessaires au fonctionnement d'Angular, mais établissent plutôt un ensemble de pratiques de code qui favorisent
la cohérence dans l'écosystème Angular. Un ensemble cohérent de pratiques facilite le partage
de code et le passage d'un projet à l'autre.

Ce guide ne couvre _pas_ TypeScript ni les pratiques de code générales sans lien avec Angular. Pour
TypeScript, consultez
le [guide de style TypeScript de Google](https://google.github.io/styleguide/tsguide.html).

### En cas de doute, privilégiez la cohérence {#when-in-doubt-prefer-consistency}

Chaque fois que vous rencontrez une situation où ces règles contredisent le style d'un fichier particulier,
donnez la priorité à la cohérence au sein du fichier. Mélanger différentes conventions de style dans un même
fichier crée plus de confusion que de s'écarter des recommandations de ce guide.

## Nommage {#naming}

### Séparez les mots des noms de fichiers par des tirets {#separate-words-in-file-names-with-hyphens}

Séparez les mots d'un nom de fichier par des tirets (`-`). Par exemple, un composant nommé `UserProfile`
a pour nom de fichier `user-profile.ts`.

### Utilisez le même nom pour les tests d'un fichier, avec `.spec` à la fin {#use-the-same-name-for-a-files-tests-with-spec-at-the-end}

Pour les tests unitaires, terminez les noms de fichiers par `.spec.ts`. Par exemple, le fichier de test unitaire
du composant `UserProfile` a pour nom de fichier `user-profile.spec.ts`.

### Faites correspondre les noms de fichiers à l'identifiant TypeScript qu'ils contiennent {#match-file-names-to-the-typescript-identifier-within}

Les noms de fichiers doivent généralement décrire le contenu du code du fichier. Quand le fichier contient une
classe TypeScript, le nom du fichier doit refléter le nom de cette classe. Par exemple, un fichier contenant un
composant nommé `UserProfile` a pour nom `user-profile.ts`.

Si le fichier contient plus d'un identifiant principal nommable, choisissez un nom qui décrit le
thème commun du code qu'il contient. Si le code d'un fichier ne relève pas d'un thème ou d'un domaine fonctionnel
commun, envisagez de le répartir dans différents fichiers. Évitez les noms de fichiers trop génériques
comme `helpers.ts`, `utils.ts` ou `common.ts`.

### Utilisez le même nom de fichier pour le TypeScript, le template et les styles d'un composant {#use-the-same-file-name-for-a-components-typescript-template-and-styles}

Les composants se composent généralement d'un fichier TypeScript, d'un fichier de template et d'un fichier de style. Ces
fichiers doivent partager le même nom avec des extensions différentes. Par exemple, un composant `UserProfile`
peut avoir les fichiers `user-profile.ts`, `user-profile.html` et `user-profile.css`.

Si un composant a plus d'un fichier de style, ajoutez au nom des mots supplémentaires qui décrivent les
styles propres à ce fichier. Par exemple, `UserProfile` pourrait avoir les fichiers de style
`user-profile-settings.css` et `user-profile-subscription.css`.

## Structure du projet {#project-structure}

### Tout le code de l'application se trouve dans un répertoire nommé `src` {#all-the-applications-code-goes-in-a-directory-named-src}

Tout votre code d'interface Angular (TypeScript, HTML et styles) doit se trouver dans un répertoire
nommé `src`. Le code sans lien avec l'interface, comme les fichiers de configuration ou les scripts, doit se trouver
en dehors du répertoire `src`.

Cela garde le répertoire racine de l'application cohérent d'un projet Angular à l'autre et crée
une séparation claire entre le code d'interface et le reste du code de votre projet.

### Lancez le bootstrap de votre application dans un fichier nommé `main.ts`, directement dans `src` {#bootstrap-your-application-in-a-file-named-maints-directly-inside-src}

Le code qui démarre une application Angular, autrement dit son **bootstrap**, doit toujours se trouver dans un fichier
nommé `main.ts`. Il représente le point d'entrée principal de l'application.

### Regroupez les fichiers étroitement liés dans le même répertoire {#group-closely-related-files-together-in-the-same-directory}

Les composants Angular se composent d'un fichier TypeScript et, éventuellement, d'un template et d'un ou plusieurs fichiers
de style. Regroupez-les dans le même répertoire.

Les tests unitaires doivent se trouver dans le même répertoire que le code testé. Évitez de rassembler des
tests sans rapport entre eux dans un unique répertoire `tests`.

### Organisez votre projet par domaine fonctionnel {#organize-your-project-by-feature-areas}

Organisez votre projet en sous-répertoires selon les fonctionnalités de votre application ou les thèmes communs
du code de ces répertoires. Par exemple, la structure du projet d'un site de cinéma,
MovieReel, pourrait ressembler à ceci :

```
src/
├─ movie-reel/
│ ├─ show-times/
│ │ ├─ film-calendar/
│ │ ├─ film-details/
│ ├─ reserve-tickets/
│ │ ├─ payment-info/
│ │ ├─ purchase-confirmation/
```

Évitez de créer des sous-répertoires selon le type de code qu'ils contiennent. Par
exemple, évitez de créer des répertoires comme `components`, `directives` et `services`.

Évitez de placer tant de fichiers dans un même répertoire qu'il devient difficile à lire ou à parcourir. À mesure que le
nombre de fichiers d'un répertoire augmente, envisagez de le découper davantage en sous-répertoires supplémentaires.

### Un concept par fichier {#one-concept-per-file}

Privilégiez des fichiers source centrés sur un seul _concept_. Pour les classes Angular en particulier, cela signifie généralement
un composant, une directive ou un service par fichier. Il est toutefois acceptable qu'un fichier contienne plus
d'un composant ou d'une directive si vos classes sont relativement petites et qu'elles forment ensemble un
seul et même concept.

En cas de doute, choisissez l'approche qui conduit à des fichiers plus petits.

## Injection de dépendances {#dependency-injection}

### Préférez la fonction `inject` à l'injection par paramètres du constructeur {#prefer-the-inject-function-over-constructor-parameter-injection}

Préférez la fonction [`inject`](/api/core/inject) à l'injection par paramètres du constructeur. La fonction [`inject`](/api/core/inject) fonctionne de la même façon que l'injection par paramètres du constructeur, mais offre plusieurs avantages de style :

- [`inject`](/api/core/inject) est généralement plus lisible, surtout quand une classe injecte de nombreuses dépendances.
- Il est syntaxiquement plus simple d'ajouter des commentaires aux dépendances injectées
- [`inject`](/api/core/inject) offre une meilleure inférence de types.
- Quand vous ciblez ES2022+ avec [`useDefineForClassFields`](https://www.typescriptlang.org/tsconfig/#useDefineForClassFields), vous évitez de séparer la déclaration et l'initialisation des champs lorsque ceux-ci lisent des dépendances injectées.

[Vous pouvez refactoriser le code existant vers `inject` avec un outil automatique](reference/migrations/inject-function).

## Composants et directives {#components-and-directives}

### Choisir les selectors des composants {#choosing-component-selectors}

Consultez
le [guide des composants pour savoir comment choisir les selectors des composants](guide/components/selectors#choosing-a-selector).

### Nommer les membres des composants et des directives {#naming-component-and-directive-members}

Consultez le guide des composants pour en savoir plus
sur le [nommage des propriétés d'input](guide/components/inputs#choosing-input-names)
et le [nommage des propriétés d'output](guide/components/outputs#choosing-event-names).

### Choisir les selectors des directives {#choosing-directive-selectors}

Les directives doivent utiliser le
même [préfixe propre à l'application](guide/components/selectors#selector-prefixes)
que vos composants.

Quand vous utilisez un attribute selector pour une directive, utilisez un nom d'attribut en camelCase. Par exemple, si
votre application s'appelle « MovieReel » et que vous créez une directive qui ajoute une infobulle à un élément,
vous pourriez utiliser le selector `[mrTooltip]`.

### Regroupez les propriétés propres à Angular avant les méthodes {#group-angular-specific-properties-before-methods}

Les composants et les directives doivent regrouper les propriétés propres à Angular, généralement vers le haut
de la déclaration de la classe. Cela inclut les dépendances injectées, les inputs, les outputs et les queries. Définissez
ces propriétés, ainsi que les autres, avant les méthodes de la classe.

Cette pratique permet de trouver plus facilement les API de template et les dépendances de la classe.

### Concentrez les composants et les directives sur la présentation {#keep-components-and-directives-focused-on-presentation}

Le code de vos composants et de vos directives doit généralement concerner l'interface affichée sur la page. Pour
le code qui a du sens par lui-même, découplé de l'interface, préférez le déplacer dans d'autres fichiers. Par
exemple, vous pouvez extraire les règles de validation de formulaire ou les transformations de données dans des fonctions ou
des classes séparées.

### Évitez une logique trop complexe dans les templates {#avoid-overly-complex-logic-in-templates}

Les templates Angular sont conçus pour
accueillir des [expressions proches de JavaScript](guide/templates/expression-syntax).
Tirez parti de ces expressions pour exprimer une logique relativement simple directement
dans les expressions de template.

Cependant, quand le code d'un template devient trop complexe, déplacez la logique dans le code TypeScript (généralement avec un [computed](guide/signals#computed-signals)).

Il n'existe pas de règle stricte et définitive qui détermine ce qui est « complexe ». Faites preuve de
discernement.

### Utilisez `protected` pour les membres de classe utilisés uniquement par le template d'un composant {#use-protected-on-class-members-that-are-only-used-by-a-components-template}

Les membres publics d'une classe de composant définissent intrinsèquement une API publique accessible via
l'injection de dépendances et les [queries](guide/components/queries). Préférez l'accès `protected`
pour tous les membres destinés à être lus depuis le template du composant.

```ts
@Component({
  ...,
  template: `<p>{{ fullName() }}</p>`,
})
export class UserProfile {
  firstName = input();
  lastName = input();

// `fullName` is not part of the component's public API, but is used in the template.
  protected fullName = computed(() => `${this.firstName()} ${this.lastName()}`);
}
```

### Utilisez `readonly` pour les propriétés qui ne doivent pas changer {#use-readonly-for-properties-that-shouldnt-change}

Marquez comme `readonly` les propriétés des composants et des directives initialisées par Angular. Cela inclut
les propriétés initialisées par `input`, `model`, `output` et les queries. Le modificateur d'accès readonly
garantit que la valeur définie par Angular n'est pas écrasée.

```ts
@Component(/* ... */)
export class UserProfile {
  readonly userId = input();
  readonly userSaved = output();
  readonly userName = model();
}
```

Pour les composants et les directives qui utilisent les API `@Input`, `@Output` et de query à base de décorateurs, ce
conseil s'applique aux propriétés d'output et aux queries, mais pas aux propriétés d'input.

```ts
@Component(/* ... */)
export class UserProfile {
  @Output() readonly userSaved = new EventEmitter<void>();
  @ViewChildren(PaymentMethod) readonly paymentMethods?: QueryList<PaymentMethod>;
}
```

### Préférez `class` et `style` à `ngClass` et `ngStyle` {#prefer-class-and-style-over-ngclass-and-ngstyle}

Préférez les bindings `class` et `style` aux directives [`NgClass`](/api/common/NgClass) et [`NgStyle`](/api/common/NgStyle).

```html {prefer}
<div [class.admin]="isAdmin" [class.dense]="density === 'high'">
  <div [style.color]="textColor" [style.background-color]="backgroundColor">
    <!-- OR -->
    <div [class]="{admin: isAdmin, dense: density === 'high'}">
      <div [style]="{'color': textColor, 'background-color': backgroundColor}"></div>
    </div>
  </div>
</div>
```

```html {avoid}
<div [ngClass]="{admin: isAdmin, dense: density === 'high'}">
  <div [ngStyle]="{'color': textColor, 'background-color': backgroundColor}"></div>
</div>
```

Les bindings `class` et `style` utilisent tous deux une syntaxe plus simple, très proche des
attributs HTML standard. Vos templates sont ainsi plus faciles à lire et à comprendre, en particulier pour
les développeurs qui connaissent les bases du HTML.

De plus, les directives `NgClass` et `NgStyle` entraînent un coût de performance supplémentaire par rapport
à la syntaxe de binding `class` et `style` intégrée.

Pour plus de détails, consultez le [guide des bindings](/guide/templates/binding#css-class-and-style-property-bindings)

### Nommez les event handlers d'après ce qu'ils _font_, pas d'après l'événement déclencheur {#name-event-handlers-for-what-they-do-not-for-the-triggering-event}

Préférez nommer les event handlers d'après l'action qu'ils effectuent plutôt que d'après l'événement déclencheur :

```html {prefer}
<button (click)="saveUserData()">Save</button>
```

```html {avoid}
<button (click)="handleClick()">Save</button>
```

Des noms parlants comme celui-ci permettent de comprendre plus facilement ce que fait un événement en lisant le
template.

Pour les événements clavier, vous pouvez utiliser les modificateurs d'événements clavier d'Angular avec des noms de handlers précis :

```html
<textarea (keydown.control.enter)="commitNotes()" (keydown.control.space)="showSuggestions()">
```

Parfois, la logique de gestion d'un événement est particulièrement longue ou complexe, ce qui rend peu pratique la déclaration d'un
seul handler bien nommé. Dans ces cas, vous pouvez vous rabattre sur un nom comme 'handleKeydown' puis
déléguer à des comportements plus précis selon les détails de l'événement :

```ts
@Component(/* ... */)
class RichText {
  handleKeydown(event: KeyboardEvent) {
    if (event.ctrlKey) {
      if (event.key === 'B') {
        this.activateBold();
      } else if (event.key === 'I') {
        this.activateItalic();
      }
      // ...
    }
  }
}
```

### Gardez les méthodes de cycle de vie simples {#keep-lifecycle-methods-simple}

Évitez de placer une logique longue ou complexe dans les lifecycle hooks comme `ngOnInit`. Préférez plutôt créer
des méthodes bien nommées pour contenir cette logique, puis _appeler ces méthodes_ dans vos lifecycle hooks.
Les noms des lifecycle hooks décrivent _quand_ ils s'exécutent, ce qui signifie que le code qu'ils contiennent n'a pas de
nom parlant qui décrit ce qu'il fait.

```ts {prefer}
ngOnInit() {
  this.startLogging();
  this.runBackgroundTask();
}
```

```ts {avoid}
ngOnInit() {
  this.logger.setMode('info');
  this.logger.monitorErrors();
  // ...and all the rest of the code that would be unrolled from these methods.
}
```

### Utilisez les interfaces des lifecycle hooks {#use-lifecycle-hook-interfaces}

Angular fournit une interface TypeScript pour chaque méthode de cycle de vie. Quand vous ajoutez un lifecycle hook à
votre classe, importez et implémentez (`implement`) ces interfaces pour garantir que les méthodes sont correctement nommées.

```ts
import {Component, OnInit} from '@angular/core';

@Component(/* ... */)
export class UserProfile implements OnInit {
  // The `OnInit` interface ensures this method is named correctly.
  ngOnInit() {
    /* ... */
  }
}
```
