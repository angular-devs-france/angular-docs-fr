<!-- source: 9c0a36680b0d9c6efffc9d1956e6dafc05b40ad9 -->
# Pipes

## Vue d'ensemble {#overview}

Les pipes sont des opérateurs spéciaux des expressions de template Angular qui vous permettent de transformer des données de manière déclarative dans votre template. Les pipes vous permettent de déclarer une fonction de transformation une seule fois, puis d'utiliser cette transformation dans plusieurs templates. Les pipes Angular utilisent le caractère barre verticale (`|`), inspiré du [pipe Unix](<https://en.wikipedia.org/wiki/Pipeline_(Unix)>).

NOTE: La syntaxe des pipes d'Angular s'écarte du JavaScript standard, qui utilise le caractère barre verticale pour l'[opérateur OU bit à bit](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Bitwise_OR). Les expressions de template Angular ne prennent pas en charge les opérateurs bit à bit.

Voici un exemple utilisant quelques pipes intégrés fournis par Angular :

```angular-ts
import {Component} from '@angular/core';
import {CurrencyPipe, DatePipe, TitleCasePipe} from '@angular/common';

@Component({
  selector: 'app-root',
  imports: [CurrencyPipe, DatePipe, TitleCasePipe],
  template: `
    <main>
      <!-- Transform the company name to title-case and
       transform the purchasedOn date to a locale-formatted string -->
      <h1>Purchases from {{ company | titlecase }} on {{ purchasedOn | date }}</h1>

      <!-- Transform the amount to a currency-formatted string -->
      <p>Total: {{ amount | currency }}</p>
    </main>
  `,
})
export class ShoppingCart {
  amount = 123.45;
  company = 'acme corporation';
  purchasedOn = '2024-07-08';
}
```

Quand Angular rend le composant, il s'assure que le format de date et la devise appropriés dépendent de la locale de l'utilisateur. Si l'utilisateur se trouve aux États-Unis, le rendu serait :

```angular-html
<main>
  <h1>Purchases from Acme Corporation on Jul 8, 2024</h1>
  <p>Total: $123.45</p>
</main>
```

Consultez le [guide approfondi sur l'i18n](/guide/i18n) pour en savoir plus sur la façon dont Angular localise les valeurs.

### Pipes intégrés {#built-in-pipes}

Angular inclut un ensemble de pipes intégrés dans le package `@angular/common` :

| Nom                                           | Description                                                                                   |
| --------------------------------------------- | --------------------------------------------------------------------------------------------- |
| [`AsyncPipe`](api/common/AsyncPipe)           | Lit la valeur d'une `Promise` ou d'un `Observable` RxJS. |
| [`CurrencyPipe`](api/common/CurrencyPipe)     | Transforme un nombre en chaîne monétaire, formatée selon les règles de la locale. |
| [`DatePipe`](api/common/DatePipe)             | Formate une valeur `Date` selon les règles de la locale. |
| [`DecimalPipe`](api/common/DecimalPipe)       | Transforme un nombre en chaîne avec un séparateur décimal, formatée selon les règles de la locale. |
| [`I18nPluralPipe`](api/common/I18nPluralPipe) | Associe une valeur à une chaîne qui met la valeur au pluriel selon les règles de la locale. |
| [`I18nSelectPipe`](api/common/I18nSelectPipe) | Associe une clé à un sélecteur personnalisé qui renvoie la valeur souhaitée. |
| [`JsonPipe`](api/common/JsonPipe)             | Transforme un objet en représentation sous forme de chaîne via `JSON.stringify`, destinée au débogage. |
| [`KeyValuePipe`](api/common/KeyValuePipe)     | Transforme un Object ou une Map en tableau de paires clé-valeur. |
| [`LowerCasePipe`](api/common/LowerCasePipe)   | Transforme le texte entièrement en minuscules. |
| [`PercentPipe`](api/common/PercentPipe)       | Transforme un nombre en chaîne de pourcentage, formatée selon les règles de la locale. |
| [`SlicePipe`](api/common/SlicePipe)           | Crée un nouvel Array ou une nouvelle String contenant un sous-ensemble (une tranche) des éléments. |
| [`TitleCasePipe`](api/common/TitleCasePipe)   | Transforme le texte en casse de titre. |
| [`UpperCasePipe`](api/common/UpperCasePipe)   | Transforme le texte entièrement en majuscules. |

## Utiliser les pipes {#using-pipes}

L'opérateur pipe d'Angular utilise le caractère barre verticale (`|`) dans une expression de template. L'opérateur pipe est un opérateur binaire : l'opérande de gauche est la valeur passée à la fonction de transformation, et l'opérande de droite est le nom du pipe suivi d'éventuels arguments supplémentaires (décrits ci-dessous).

```angular-html
<p>Total: {{ amount | currency }}</p>
```

Dans cet exemple, la valeur de `amount` est passée au `CurrencyPipe`, dont le nom de pipe est `currency`. Il affiche ensuite la devise par défaut correspondant à la locale de l'utilisateur.

### Combiner plusieurs pipes dans la même expression {#combining-multiple-pipes-in-the-same-expression}

Vous pouvez appliquer plusieurs transformations à une valeur en utilisant plusieurs opérateurs pipe. Angular exécute les pipes de gauche à droite.

L'exemple suivant montre une combinaison de pipes pour afficher une date localisée entièrement en majuscules :

```angular-html
<p>The event will occur on {{ scheduledOn | date | uppercase }}.</p>
```

### Passer des paramètres aux pipes {#passing-parameters-to-pipes}

Certains pipes acceptent des paramètres pour configurer la transformation. Pour spécifier un paramètre, ajoutez après le nom du pipe un deux-points (`:`) suivi de la valeur du paramètre.

Par exemple, le `DatePipe` peut prendre des paramètres pour formater la date d'une manière spécifique.

```angular-html
<p>The event will occur at {{ scheduledOn | date: 'hh:mm' }}.</p>
```

Certains pipes peuvent accepter plusieurs paramètres. Vous pouvez spécifier des valeurs de paramètres supplémentaires séparées par le caractère deux-points (`:`).

Par exemple, nous pouvons aussi passer un second paramètre optionnel pour contrôler le fuseau horaire.

```angular-html
<p>The event will occur at {{ scheduledOn | date: 'hh:mm' : 'UTC' }}.</p>
```

## Comment fonctionnent les pipes {#how-pipes-work}

Conceptuellement, les pipes sont des fonctions qui acceptent une valeur d'entrée et renvoient une valeur transformée.

```angular-ts
import {Component} from '@angular/core';
import {CurrencyPipe} from '@angular/common';

@Component({
  selector: 'app-root',
  imports: [CurrencyPipe],
  template: `
    <main>
      <p>Total: {{ amount | currency }}</p>
    </main>
  `,
})
export class AppComponent {
  amount = 123.45;
}
```

Dans cet exemple :

1. `CurrencyPipe` est importé depuis `@angular/common`
1. `CurrencyPipe` est ajouté au tableau `imports`
1. La donnée `amount` est passée au pipe `currency`

### Priorité de l'opérateur pipe {#pipe-operator-precedence}

L'opérateur pipe a une priorité plus faible que les autres opérateurs binaires, dont `+`, `-`, `*`, `/`, `%`, `&&`, `||` et `??`.

```angular-html
<!-- firstName and lastName are concatenated before the result is passed to the uppercase pipe -->
{{ firstName + lastName | uppercase }}
```

L'opérateur pipe a une priorité plus élevée que l'opérateur conditionnel (ternaire).

```angular-html
{{ (isAdmin ? 'Access granted' : 'Access denied') | uppercase }}
```

Si la même expression était écrite sans parenthèses :

<!-- prettier-ignore -->
```angular-html
{{ isAdmin ? 'Access granted' : 'Access denied' | uppercase }}
```

Elle serait plutôt analysée comme :

```angular-html
{{ isAdmin ? 'Access granted' : ('Access denied' | uppercase) }}
```

Utilisez toujours des parenthèses dans vos expressions quand la priorité des opérateurs peut être ambiguë.

### Détection de changements avec les pipes {#change-detection-with-pipes}

Par défaut, tous les pipes sont considérés comme `pure`, ce qui signifie qu'ils ne s'exécutent que lorsqu'une valeur d'entrée primitive (comme un `String`, `Number`, `Boolean` ou `Symbol`) ou une référence d'objet (comme un `Array`, `Object`, `Function` ou `Date`) change. Les pipes purs offrent un avantage en matière de performances, car Angular peut éviter d'appeler la fonction de transformation si la valeur passée n'a pas changé.

Par conséquent, les mutations des propriétés d'un objet ou des éléments d'un tableau ne sont pas détectées, à moins que la référence de l'objet ou du tableau entier ne soit remplacée par une autre instance. Si vous voulez ce niveau de détection de changements, consultez [détecter les changements dans les tableaux ou les objets](#detecting-change-within-arrays-or-objects).

## Créer des pipes personnalisés {#creating-custom-pipes}

Vous pouvez définir un pipe personnalisé en implémentant une classe TypeScript avec le décorateur `@Pipe`. Un pipe doit avoir deux choses :

- Un nom, spécifié dans le décorateur du pipe
- Une méthode nommée `transform` qui effectue la transformation de la valeur.

La classe TypeScript doit en plus implémenter l'interface `PipeTransform` pour garantir qu'elle respecte la signature de type d'un pipe.

Voici un exemple de pipe personnalisé qui transforme des chaînes en kebab case :

```angular-ts {header: "kebab-case.pipe.ts"}
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  name: 'kebabCase',
})
export class KebabCasePipe implements PipeTransform {
  transform(value: string): string {
    return value.toLowerCase().replace(/ /g, '-');
  }
}
```

### Utiliser le décorateur `@Pipe` {#using-the-pipe-decorator}

Quand vous créez un pipe personnalisé, importez `Pipe` depuis le package `@angular/core` et utilisez-le comme décorateur de la classe TypeScript.

```angular-ts
import {Pipe} from '@angular/core';

@Pipe({
  name: 'myCustomTransformation',
})
export class MyCustomTransformationPipe {}
```

Le décorateur `@Pipe` requiert un `name` qui détermine comment le pipe est utilisé dans un template.

### Convention de nommage des pipes personnalisés {#naming-convention-for-custom-pipes}

La convention de nommage des pipes personnalisés se compose de deux conventions :

- `name` - le camelCase est recommandé. N'utilisez pas de tirets.
- `class name` - version PascalCase du `name`, suivie du suffixe `Pipe`

### Implémenter l'interface `PipeTransform` {#implement-the-pipetransform-interface}

En plus du décorateur `@Pipe`, les pipes personnalisés doivent toujours implémenter l'interface `PipeTransform` de `@angular/core`.

```angular-ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  name: 'myCustomTransformation',
})
export class MyCustomTransformationPipe implements PipeTransform {}
```

Implémenter cette interface garantit que votre classe de pipe a la bonne structure.

### Transformer la valeur d'un pipe {#transforming-the-value-of-a-pipe}

Chaque transformation est invoquée par la méthode `transform`, dont le premier paramètre est la valeur passée et dont la valeur de retour est la valeur transformée.

```angular-ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  name: 'myCustomTransformation',
})
export class MyCustomTransformationPipe implements PipeTransform {
  transform(value: string): string {
    return `My custom transformation of ${value}.`;
  }
}
```

### Ajouter des paramètres à un pipe personnalisé {#adding-parameters-to-a-custom-pipe}

Vous pouvez ajouter des paramètres à votre transformation en ajoutant des paramètres supplémentaires à la méthode `transform` :

```angular-ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  name: 'myCustomTransformation',
})
export class MyCustomTransformationPipe implements PipeTransform {
  transform(value: string, format: string): string {
    let msg = `My custom transformation of ${value}.`;

    if (format === 'uppercase') {
      return msg.toUpperCase();
    } else {
      return msg;
    }
  }
}
```

### Détecter les changements dans les tableaux ou les objets {#detecting-change-within-arrays-or-objects}

Quand vous voulez qu'un pipe détecte les changements dans des tableaux ou des objets, il doit être marqué comme fonction impure en passant l'option `pure` avec la valeur `false`.

IMPORTANT: Évitez de créer des pipes impurs sauf en cas d'absolue nécessité, car ils peuvent entraîner une pénalité de performances importante s'ils sont utilisés sans précaution.

```angular-ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({
  name: 'joinNamesImpure',
  pure: false,
})
export class JoinNamesImpurePipe implements PipeTransform {
  transform(names: string[]): string {
    return names.join();
  }
}
```

Les développeurs Angular adoptent souvent la convention d'inclure `Impure` dans le `name` du pipe et dans le nom de la classe, pour signaler aux autres développeurs le risque potentiel en matière de performances.

## Utiliser la logique d'un pipe en dehors des templates {#using-pipe-logic-outside-templates}

Les pipes sont des opérateurs de template conçus pour transformer des données de manière déclarative dans les templates. Quand vous avez besoin de la même logique de transformation dans un service, une classe utilitaire ou tout autre contexte hors template, **évitez d'injecter la classe du pipe**. Les pipes ne sont pas conçus comme des services injectables.

### Extraire la logique des pipes personnalisés {#extract-the-logic-from-custom-pipes}

Quand vous créez un pipe personnalisé, extrayez la transformation dans une fonction standalone. Faites déléguer la méthode `transform()` du pipe à cette fonction, et importez directement la fonction partout où vous en avez besoin par ailleurs.

```ts {header: "kebab-case.ts"}
export function toKebabCase(value: string): string {
  return value.toLowerCase().replace(/ /g, '-');
}
```

```ts {header: "kebab-case.pipe.ts" , prefer}
import {Pipe, PipeTransform} from '@angular/core';
import {toKebabCase} from './kebab-case';

@Pipe({name: 'kebabCase'})
export class KebabCasePipe implements PipeTransform {
  transform(value: string): string {
    return toKebabCase(value);
  }
}
```

```ts {header: "formatter.service.ts" , prefer}
import {Service} from '@angular/core';
import {toKebabCase} from './kebab-case';

@Service()
export class FormatterService {
  formatSlug(title: string): string {
    return toKebabCase(title);
  }
}
```

```ts {header: "formatter.service.ts" , avoid}
import {inject, Service} from '@angular/core';
import {KebabCasePipe} from './kebab-case.pipe';

@Service()
export class FormatterService {
  // Avoid injecting the pipe class into services or other classes.
  private kebabCasePipe = inject(KebabCasePipe);

  formatSlug(title: string): string {
    return this.kebabCasePipe.transform(title);
  }
}
```

### Utiliser les fonctions de formatage pour les pipes intégrés {#use-formatting-functions-for-built-in-pipes}

Les pipes intégrés d'Angular qui tiennent compte de la locale ont chacun une fonction de formatage standalone correspondante, exportée depuis `@angular/common`. Utilisez ces fonctions plutôt que d'injecter la classe du pipe.

| Pipe           | Fonction standalone |
| -------------- | ------------------- |
| `DatePipe`     | `formatDate`        |
| `CurrencyPipe` | `formatCurrency`    |
| `DecimalPipe`  | `formatNumber`      |
| `PercentPipe`  | `formatPercent`     |

```ts {prefer}
import {Service, LOCALE_ID, inject} from '@angular/core';
import {formatNumber} from '@angular/common';

@Service()
export class PriceService {
  private locale = inject(LOCALE_ID);

  format(value: number) {
    return formatNumber(value, this.locale, '1.2-2');
  }
}
```

```ts {avoid}
import {inject, Service} from '@angular/core';
import {DecimalPipe} from '@angular/common';

@Service()
export class PriceService {
  private decimalPipe = inject(DecimalPipe);

  format(value: number) {
    return this.decimalPipe.transform(value, '1.2-2');
  }
}
```
