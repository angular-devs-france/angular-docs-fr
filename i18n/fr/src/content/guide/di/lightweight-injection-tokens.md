<!-- source: 314580ed5e11fe288738215cf5b7bdb083e2d888 -->
# Optimiser la taille de l'application cliente avec des jetons d'injection légers

Cette page présente une vue d'ensemble conceptuelle d'une technique d'injection de dépendances recommandée pour les développeurs de libraries.
Concevoir votre library avec des _jetons d'injection légers_ aide à optimiser la taille du bundle des applications clientes qui utilisent votre library.

Vous pouvez gérer la structure des dépendances entre vos composants et vos services injectables pour optimiser la taille du bundle en utilisant des providers compatibles avec le tree-shaking.
Normalement, cela garantit que si un composant ou un service fourni n'est jamais réellement utilisé par l'application, le compilateur peut retirer son code du bundle.

En raison de la façon dont Angular stocke les jetons d'injection, il est possible qu'un tel composant ou service inutilisé se retrouve quand même dans le bundle.
Cette page décrit un design pattern d'injection de dépendances qui permet un tree-shaking correct grâce aux jetons d'injection légers.

Le design pattern des jetons d'injection légers est particulièrement important pour les développeurs de libraries.
Il garantit que, quand une application n'utilise qu'une partie des fonctionnalités de votre library, le code inutilisé peut être éliminé du bundle de l'application cliente.

Quand une application utilise votre library, il se peut que votre library fournisse certains services que l'application cliente n'utilise pas.
Dans ce cas, le développeur de l'application s'attend à ce que ce service soit éliminé par le tree-shaking et ne contribue pas à la taille de l'application compilée.
Comme le développeur de l'application ne peut ni connaître ni corriger un problème de tree-shaking dans la library, c'est au développeur de la library de s'en charger.
Pour éviter la rétention de composants inutilisés, votre library doit utiliser le design pattern des jetons d'injection légers.

## Quand les jetons sont conservés {#when-tokens-are-retained}

Pour mieux expliquer dans quelles conditions un jeton est conservé, prenons une library qui fournit un composant de carte.
Ce composant contient un corps et peut contenir un en-tête facultatif :

```html
<lib-card>
  <lib-header>…</lib-header>
</lib-card>
```

Dans une implémentation probable, le composant `<lib-card>` utilise `contentChild` ou `contentChildren` pour obtenir `<lib-header>` et `<lib-body>`, comme ceci :

```ts {highlight: [14]}
import {Component, contentChild} from '@angular/core';

@Component({
  selector: 'lib-header',
  …,
})
class LibHeader {}

@Component({
  selector: 'lib-card',
  …,
})
class LibCard {
  readonly header = contentChild(LibHeader);
}
```

Comme `<lib-header>` est facultatif, l'élément peut apparaître dans le template sous sa forme minimale, `<lib-card />`.
Dans ce cas, `<lib-header>` n'est pas utilisé et vous vous attendriez à ce qu'il soit éliminé par le tree-shaking, mais ce n'est pas ce qui se passe.
En effet, `LibCard` contient en réalité deux références à `LibHeader` :

```ts
readonly header = contentChild(LibHeader);
```

- L'une de ces références est en _position de type_, c'est-à-dire qu'elle spécifie `LibHeader` comme type : `readonly header: Signal<LibHeader|undefined>`.
- L'autre référence est en _position de valeur_, c'est-à-dire que `LibHeader` est la valeur passée à la fonction `contentChild` : `contentChild(LibHeader)`.

Le compilateur traite différemment les références de jetons selon leur position :

- Le compilateur efface les références en _position de type_ après la conversion depuis TypeScript : elles n'ont donc aucun impact sur le tree-shaking.
- Le compilateur doit conserver les références en _position de valeur_ à l'exécution, ce qui **empêche** le composant d'être éliminé par le tree-shaking.

Dans l'exemple, le compilateur conserve le jeton `LibHeader` qui apparaît en position de valeur.
Cela empêche le composant référencé d'être éliminé par le tree-shaking, même si l'application n'utilise en réalité `<lib-header>` nulle part.
Si le code, le template et les styles de `LibHeader` deviennent trop volumineux, l'inclure inutilement peut augmenter considérablement la taille de l'application cliente.

## Quand utiliser le pattern des jetons d'injection légers {#when-to-use-the-lightweight-injection-token-pattern}

Le problème de tree-shaking survient quand un composant est utilisé comme jeton d'injection.
Cela peut arriver dans deux cas :

- Le jeton est utilisé en position de valeur d'une [content query](guide/components/queries#content-queries).
- Le jeton est utilisé avec la fonction `inject`.

Dans l'exemple suivant, les deux utilisations du jeton `CustomOther` entraînent la conservation de `CustomOther`, ce qui l'empêche d'être éliminé par le tree-shaking quand il n'est pas utilisé :

```ts {highlight: [[2],[4]]}
class App {
  private readonly other = inject(CustomOther, {optional: true});

  readonly header = contentChild(CustomOther);
}
```

Si les jetons utilisés uniquement comme spécificateurs de type sont supprimés lors de la conversion en JavaScript, tous les jetons utilisés pour l'injection de dépendances sont nécessaires à l'exécution.
Quand vous utilisez `inject(CustomOther)`, `CustomOther` est passé en tant qu'argument valeur.
Le jeton est alors en position de valeur, ce qui amène le tree-shaker à conserver la référence.

HELPFUL: Les libraries doivent utiliser des [providers compatibles avec le tree-shaking](guide/di/defining-dependency-providers) pour tous leurs services, en fournissant les dépendances au niveau racine plutôt que dans des composants ou des modules.

## Utiliser des jetons d'injection légers {#using-lightweight-injection-tokens}

Le design pattern des jetons d'injection légers consiste à utiliser une petite classe abstraite comme jeton d'injection, puis à fournir l'implémentation réelle plus tard.
La classe abstraite est conservée, elle n'est pas éliminée par le tree-shaking, mais elle est petite et n'a pas d'impact notable sur la taille de l'application.

L'exemple suivant montre comment cela fonctionne pour `LibHeader` :

```ts {highlight: [[1],[5], [15]]}
abstract class LibHeaderToken {}

@Component({
  selector: 'lib-header',
  providers: [{provide: LibHeaderToken, useExisting: LibHeader}],
  …,
})
class LibHeader extends LibHeaderToken {}

@Component({
  selector: 'lib-card',
  …,
})
class LibCard {
  readonly header = contentChild(LibHeaderToken);
}
```

Dans cet exemple, l'implémentation de `LibCard` ne fait plus référence à `LibHeader`, ni en position de type, ni en position de valeur.
Cela permet à `LibHeader` d'être entièrement éliminé par le tree-shaking.
`LibHeaderToken` est conservé, mais ce n'est qu'une déclaration de classe, sans implémentation concrète.
Il est petit et n'a pas d'impact notable sur la taille de l'application quand il est conservé après la compilation.

C'est `LibHeader` lui-même qui implémente la classe abstraite `LibHeaderToken`.
Vous pouvez utiliser ce jeton en toute sécurité comme provider dans la définition du composant, ce qui permet à Angular d'injecter correctement le type concret.

En résumé, le pattern des jetons d'injection légers se compose des éléments suivants :

1. Un jeton d'injection léger représenté par une classe abstraite.
2. Une définition de composant qui implémente la classe abstraite.
3. L'injection du pattern léger, avec `contentChild` ou `contentChildren`.
4. Un provider dans l'implémentation du jeton d'injection léger, qui associe le jeton d'injection léger à l'implémentation.

### Utiliser le jeton d'injection léger pour définir une API {#use-the-lightweight-injection-token-for-api-definition}

Un composant qui injecte un jeton d'injection léger peut avoir besoin d'appeler une méthode de la classe injectée.
Le jeton est désormais une classe abstraite. Comme le composant injectable implémente cette classe, vous devez aussi déclarer une méthode abstraite dans la classe abstraite du jeton d'injection léger.
L'implémentation de la méthode, avec tout le code qu'elle ajoute, réside dans le composant injectable, qui peut être éliminé par le tree-shaking.
Cela permet au parent de communiquer avec l'enfant, s'il est présent, de manière typée.

Par exemple, `LibCard` interroge désormais `LibHeaderToken` plutôt que `LibHeader`.
L'exemple suivant montre comment le pattern permet à `LibCard` de communiquer avec `LibHeader` sans jamais faire référence à `LibHeader` :

```ts {highlight: [[2],[7],[11],[19]]}
abstract class LibHeaderToken {
  abstract doSomething(): void;
}

@Component({
  selector: 'lib-header',
  providers: [{provide: LibHeaderToken, useExisting: LibHeader}],
})
class LibHeader extends LibHeaderToken {
  doSomething(): void {
    // Concrete implementation of `doSomething`
  }
}

@Component({
  selector: 'lib-card',
})
class LibCard implements AfterContentInit {
  readonly header = contentChild(LibHeaderToken);

  ngAfterContentInit(): void {
    if (this.header() !== undefined) {
      this.header()!.doSomething();
    }
  }
}
```

Dans cet exemple, le parent interroge le jeton pour obtenir le composant enfant et stocke la référence du composant obtenu s'il est présent.
Avant d'appeler une méthode de l'enfant, le composant parent vérifie que le composant enfant est présent.
Si le composant enfant a été éliminé par le tree-shaking, il n'existe aucune référence à celui-ci à l'exécution, ni aucun appel à sa méthode.

### Nommer votre jeton d'injection léger {#naming-your-lightweight-injection-token}

Les jetons d'injection légers ne sont utiles qu'avec des composants.
Le [guide de style Angular](style-guide) recommande de nommer les composants sans le suffixe `Component`.
L'exemple `LibHeader` suit cette convention.

Vous devez conserver le lien entre le composant et son jeton tout en les distinguant.
Le style recommandé consiste à utiliser le nom de base du composant suivi du suffixe `Token` pour nommer vos jetons d'injection légers : `LibHeaderToken`.
