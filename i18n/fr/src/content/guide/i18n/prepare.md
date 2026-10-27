<!-- source: 970c1b68a1acc473434361473fbc1684b1f4d1be -->
# Préparer un composant pour la traduction

Pour préparer votre projet à la traduction, effectuez les actions suivantes.

- Utilisez l'attribut `i18n` pour marquer du texte dans les templates des composants
- Utilisez l'attribut `i18n-` pour marquer les chaînes de texte des attributs dans les templates des composants
- Utilisez la chaîne de message balisée avec `$localize` pour marquer des chaînes de texte dans le code des composants

## Marquer du texte dans le template d'un composant {#mark-text-in-component-template}

Dans le template d'un composant, les métadonnées i18n sont la valeur de l'attribut `i18n`.

```html
<element i18n="{i18n_metadata}">{string_to_translate}</element>
```

Utilisez l'attribut `i18n` pour marquer un message texte statique à traduire dans les templates de vos composants.
Placez-le sur chaque balise d'élément contenant du texte fixe que vous voulez traduire.

HELPFUL: L'attribut `i18n` est un attribut personnalisé que les outils et compilateurs Angular reconnaissent.

### Exemple `i18n` {#i18n-example}

La balise `<h1>` suivante affiche une simple salutation en anglais, « Hello i18n ! ».

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="greeting"/>

Pour marquer la salutation à traduire, ajoutez l'attribut `i18n` à la balise `<h1>`.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-attribute"/>

### Utiliser une instruction conditionnelle avec `i18n` {#using-conditional-statement-with-i18n}

La balise `<div>` suivante affiche du texte traduit dans le contenu du `div` et dans `aria-label` selon l'état du basculement

<docs-code-multifile>
    <docs-code header="app.component.html" path="adev/src/content/examples/i18n/src/app/app.component.html"  region="i18n-conditional"/>
    <docs-code header="app.component.ts" path="adev/src/content/examples/i18n/src/app/app.component.ts" visibleLines="[[13,18],[32,34]]"/>
</docs-code-multifile>

### Traduire du texte en ligne sans élément HTML {#translate-inline-text-without-html-element}

Utilisez l'élément `<ng-container>` pour associer un comportement de traduction à un texte spécifique sans changer la façon dont le texte est affiché.

HELPFUL: Chaque élément HTML crée un nouvel élément du DOM.
Pour éviter de créer un nouvel élément du DOM, enveloppez le texte dans un élément `<ng-container>`.
L'exemple suivant montre l'élément `<ng-container>` transformé en commentaire HTML non affiché.

<docs-code path="adev/src/content/examples/i18n/src/app/app.component.html" region="i18n-ng-container"/>

### Nommer le placeholder d'interpolation {#name-the-interpolation-placeholder}

Par défaut, Angular génère un nom de placeholder pour chaque interpolation d'un message traduit. Pour lui donner un nom explicite qui aide les traducteurs à comprendre le contexte, ajoutez un commentaire `//i18n(ph="name")` à l'intérieur de l'interpolation.

```html
<element i18n>{{ expression //i18n(ph="placeholder_name") }}</element>
```

Par exemple :

```html
<p i18n>Hello, {{ username //i18n(ph="name") }}!</p>
```

C'est l'équivalent, dans le template, du nommage d'un placeholder dans le code d'un composant avec [`$localize`][ApiLocalizeInitLocalize] :

```ts
$localize`Hello, ${username}:name:!`;
```

## Marquer les attributs des éléments pour la traduction {#mark-element-attributes-for-translations}

Dans le template d'un composant, les métadonnées i18n sont la valeur de l'attribut `i18n-{attribute_name}`.

```html
<element i18n-{attribute_name}="{i18n_metadata}" {attribute_name}="{attribute_value}" />
```

Les attributs des éléments HTML contiennent du texte qui doit être traduit avec le reste du texte affiché dans le template du composant.

Utilisez `i18n-{attribute_name}` avec n'importe quel attribut de n'importe quel élément et remplacez `{attribute_name}` par le nom de l'attribut.
Utilisez la syntaxe suivante pour attribuer une signification, une description et un identifiant personnalisé.

```html
i18n-{attribute_name}="{meaning}|{description}@@{id}"
```

### Exemple `i18n-title` {#i18n-title-example}

Pour traduire le titre d'une image, examinez cet exemple.
L'exemple suivant affiche une image avec un attribut `title`.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-title"/>

Pour marquer l'attribut title à traduire, effectuez l'action suivante.

Ajoutez l'attribut `i18n-title`

L'exemple suivant montre comment marquer l'attribut `title` de la balise `img` en ajoutant `i18n-title`.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/src/app/app.component.html" region="i18n-title-translate"/>

## Marquer du texte dans le code d'un composant {#mark-text-in-component-code}

Dans le code d'un composant, le texte source à traduire et les métadonnées sont entourés de caractères backtick \(<code>&#96;</code>\).

Utilisez la chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize] pour marquer une chaîne à traduire dans votre code.

```ts
$localize`string_to_translate`;
```

Les métadonnées i18n sont entourées de caractères deux-points \(`:`\) et précèdent le texte source à traduire.

```ts
$localize`:{i18n_metadata}:string_to_translate`;
```

### Inclure du texte interpolé {#include-interpolated-text}

Incluez des [interpolations](guide/templates/binding#render-dynamic-text-with-text-interpolation) dans une chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize].

```ts
$localize`string_to_translate ${variable_name}`;
```

### Nommer le placeholder d'interpolation {#name-the-interpolation-placeholder-in-code}

```ts
$localize`string_to_translate ${variable_name}:placeholder_name:`;
```

### Syntaxe conditionnelle pour les traductions {#conditional-syntax-for-translations}

```ts
return this.show ? $localize`Show Tabs` : $localize`Hide tabs`;
```

## Métadonnées i18n pour la traduction {#i18n-metadata-for-translation}

```html
{meaning}|{description}@@{custom_id}
```

Les paramètres suivants fournissent du contexte et des informations supplémentaires pour éviter toute confusion au traducteur.

| Paramètre de métadonnées | Détails                                                               |
| :----------------- | :-------------------------------------------------------------------- |
| Identifiant personnalisé          | Fournit un identifiant personnalisé                                           |
| Description        | Fournit des informations ou un contexte supplémentaires                             |
| Signification            | Fournit la signification ou l'intention du texte dans le contexte spécifique |

Pour en savoir plus sur les identifiants personnalisés, consultez [Gérer le texte marqué avec des identifiants personnalisés][GuideI18nOptionalManageMarkedText].

### Ajouter des descriptions et des significations utiles {#add-helpful-descriptions-and-meanings}

Pour traduire un message texte avec précision, fournissez des informations ou un contexte supplémentaires au traducteur.

Ajoutez une _description_ du message texte comme valeur de l'attribut `i18n` ou de la chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize].

L'exemple suivant montre la valeur de l'attribut `i18n`.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-attribute-desc"/>

L'exemple suivant montre la valeur de la chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize] avec une description.

```ts
$localize`:An introduction header for this sample:Hello i18n!`;
```

Le traducteur peut aussi avoir besoin de connaître la signification ou l'intention du message texte dans le contexte particulier de cette application, afin de le traduire de la même façon que d'autres textes ayant la même signification.
Commencez la valeur de l'attribut `i18n` par la _signification_ et séparez-la de la _description_ avec le caractère `|` : `{meaning}|{description}`.

#### Exemple `h1` {#h1-example}

Par exemple, vous pouvez vouloir préciser que la balise `<h1>` est un en-tête de site qui doit être traduit de la même façon, qu'il soit utilisé comme en-tête ou référencé dans une autre section du texte.

L'exemple suivant montre comment préciser que la balise `<h1>` doit être traduite comme un en-tête ou référencée ailleurs.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-attribute-meaning"/>

Résultat : tout texte marqué avec `site header` comme _signification_ est traduit exactement de la même façon.

L'exemple de code suivant montre la valeur de la chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize] avec une signification et une description.

```ts
$localize`:site header|An introduction header for this sample:Hello i18n!`;
```

<docs-callout title="Comment les significations contrôlent l'extraction et la fusion du texte">

L'outil d'extraction d'Angular génère une entrée d'unité de traduction pour chaque attribut `i18n` d'un template.
L'outil d'extraction d'Angular attribue à chaque unité de traduction un identifiant unique fondé sur son texte source et sa _signification_. La _description_ n'affecte pas l'identifiant.

HELPFUL: Pour en savoir plus sur l'outil d'extraction d'Angular, consultez [Travailler avec les fichiers de traduction](guide/i18n/translation-files).

Des éléments de texte identiques avec des _significations_ différentes sont extraits avec des identifiants différents.
Par exemple, si le mot « right » est utilisé avec les deux définitions suivantes à deux endroits différents, le mot est traduit différemment et réintégré dans l'application sous forme d'entrées de traduction distinctes.

- `correct` comme dans « you are right »
- `direction` comme dans « turn right »

Si des éléments de texte identiques remplissent les conditions suivantes, ils ne sont extraits qu'une seule fois et utilisent le même identifiant.

- Même signification ou définition
- Descriptions différentes

Cette unique entrée de traduction est réintégrée dans l'application partout où ces éléments de texte identiques apparaissent.

</docs-callout>

## Expressions ICU {#icu-expressions}

Les expressions ICU vous aident à marquer du texte alternatif dans les templates des composants selon des conditions.
Une expression ICU comprend une propriété de composant, une clause ICU et les instructions case, entourées d'une accolade ouvrante \(`{`\) et d'une accolade fermante \(`}`\).

```html
{ component_property, icu_clause, case_statements }
```

La propriété du composant définit la variable.
Une clause ICU définit le type de texte conditionnel.

| Clause ICU                                                           | Détails                                                             |
| :------------------------------------------------------------------- | :------------------------------------------------------------------ |
| [`plural`][GuideI18nCommonPrepareMarkPlurals]                        | Marque l'utilisation de nombres au pluriel                                      |
| [`select`][GuideI18nCommonPrepareMarkAlternatesAndNestedExpressions] | Marque des choix de texte alternatif selon les valeurs de chaîne que vous définissez |

Pour simplifier la traduction, utilisez les clauses International Components for Unicode \(clauses ICU\) avec des expressions régulières.

HELPFUL: Les clauses ICU respectent le [format de message ICU][GithubUnicodeOrgIcuUserguideFormatParseMessages] spécifié dans les [règles de pluralisation du CLDR][UnicodeCldrIndexCldrSpecPluralRules].

### Marquer les pluriels {#mark-plurals}

Les langues ont des règles de pluralisation différentes, ce qui rend la traduction plus difficile.
Comme les autres locales expriment la cardinalité différemment, vous devrez peut-être définir des catégories de pluralisation qui ne correspondent pas à celles de l'anglais.
Utilisez la clause `plural` pour marquer les expressions qui risquent de ne pas avoir de sens si elles sont traduites mot à mot.

```html
{ component_property, plural, pluralization_categories }
```

Après la catégorie de pluralisation, saisissez le texte par défaut \(en anglais\) entouré d'une accolade ouvrante \(`{`\) et d'une accolade fermante \(`}`\).

```html
pluralization_category { }
```

Les catégories de pluralisation suivantes sont disponibles pour l'anglais et peuvent changer selon la locale.

| Catégorie de pluralisation | Détails                    | Exemple                    |
| :--------------------- | :------------------------- | :------------------------- |
| `zero`                 | La quantité est zéro           | `=0 { }` <br /> `zero { }` |
| `one`                  | La quantité est 1              | `=1 { }` <br /> `one { }`  |
| `two`                  | La quantité est 2              | `=2 { }` <br /> `two { }`  |
| `few`                  | La quantité est 2 ou plus      | `few { }`                  |
| `many`                 | La quantité est un grand nombre | `many { }`                 |
| `other`                | La quantité par défaut       | `other { }`                |

Si aucune des catégories de pluralisation ne correspond, Angular utilise `other`, qui correspond au repli standard pour une catégorie manquante.

```html
other { default_quantity }
```

HELPFUL: Pour en savoir plus sur les catégories de pluralisation, consultez [Choosing plural category names][UnicodeCldrIndexCldrSpecPluralRulesTocChoosingPluralCategoryNames] dans le [CLDR - Unicode Common Locale Data Repository][UnicodeCldrMain].

<docs-callout title="Contexte : certaines locales ne prennent pas en charge certaines catégories de pluralisation">

De nombreuses locales ne prennent pas en charge certaines catégories de pluralisation.
La locale par défaut \(`en-US`\) utilise une fonction `plural()` très simple qui ne prend pas en charge la catégorie de pluralisation `few`.
Une autre locale avec une fonction `plural()` simple est `es`.
L'exemple de code suivant montre la fonction [`plural()` de en-US][GithubAngularAngularBlobEcffc3557fe1bff9718c01277498e877ca44588dPackagesCoreSrcI18nLocaleEnTsL14L18].

<docs-code path="adev/src/content/examples/i18n/doc-files/locale_plural_function.ts" class="no-box" hideCopy/>

La fonction `plural()` ne renvoie que 1 \(`one`\) ou 5 \(`other`\).
La catégorie `few` ne correspond jamais.

</docs-callout>

#### Exemple `minutes` {#minutes-example}

Supposons que vous vouliez afficher la phrase suivante en anglais, où `x` est un nombre.

<!--todo: replace output docs-code with screen capture image --->

```html
updated x minutes ago
```

Et que vous vouliez aussi afficher les phrases suivantes selon la cardinalité de `x`.

<!--todo: replace output docs-code with screen capture image --->

```html
updated just now
```

<!--todo: replace output docs-code with screen capture image --->

```html
updated one minute ago
```

Utilisez du balisage HTML et des [interpolations](guide/templates/binding#render-dynamic-text-with-text-interpolation).
L'exemple de code suivant montre comment utiliser la clause `plural` pour exprimer les trois situations précédentes dans un élément `<span>`.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/src/app/app.component.html" region="i18n-plural"/>

Examinez les détails suivants de l'exemple de code précédent.

| Paramètres                        | Détails                                                                                                               |
| :-------------------------------- | :-------------------------------------------------------------------------------------------------------------------- |
| `minutes`                         | Le premier paramètre indique que la propriété du composant est `minutes` et détermine le nombre de minutes.               |
| `plural`                          | Le deuxième paramètre indique que la clause ICU est `plural`.                                                            |
| `=0 {just now}`                   | Pour zéro minute, la catégorie de pluralisation est `=0`. La valeur est `just now`.                                        |
| `=1 {one minute}`                 | Pour une minute, la catégorie de pluralisation est `=1`. La valeur est `one minute`.                                        |
| `other {{{minutes}} minutes ago}` | Pour toute cardinalité sans correspondance, la catégorie de pluralisation par défaut est `other`. La valeur est `{{minutes}} minutes ago`. |

`{{minutes}}` est une [interpolation](guide/templates/binding#render-dynamic-text-with-text-interpolation).

### Marquer les alternatives et les expressions imbriquées {#mark-alternates-and-nested-expressions}

La clause `select` marque des choix de texte alternatif selon les valeurs de chaîne que vous définissez.

```html
{ component_property, select, selection_categories }
```

Traduisez toutes les alternatives pour afficher un texte alternatif selon la valeur d'une variable.

Après la catégorie de sélection, saisissez le texte \(en anglais\) entouré d'une accolade ouvrante \(`{`\) et d'une accolade fermante \(`}`\).

```html
selection_category { text }
```

Les locales ont des constructions grammaticales différentes, ce qui rend la traduction plus difficile.
Utilisez du balisage HTML.
Si aucune des catégories de sélection ne correspond, Angular utilise `other`, qui correspond au repli standard pour une catégorie manquante.

```html
other { default_value }
```

#### Exemple `gender` {#gender-example}

Supposons que vous vouliez afficher la phrase suivante en anglais.

<!--todo: replace output docs-code with screen capture image --->

```html
The author is other
```

Et que vous vouliez aussi afficher les phrases suivantes selon la propriété `gender` du composant.

<!--todo: replace output docs-code with screen capture image --->

```html
The author is female
```

<!--todo: replace output docs-code with screen capture image --->

```html
The author is male
```

L'exemple de code suivant montre comment lier la propriété `gender` du composant et utiliser la clause `select` pour exprimer les trois situations précédentes dans un élément `<span>`.

La propriété `gender` associe les sorties à chacune des valeurs de chaîne suivantes.

| Valeur  | Valeur en anglais |
| :----- | :------------ |
| female | `female`      |
| male   | `male`        |
| other  | `other`       |

La clause `select` associe les valeurs aux traductions appropriées.
L'exemple de code suivant montre la propriété `gender` utilisée avec la clause select.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/src/app/app.component.html" region="i18n-select"/>

#### Exemple `gender` et `minutes` {#gender-and-minutes-example}

Combinez différentes clauses, comme les clauses `plural` et `select`.
L'exemple de code suivant montre des clauses imbriquées fondées sur les exemples `gender` et `minutes`.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/src/app/app.component.html" region="i18n-nested"/>

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/i18n/translation-files" title="Travailler avec les fichiers de traduction"/>
</docs-pill-row>

[ApiLocalizeInitLocalize]: api/localize/init/$localize '$localize | init - localize - API  | Angular'
[GuideI18nCommonPrepareMarkAlternatesAndNestedExpressions]: guide/i18n/prepare#mark-alternates-and-nested-expressions 'Mark alternates and nested expressions - Prepare templates for translation | Angular'
[GuideI18nCommonPrepareMarkPlurals]: guide/i18n/prepare#mark-plurals 'Mark plurals - Prepare component for translation | Angular'
[GuideI18nOptionalManageMarkedText]: guide/i18n/manage-marked-text 'Manage marked text with custom IDs | Angular'
[GithubAngularAngularBlobEcffc3557fe1bff9718c01277498e877ca44588dPackagesCoreSrcI18nLocaleEnTsL14L18]: https://github.com/angular/angular/blob/ecffc3557fe1bff9718c01277498e877ca44588d/packages/core/src/i18n/locale_en.ts#L14-L18 'Line 14 to 18 - angular/packages/core/src/i18n/locale_en.ts | angular/angular | GitHub'
[GithubUnicodeOrgIcuUserguideFormatParseMessages]: https://unicode-org.github.io/icu/userguide/format_parse/messages 'ICU Message Format - ICU Documentation | Unicode | GitHub'
[UnicodeCldrMain]: https://cldr.unicode.org 'Unicode CLDR Project'
[UnicodeCldrIndexCldrSpecPluralRules]: https://cldr.unicode.org/index/cldr-spec/plural-rules 'Plural Rules | CLDR - Unicode Common Locale Data Repository | Unicode'
[UnicodeCldrIndexCldrSpecPluralRulesTocChoosingPluralCategoryNames]: https://cldr.unicode.org/index/cldr-spec/plural-rules#TOC-Choosing-Plural-Category-Names 'Choosing Plural Category Names - Plural Rules | CLDR - Unicode Common Locale Data Repository | Unicode'
