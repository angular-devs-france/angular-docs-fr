<!-- source: cb2b96a9dbb76d168ae639f0d8acfaaf752f2fa1 -->
# Compilation ahead-of-time (AOT)

Une application Angular se compose principalement de composants et de leurs templates HTML.
Comme le navigateur ne peut pas comprendre directement les composants et les templates fournis par Angular, les applications Angular nécessitent un processus de compilation avant de pouvoir s'exécuter dans un navigateur.

Le compilateur ahead-of-time (AOT) d'Angular convertit votre code HTML et TypeScript Angular en code JavaScript efficace pendant la phase de build, _avant_ que le navigateur ne télécharge et n'exécute ce code.
Compiler votre application pendant le processus de build permet un rendu plus rapide dans le navigateur.

Ce guide explique comment spécifier les métadonnées et appliquer les options de compilation disponibles pour compiler efficacement vos applications avec le compilateur AOT.

HELPFUL: [Regardez Alex Rickabaugh expliquer le compilateur Angular](https://www.youtube.com/watch?v=anphffaCZrQ) à AngularConnect 2019.

Voici quelques raisons pour lesquelles vous pourriez vouloir utiliser AOT.

| Raisons | Détails |
| :-------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Rendu plus rapide | Avec AOT, le navigateur télécharge une version précompilée de l'application. Le navigateur charge du code exécutable et peut donc afficher l'application immédiatement, sans attendre de devoir la compiler au préalable. |
| Moins de requêtes asynchrones | Le compilateur _intègre_ les templates HTML et les feuilles de style CSS externes dans le JavaScript de l'application, ce qui élimine les requêtes ajax séparées pour ces fichiers sources. |
| Téléchargement du framework Angular plus léger | Il n'est pas nécessaire de télécharger le compilateur Angular si l'application est déjà compilée. Le compilateur représente environ la moitié d'Angular lui-même : l'omettre réduit donc considérablement le poids de l'application. |
| Détection plus précoce des erreurs de template | Le compilateur AOT détecte et signale les erreurs de binding dans les templates pendant l'étape de build, avant que les utilisateurs ne puissent les voir. |
| Meilleure sécurité | AOT compile les templates HTML et les composants en fichiers JavaScript bien avant qu'ils ne soient servis au client. Sans template à lire ni évaluation risquée de HTML ou de JavaScript côté client, les occasions d'attaques par injection sont moins nombreuses. |

## Choisir un compilateur {#choosing-a-compiler}

Angular propose deux façons de compiler votre application :

| Compilation Angular | Détails |
| :-------------------- | :------------------------------------------------------------------------------------------------ |
| Just-in-Time \(JIT\) | Compile votre application dans le navigateur à l'exécution. C'était le comportement par défaut jusqu'à Angular 8. |
| Ahead-of-Time \(AOT\) | Compile votre application et vos libraries au moment du build. C'est le comportement par défaut depuis Angular 9. |

Lorsque vous lancez les commandes CLI [`ng build`](cli/build) \(build uniquement\) ou [`ng serve`](cli/serve) \(build et service en local\), le type de compilation \(JIT ou AOT\) dépend de la valeur de la propriété `aot` dans votre configuration de build spécifiée dans `angular.json`.
Par défaut, `aot` vaut `true` pour les nouvelles applications CLI.

Consultez la [référence des commandes CLI](cli) et [Compiler et servir des applications Angular](tools/cli/build) pour plus d'informations.

## Fonctionnement d'AOT {#how-aot-works}

Le compilateur AOT d'Angular extrait des **métadonnées** pour interpréter les parties de l'application qu'Angular est censé gérer.
Vous pouvez spécifier les métadonnées explicitement dans des **décorateurs** comme `@Component()`, ou implicitement dans les déclarations de constructeur des classes décorées.
Les métadonnées indiquent à Angular comment construire les instances des classes de votre application et interagir avec elles à l'exécution.

Dans l'exemple suivant, l'objet de métadonnées de `@Component()` et le constructeur de la classe indiquent à Angular comment créer et afficher une instance de `Typical`.

```angular-ts
@Component({
  selector: 'app-typical',
  template: '<div>A typical component for {{data().name}}</div>',
})
export class Typical {
  data = input.required<TypicalData>();
  private someService = inject(SomeService);
}
```

Le compilateur Angular extrait les métadonnées _une seule fois_ et génère une _fabrique_ pour `Typical`.
Lorsqu'il doit créer une instance de `Typical`, Angular appelle la fabrique, qui produit un nouvel élément visuel, lié à une nouvelle instance de la classe du composant avec sa dépendance injectée.

### Phases de compilation {#compilation-phases}

La compilation AOT comporte trois phases.

| | Phase | Détails |
| :-- | :--------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1 | analyse du code | Dans cette phase, le compilateur TypeScript et le _collecteur AOT_ créent une représentation du code source. Le collecteur ne tente pas d'interpréter les métadonnées qu'il collecte. Il représente les métadonnées du mieux qu'il peut et enregistre des erreurs lorsqu'il détecte une violation de la syntaxe des métadonnées. |
| 2 | génération du code | Dans cette phase, le `StaticReflector` du compilateur interprète les métadonnées collectées en phase 1, effectue une validation supplémentaire des métadonnées et lève une erreur s'il détecte une violation des restrictions sur les métadonnées. |
| 3 | vérification de types des templates | Dans cette phase, le _compilateur de templates_ d'Angular utilise le compilateur TypeScript pour valider les expressions de binding dans les templates. La vérification stricte dans cette phase est activée par défaut et contrôlée par l'option de configuration `strictTemplates` ; consultez [Options du compilateur Angular](reference/configs/angular-compiler-options). |

### Restrictions sur les métadonnées {#metadata-restrictions}

Vous écrivez les métadonnées dans un _sous-ensemble_ de TypeScript qui doit respecter les contraintes générales suivantes :

- Limiter la [syntaxe des expressions](#expression-syntax-limitations) au sous-ensemble de JavaScript pris en charge
- Ne référencer que des symboles exportés après le [code folding](#code-folding)
- N'appeler que des [fonctions prises en charge](#supported-classes-and-functions) par le compilateur
- Les inputs/outputs et les membres de classe liés aux données doivent être public ou protected. Pour d'autres conseils et instructions sur la préparation d'une application à la compilation AOT, consultez [Angular : écrire des applications compatibles AOT](https://medium.com/sparkles-blog/angular-writing-aot-friendly-applications-7b64c8afbe3f).

HELPFUL: Les erreurs de compilation AOT surviennent généralement à cause de métadonnées qui ne respectent pas les exigences du compilateur \(décrites plus en détail ci-dessous\).
Pour vous aider à comprendre et à résoudre ces problèmes, consultez [Erreurs de métadonnées AOT](tools/cli/aot-metadata-errors).

### Configurer la compilation AOT {#configuring-aot-compilation}

Vous pouvez fournir des options dans le [fichier de configuration TypeScript](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html) qui contrôle le processus de compilation.
Consultez [Options du compilateur Angular](reference/configs/angular-compiler-options) pour la liste complète des options disponibles.

## Phase 1 : analyse du code {#phase-1-code-analysis}

Le compilateur TypeScript effectue une partie du travail d'analyse de la première phase.
Il émet les _fichiers de définition de types_ `.d.ts` avec les informations de type dont le compilateur AOT a besoin pour générer le code de l'application.
En parallèle, le **collecteur** AOT analyse les métadonnées enregistrées dans les décorateurs Angular et produit les informations de métadonnées dans des fichiers **`.metadata.json`**, un par fichier `.d.ts`.

Vous pouvez considérer `.metadata.json` comme un diagramme de la structure globale des métadonnées d'un décorateur, représenté sous forme d'[arbre syntaxique abstrait (AST)](https://en.wikipedia.org/wiki/Abstract_syntax_tree).

HELPFUL: Le fichier [schema.ts](https://github.com/angular/angular/blob/12.2.x/packages/compiler-cli/src/metadata/schema.ts) d'Angular décrit le format JSON sous forme d'une collection d'interfaces TypeScript.

### Limitations de la syntaxe des expressions {#expression-syntax-limitations}

Le collecteur AOT ne comprend qu'un sous-ensemble de JavaScript.
Définissez les objets de métadonnées avec la syntaxe limitée suivante :

| Syntaxe | Exemple |
| :------------------------ | :--------------------------------------------------------- |
| Objet littéral | `{cherry: true, apple: true, mincemeat: false}` |
| Tableau littéral | `['cherries', 'flour', 'sugar']` |
| Spread dans un tableau littéral | `['apples', 'flour', …]` |
| Appels | `bake(ingredients)` |
| New | `new Oven()` |
| Accès à une propriété | `pie.slice` |
| Index de tableau | `ingredients[0]` |
| Référence à un identifiant | `Component` |
| Une template string | <code>`pie is ${multiplier} times better than cake`</code> |
| Chaîne littérale | `'pi'` |
| Nombre littéral | `3.14153265` |
| Booléen littéral | `true` |
| Null littéral | `null` |
| Opérateur préfixe pris en charge | `!cake` |
| Opérateur binaire pris en charge | `a+b` |
| Opérateur conditionnel | `a ? b : c` |
| Parenthèses | `(a+b)` |

Si une expression utilise une syntaxe non prise en charge, le collecteur écrit un nœud d'erreur dans le fichier `.metadata.json`.
Le compilateur signale ensuite l'erreur s'il a besoin de ce morceau de métadonnées pour générer le code de l'application.

HELPFUL: Si vous voulez que `ngc` signale immédiatement les erreurs de syntaxe au lieu de produire un fichier `.metadata.json` contenant des erreurs, définissez l'option `strictMetadataEmit` dans le fichier de configuration TypeScript.

```json

"angularCompilerOptions": {
  …
  "strictMetadataEmit" : true
}

```

Les libraries Angular utilisent cette option pour garantir que tous les fichiers `.metadata.json` d'Angular sont propres, et c'est une bonne pratique de faire de même lorsque vous compilez vos propres libraries.

### Code folding {#code-folding}

Le compilateur ne peut résoudre que les références à des symboles **_exportés_**.
Le collecteur, en revanche, peut évaluer une expression pendant la collecte et enregistrer le résultat dans le `.metadata.json`, plutôt que l'expression d'origine.
Cela vous permet d'utiliser de façon limitée des symboles non exportés dans les expressions.

Par exemple, le collecteur peut évaluer l'expression `1 + 2 + 3 + 4` et la remplacer par le résultat, `10`.
Ce processus s'appelle le _folding_.
Une expression qui peut être réduite de cette manière est dite _foldable_.

Le collecteur peut évaluer les références aux déclarations `const` locales au module et aux déclarations `var` et `let` initialisées, ce qui les retire de fait du fichier `.metadata.json`.

Considérez la définition de composant suivante :

```angular-ts
const template = '<div>{{hero().name}}</div>';

@Component({
  selector: 'app-hero',
  template: template,
})
export class Hero {
  hero = input.required<Hero>();
}
```

Le compilateur ne pourrait pas faire référence à la constante `template`, car elle n'est pas exportée.
Le collecteur, en revanche, peut appliquer le folding à la constante `template` dans la définition des métadonnées en intégrant son contenu.
Le résultat est le même que si vous aviez écrit :

```angular-ts
@Component({
  selector: 'app-hero',
  template: '<div>{{hero().name}}</div>',
})
export class Hero {
  hero = input.required<Hero>();
}
```

Il n'y a plus de référence à `template` et donc plus rien pour gêner le compilateur lorsqu'il interprétera ensuite la sortie du _collecteur_ dans `.metadata.json`.

Vous pouvez aller plus loin avec cet exemple en incluant la constante `template` dans une autre expression :

```angular-ts
const template = '<div>{{hero().name}}</div>';

@Component({
  selector: 'app-hero',
  template: template + '<div>{{hero().title}}</div>',
})
export class Hero {
  hero = input.required<Hero>();
}
```

Le collecteur réduit cette expression à la chaîne équivalente obtenue par _folding_ :

```angular-ts
'<div>{{hero().name}}</div><div>{{hero().title}}</div>';
```

#### Syntaxe foldable {#foldable-syntax}

Le tableau suivant indique les expressions auxquelles le collecteur peut ou non appliquer le folding :

| Syntaxe | Foldable |
| :------------------------------- | :--------------------------------------- |
| Objet littéral | oui |
| Tableau littéral | oui |
| Spread dans un tableau littéral | non |
| Appels | non |
| New | non |
| Accès à une propriété | oui, si la cible est foldable |
| Index de tableau | oui, si la cible et l'index sont foldables |
| Référence à un identifiant | oui, s'il s'agit d'une référence à une variable locale |
| Un template sans substitution | oui |
| Un template avec substitutions | oui, si les substitutions sont foldables |
| Chaîne littérale | oui |
| Nombre littéral | oui |
| Booléen littéral | oui |
| Null littéral | oui |
| Opérateur préfixe pris en charge | oui, si l'opérande est foldable |
| Opérateur binaire pris en charge | oui, si les opérandes gauche et droit sont foldables |
| Opérateur conditionnel | oui, si la condition est foldable |
| Parenthèses | oui, si l'expression est foldable |

Si une expression n'est pas foldable, le collecteur l'écrit dans `.metadata.json` sous forme d'[AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree) que le compilateur devra résoudre.

## Phase 2 : génération du code {#phase-2-code-generation}

Le collecteur ne tente pas de comprendre les métadonnées qu'il collecte et écrit dans `.metadata.json`.
Il représente les métadonnées du mieux qu'il peut et enregistre des erreurs lorsqu'il détecte une violation de la syntaxe des métadonnées.
C'est au compilateur d'interpréter le `.metadata.json` pendant la phase de génération du code.

Le compilateur comprend toutes les formes syntaxiques que le collecteur prend en charge, mais il peut rejeter des métadonnées _syntaxiquement_ correctes si leur _sémantique_ enfreint les règles du compilateur.

### Symboles public ou protected {#public-or-protected-symbols}

Le compilateur ne peut référencer que des _symboles exportés_.

- Les membres des classes de composants décorées doivent être public ou protected.
  Vous ne pouvez pas rendre une propriété `input()` private.

- Les propriétés liées aux données doivent elles aussi être public ou protected

### Classes et fonctions prises en charge {#supported-classes-and-functions}

Le collecteur peut représenter un appel de fonction ou une création d'objet avec `new` tant que la syntaxe est valide.
Le compilateur, en revanche, peut ensuite refuser de générer un appel à une fonction _particulière_ ou la création d'un objet _particulier_.

Le compilateur ne peut créer des instances que de certaines classes, ne prend en charge que les décorateurs du cœur et ne prend en charge que les appels à des macros \(fonctions ou méthodes statiques\) qui renvoient des expressions.

| Action du compilateur | Détails |
| :------------------- | :------------------------------------------------------------------------------------------------------------------- |
| Nouvelles instances | Le compilateur n'autorise que les métadonnées qui créent des instances de la classe `InjectionToken` de `@angular/core`. |
| Décorateurs pris en charge | Le compilateur ne prend en charge que les métadonnées des [décorateurs Angular du module `@angular/core`](/api?type=decorator). |

### Appels de fonctions et de méthodes statiques {#functions-and-static-method-calls}

Le collecteur accepte toute fonction ou méthode statique qui contient une seule instruction `return`.
Le compilateur, en revanche, ne prend en charge que les macros sous forme de fonctions ou de méthodes statiques qui renvoient une _expression_.

Par exemple, considérez la fonction suivante :

```ts
export function wrapInArray<T>(value: T): T[] {
  return [value];
}
```

Vous pouvez appeler `wrapInArray` dans une définition de métadonnées, car elle renvoie la valeur d'une expression conforme au sous-ensemble restreint de JavaScript accepté par le compilateur.

Vous pourriez utiliser `wrapInArray()` ainsi :

```ts
@NgModule({
  declarations: wrapInArray(Typical),
})
export class TypicalModule {}
```

Le compilateur traite cette utilisation comme si vous aviez écrit :

```ts
@NgModule({
  declarations: [Typical],
})
export class TypicalModule {}
```

Le [`RouterModule`](api/router/RouterModule) d'Angular exporte deux méthodes statiques macros, `forRoot` et `forChild`, pour aider à déclarer les routes racines et les routes enfants.
Consultez le [code source](https://github.com/angular/angular/blob/main/packages/router/src/router_module.ts#L139 'RouterModule.forRoot source code')
de ces méthodes pour voir comment les macros peuvent simplifier la configuration de [NgModules](guide/ngmodules/overview) complexes.

## Phase 3 : vérification de types des templates {#phase-3-template-type-checking}

L'une des fonctionnalités les plus utiles du compilateur Angular est sa capacité à vérifier les types des expressions dans les templates et à détecter toute erreur avant qu'elle ne provoque un plantage à l'exécution.
Pendant la phase de vérification de types des templates, le compilateur de templates d'Angular utilise le compilateur TypeScript pour valider les expressions de binding dans les templates.

La vérification stricte dans cette phase est activée par défaut et contrôlée par l'option de compilation `"strictTemplates"` dans les `"angularCompilerOptions"` du fichier de configuration TypeScript du projet
(consultez [Options du compilateur Angular](reference/configs/angular-compiler-options)).

La validation des templates produit des messages d'erreur lorsqu'une erreur de type est détectée dans une expression
de binding d'un template, de la même manière que le compilateur TypeScript signale les erreurs de type dans le code d'un fichier `.ts`
.

Par exemple, considérez le composant suivant :

```angular-ts
@Component({
  selector: 'my-component',
  template: '{{person.addresss.street}}',
})
class MyComponent {
  person?: Person;
}
```

Cela produit l'erreur suivante :

```shell {hideCopy}

my.component.ts.MyComponent.html(1,1): : Property 'addresss' does not exist on type 'Person'. Did you mean 'address'?

```

Le nom de fichier indiqué dans le message d'erreur, `my.component.ts.MyComponent.html`, est un fichier synthétique
généré par le compilateur de templates, qui contient le template de la classe `MyComponent`.
Le compilateur n'écrit jamais ce fichier sur le disque.
Les numéros de ligne et de colonne sont relatifs à la chaîne du template dans l'annotation `@Component` de la classe, `MyComponent` dans ce cas.
Si un composant utilise `templateUrl` au lieu de `template`, les erreurs sont signalées dans le fichier HTML référencé par `templateUrl` au lieu d'un fichier synthétique.

L'emplacement de l'erreur est le début du nœud texte qui contient l'expression d'interpolation en erreur.
Si l'erreur se trouve dans un attribute binding comme `[value]="person.address.street"`, l'emplacement
de l'erreur est celui de l'attribut qui contient l'erreur.

La validation utilise le vérificateur de types de TypeScript et les options fournies au compilateur TypeScript pour contrôler le niveau de détail de la validation des types.
Par exemple, si l'option TypeScript `strictNullChecks` est activée, l'erreur

```shell {hideCopy}

my.component.ts.MyComponent.html(1,1): : Object is possibly 'undefined'

```

est signalée en plus du message d'erreur ci-dessus.

### Affinage de type {#type-narrowing}

L'expression utilisée dans une directive `ngIf` sert à affiner les unions de types dans le compilateur
de templates d'Angular, de la même manière que l'expression `if` en TypeScript.
Par exemple, pour éviter l'erreur `Object is possibly 'undefined'` dans le template ci-dessus, modifiez-le pour n'émettre l'interpolation que si la valeur de `person` est initialisée, comme ci-dessous :

```angular-ts
@Component({
  selector: 'my-component',
  template: '<span *ngIf="person"> {{person.address.street}} </span>',
})
class MyComponent {
  person?: Person;
}
```

Utiliser `*ngIf` permet au compilateur TypeScript de déduire que la variable `person` utilisée dans l'expression de binding ne sera jamais `undefined`.

Pour plus d'informations sur l'affinage de type des inputs, consultez [Améliorer la vérification de types des templates pour les directives personnalisées](/guide/directives/structural-directives#improving-template-type-checking-for-custom-directives).

### Opérateur d'assertion non-null {#non-null-type-assertion-operator}

Utilisez l'opérateur d'assertion de type non-null pour supprimer l'erreur `Object is possibly 'undefined'` lorsqu'il n'est pas pratique d'utiliser `*ngIf` ou lorsqu'une contrainte du composant garantit que l'expression est toujours non nulle au moment où l'expression de binding est interpolée.

Dans l'exemple suivant, les propriétés `person` et `address` sont toujours définies ensemble, ce qui implique que `address` est toujours non nulle si `person` est non nulle.
Il n'existe aucun moyen pratique de décrire cette contrainte à TypeScript et au compilateur de templates, mais l'erreur est supprimée dans l'exemple en utilisant `address!.street`.

```angular-ts
@Component({
  selector: 'my-component',
  template: '<span *ngIf="person"> {{person.name}} lives on {{address!.street}} </span>',
})
class MyComponent {
  person?: Person;
  address?: Address;

  setData(person: Person, address: Address) {
    this.person = person;
    this.address = address;
  }
}
```

L'opérateur d'assertion non-null doit être utilisé avec parcimonie, car un refactoring du composant pourrait rompre cette contrainte.

Dans cet exemple, il est recommandé d'inclure la vérification de `address` dans le `*ngIf`, comme ci-dessous :

```angular-ts
@Component({
  selector: 'my-component',
  template: '<span *ngIf="person && address"> {{person.name}} lives on {{address.street}} </span>',
})
class MyComponent {
  person?: Person;
  address?: Address;

  setData(person: Person, address: Address) {
    this.person = person;
    this.address = address;
  }
}
```
