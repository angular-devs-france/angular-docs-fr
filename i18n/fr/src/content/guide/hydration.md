<!-- source: a86cb18c482dfc14da70e15a52a6bd98c6987f8e -->
# Hydratation

## Qu'est-ce que l'hydratation {#what-is-hydration}

L'hydratation est le processus qui restaure côté client l'application rendue côté serveur. Cela inclut notamment la réutilisation des structures DOM rendues par le serveur, la conservation de l'état de l'application, le transfert des données de l'application déjà récupérées par le serveur, ainsi que d'autres processus.

## Pourquoi l'hydratation est-elle importante ? {#why-is-hydration-important}

L'hydratation améliore les performances de l'application en évitant le travail supplémentaire de recréation des nœuds DOM. Angular essaie plutôt de faire correspondre, à l'exécution, les éléments DOM existants à la structure de l'application et réutilise les nœuds DOM lorsque c'est possible. Il en résulte une amélioration des performances mesurable à l'aide des statistiques [Core Web Vitals (CWV)](https://web.dev/learn-core-web-vitals/), comme la réduction du First Input Delay ([FID](https://web.dev/fid/)) et du Largest Contentful Paint ([LCP](https://web.dev/lcp/)), ainsi que du Cumulative Layout Shift ([CLS](https://web.dev/cls/)). L'amélioration de ces chiffres a aussi un effet sur des aspects comme les performances SEO.

Sans hydratation activée, les applications Angular rendues côté serveur détruisent et rendent à nouveau le DOM de l'application, ce qui peut provoquer un scintillement visible de l'interface. Ce nouveau rendu peut avoir un impact négatif sur les [Core Web Vitals](https://web.dev/learn-core-web-vitals/) comme le [LCP](https://web.dev/lcp/) et provoquer un décalage de mise en page. Activer l'hydratation permet de réutiliser le DOM existant et évite ce scintillement.

## Comment activer l'hydratation dans Angular {#how-do-you-enable-hydration-in-angular}

L'hydratation ne peut être activée que pour les applications rendues côté serveur (SSR). Suivez d'abord le [guide SSR d'Angular](guide/ssr) pour activer le rendu côté serveur.

### Avec Angular CLI {#using-angular-cli}

Si vous avez utilisé Angular CLI pour activer le SSR (soit lors de la création de l'application, soit plus tard via `ng add @angular/ssr`), le code qui active l'hydratation devrait déjà être inclus dans votre application.

### Configuration manuelle {#manual-setup}

Si vous avez une configuration personnalisée et n'avez pas utilisé Angular CLI pour activer le SSR, vous pouvez activer l'hydratation manuellement en ouvrant le composant ou le module principal de votre application et en important `provideClientHydration` depuis `@angular/platform-browser`. Ajoutez ensuite ce provider à la liste des providers du bootstrap de votre application.

```typescript
import {
  bootstrapApplication,
  provideClientHydration,
} from '@angular/platform-browser';
...

bootstrapApplication(App, {
  providers: [provideClientHydration()]
});
```

Si vous utilisez des NgModules, ajoutez plutôt `provideClientHydration` à la liste des providers du module racine de votre application.

```typescript
import {provideClientHydration} from '@angular/platform-browser';
import {NgModule} from '@angular/core';

@NgModule({
  declarations: [App],
  exports: [App],
  bootstrap: [App],
  providers: [provideClientHydration()],
})
export class AppModule {}
```

IMPORTANT: Assurez-vous que l'appel à `provideClientHydration()` est aussi inclus dans l'ensemble des providers utilisé pour le bootstrap de l'application sur le **serveur**. Dans les applications qui suivent la structure de projet par défaut (générée par la commande `ng new`), ajouter l'appel au `AppModule` racine devrait suffire, puisque ce module est importé par le module serveur. Si vous utilisez une configuration personnalisée, ajoutez l'appel à `provideClientHydration()` à la liste des providers de la configuration du bootstrap côté serveur.

### Vérifier que l'hydratation est activée {#verify-that-hydration-is-enabled}

Après avoir configuré l'hydratation et démarré votre serveur, chargez votre application dans le navigateur.

HELPFUL: Vous devrez probablement corriger les cas de manipulation directe du DOM avant que l'hydratation fonctionne pleinement, soit en passant à des constructions Angular, soit en utilisant `ngSkipHydration`. Consultez [Contraintes](#constraints), [Manipulation directe du DOM](#direct-dom-manipulation) et [Comment ignorer l'hydratation pour certains composants](#how-to-skip-hydration-for-particular-components) pour plus de détails.

Lorsque l'application s'exécute en mode développement, vous pouvez confirmer que l'hydratation est activée en ouvrant les outils de développement de votre navigateur et en consultant la console. Vous devriez voir un message contenant des statistiques liées à l'hydratation, comme le nombre de composants et de nœuds hydratés. Angular calcule ces statistiques à partir de tous les composants rendus sur une page, y compris ceux qui proviennent de libraries tierces.

Vous pouvez aussi utiliser [l'extension de navigateur Angular DevTools](tools/devtools) pour voir l'état d'hydratation des composants d'une page. Angular DevTools permet également d'activer une surcouche indiquant quelles parties de la page ont été hydratées. En cas d'erreur d'incohérence d'hydratation, DevTools met aussi en évidence le composant à l'origine de l'erreur.

## Capturer et rejouer les événements {#capturing-and-replaying-events}

Lorsqu'une application est rendue sur le serveur, elle est visible dans le navigateur dès que le HTML produit est chargé. Les utilisateurs peuvent penser qu'ils peuvent interagir avec la page, mais les event listeners ne sont attachés qu'une fois l'hydratation terminée. La fonctionnalité Event Replay capture tous les événements qui se produisent avant l'hydratation et les rejoue une fois l'hydratation terminée.

L'event replay est activé en même temps que [l'hydratation incrémentale](guide/incremental-hydration). Si vous désactivez l'hydratation incrémentale avec `withNoIncrementalHydration()`, vous pouvez toujours activer l'event replay à l'aide de la fonction `withEventReplay()` :

```typescript
import {
  provideClientHydration,
  withEventReplay,
  withNoIncrementalHydration,
  bootstrapApplication,
} from '@angular/platform-browser';

bootstrapApplication(App, {
  providers: [provideClientHydration(withNoIncrementalHydration(), withEventReplay())],
});
```

### Fonctionnement de l'event replay {#how-event-replay-works}

Event Replay est une fonctionnalité qui améliore l'expérience utilisateur en capturant les événements utilisateur déclenchés avant la fin du processus d'hydratation. Ces événements sont ensuite rejoués, ce qui garantit qu'aucune de ces interactions n'est perdue.

L'Event Replay se décompose en trois phases principales :

- **Capture des interactions utilisateur**<br>
  Avant l'**hydratation**, l'Event Replay capture et stocke toutes les interactions que l'utilisateur peut effectuer, comme les clics et d'autres événements natifs du navigateur.

- **Stockage des événements**<br>
  L'**Event Contract** conserve en mémoire toutes les interactions enregistrées à l'étape précédente, ce qui garantit qu'elles ne sont pas perdues et pourront être rejouées plus tard.

- **Relance des événements**<br>
  Une fois l'**hydratation** terminée, Angular invoque à nouveau les événements capturés.

L'event replay prend en charge les _événements natifs du navigateur_, par exemple `click`, `mouseover` et `focusin`. Pour en savoir plus sur JSAction, la library sur laquelle repose l'event replay, consultez [son readme](https://github.com/angular/angular/tree/main/packages/core/primitives/event-dispatch#readme).

Cette fonctionnalité garantit une expérience utilisateur cohérente, en évitant que les actions effectuées par l'utilisateur avant l'hydratation soient ignorées.

## Contraintes {#constraints}

L'hydratation impose à votre application quelques contraintes qui n'existent pas sans hydratation. Votre application doit générer la même structure DOM à la fois sur le serveur et sur le client. Le processus d'hydratation s'attend à ce que l'arbre DOM ait la même structure aux deux endroits. Cela inclut aussi les espaces blancs et les nœuds de commentaire qu'Angular produit pendant le rendu sur le serveur. Ces espaces blancs et ces nœuds doivent être présents dans le HTML généré par le processus de rendu côté serveur.

IMPORTANT: Le HTML produit par l'opération de rendu côté serveur **ne doit pas** être modifié entre le serveur et le client.

En cas d'incohérence entre les structures de l'arbre DOM du serveur et du client, le processus d'hydratation rencontrera des problèmes en essayant de faire correspondre ce qui était attendu avec ce qui est réellement présent dans le DOM. Les composants qui manipulent directement le DOM à l'aide des API DOM natives en sont la cause la plus fréquente.

### Manipulation directe du DOM {#direct-dom-manipulation}

Si vous avez des composants qui manipulent le DOM à l'aide des API DOM natives ou qui utilisent `innerHTML` ou `outerHTML`, le processus d'hydratation rencontrera des erreurs. Les cas précis où la manipulation du DOM pose problème sont, par exemple, l'accès au `document`, la recherche d'éléments spécifiques et l'injection de nœuds supplémentaires avec `appendChild`. Détacher des nœuds DOM et les déplacer à d'autres endroits provoque également des erreurs.

En effet, Angular n'a pas connaissance de ces modifications du DOM et ne peut pas les résoudre pendant le processus d'hydratation. Angular s'attend à une certaine structure, mais en rencontre une différente au moment d'hydrater. Cette incohérence entraîne un échec de l'hydratation et lève une erreur d'incohérence DOM ([voir ci-dessous](#errors)).

Le mieux est de refactoriser votre composant pour éviter ce type de manipulation du DOM. Essayez, si possible, d'utiliser les API Angular pour effectuer ce travail. Si vous ne pouvez pas refactoriser ce comportement, utilisez l'attribut `ngSkipHydration` ([décrit ci-dessous](#how-to-skip-hydration-for-particular-components)) jusqu'à ce que vous puissiez passer à une solution compatible avec l'hydratation.

### Structure HTML valide {#valid-html-structure}

Dans certains cas, un template de composant dont la structure HTML n'est pas valide peut provoquer une erreur d'incohérence DOM pendant l'hydratation.

Voici, à titre d'exemple, quelques-uns des cas les plus courants de ce problème.

- `<table>` sans `<tbody>`
- `<div>` à l'intérieur d'un `<p>`
- `<a>` à l'intérieur d'un autre `<a>`

Si vous n'êtes pas sûr que votre HTML soit valide, vous pouvez utiliser un [validateur de syntaxe](https://validator.w3.org/) pour le vérifier.

NOTE: Bien que le standard HTML n'exige pas l'élément `<tbody>` dans les tableaux, les navigateurs modernes créent automatiquement un élément `<tbody>` dans les tableaux qui n'en déclarent pas. En raison de cette incohérence, déclarez toujours explicitement un élément `<tbody>` dans vos tableaux pour éviter les erreurs d'hydratation.

### Configuration de la conservation des espaces blancs {#preserve-whitespaces-configuration}

Lorsque vous utilisez l'hydratation, nous recommandons de conserver le réglage par défaut `false` pour `preserveWhitespaces`. Si ce réglage ne figure pas dans votre tsconfig, la valeur sera `false` et aucune modification n'est nécessaire. Si vous choisissez d'activer la conservation des espaces blancs en ajoutant `preserveWhitespaces: true` à votre tsconfig, vous risquez de rencontrer des problèmes d'hydratation. Il ne s'agit pas encore d'une configuration entièrement prise en charge.

HELPFUL: Assurez-vous que ce réglage est défini de manière **cohérente** dans `tsconfig.server.json` pour votre serveur et dans `tsconfig.app.json` pour vos builds navigateur. Une valeur différente entre les deux casse l'hydratation.

Si vous choisissez de définir ce réglage dans votre tsconfig, nous recommandons de le définir uniquement dans `tsconfig.app.json`, dont `tsconfig.server.json` hérite par défaut.

### Les implémentations personnalisées ou noop de Zone.js ne sont pas encore prises en charge {#custom-or-noop-zonejs-are-not-yet-supported}

L'hydratation s'appuie sur un signal de Zone.js indiquant que l'application devient stable, afin qu'Angular puisse démarrer le processus de sérialisation sur le serveur ou le nettoyage post-hydratation sur le client pour supprimer les nœuds DOM restés non réclamés.

Fournir une implémentation personnalisée ou « noop » de Zone.js peut modifier le moment où survient l'événement « stable », et donc déclencher la sérialisation ou le nettoyage trop tôt ou trop tard. Il ne s'agit pas encore d'une configuration entièrement prise en charge, et vous devrez peut-être ajuster le moment de l'événement `onStable` dans l'implémentation personnalisée de Zone.js.

## Erreurs {#errors}

Vous pouvez rencontrer plusieurs erreurs liées à l'hydratation, depuis des incohérences de nœuds jusqu'aux cas où `ngSkipHydration` a été utilisé sur un nœud host invalide. Le cas d'erreur le plus courant est dû à une manipulation directe du DOM via des API natives, qui empêche l'hydratation de trouver ou de faire correspondre, côté client, la structure d'arbre DOM attendue qui a été rendue par le serveur. L'autre cas où vous pouvez rencontrer ce type d'erreur a été mentionné plus haut, dans la section [Structure HTML valide](#valid-html-structure). Assurez-vous donc que le HTML de vos templates utilise une structure valide, et vous éviterez ce cas d'erreur.

Pour une référence complète des erreurs liées à l'hydratation, consultez le [guide de référence des erreurs](/errors).

## Comment ignorer l'hydratation pour certains composants {#how-to-skip-hydration-for-particular-components}

Certains composants peuvent ne pas fonctionner correctement avec l'hydratation activée, en raison de l'un des problèmes mentionnés précédemment, comme la [manipulation directe du DOM](#direct-dom-manipulation). Pour contourner le problème, vous pouvez ajouter l'attribut `ngSkipHydration` à la balise d'un composant afin d'ignorer l'hydratation de l'ensemble du composant.

```angular-html
<app-example ngSkipHydration />
```

Vous pouvez aussi définir `ngSkipHydration` sous forme de host binding.

```typescript
@Component({
  ...
  host: {ngSkipHydration: 'true'},
})
class ExampleComponent {}
```

L'attribut `ngSkipHydration` force Angular à ignorer l'hydratation de l'ensemble du composant et de ses enfants. Utiliser cet attribut signifie que le composant se comporte comme si l'hydratation n'était pas activée : il se détruit et effectue à nouveau son rendu.

HELPFUL: Cela corrige les problèmes de rendu, mais vous perdez les avantages de l'hydratation pour ce composant (et ses enfants). Vous devrez adapter l'implémentation de votre composant pour éviter les patterns qui cassent l'hydratation (par exemple la manipulation directe du DOM) afin de pouvoir supprimer l'annotation qui ignore l'hydratation.

L'attribut `ngSkipHydration` ne peut être utilisé que sur les nœuds host de composants. Angular lève une erreur si cet attribut est ajouté à d'autres nœuds.

Gardez à l'esprit qu'ajouter l'attribut `ngSkipHydration` au composant racine de votre application désactiverait de fait l'hydratation pour toute votre application. Utilisez cet attribut avec prudence et discernement. Il est conçu comme un contournement de dernier recours. Les composants qui cassent l'hydratation doivent être considérés comme des bugs à corriger.

## Moment de l'hydratation et stabilité de l'application {#hydration-timing-and-application-stability}

La stabilité de l'application est un élément important du processus d'hydratation. L'hydratation et tous les processus post-hydratation n'ont lieu qu'une fois que l'application a signalé sa stabilité. La stabilité peut être retardée de plusieurs façons, par exemple par la définition de timeouts et d'intervalles, des promises non résolues ou des microtâches en attente. Dans ces cas, vous pouvez rencontrer l'erreur [Application remains unstable](errors/NG0506), qui indique que votre application n'a pas atteint l'état stable au bout de 10 secondes. Si vous constatez que votre application ne s'hydrate pas tout de suite, examinez ce qui affecte la stabilité de l'application et refactorisez votre code pour éviter ces délais.

### Déboguer la stabilité de l'application {#debugging-application-stability}

L'utilitaire `provideStabilityDebugging` aide à identifier pourquoi votre application ne parvient pas à se stabiliser. Cet utilitaire est fourni par défaut en mode développement lorsque vous utilisez `provideClientHydration`. Vous pouvez aussi l'ajouter manuellement aux providers de l'application, par exemple pour l'utiliser dans des bundles de production ou avec le SSR sans hydratation. La fonctionnalité affiche des informations dans la console si l'application met plus de temps que prévu à se stabiliser.

```typescript
import {provideStabilityDebugging} from '@angular/core';
import {bootstrapApplication} from '@angular/platform-browser';
import 'zone.js/plugins/task-tracking'; // Use if you have Zone.js with `provideZoneChangeDetection`

bootstrapApplication(App, {
  providers: [provideStabilityDebugging()],
});
```

Une fois activé, l'utilitaire affiche dans la console les tâches en attente (`PendingTasks`). Si votre application utilise Zone.js, vous pouvez aussi importer `zone.js/plugins/task-tracking` pour voir quelles macrotâches empêchent la zone Angular de se stabiliser. Ce plugin fournit la stack trace de la création de la macrotâche, ce qui vous aide efficacement à identifier l'origine du délai.

IMPORTANT: Angular ne retire pas le plugin de suivi des tâches de zone.js ni cet utilitaire des bundles de production. Utilisez-les uniquement pour déboguer temporairement des problèmes de stabilité pendant le développement, y compris pour les builds de production optimisés.

## I18N {#i18n}

HELPFUL: Par défaut, Angular ignore l'hydratation des composants qui utilisent des blocs i18n, ce qui revient à refaire entièrement le rendu de ces composants.

Pour activer l'hydratation des blocs i18n, vous pouvez ajouter [`withI18nSupport`](/api/platform-browser/withI18nSupport) à votre appel à `provideClientHydration`.

```typescript
import {
  bootstrapApplication,
  provideClientHydration,
  withI18nSupport,
} from '@angular/platform-browser';
...

bootstrapApplication(App, {
  providers: [provideClientHydration(withI18nSupport())]
});
```

## Un rendu cohérent côté serveur et côté client {#consistent-rendering-across-server-side-and-client-side}

Évitez d'introduire des blocs `@if` et d'autres conditions qui affichent un contenu différent lors du rendu côté serveur et lors du rendu côté client, par exemple un bloc `@if` utilisant la fonction `isPlatformBrowser` d'Angular. Ces différences de rendu provoquent des décalages de mise en page, ce qui nuit à l'expérience de l'utilisateur final et aux Core Web Vitals.

## Libraries tierces qui manipulent le DOM {#third-party-libraries-with-dom-manipulation}

Un certain nombre de libraries tierces dépendent de la manipulation du DOM pour effectuer leur rendu. Les graphiques D3 en sont un exemple typique. Ces libraries fonctionnaient sans hydratation, mais elles peuvent provoquer des erreurs d'incohérence DOM lorsque l'hydratation est activée. Pour l'instant, si vous rencontrez des erreurs d'incohérence DOM avec l'une de ces libraries, vous pouvez ajouter l'attribut `ngSkipHydration` au composant qui effectue son rendu à l'aide de cette library.

## Scripts tiers qui manipulent le DOM {#third-party-scripts-with-dom-manipulation}

De nombreux scripts tiers, comme les traceurs publicitaires et les outils d'analytics, modifient le DOM avant que l'hydratation puisse avoir lieu. Ces scripts peuvent provoquer des erreurs d'hydratation, car la page ne correspond plus à la structure attendue par Angular. Dans la mesure du possible, différez plutôt ce type de script jusqu'après l'hydratation. Envisagez d'utiliser [`AfterNextRender`](api/core/afterNextRender) pour retarder le script jusqu'à ce que les processus post-hydratation aient eu lieu.

## Hydratation incrémentale {#incremental-hydration}

L'hydratation incrémentale est une forme avancée d'hydratation qui permet un contrôle plus fin du moment où l'hydratation a lieu. Consultez le [guide de l'hydratation incrémentale](guide/incremental-hydration) pour plus d'informations.
