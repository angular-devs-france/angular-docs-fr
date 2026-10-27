<!-- source: d5759321dd07f924ed842c42e2c5d8e29b8fa3e3 -->
# Désigner les locales par leur identifiant

Angular utilise l'_identifiant de locale_ Unicode \(Unicode locale ID\) pour trouver les données de locale correctes pour l'internationalisation des chaînes de texte.

<docs-callout title="Identifiant de locale Unicode">

- Un identifiant de locale est conforme à la [spécification de base du Unicode Common Locale Data Repository (CLDR)][UnicodeCldrDevelopmentCoreSpecification].
  Pour en savoir plus sur les identifiants de locale, consultez [Unicode Language and Locale Identifiers][UnicodeCldrDevelopmentCoreSpecificationLocaleID].

- Le CLDR et Angular utilisent les [balises BCP 47][RfcEditorInfoBcp47] comme base de l'identifiant de locale

</docs-callout>

Un identifiant de locale spécifie la langue, le pays et un code facultatif pour d'autres variantes ou subdivisions.
Un identifiant de locale se compose de l'identifiant de langue, d'un trait d'union \(`-`\) et de l'extension de locale.

```html
{language_id}-{locale_extension}
```

HELPFUL: Pour traduire correctement votre projet Angular, vous devez décider quelles langues et quelles locales vous ciblez pour l'internationalisation.

De nombreux pays partagent la même langue, mais avec des usages différents.
Les différences concernent la grammaire, la ponctuation, les formats des devises, des nombres décimaux, des dates, etc.

Pour les exemples de ce guide, utilisez les langues et locales suivantes.

| Langue | Locale                   | Identifiant de locale Unicode |
| :------- | :----------------------- | :---------------- |
| Anglais  | Canada                   | `en-CA`           |
| Anglais  | États-Unis d'Amérique | `en-US`           |
| Français   | Canada                   | `fr-CA`           |
| Français   | France                   | `fr-FR`           |

Le [dépôt Angular][GithubAngularAngularTreeMasterPackagesCommonLocales] inclut les locales courantes.

<docs-callout>
Pour une liste des codes de langue, consultez [ISO 639-2](https://www.loc.gov/standards/iso639-2).
</docs-callout>

## Définir l'identifiant de la locale source {#set-the-source-locale-id}

Utilisez l'Angular CLI pour définir la langue source dans laquelle vous écrivez le template et le code du composant.

Par défaut, Angular utilise `en-US` comme locale source de votre projet.

Pour changer la locale source de votre projet pour le build, effectuez les actions suivantes.

1. Ouvrez le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig].
2. Ajoutez ou modifiez le champ `sourceLocale` dans la section `i18n` :

```json
{
  "projects": {
    "your-project": {
      "i18n": {
        "sourceLocale": "ca" // Use your desired locale code
      }
    }
  }
}
```

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/i18n/format-data-locale" title="Formater les données selon la locale"/>
</docs-pill-row>

[GuideWorkspaceConfig]: reference/configs/workspace-config 'Angular workspace configuration | Angular'
[GithubAngularAngularTreeMasterPackagesCommonLocales]: https://github.com/angular/angular/tree/main/packages/common/locales 'angular/packages/common/locales | angular/angular | GitHub'
[RfcEditorInfoBcp47]: https://www.rfc-editor.org/info/bcp47 'BCP 47 | RFC Editor'
[UnicodeCldrDevelopmentCoreSpecification]: https://cldr.unicode.org/index/cldr-spec 'Core Specification | Unicode CLDR Project'
[UnicodeCldrDevelopmentCoreSpecificationLocaleID]: https://cldr.unicode.org/index/cldr-spec/picking-the-right-language-code 'Unicode Language and Locale Identifiers - Core Specification | Unicode CLDR Project'
