<!-- source: 52b03362d6012f4456f491d6053a94ca0cb3dde2 -->
# Diagnostics étendus

De nombreux motifs de code sont techniquement valides pour le compilateur ou à l'exécution, mais peuvent comporter des subtilités ou des pièges.
Ces motifs peuvent ne pas avoir l'effet attendu par le développeur, ce qui conduit souvent à des bugs.
Le compilateur Angular inclut des « diagnostics étendus » qui identifient bon nombre de ces motifs, afin d'avertir les développeurs des problèmes potentiels et de faire respecter des bonnes pratiques courantes dans une base de code.

## Diagnostics {#diagnostics}

Actuellement, Angular prend en charge les diagnostics étendus suivants :

| Code     | Nom                                                                   |
| :------- | :-------------------------------------------------------------------- |
| `NG8101` | [`invalidBananaInBox`](extended-diagnostics/NG8101)                   |
| `NG8102` | [`nullishCoalescingNotNullable`](extended-diagnostics/NG8102)         |
| `NG8103` | [`missingControlFlowDirective`](extended-diagnostics/NG8103)          |
| `NG8104` | [`textAttributeNotBinding`](extended-diagnostics/NG8104)              |
| `NG8105` | [`missingNgForOfLet`](extended-diagnostics/NG8105)                    |
| `NG8106` | [`suffixNotSupported`](extended-diagnostics/NG8106)                   |
| `NG8107` | [`optionalChainNotNullable`](extended-diagnostics/NG8107)             |
| `NG8108` | [`skipHydrationNotStatic`](extended-diagnostics/NG8108)               |
| `NG8109` | [`interpolatedSignalNotInvoked`](extended-diagnostics/NG8109)         |
| `NG8111` | [`uninvokedFunctionInEventBinding`](extended-diagnostics/NG8111)      |
| `NG8112` | [`unusedLetDeclaration`](extended-diagnostics/NG8112)                 |
| `NG8113` | [`unusedStandaloneImports`](extended-diagnostics/NG8113)              |
| `NG8114` | [`unparenthesizedNullishCoalescing`](extended-diagnostics/NG8114)     |
| `NG8115` | [`uninvokedTrackFunction`](extended-diagnostics/NG8115)               |
| `NG8116` | [`missingStructuralDirective`](extended-diagnostics/NG8116)           |
| `NG8117` | [`uninvokedFunctionInTextInterpolation`](extended-diagnostics/NG8117) |
| `NG8021` | [`deferTriggerMisconfiguration`](extended-diagnostics/NG8021)         |

## Configuration {#configuration}

Les diagnostics étendus sont des avertissements par défaut et ne bloquent pas la compilation.
Chaque diagnostic peut être configuré avec l'une des catégories suivantes :

| Catégorie d'erreur | Effet |
| :------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `warning`      | Par défaut - Le compilateur émet le diagnostic sous forme d'avertissement mais ne bloque pas la compilation. Le compilateur se termine tout de même avec le code de statut 0, même si des avertissements sont émis. |
| `error`        | Le compilateur émet le diagnostic sous forme d'erreur et la compilation échoue. Le compilateur se termine avec un code de statut non nul si une ou plusieurs erreurs sont émises. |
| `suppress`     | Le compilateur n'émet _pas du tout_ le diagnostic. |

La sévérité des vérifications peut être configurée via une [option du compilateur Angular](reference/configs/angular-compiler-options) :

```json
{
  "angularCompilerOptions": {
    "extendedDiagnostics": {
      "checks": {
        "invalidBananaInBox": "suppress"
      },
      "defaultCategory": "error"
    }
  }
}
```

Le champ `checks` associe le nom de chaque diagnostic à sa catégorie.
Consultez [Diagnostics](#diagnostics) pour la liste complète des diagnostics étendus et le nom à utiliser pour les configurer.

Le champ `defaultCategory` s'applique à tous les diagnostics qui ne sont pas explicitement listés dans `checks`.
S'il n'est pas défini, ces diagnostics sont traités comme `warning`.

Les diagnostics étendus sont émis lorsque [`strictTemplates`](tools/cli/template-typecheck#strict-mode) est activé.
C'est nécessaire pour que le compilateur comprenne mieux les types dans chaque template Angular et fournisse des diagnostics précis et pertinents.

## Versionnement sémantique {#semantic-versioning}

L'équipe Angular prévoit d'ajouter ou d'activer de nouveaux diagnostics étendus dans les versions **mineures** d'Angular (voir [semver](https://docs.npmjs.com/about-semantic-versioning)).
Cela signifie que la mise à jour d'Angular peut faire apparaître de nouveaux avertissements dans votre base de code existante.
Cela permet à l'équipe de livrer des fonctionnalités plus rapidement et de rendre les diagnostics étendus plus accessibles aux développeurs.

Cependant, définir `"defaultCategory": "error"` transforme ces avertissements en véritables erreurs.
Une mise à jour de version mineure peut alors introduire des erreurs de compilation, ce qui peut être perçu comme un breaking change non conforme à semver.
Tout nouveau diagnostic peut être désactivé ou rétrogradé en avertissement via la [configuration](#configuration) ci-dessus, de sorte que l'impact d'un nouveau diagnostic devrait être minime pour
les projets qui traitent les diagnostics étendus comme des erreurs par défaut.
Utiliser l'erreur par défaut est un outil très puissant ; tenez simplement compte de cette mise en garde liée à semver au moment de décider si `error` est la bonne valeur par défaut pour votre projet.

## Nouveaux diagnostics {#new-diagnostics}

L'équipe Angular est toujours ouverte aux suggestions de nouveaux diagnostics à ajouter.
Les diagnostics étendus doivent généralement :

- Détecter une erreur de développement courante et non évidente dans les templates Angular
- Expliquer clairement pourquoi ce motif peut conduire à des bugs ou à un comportement non souhaité
- Proposer une ou plusieurs solutions claires
- Avoir un taux de faux positifs faible, idéalement nul
- S'appliquer à la grande majorité des applications Angular (pas spécifique à une library non officielle)
- Améliorer la justesse ou les performances du programme (pas le style, cette responsabilité revient à un linter)

Si vous avez une idée de diagnostic étendu qui répond à ces critères, envisagez de soumettre une [demande de fonctionnalité](https://github.com/angular/angular/issues/new?template=2-feature-request.yaml).
