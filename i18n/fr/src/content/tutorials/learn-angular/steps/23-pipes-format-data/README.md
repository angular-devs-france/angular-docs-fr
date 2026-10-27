<!-- source: 3dc6176cb38f2d322c5b8f4e932bb4542cb87bec -->
# Formater des données avec les pipes

Vous pouvez aller encore plus loin dans l'utilisation des pipes en les configurant. Pour configurer un pipe, passez-lui des options.

NOTE: Pour en savoir plus, consultez [le formatage des données avec les pipes dans le guide approfondi](/guide/templates/pipes).

Dans cette activité, vous allez manipuler quelques pipes et leurs paramètres.

<hr>

Pour passer des paramètres à un pipe, utilisez la syntaxe `:` suivie de la valeur du paramètre. Voici un exemple :

```angular-html
template: `{{ date | date: 'medium' }}`;
```

Le résultat est `Jun 15, 2015, 9:43:11 PM`.

Place à la personnalisation de la sortie de quelques pipes :

<docs-workflow>

<docs-step title="Formater un nombre avec `DecimalPipe` {#format-a-number-with-decimalpipe}">

Dans `app.ts`, mettez à jour le template pour ajouter un paramètre au pipe `decimal`.

```angular-html {highlight:[2]}
template: ` ...
<li>Number with "decimal" {{ num | number: '3.2-2' }}</li>
`
```

NOTE: C'est quoi, ce format ? Le paramètre du `DecimalPipe` s'appelle `digitsInfo` ; il utilise le format : `{minIntegerDigits}.{minFractionDigits}-{maxFractionDigits}`

</docs-step>

<docs-step title="Formater une date avec `DatePipe` {#format-a-date-with-datepipe}">

Mettez maintenant à jour le template pour utiliser le pipe `date`.

```angular-html {highlight:[2]}
template: ` ...
<li>Date with "date" {{ birthday | date: 'medium' }}</li>
`
```

Pour vous amuser un peu plus, essayez différents paramètres pour `date`. Vous trouverez plus d'informations dans la [documentation Angular](guide/templates/pipes).

</docs-step>

<docs-step title="Formater une devise avec `CurrencyPipe` {#format-a-currency-with-currencypipe}">

Pour votre dernière tâche, mettez à jour le template pour utiliser le pipe `currency`.

```angular-html {highlight:[2]}
template: ` ...
<li>Currency with "currency" {{ cost | currency }}</li>
`
```

Vous pouvez aussi essayer différents paramètres pour `currency`. Vous trouverez plus d'informations dans la [documentation Angular](guide/templates/pipes).

</docs-step>

</docs-workflow>

Beau travail avec les pipes. Vous avez déjà fait de beaux progrès.

Il existe encore bien d'autres pipes intégrés que vous pouvez utiliser dans vos applications. Vous en trouverez la liste dans la [documentation Angular](guide/templates/pipes).

Si les pipes intégrés ne couvrent pas vos besoins, vous pouvez aussi créer un pipe personnalisé. Consultez la leçon suivante pour en savoir plus.
