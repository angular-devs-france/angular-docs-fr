<!-- source: b592eb1cd64a1a294038bf3af4116d3b443b192e -->
# Fusionner les traductions dans l'application

Pour fusionner les traductions terminées dans votre projet, effectuez les actions suivantes

1. Utilisez l'[Angular CLI][CliMain] pour construire une copie des fichiers distribuables de votre projet
1. Utilisez l'option `"localize"` pour remplacer tous les messages i18n par les traductions valides et construire une variante localisée de l'application.
   Une variante de l'application est une copie complète des fichiers distribuables de votre application, traduite pour une seule locale.

Après avoir fusionné les traductions, servez chaque copie distribuable de l'application en utilisant la détection de la langue côté serveur ou des sous-répertoires différents.

HELPFUL: Pour en savoir plus sur la façon de servir chaque copie distribuable de l'application, consultez [déployer plusieurs locales](guide/i18n/deploy).

Pour une traduction de l'application à la compilation, le processus de build utilise la compilation ahead-of-time (AOT) pour produire une application petite, rapide et prête à l'exécution.

HELPFUL: Pour une explication détaillée du processus de build, consultez [Construire et servir des applications Angular][GuideBuild].
Le processus de build fonctionne avec les fichiers de traduction au format `.xlf` ou dans un autre format qu'Angular comprend, comme `.xtb`.
Pour en savoir plus sur les formats de fichiers de traduction utilisés par Angular, consultez [Changer le format du fichier de langue source][GuideI18nCommonTranslationFilesChangeTheSourceLanguageFileFormat]

Pour construire une copie distribuable distincte de l'application pour chaque locale, [définissez les locales dans la configuration de build][GuideI18nCommonMergeDefineLocalesInTheBuildConfiguration] dans le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig] de votre projet.

Cette méthode raccourcit le processus de build en supprimant la nécessité d'effectuer un build complet de l'application pour chaque locale.

Pour [générer des variantes de l'application pour chaque locale][GuideI18nCommonMergeGenerateApplicationVariantsForEachLocale], utilisez l'option `"localize"` dans le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig].
De plus, pour [construire depuis la ligne de commande][GuideI18nCommonMergeBuildFromTheCommandLine], utilisez la commande [`build`][CliBuild] de l'[Angular CLI][CliMain] avec l'option `--localize`.

HELPFUL: Vous pouvez éventuellement [appliquer des options de build spécifiques à une seule locale][GuideI18nCommonMergeApplySpecificBuildOptionsForJustOneLocale] pour une configuration de locale personnalisée.

## Définir les locales dans la configuration de build {#define-locales-in-the-build-configuration}

Utilisez l'option de projet `i18n` dans le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig] de votre projet pour définir les locales d'un projet.

Les sous-options suivantes identifient la langue source et indiquent au compilateur où trouver les traductions prises en charge pour le projet.

| Sous-option      | Détails                                                                                                                                                |
| :------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------- |
| `sourceLocale` | La locale que vous utilisez dans le code source de l'application \(`en-US` par défaut\). Peut aussi être un objet avec les propriétés `code`, `baseHref` et `subPath`. |
| `locales`      | Une map associant des identifiants de locale à des fichiers de traduction. Chaque entrée peut aussi être un objet avec les propriétés `translation`, `baseHref` et `subPath`.           |

Pour la liste complète des propriétés `i18n` et de leurs types, consultez [Options i18n][GuideWorkspaceConfigI18n].

### Exemple d'`angular.json` pour `en-US` et `fr` {#angularjson-for-en-us-and-fr-example}

Par exemple, l'extrait suivant d'un fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig] définit la locale source sur `en-US` et fournit le chemin du fichier de traduction de la locale française \(`fr`\).

<docs-code header="angular.json" path="adev/src/content/examples/i18n/angular.json" region="locale-config"/>

## Générer des variantes de l'application pour chaque locale {#generate-application-variants-for-each-locale}

Pour utiliser votre définition de locales dans la configuration de build, utilisez l'option `"localize"` dans le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig] pour indiquer à la CLI quelles locales générer pour la configuration de build.

- Définissez `"localize"` sur `true` pour toutes les locales définies au préalable dans la configuration de build.
- Définissez `"localize"` sur un tableau contenant un sous-ensemble des identifiants de locale définis au préalable pour ne construire que les versions de ces locales.
- Définissez `"localize"` sur `false` pour désactiver la localisation et ne générer aucune version spécifique à une locale.

HELPFUL: La compilation ahead-of-time (AOT) est requise pour localiser les templates des composants.

Si vous avez modifié ce paramètre, définissez `"aot"` sur `true` pour utiliser l'AOT.

HELPFUL: En raison de la complexité du déploiement de l'i18n et de la nécessité de réduire le temps de rebuild, le serveur de développement ne prend en charge la localisation que d'une seule locale à la fois.
Si vous définissez l'option `"localize"` sur `true` ou sur un tableau de plusieurs locales et que vous utilisez `ng serve`, le serveur de développement affiche un avertissement et désactive la localisation.
Si vous voulez développer avec une locale spécifique, définissez l'option `"localize"` sur cette locale.
Par exemple, pour le français \(`fr`\), spécifiez `"localize": ["fr"]`.

La CLI charge et enregistre les données de locale, place chaque version générée dans un répertoire propre à la locale pour la séparer des versions des autres locales, et place ces répertoires dans le `outputPath` configuré pour le projet.
Pour chaque variante de l'application, l'attribut `lang` de l'élément `html` est défini sur la locale.
La CLI ajuste aussi le HREF de base HTML de chaque version de l'application en ajoutant la locale au `baseHref` configuré.

Définissez la propriété `"localize"` dans une configuration partagée pour qu'elle soit héritée par toutes les configurations.
Définissez aussi la propriété pour remplacer d'autres configurations.

### Exemple d'`angular.json` incluant toutes les locales du build {#angularjson-include-all-locales-from-build-example}

L'exemple suivant montre l'option `"localize"` définie sur `true` dans le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig], de sorte que toutes les locales définies dans la configuration de build soient construites.

<docs-code header="angular.json" path="adev/src/content/examples/i18n/angular.json" region="build-localize-true"/>

## Construire depuis la ligne de commande {#build-from-the-command-line}

Vous pouvez aussi utiliser l'option `--localize` avec la commande [`ng build`][CliBuild] et votre configuration `production` existante.
La CLI construit toutes les locales définies dans la configuration de build.
Si vous définissez les locales dans la configuration de build, cela revient à définir l'option `"localize"` sur `true`.

HELPFUL: Pour en savoir plus sur la façon de définir les locales, consultez [Générer des variantes de l'application pour chaque locale][GuideI18nCommonMergeGenerateApplicationVariantsForEachLocale].

<docs-code path="adev/src/content/examples/i18n/doc-files/commands.sh" region="build-localize"/>

## Appliquer des options de build spécifiques à une seule locale {#apply-specific-build-options-for-just-one-locale}

Pour appliquer des options de build spécifiques à une seule locale, spécifiez une seule locale afin de créer une configuration personnalisée propre à cette locale.

IMPORTANT: N'utilisez le serveur de développement de l'[Angular CLI][CliMain] \(`ng serve`\) qu'avec une seule locale.

### Exemple de build pour le français {#build-for-french-example}

L'exemple suivant montre une configuration personnalisée propre à une locale, utilisant une seule locale.

<docs-code header="angular.json" path="adev/src/content/examples/i18n/angular.json" region="build-single-locale"/>

Passez cette configuration aux commandes `ng serve` ou `ng build`.
L'exemple de code suivant montre comment servir le fichier de langue française.

<docs-code path="adev/src/content/examples/i18n/doc-files/commands.sh" region="serve-french"/>

Pour les builds de production, utilisez la composition de configurations pour exécuter les deux configurations.

<docs-code path="adev/src/content/examples/i18n/doc-files/commands.sh" region="build-production-french"/>

<docs-code header="angular.json" path="adev/src/content/examples/i18n/angular.json" region="build-production-french" />

## Signaler les traductions manquantes {#report-missing-translations}

Lorsqu'une traduction est manquante, le build réussit mais génère un avertissement tel que `Missing translation for message "{translation_text}"`.
Pour configurer le niveau d'avertissement généré par le compilateur Angular, spécifiez l'un des niveaux suivants.

| Niveau d'avertissement | Détails                                              | Sortie                                                 |
| :------------ | :--------------------------------------------------- | :----------------------------------------------------- |
| `error`       | Lève une erreur et le build échoue                   | n/a                                                    |
| `ignore`      | Ne fait rien                                           | n/a                                                    |
| `warning`     | Affiche l'avertissement par défaut dans la console ou le shell | `Missing translation for message "{translation_text}"` |

Spécifiez le niveau d'avertissement dans la section `options` de la cible `build` de votre fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig].

### Exemple d'avertissement `error` dans `angular.json` {#angularjson-error-warning-example}

L'exemple suivant montre comment définir le niveau d'avertissement sur `error`.

<docs-code header="angular.json" path="adev/src/content/examples/i18n/angular.json" region="missing-translation-error" />

HELPFUL: Lorsque vous compilez votre projet Angular en application Angular, les occurrences de l'attribut `i18n` sont remplacées par des occurrences de la chaîne de message balisée avec [`$localize`][ApiLocalizeInitLocalize].
Cela signifie que votre application Angular est traduite après la compilation.
Cela signifie aussi que vous pouvez créer des versions localisées de votre application Angular sans recompiler l'ensemble de votre projet Angular pour chaque locale.

Lorsque vous traduisez votre application Angular, la _transformation de traduction_ remplace et réordonne les parties \(chaînes statiques et expressions\) du template literal avec des chaînes issues d'une collection de traductions.
Pour en savoir plus, consultez [`$localize`][ApiLocalizeInitLocalize].

TLDR: Compilez une fois, puis traduisez pour chaque locale.

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/i18n/deploy" title="Déployer plusieurs locales"/>
</docs-pill-row>

[ApiLocalizeInitLocalize]: api/localize/init/$localize '$localize | init - localize - API | Angular'
[CliMain]: cli 'CLI Overview and Command Reference | Angular'
[CliBuild]: cli/build 'ng build | CLI | Angular'
[GuideBuild]: tools/cli/build 'Building and serving Angular apps | Angular'
[GuideI18nCommonMergeApplySpecificBuildOptionsForJustOneLocale]: guide/i18n/merge#apply-specific-build-options-for-just-one-locale 'Apply specific build options for just one locale - Merge translations into the application | Angular'
[GuideI18nCommonMergeBuildFromTheCommandLine]: guide/i18n/merge#build-from-the-command-line 'Build from the command line - Merge translations into the application | Angular'
[GuideI18nCommonMergeDefineLocalesInTheBuildConfiguration]: guide/i18n/merge#define-locales-in-the-build-configuration 'Define locales in the build configuration - Merge translations into the application | Angular'
[GuideI18nCommonMergeGenerateApplicationVariantsForEachLocale]: guide/i18n/merge#generate-application-variants-for-each-locale 'Generate application variants for each locale - Merge translations into the application | Angular'
[GuideI18nCommonTranslationFilesChangeTheSourceLanguageFileFormat]: guide/i18n/translation-files#change-the-source-language-file-format 'Change the source language file format - Work with translation files | Angular'
[GuideWorkspaceConfig]: reference/configs/workspace-config 'Angular workspace configuration | Angular'
[GuideWorkspaceConfigI18n]: reference/configs/workspace-config#i18n-options 'i18n options - Angular workspace configuration | Angular'
