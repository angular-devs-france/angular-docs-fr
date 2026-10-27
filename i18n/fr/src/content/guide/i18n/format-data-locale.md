<!-- source: 122f32b6fee4a86a5422571c86ea891e730eb784 -->
# Formater les données selon la locale

Angular fournit les [pipes](guide/templates/pipes) intégrés de transformation de données suivants.
Les pipes de transformation de données utilisent le jeton [`LOCALE_ID`][ApiCoreLocaleId] pour formater les données selon les règles de chaque locale.

| Pipe de transformation de données                | Détails                                           |
| :-------------------------------------- | :------------------------------------------------ |
| [`DatePipe`][ApiCommonDatepipe]         | Formate une date.                             |
| [`CurrencyPipe`][ApiCommonCurrencypipe] | Transforme un nombre en chaîne de devise.       |
| [`DecimalPipe`][ApiCommonDecimalpipe]   | Transforme un nombre en chaîne de nombre décimal. |
| [`PercentPipe`][ApiCommonPercentpipe]   | Transforme un nombre en chaîne de pourcentage.     |

## Utiliser DatePipe pour afficher la date du jour {#use-datepipe-to-display-the-current-date}

Pour afficher la date du jour dans le format de la locale courante, utilisez le format suivant pour le `DatePipe`.

```angular-html
{{ today | date }}
```

## Remplacer la locale courante pour CurrencyPipe {#override-current-locale-for-currencypipe}

Ajoutez le paramètre `locale` au pipe pour remplacer la valeur courante du jeton `LOCALE_ID`.

Pour forcer la devise à utiliser l'anglais américain \(`en-US`\), utilisez le format suivant pour le `CurrencyPipe`

```angular-html
{{ amount | currency: 'USD' : 'symbol' : '1.2-2' : 'en-US' }}
```

HELPFUL: La locale spécifiée pour le `CurrencyPipe` remplace le jeton global `LOCALE_ID` de votre application.

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/i18n/prepare" title="Préparer un composant pour la traduction"/>
</docs-pill-row>

[ApiCommonCurrencypipe]: api/common/CurrencyPipe 'CurrencyPipe | Common - API | Angular'
[ApiCommonDatepipe]: api/common/DatePipe 'DatePipe | Common - API | Angular'
[ApiCommonDecimalpipe]: api/common/DecimalPipe 'DecimalPipe | Common - API | Angular'
[ApiCommonPercentpipe]: api/common/PercentPipe 'PercentPipe | Common - API | Angular'
[ApiCoreLocaleId]: api/core/LOCALE_ID 'LOCALE_ID | Core - API | Angular'
