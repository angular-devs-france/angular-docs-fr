<!-- source: 4ba3e72230a9de43a008ea54e329e9b72417a299 -->
# Syntaxe des expressions

Les expressions Angular sont basées sur JavaScript, mais en diffèrent sur quelques points clés. Ce guide présente les similitudes et les différences entre les expressions Angular et le JavaScript standard.

## Valeurs littérales {#value-literals}

Angular prend en charge un sous-ensemble des [valeurs littérales](https://developer.mozilla.org/en-US/docs/Glossary/Literal) de JavaScript.

### Valeurs littérales prises en charge {#supported-value-literals}

| Type de littéral       | Exemples de valeurs             |
| ---------------------- | ------------------------------- |
| Chaîne de caractères   | `'Hello'`, `"World"`            |
| Booléen                | `true`, `false`                 |
| Nombre                 | `123`, `3.14`                   |
| Objet                  | `{name: 'Alice'}`               |
| Tableau                | `['Onion', 'Cheese', 'Garlic']` |
| null                   | `null`                          |
| RegExp                 | `/\d+/`                         |
| Template string        | `` `Hello ${name}` ``           |
| Template string étiquetée | `` tag`Hello ${name}` ``        |

### Valeurs littérales non prises en charge {#unsupported-value-literals}

| Type de littéral | Exemples de valeurs |
| ------------ | -------------- |
| BigInt       | `1n`           |

## Globales {#globals}

Les expressions Angular prennent en charge les [globales](https://developer.mozilla.org/en-US/docs/Glossary/Global_object) suivantes :

- [undefined](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/undefined)
- [$any](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#any)

Aucune autre globale JavaScript n'est prise en charge. Parmi les globales JavaScript courantes figurent `Number`, `Boolean`, `NaN`, `Infinity`, `parseInt`, etc.

## Variables locales {#local-variables}

Angular met automatiquement à disposition, dans certains contextes, des variables locales spéciales utilisables dans les expressions. Ces variables spéciales commencent toujours par le caractère dollar (`$`).

Par exemple, les blocs `@for` fournissent plusieurs variables locales correspondant à des informations sur la boucle, comme `$index`.

## Quels opérateurs sont pris en charge ? {#what-operators-are-supported}

### Opérateurs pris en charge {#supported-operators}

Angular prend en charge les opérateurs suivants du JavaScript standard.

| Opérateur                     | Exemple(s)                                     |
| ----------------------------- | ---------------------------------------------- |
| Addition / Concaténation      | `1 + 2`                                        |
| Soustraction                  | `52 - 3`                                       |
| Multiplication                | `41 * 6`                                       |
| Division                      | `20 / 4`                                       |
| Reste (modulo)                | `17 % 5`                                       |
| Exponentiation                | `10 ** 3`                                      |
| Parenthèses                   | `9 * (8 + 4)`                                  |
| Conditionnel (ternaire)       | `a > b ? true : false`                         |
| ET (logique)                  | `&&`                                           |
| OU (logique)                  | `\|\|`                                         |
| NON (logique)                 | `!`                                            |
| Nullish coalescing            | `possiblyNullValue ?? 'default'`               |
| Opérateurs de comparaison     | `<`, `<=`, `>`, `>=`, `==`, `===`, `!==`, `!=` |
| Négation unaire               | `-x`                                           |
| Plus unaire                   | `+y`                                           |
| Accesseur de propriété        | `person['name']`                               |
| typeof                        | `typeof 42`                                    |
| void                          | `void 1`                                       |
| in                            | `'model' in car`                               |
| instanceof                    | `car instanceof Automobile`                    |
| Affectation                   | `a = b`                                        |
| Affectation après addition    | `a += b`                                       |
| Affectation après soustraction | `a -= b`                                       |
| Affectation après multiplication | `a *= b`                                       |
| Affectation après division    | `a /= b`                                       |
| Affectation du reste          | `a %= b`                                       |
| Affectation après exponentiation | `a **= b`                                      |
| Affectation après ET logique  | `a &&= b`                                      |
| Affectation après OU logique  | `a \|\|= b`                                    |
| Affectation de nullish coalescing   | `a ??= b`                                      |
| Décomposition dans les littéraux d'objet | `{...obj, foo: 'bar'}`                         |
| Décomposition dans les littéraux de tableau | `[...arr, 1, 2, 3]`                            |
| Décomposition dans les appels de fonction | `fn(...args)`                                  |
| Fonction fléchée              | `items.filter((i) => i.active)`                |

NOTE: Les opérateurs d'affectation et d'incrémentation/décrémentation ne sont valides que dans les [instructions d'event listener](#event-listener-statements). Les utiliser dans un binding ou une interpolation provoque une erreur de compilation, sauf s'ils se trouvent dans une fonction fléchée.

Les fonctions fléchées doivent avoir une expression comme corps. Les pipes ne peuvent pas être utilisés dans une fonction fléchée, mais vous pouvez passer une fonction fléchée en argument à un pipe.

Les expressions Angular prennent également en charge les opérateurs non standard suivants :

| Opérateur                       | Exemple(s)                     |
| ------------------------------- | ------------------------------ |
| [Pipe](/guide/templates/pipes)  | `{{ total \| currency }}`      |
| Chaînage optionnel\*            | `someObj.someProp?.nestedProp` |
| Assertion non nulle (TypeScript) | `someObj!.someProp`            |

### Migration de la navigation sécurisée {#safe-navigation-migration}

Avant Angular 22, l'opérateur de chaînage optionnel (`?.`) renvoyait `null` lorsque l'opérande de gauche valait `null` ou `undefined`, alors que l'opérateur `?.` du JavaScript standard renvoie `undefined`.
Depuis Angular 22, le comportement de l'opérateur de chaînage optionnel dans les expressions Angular est aligné sur celui du JavaScript standard.

Lors de la migration vers la v22, les schematics de `ng update` ont ajouté une fonction magique `$safeNavigationMigration` aux expressions existantes pour préserver l'ancien comportement qui renvoie `null`.

```html
{{ $safeNavigationMigration(foo?.bar) }}
```

`$safeNavigationMigration` est **uniquement une aide temporaire à la migration**. Elle indique au compilateur de compiler l'expression de navigation sécurisée qu'elle entoure avec l'ancienne sémantique qui renvoie null plutôt qu'avec la sémantique `?.` du JavaScript standard. Ce n'est pas une vraie fonction et elle ne peut pas être appelée depuis TypeScript.

NOTE: Privilégiez la migration des expressions pour qu'elles ne dépendent plus de la distinction entre `null` et `undefined`, afin de pouvoir retirer `$safeNavigationMigration`. Cette fonction pourrait être supprimée dans une future version d'Angular.

### Opérateurs non pris en charge {#unsupported-operators}

| Opérateur             | Exemple(s)                        |
| --------------------- | --------------------------------- |
| Tous les opérateurs bit à bit | `&`, `&=`, `~`, `\|=`, `^=`, etc. |
| Déstructuration d'objet | `const { name } = person`         |
| Déstructuration de tableau | `const [firstItem] = items`       |
| Opérateur virgule     | `x = (x++, x)`                    |
| new                   | `new Car()`                       |

## Contexte lexical des expressions {#lexical-context-for-expressions}

Les expressions Angular sont évaluées dans le contexte de la classe du composant ainsi que des [variables de template](/guide/templates/variables), variables locales et globales pertinentes.

Quand vous faites référence aux membres de la classe du composant, `this` est toujours implicite. Cependant, si un template déclare une [variable de template](guide/templates/variables) portant le même nom qu'un membre, la variable masque ce membre. Vous pouvez faire référence sans ambiguïté à un tel membre de la classe en utilisant explicitement `this.`. Cela peut être utile lors de la création d'une déclaration `@let` qui masque un membre de la classe, par exemple pour affiner le type d'un signal.

## Déclarations {#declarations}

De manière générale, les déclarations ne sont pas prises en charge dans les expressions Angular. Cela inclut, sans s'y limiter :

| Déclarations                      | Exemple(s)                                  |
| --------------------------------- | ------------------------------------------- |
| Variables                         | `let label = 'abc'`, `const item = 'apple'` |
| Fonctions                         | `function myCustomFunction() { }`           |
| Fonctions fléchées avec un corps en bloc | `() => { }`                                 |
| Classes                           | `class Rectangle { }`                       |

## Instructions d'event listener {#event-listener-statements}

Les event handlers sont des **instructions** et non des expressions. Bien qu'ils prennent en charge toute la syntaxe des expressions Angular, il existe deux différences clés :

1. Les instructions **prennent en charge** les opérateurs d'affectation (mais pas les affectations par déstructuration)
1. Les instructions **ne prennent pas en charge** les pipes
