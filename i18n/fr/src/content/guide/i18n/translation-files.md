<!-- source: d082942d778d77771ff18f15bb505ce30188b93b -->
# Travailler avec les fichiers de traduction

Après avoir préparé un composant pour la traduction, utilisez la commande [`extract-i18n`][CliExtractI18n] de l'[Angular CLI][CliMain] pour extraire le texte marqué du composant dans un fichier de _langue source_.

Le texte marqué comprend le texte marqué avec `i18n`, les attributs marqués avec `i18n-`_attribut_ et le texte balisé avec `$localize`, comme décrit dans [Préparer un composant pour la traduction][GuideI18nCommonPrepare].

Suivez les étapes ci-dessous pour créer et mettre à jour les fichiers de traduction de votre projet.

1. [Extrayez le fichier de langue source][GuideI18nCommonTranslationFilesExtractTheSourceLanguageFile].
   1. Si vous le souhaitez, changez l'emplacement, le format et le nom.
1. Copiez le fichier de langue source pour [créer un fichier de traduction pour chaque langue][GuideI18nCommonTranslationFilesCreateATranslationFileForEachLanguage].
1. [Traduisez chaque fichier de traduction][GuideI18nCommonTranslationFilesTranslateEachTranslationFile].
1. Traduisez séparément les pluriels et les expressions alternatives.
   1. [Traduisez les pluriels][GuideI18nCommonTranslationFilesTranslatePlurals].
   1. [Traduisez les expressions alternatives][GuideI18nCommonTranslationFilesTranslateAlternateExpressions].
   1. [Traduisez les expressions imbriquées][GuideI18nCommonTranslationFilesTranslateNestedExpressions].

## Extraire le fichier de langue source {#extract-the-source-language-file}

Pour extraire le fichier de langue source, effectuez les actions suivantes.

1. Ouvrez une fenêtre de terminal.
1. Placez-vous dans le répertoire racine de votre projet.
1. Exécutez la commande CLI suivante.

<docs-code path="adev/src/content/examples/i18n/doc-files/commands.sh" region="extract-i18n-default"/>

La commande `extract-i18n` crée un fichier de langue source nommé `messages.xlf` dans le répertoire racine de votre projet.
Pour en savoir plus sur le XML Localization Interchange File Format \(XLIFF, version 1.2\), consultez [XLIFF][WikipediaWikiXliff].

Utilisez les options suivantes de la commande [`extract-i18n`][CliExtractI18n] pour changer l'emplacement, le format et le nom du fichier de langue source.

| Option de commande  | Détails                              |
| :-------------- | :----------------------------------- |
| `--format`      | Définit le format du fichier de sortie    |
| `--out-file`    | Définit le nom du fichier de sortie      |
| `--output-path` | Définit le chemin du répertoire de sortie |

### Changer l'emplacement du fichier de langue source {#change-the-source-language-file-location}

Pour créer un fichier dans le répertoire `src/locale`, spécifiez le chemin de sortie en option.

#### Exemple `extract-i18n --output-path` {#extract-i18n---output-path-example}

L'exemple suivant spécifie le chemin de sortie en option.

<docs-code path="adev/src/content/examples/i18n/doc-files/commands.sh" region="extract-i18n-output-path"/>

### Changer le format du fichier de langue source {#change-the-source-language-file-format}

La commande `extract-i18n` crée des fichiers dans les formats de traduction suivants.

| Format de traduction | Détails                                                                                                          | Extension de fichier    |
| :----------------- | :--------------------------------------------------------------------------------------------------------------- | :---------------- |
| ARB                | [Application Resource Bundle][GithubGoogleAppResourceBundleWikiApplicationresourcebundlespecification]           | `.arb`            |
| JSON               | [JavaScript Object Notation][JsonMain]                                                                           | `.json`           |
| XLIFF 1.2          | [XML Localization Interchange File Format, version 1.2][OasisOpenDocsXliffXliffCoreXliffCoreHtml]                | `.xlf`            |
| XLIFF 2            | [XML Localization Interchange File Format, version 2][OasisOpenDocsXliffXliffCoreV20Cos01XliffCoreV20Cose01Html] | `.xlf`            |
| XMB                | [XML Message Bundle][UnicodeCldrDevelopmentDevelopmentProcessDesignProposalsXmb]                                 | `.xmb` \(`.xtb`\) |

Spécifiez explicitement le format de traduction avec l'option de commande `--format`.

HELPFUL: Le format XMB génère des fichiers de langue source `.xmb`, mais utilise des fichiers de traduction `.xtb`.

#### Exemple `extract-i18n --format` {#extract-i18n---format-example}

L'exemple suivant présente plusieurs formats de traduction.

<docs-code path="adev/src/content/examples/i18n/doc-files/commands.sh" region="extract-i18n-formats"/>

### Changer le nom du fichier de langue source {#change-the-source-language-file-name}

Pour changer le nom du fichier de langue source généré par l'outil d'extraction, utilisez l'option de commande `--out-file`.

#### Exemple `extract-i18n --out-file` {#extract-i18n---out-file-example}

L'exemple suivant montre comment nommer le fichier de sortie.

<docs-code path="adev/src/content/examples/i18n/doc-files/commands.sh" region="extract-i18n-out-file"/>

## Créer un fichier de traduction pour chaque langue {#create-a-translation-file-for-each-language}

Pour créer un fichier de traduction pour une locale ou une langue, effectuez les actions suivantes.

1. [Extrayez le fichier de langue source][GuideI18nCommonTranslationFilesExtractTheSourceLanguageFile].
1. Faites une copie du fichier de langue source pour créer un fichier de _traduction_ pour chaque langue.
1. Renommez le fichier de _traduction_ pour y ajouter la locale.

   ```file {hideCopy}

   messages.xlf --> messages.{locale}.xlf

   ```

1. Créez un nouveau répertoire nommé `locale` à la racine de votre projet.

   ```file {hideCopy}

   src/locale

   ```

1. Déplacez le fichier de _traduction_ dans le nouveau répertoire.
1. Envoyez le fichier de _traduction_ à votre traducteur.
1. Répétez les étapes ci-dessus pour chaque langue que vous voulez ajouter à votre application.

### Exemple `extract-i18n` pour le français {#extract-i18n-example-for-french}

Par exemple, pour créer un fichier de traduction français, effectuez les actions suivantes.

1. Exécutez la commande `extract-i18n`.
1. Faites une copie du fichier de langue source `messages.xlf`.
1. Renommez la copie en `messages.fr.xlf` pour la traduction en français \(`fr`\).
1. Déplacez le fichier de traduction `fr` dans le répertoire `src/locale`.
1. Envoyez le fichier de traduction `fr` au traducteur.

## Traduire chaque fichier de traduction {#translate-each-translation-file}

À moins que vous ne maîtrisiez la langue et ayez le temps de modifier les traductions, vous suivrez probablement les étapes suivantes.

1. Envoyez chaque fichier de traduction à un traducteur.
1. Le traducteur utilise un éditeur de fichiers XLIFF pour effectuer les actions suivantes.
   1. Créer la traduction.
   1. Modifier la traduction.

### Exemple de processus de traduction pour le français {#translation-process-example-for-french}

Pour illustrer le processus, examinez le fichier `messages.fr.xlf` de l'[exemple d'application Angular internationalisée][GuideI18nExample]. L'[exemple d'application Angular internationalisée][GuideI18nExample] inclut une traduction française que vous pouvez modifier sans éditeur XLIFF particulier ni connaissance du français.

Les actions suivantes décrivent le processus de traduction pour le français.

1. Ouvrez `messages.fr.xlf` et trouvez le premier élément `<trans-unit>`.
   Il s'agit d'une _unité de traduction_, aussi appelée _nœud de texte_, qui représente la traduction de la balise de salutation `<h1>` marquée précédemment avec l'attribut `i18n`.

   <docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translated-hello-before"/>

   `id="introductionHeader"` est un [identifiant personnalisé][GuideI18nOptionalManageMarkedText], mais sans le préfixe `@@` requis dans le HTML source.

1. Dupliquez l'élément `<source>... </source>` dans le nœud de texte, renommez-le en `target`, puis remplacez son contenu par le texte français.

   <docs-code header="src/locale/messages.fr.xlf (<trans-unit>, après traduction)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translated-hello"/>

   Dans une traduction plus complexe, les informations et le contexte fournis par les [éléments de description et de signification][GuideI18nCommonPrepareAddHelpfulDescriptionsAndMeanings] vous aident à choisir les bons mots pour la traduction.

1. Traduisez les autres nœuds de texte.
   L'exemple suivant montre comment traduire.

   <docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translated-other-nodes"/>

IMPORTANT: Ne modifiez pas les identifiants des unités de traduction.
Chaque attribut `id` est généré par Angular et dépend du contenu du texte du composant et de la signification attribuée.

Si vous modifiez le texte ou la signification, l'attribut `id` change.
Pour en savoir plus sur la gestion des modifications de texte et des identifiants, consultez [identifiants personnalisés][GuideI18nOptionalManageMarkedText].

## Traduire les pluriels {#translate-plurals}

Ajoutez ou supprimez des cas de pluriel selon les besoins de chaque langue.

HELPFUL: Pour les règles de pluriel des langues, consultez [les règles de pluriel du CLDR][GithubUnicodeOrgCldrStagingChartsLatestSupplementalLanguagePluralRulesHtml].

### Exemple `minute` `plural` {#minute-plural-example}

Pour traduire un `plural`, traduisez les valeurs de correspondance du format ICU.

- `just now`
- `one minute ago`
- `<x id="INTERPOLATION" equiv-text="{{minutes}}"/> minutes ago`

L'exemple suivant montre comment traduire.

<docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translated-plural"/>

## Traduire les expressions alternatives {#translate-alternate-expressions}

Angular extrait aussi les expressions ICU alternatives `select` sous forme d'unités de traduction distinctes.

### Exemple `gender` `select` {#gender-select-example}

L'exemple suivant montre une expression ICU `select` dans le template du composant.

<docs-code header="app.component.html" path="adev/src/content/examples/i18n/src/app/app.component.html" region="i18n-select"/>

Dans cet exemple, Angular extrait l'expression en deux unités de traduction.
La première contient le texte situé hors de la clause `select` et utilise un placeholder pour `select` \(`<x id="ICU">`\) :

<docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translate-select-1"/>

IMPORTANT: Lorsque vous traduisez le texte, déplacez le placeholder si nécessaire, mais ne le supprimez pas.
Si vous supprimez le placeholder, l'expression ICU est supprimée de votre application traduite.

L'exemple suivant montre la seconde unité de traduction, qui contient la clause `select`.

<docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translate-select-2"/>

L'exemple suivant montre les deux unités de traduction une fois la traduction terminée.

<docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translated-select"/>

## Traduire les expressions imbriquées {#translate-nested-expressions}

Angular traite une expression imbriquée de la même manière qu'une expression alternative.
Angular extrait l'expression en deux unités de traduction.

### Exemple de `plural` imbriqué {#nested-plural-example}

L'exemple suivant montre la première unité de traduction, qui contient le texte situé hors de l'expression imbriquée.

<docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translate-nested-1"/>

L'exemple suivant montre la seconde unité de traduction, qui contient l'expression imbriquée complète.

<docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translate-nested-2"/>

L'exemple suivant montre les deux unités de traduction après traduction.

<docs-code header="src/locale/messages.fr.xlf (<trans-unit>)" path="adev/src/content/examples/i18n/doc-files/messages.fr.xlf" region="translate-nested"/>

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/i18n/merge" title="Fusionner les traductions dans l'application"/>
</docs-pill-row>

[CliMain]: cli 'CLI Overview and Command Reference | Angular'
[CliExtractI18n]: cli/extract-i18n 'ng extract-i18n | CLI | Angular'
[GuideI18nCommonPrepare]: guide/i18n/prepare 'Prepare component for translation | Angular'
[GuideI18nCommonPrepareAddHelpfulDescriptionsAndMeanings]: guide/i18n/prepare#add-helpful-descriptions-and-meanings 'Add helpful descriptions and meanings - Prepare component for translation | Angular'
[GuideI18nCommonTranslationFilesCreateATranslationFileForEachLanguage]: guide/i18n/translation-files#create-a-translation-file-for-each-language 'Create a translation file for each language - Work with translation files | Angular'
[GuideI18nCommonTranslationFilesExtractTheSourceLanguageFile]: guide/i18n/translation-files#extract-the-source-language-file 'Extract the source language file - Work with translation files | Angular'
[GuideI18nCommonTranslationFilesTranslateAlternateExpressions]: guide/i18n/translation-files#translate-alternate-expressions 'Translate alternate expressions - Work with translation files | Angular'
[GuideI18nCommonTranslationFilesTranslateEachTranslationFile]: guide/i18n/translation-files#translate-each-translation-file 'Translate each translation file - Work with translation files | Angular'
[GuideI18nCommonTranslationFilesTranslateNestedExpressions]: guide/i18n/translation-files#translate-nested-expressions 'Translate nested expressions - Work with translation files | Angular'
[GuideI18nCommonTranslationFilesTranslatePlurals]: guide/i18n/translation-files#translate-plurals 'Translate plurals - Work with translation files | Angular'
[GuideI18nExample]: guide/i18n/example 'Example Angular Internationalization application | Angular'
[GuideI18nOptionalManageMarkedText]: guide/i18n/manage-marked-text 'Manage marked text with custom IDs | Angular'
[GithubGoogleAppResourceBundleWikiApplicationresourcebundlespecification]: https://github.com/google/app-resource-bundle/wiki/ApplicationResourceBundleSpecification 'ApplicationResourceBundleSpecification | google/app-resource-bundle | GitHub'
[GithubUnicodeOrgCldrStagingChartsLatestSupplementalLanguagePluralRulesHtml]: https://cldr.unicode.org/index/cldr-spec/plural-rules 'Language Plural Rules - CLDR Charts | Unicode | GitHub'
[JsonMain]: https://www.json.org 'Introducing JSON | JSON'
[OasisOpenDocsXliffXliffCoreXliffCoreHtml]: https://docs.oasis-open.org/xliff/v1.2/os/xliff-core.html 'XLIFF Version 1.2 Specification | Oasis Open Docs'
[OasisOpenDocsXliffXliffCoreV20Cos01XliffCoreV20Cose01Html]: https://docs.oasis-open.org/xliff/xliff-core/v2.0/cos01/xliff-core-v2.0-cos01.html 'XLIFF Version 2.0 | Oasis Open Docs'
[UnicodeCldrDevelopmentDevelopmentProcessDesignProposalsXmb]: https://cldr.unicode.org/development/development-process/design-proposals/xmb 'XMB | CLDR - Unicode Common Locale Data Repository | Unicode'
[WikipediaWikiXliff]: https://en.wikipedia.org/wiki/XLIFF 'XLIFF | Wikipedia'
