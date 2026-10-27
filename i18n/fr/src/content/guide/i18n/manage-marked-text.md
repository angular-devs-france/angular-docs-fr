<!-- source: 453f583a331a501e99dce0a61d956cdae4d0b20c -->
# Gérer le texte marqué avec des identifiants personnalisés

L'extracteur Angular génère un fichier avec une entrée d'unité de traduction pour chacune des occurrences suivantes.

- Chaque attribut `i18n` dans le template d'un composant
- Chaque chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize] dans le code d'un composant

Comme décrit dans [Comment les significations contrôlent l'extraction et la fusion du texte][GuideI18nCommonPrepareHowMeaningsControlTextExtractionAndMerges], Angular attribue à chaque unité de traduction un identifiant unique.

L'exemple suivant montre des unités de traduction avec des identifiants uniques.

<docs-code header="messages.fr.xlf" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="generated-id"/>

Lorsque vous modifiez le texte traduisible, l'extracteur génère un nouvel identifiant pour cette unité de traduction.
Dans la plupart des cas, une modification du texte source exige aussi une modification de la traduction.
Ainsi, l'utilisation d'un nouvel identifiant maintient la modification du texte synchronisée avec les traductions.

Cependant, certains systèmes de traduction exigent une forme ou une syntaxe spécifique pour l'identifiant.
Pour répondre à cette exigence, utilisez un identifiant personnalisé pour marquer le texte.
La plupart des développeurs n'ont pas besoin d'utiliser d'identifiant personnalisé.
Si vous voulez utiliser une syntaxe unique pour transmettre des métadonnées supplémentaires, utilisez un identifiant personnalisé.
Les métadonnées supplémentaires peuvent inclure la library, le composant ou la zone de l'application dans laquelle le texte apparaît.

Pour spécifier un identifiant personnalisé dans l'attribut `i18n` ou dans une chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize], utilisez le préfixe `@@`.
L'exemple suivant définit l'identifiant personnalisé `introductionHeader` dans un élément de titre.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-attribute-solo-id"/>

L'exemple suivant définit l'identifiant personnalisé `introductionHeader` pour une variable.

```ts
variableText1 = $localize`:@@introductionHeader:Hello i18n!`;
```

Lorsque vous spécifiez un identifiant personnalisé, l'extracteur génère une unité de traduction avec cet identifiant personnalisé.

<docs-code header="messages.fr.xlf" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="custom-id"/>

Si vous modifiez le texte, l'extracteur ne modifie pas l'identifiant.
Vous n'avez donc pas à effectuer l'étape supplémentaire de mise à jour de la traduction.
L'inconvénient des identifiants personnalisés est que, si vous modifiez le texte, votre traduction risque de ne plus être synchronisée avec le texte source nouvellement modifié.

## Utiliser un identifiant personnalisé avec une description {#use-a-custom-id-with-a-description}

Utilisez un identifiant personnalisé combiné à une description et à une signification pour aider davantage le traducteur.

L'exemple suivant inclut une description, suivie de l'identifiant personnalisé.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-attribute-id"/>

L'exemple suivant définit l'identifiant personnalisé `introductionHeader` et une description pour une variable.

```ts
variableText2 = $localize`:An introduction header for this sample@@introductionHeader:Hello i18n!`;
```

L'exemple suivant ajoute une signification.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-attribute-meaning-and-id"/>

L'exemple suivant définit l'identifiant personnalisé `introductionHeader` pour une variable.

```ts
variableText3 = $localize`:site header|An introduction header for this sample@@introductionHeader:Hello i18n!`;
```

### Définir des identifiants personnalisés uniques {#define-unique-custom-ids}

Veillez à définir des identifiants personnalisés uniques.
Si vous utilisez le même identifiant pour deux éléments de texte différents, l'outil d'extraction n'extrait que le premier, et Angular utilise la traduction à la place des deux éléments de texte d'origine.

Par exemple, dans l'extrait de code suivant, le même identifiant personnalisé `myId` est défini pour deux éléments de texte différents.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/doc-files/app.component.html" region="i18n-duplicate-custom-id"/>

L'exemple suivant montre la traduction en français.

<docs-code header="src/locale/messages.fr.xlf" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="i18n-duplicate-custom-id"/>

Les deux éléments utilisent désormais la même traduction \(`Bonjour`\), car tous deux ont été définis avec le même identifiant personnalisé.

<docs-code path="adev/src/content/examples/i18n/doc-files/rendered-output.html"/>

[ApiLocalizeInitLocalize]: api/localize/init/$localize '$localize | init - localize - API | Angular'
[GuideI18nCommonPrepareHowMeaningsControlTextExtractionAndMerges]: guide/i18n/prepare#h1-example 'How meanings control text extraction and merges - Prepare components for translations | Angular'
