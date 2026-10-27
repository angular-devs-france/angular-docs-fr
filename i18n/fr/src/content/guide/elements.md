<!-- source: caf6395109874df083273e562e1b5789456584a1 -->
# Présentation d'Angular Elements

Les _Angular Elements_ sont des composants Angular packagés sous forme de _custom elements_ \(aussi appelés Web Components\), un standard du web qui permet de définir de nouveaux éléments HTML indépendamment de tout framework.

Les [custom elements](https://developer.mozilla.org/docs/Web/Web_Components/Using_custom_elements) sont une fonctionnalité de la plateforme web disponible dans tous les navigateurs pris en charge par Angular.
Un custom element étend le HTML en vous permettant de définir une balise dont le contenu est créé et contrôlé par du code JavaScript.
Le navigateur maintient un `CustomElementRegistry` des custom elements définis, qui associe une classe JavaScript instanciable à une balise HTML.

Le package `@angular/elements` exporte une API `createCustomElement()` qui fait le pont entre l'interface des composants Angular et sa détection de changements, d'une part, et l'API DOM native, d'autre part.

Transformer un composant en custom element met toute l'infrastructure Angular nécessaire à la disposition du navigateur.
Créer un custom element est simple et direct, et connecte automatiquement la vue définie par votre composant à la détection de changements et au data binding, en associant les fonctionnalités d'Angular à leurs équivalents HTML natifs.

## Utiliser des custom elements {#using-custom-elements}

Les custom elements assurent eux-mêmes leur bootstrap : ils démarrent quand ils sont ajoutés au DOM et sont détruits quand ils en sont retirés.
Une fois qu'un custom element est ajouté au DOM d'une page, il ressemble à n'importe quel autre élément HTML et se comporte comme lui, sans nécessiter de connaissance particulière des termes ou des conventions d'usage d'Angular.

Pour ajouter le package `@angular/elements` à votre workspace, exécutez la commande suivante :

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install @angular/elements
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add @angular/elements
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add @angular/elements
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add @angular/elements
  </docs-code>
</docs-code-multifile>

### Fonctionnement {#how-it-works}

La fonction `createCustomElement()` convertit un composant en une classe qui peut être enregistrée auprès du navigateur comme custom element.
Après avoir enregistré votre classe configurée dans le registre des custom elements du navigateur, utilisez le nouvel élément comme un élément HTML natif dans le contenu que vous ajoutez directement au DOM :

```html
<my-popup message="Use Angular!" />
```

Quand votre custom element est placé sur une page, le navigateur crée une instance de la classe enregistrée et l'ajoute au DOM.
Le contenu est fourni par le template du composant, qui utilise la syntaxe de template d'Angular, et il est rendu à partir des données du composant et du DOM.
Les propriétés d'input du composant correspondent aux attributs d'entrée de l'élément.

## Transformer des composants en custom elements {#transforming-components-to-custom-elements}

Angular fournit la fonction `createCustomElement()` pour convertir un composant Angular, avec ses dépendances, en custom element.

Le processus de conversion implémente l'interface `NgElementConstructor` et crée une
classe constructeur configurée pour produire une instance de votre composant qui assure son propre bootstrap.

Utilisez la fonction native du navigateur [`customElements.define()`](https://developer.mozilla.org/docs/Web/API/CustomElementRegistry/define) pour enregistrer le constructeur configuré et la balise de custom element associée dans le [`CustomElementRegistry`](https://developer.mozilla.org/docs/Web/API/CustomElementRegistry) du navigateur.
Quand le navigateur rencontre la balise de l'élément enregistré, il utilise le constructeur pour créer une instance du custom element.

IMPORTANT: Évitez d'utiliser le selector du composant comme nom de balise du custom element.
Cela peut entraîner un comportement inattendu, car Angular crée alors deux instances du composant pour un seul élément du DOM :
un composant Angular classique et un second qui utilise le custom element.

### Correspondance {#mapping}

Un custom element _héberge_ un composant Angular et fait le pont entre les données et la logique définies dans le composant et les API DOM standard.
Les propriétés et la logique du composant sont directement associées aux attributs HTML et au système d'événements du navigateur.

- L'API de création analyse le composant à la recherche de propriétés d'input et définit les attributs correspondants pour le custom element.
  Elle transforme les noms de propriétés pour les rendre compatibles avec les custom elements, qui ne distinguent pas la casse.
  Les noms d'attributs obtenus sont en minuscules, séparés par des tirets.
  Par exemple, pour un composant avec `inputProp = input('', {alias: 'myInputProp'})`, le custom element correspondant définit un attribut `my-input-prop`.

- Les outputs du composant sont émis sous forme d'[événements personnalisés (Custom Events)](https://developer.mozilla.org/docs/Web/API/CustomEvent) HTML, dont le nom correspond au nom de l'output.
  Par exemple, pour un composant avec `valueChanged = output()`, le custom element correspondant émet des événements nommés « valueChanged », et les données émises sont stockées dans la propriété `detail` de l'événement.
  Si vous fournissez un alias, c'est cette valeur qui est utilisée ; par exemple, `clicks = output<string>({alias: 'myClick'});` émet des événements nommés « myClick ».

Pour plus d'informations, consultez la documentation des Web Components sur la [création d'événements personnalisés](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Events#creating_custom_events).

## Exemple : un service de popup {#example-a-popup-service}

Pour ajouter un composant à une application à l'exécution, vous pouvez le [rendre par programmation](guide/components/programmatic-rendering) avec l'API `createComponent`.
Avec cette approche, l'infrastructure qui l'entoure est à votre charge : attacher la host view du composant à l'`ApplicationRef` pour que la détection de changements s'exécute, définir ses inputs, s'abonner à ses outputs, puis détacher et nettoyer la vue quand le composant est retiré.

Utiliser un custom element Angular rend le processus plus simple et plus transparent, en fournissant automatiquement toute cette infrastructure : il vous suffit de définir le type de gestion d'événements souhaité.

L'application d'exemple Popup Service suivante définit un composant que vous pouvez soit charger dynamiquement, soit convertir en custom element.

| Fichiers           | Détails                                                                                                                                                                                                             |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `popup.ts`         | Définit un élément popup simple qui affiche un message reçu en input, avec un peu d'animation et de style.                                                                                                          |
| `popup.service.ts` | Crée un service injectable qui fournit deux façons différentes d'invoquer `Popup` : comme composant dynamique ou comme custom element. Remarquez combien la méthode de chargement dynamique demande plus de configuration. |
| `app.ts`           | Définit le composant racine de l'application, qui utilise `PopupService` pour ajouter la popup au DOM à l'exécution. Quand l'application s'exécute, le constructeur du composant racine convertit `Popup` en custom element. |

À titre de comparaison, la démo montre les deux méthodes.
Un bouton ajoute la popup avec la méthode de chargement dynamique, et l'autre utilise le custom element.
Le résultat est le même, mais la préparation diffère.

<docs-code-multifile>
    <docs-code language="angular-ts" header="popup.ts" path="adev/src/content/examples/elements/src/app/popup.ts"/>
    <docs-code header="popup.service.ts" path="adev/src/content/examples/elements/src/app/popup.service.ts"/>
    <docs-code header="app.ts" path="adev/src/content/examples/elements/src/app/app.ts"/>
</docs-code-multifile>

## Typage des custom elements {#typings-for-custom-elements}

Les API DOM génériques, comme `document.createElement()` ou `document.querySelector()`, renvoient un type d'élément adapté aux arguments fournis.
Par exemple, appeler `document.createElement('a')` renvoie un `HTMLAnchorElement`, dont TypeScript sait qu'il possède une propriété `href`.
De même, `document.createElement('div')` renvoie un `HTMLDivElement`, dont TypeScript sait qu'il ne possède pas de propriété `href`.

Appelées avec des éléments inconnus, comme un nom de custom element \(`popup-element` dans notre exemple\), ces méthodes renvoient un type générique, comme `HTMLElement`, car TypeScript ne peut pas déduire le type exact de l'élément renvoyé.

Les custom elements créés avec Angular étendent `NgElement` \(qui étend lui-même `HTMLElement`\).
De plus, ces custom elements possèdent une propriété pour chaque input du composant correspondant.
Par exemple, notre `popup-element` possède une propriété `message` de type `string`.

Plusieurs options s'offrent à vous pour obtenir des types corrects pour vos custom elements.
Supposons que vous créiez un custom element `my-dialog` à partir du composant suivant :

```ts
@Component(/* ... */)
class MyDialog {
  content = input('');
}
```

La façon la plus directe d'obtenir un typage exact est de caster la valeur renvoyée par les méthodes DOM concernées vers le bon type.
Pour cela, utilisez les types `NgElement` et `WithProperties` \(tous deux exportés depuis `@angular/elements`\) :

```ts
const aDialog = document.createElement('my-dialog') as NgElement &
  WithProperties<{content: string}>;
aDialog.content = 'Hello, world!';
aDialog.content = 123; // <-- ERROR: TypeScript knows this should be a string.
aDialog.body = 'News'; // <-- ERROR: TypeScript knows there is no `body` property on `aDialog`.
```

C'est un bon moyen de bénéficier rapidement des fonctionnalités de TypeScript, comme la vérification de types et l'autocomplétion, pour votre custom element.
Mais cela peut devenir fastidieux si vous en avez besoin à plusieurs endroits, car vous devez caster le type de retour à chaque occurrence.

Une autre façon, qui ne demande de définir le type de chaque custom element qu'une seule fois, consiste à augmenter `HTMLElementTagNameMap`, que TypeScript utilise pour déduire le type d'un élément renvoyé à partir de son nom de balise \(pour les méthodes DOM comme `document.createElement()`, `document.querySelector()`, etc.\) :

```ts

declare global {
  interface HTMLElementTagNameMap {
    'my-dialog': NgElement & WithProperties<{content: string}>;
    'my-other-element': NgElement & WithProperties<{foo: 'bar'}>;
    …
  }
}

```

Désormais, TypeScript peut déduire le type correct de la même manière que pour les éléments natifs :

```ts
document.createElement('div'); //--> HTMLDivElement (built-in element)
document.querySelector('foo'); //--> Element        (unknown element)
document.createElement('my-dialog'); //--> NgElement & WithProperties<{content: string}> (custom element)
document.querySelector('my-other-element'); //--> NgElement & WithProperties<{foo: 'bar'}>      (custom element)
```

## Limitations {#limitations}

Faites attention lorsque vous détruisez puis rattachez des custom elements créés avec `@angular/elements`, en raison de problèmes liés au callback [disconnect()](https://github.com/angular/angular/issues/38778). Vous pouvez rencontrer ce problème dans les cas suivants :

- Rendu d'un composant dans un `ng-if` ou un `ng-repeat` dans `AngularJS`
- Détachement et rattachement manuels d'un élément au DOM
