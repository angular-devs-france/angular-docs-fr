<!-- source: 3e92a5a4002f63ce8ef7d2cf3a3cec2154dc42c0 -->
# Angular sans ZoneJS (zoneless)

## Pourquoi utiliser le mode zoneless ? {#why-use-zoneless}

Les principaux avantages à retirer ZoneJS des dépendances sont :

- **Performances améliorées** : ZoneJS utilise les événements DOM et les tâches asynchrones comme indicateurs du moment où l'état de l'application _pourrait_ avoir changé, puis déclenche une synchronisation de l'application pour exécuter la détection de changements sur ses vues. ZoneJS ne sait pas si l'état de l'application a réellement changé, et cette synchronisation est donc déclenchée plus souvent que nécessaire.
- **Core Web Vitals améliorés** : ZoneJS ajoute une surcharge non négligeable, à la fois en taille de payload et en temps de démarrage.
- **Débogage facilité** : ZoneJS rend le débogage du code plus difficile. Les stack traces sont plus difficiles à comprendre avec ZoneJS. Il est aussi difficile de comprendre quand du code casse parce qu'il s'exécute en dehors de la zone Angular.
- **Meilleure compatibilité avec l'écosystème** : ZoneJS fonctionne en patchant les API du navigateur, mais ne dispose pas automatiquement de patches pour chaque nouvelle API du navigateur. Certaines API, comme `async`/`await`, ne peuvent pas être patchées efficacement et doivent être transpilées vers une version antérieure (downleveling) pour fonctionner avec ZoneJS. Il arrive aussi que des libraries de l'écosystème soient incompatibles avec la façon dont ZoneJS patche les API natives. Retirer ZoneJS des dépendances garantit une meilleure compatibilité à long terme en supprimant une source de complexité, de monkey patching et de maintenance continue.

## Activer le mode zoneless dans une application {#enabling-zoneless-in-an-application}

Le mode zoneless est le comportement par défaut depuis Angular v21 : vous n'avez rien à faire pour l'activer. Vérifiez que `provideZoneChangeDetection` n'est utilisé nulle part pour remplacer la configuration par défaut.

Si vous utilisez Angular v20, activez la détection de changements zoneless en ajoutant `provideZonelessChangeDetection()` au bootstrap :

```ts {header: 'standalone bootstrap'}
bootstrapApplication(MyApp, {providers: [provideZonelessChangeDetection()]});
```

```ts {header: 'NgModule bootstrap'}
platformBrowser().bootstrapModule(AppModule);

@NgModule({
  providers: [provideZonelessChangeDetection()],
})
export class AppModule {}
```

## Retirer ZoneJS {#removing-zonejs}

Les applications zoneless devraient retirer complètement ZoneJS du build pour réduire la taille du bundle. ZoneJS est généralement
chargé via l'option `polyfills` dans `angular.json`, à la fois dans les cibles `build` et `test`. Retirez `zone.js`
et `zone.js/testing` des deux pour l'exclure du build. Les projets qui utilisent un fichier `polyfills.ts` explicite
doivent retirer `import 'zone.js';` et `import 'zone.js/testing';` de ce fichier.

Une fois ZoneJS retiré du build, la dépendance `zone.js` n'est plus nécessaire non plus et le
package peut être entièrement supprimé :

```shell
npm uninstall zone.js
```

## Prérequis pour la compatibilité zoneless {#requirements-for-zoneless-compatibility}

Angular s'appuie sur des notifications provenant des API du cœur pour déterminer quand exécuter la détection de changements et sur quelles vues.
Ces notifications comprennent :

- `ChangeDetectorRef.markForCheck` (appelé automatiquement par `AsyncPipe`)
- `ComponentRef.setInput`
- La mise à jour d'un signal lu dans un template
- Les callbacks des listeners liés au host ou au template
- L'attachement d'une vue marquée comme modifiée (dirty) par l'un des cas ci-dessus

### Composants compatibles `OnPush` {#onpush-compatible-components}

Une façon de garantir qu'un composant utilise les bons mécanismes de notification ci-dessus consiste à
utiliser [ChangeDetectionStrategy.OnPush](/best-practices/skipping-subtrees#using-onpush).

La stratégie de détection de changements `OnPush` n'est pas obligatoire, mais c'est une étape recommandée vers la compatibilité zoneless pour les composants d'application. Il n'est pas toujours possible pour les composants de library d'utiliser `ChangeDetectionStrategy.OnPush`.
Lorsqu'un composant de library est le host de composants utilisateur susceptibles d'utiliser `ChangeDetectionStrategy.Eager`/`Default`, il ne peut pas utiliser `OnPush`, car cela empêcherait le composant enfant d'être rafraîchi s'il n'est pas compatible `OnPush` et s'appuie sur ZoneJS pour déclencher la détection de changements. Les composants peuvent utiliser la stratégie `Default` tant qu'ils notifient Angular lorsque la détection de changements doit s'exécuter (en appelant `markForCheck`, en utilisant des signals, `AsyncPipe`, etc.).
Être le host d'un composant utilisateur signifie utiliser une API comme `ViewContainerRef.createComponent`, et pas seulement héberger une portion du template d'un composant utilisateur (c'est-à-dire la projection de contenu ou un input recevant une référence de template).

### Retirer `NgZone.onMicrotaskEmpty`, `NgZone.onUnstable`, `NgZone.isStable` ou `NgZone.onStable` {#remove-ngzoneonmicrotaskempty-ngzoneonunstable-ngzoneisstable-or-ngzoneonstable}

Les applications et les libraries doivent supprimer les utilisations de `NgZone.onMicrotaskEmpty`, `NgZone.onUnstable` et `NgZone.onStable`.
Ces observables n'émettront jamais lorsqu'une application active la détection de changements zoneless.
De même, `NgZone.isStable` vaudra toujours `true` et ne doit pas servir de condition à l'exécution de code.

Les observables `NgZone.onMicrotaskEmpty` et `NgZone.onStable` servent le plus souvent à attendre qu'Angular
termine la détection de changements avant d'effectuer une tâche. Ils peuvent être remplacés par `afterNextRender`
s'il faut attendre une seule détection de changements, ou par `afterEveryRender` si une condition peut s'étendre
sur plusieurs cycles de détection de changements. Dans d'autres cas, ces observables étaient utilisés parce qu'ils se trouvaient être
familiers et avaient un timing proche de celui nécessaire. Des API DOM plus simples ou plus directes peuvent être utilisées à la place,
comme `MutationObserver` lorsque le code doit attendre un certain état du DOM (plutôt que de l'attendre indirectement
via les hooks de rendu d'Angular).

<docs-callout title="NgZone.run et NgZone.runOutsideAngular sont compatibles avec le mode zoneless">
`NgZone.run` et `NgZone.runOutsideAngular` n'ont pas besoin d'être retirés pour que le code soit compatible avec
les applications zoneless. En réalité, retirer ces appels peut entraîner des régressions de performances pour les libraries
utilisées dans des applications qui s'appuient encore sur ZoneJS.
</docs-callout>

### `PendingTasks` pour le rendu côté serveur (SSR) {#pendingtasks-for-server-side-rendering-ssr}

Si vous utilisez le SSR avec Angular, vous savez peut-être qu'il s'appuie sur ZoneJS pour déterminer quand l'application
est « stable » et peut être sérialisée. S'il existe des tâches asynchrones qui doivent empêcher la sérialisation, une application
qui n'utilise pas ZoneJS doit les signaler à Angular avec le service [PendingTasks](/api/core/PendingTasks). La sérialisation
attendra le premier moment où toutes les tâches en attente auront été retirées.

Les deux utilisations les plus simples des tâches en attente passent par la méthode `run` :

```typescript
const taskService = inject(PendingTasks);
taskService.run(async () => {
  const someResult = await doSomeWorkThatNeedsToBeRendered();
  this.someState.set(someResult);
});
```

Pour des cas d'usage plus complexes, vous pouvez ajouter et retirer manuellement une tâche en attente :

```typescript
const taskService = inject(PendingTasks);
const taskCleanup = taskService.add();
try {
  await doSomeWorkThatNeedsToBeRendered();
} catch {
  // handle error
} finally {
  taskCleanup();
}
```

De plus, le helper [pendingUntilEvent](/api/core/rxjs-interop/pendingUntilEvent) de `rxjs-interop` garantit
que l'application reste instable jusqu'à ce que l'observable émette, se termine, lève une erreur ou soit désabonné.

```typescript
readonly myObservableState = someObservable.pipe(pendingUntilEvent());
```

Le framework utilise aussi ce service en interne pour empêcher la sérialisation tant que des tâches asynchrones ne sont pas terminées. Cela inclut notamment une navigation du router en cours et une requête `HttpClient` non terminée.

### Reactive forms dans les applications zoneless {#reactive-forms-in-zoneless-applications}

Les mises à jour du model des reactive forms (`setValue`, `patchValue`, `FormArray.push` et API similaires) mettent à jour l'état du formulaire et émettent les observables du formulaire, mais elles ne planifient pas automatiquement la détection de changements du composant.

Si un template dépend de l'état des reactive forms, connectez les observables du formulaire à une notification de détection de changements (par exemple `ChangeDetectorRef.markForCheck()`), ou reflétez les données via des signals consommés par le template.

## Tests et débogage {#testing-and-debugging}

### Utiliser le mode zoneless dans `TestBed` {#using-zoneless-in-testbed}

`TestBed` utilise par défaut la détection de changements zoneless, même lorsque `zone.js` est chargé via les `polyfills`.

Pour utiliser la détection de changements basée sur Zone dans un test, ajoutez `provideZoneChangeDetection()` aux providers du `TestBed`. Ajouter `provideZonelessChangeDetection()` est facultatif :

```typescript
TestBed.configureTestingModule({
  // Optional: include the provider to force the testing environment
  // uses the same zoneless behavior as a zoneless application.
  providers: [provideZonelessChangeDetection()],
});

const fixture = TestBed.createComponent(MyComponent);
await fixture.whenStable();
```

Pour que les tests aient un comportement le plus proche possible du code de production,
évitez autant que possible d'utiliser `fixture.detectChanges()`. Cela force
la détection de changements à s'exécuter alors qu'Angular ne l'aurait peut-être pas
planifiée. Les tests doivent s'assurer que ces notifications
ont bien lieu et laisser Angular décider quand synchroniser
l'état, plutôt que de le forcer manuellement dans le test.

Pour les suites de tests existantes, utiliser `fixture.detectChanges()` est un pattern courant
et il ne vaut probablement pas la peine de les convertir en
`await fixture.whenStable()`. `TestBed` vérifiera toujours que le
composant de la fixture est compatible `OnPush` et lève `ExpressionChangedAfterItHasBeenCheckedError`
s'il constate que des valeurs du template ont été mises à jour sans
notification de changement (par exemple `fixture.componentInstance.someValue = 'newValue';`).
Si le composant est utilisé en production, ce problème doit être corrigé en modifiant
le composant pour qu'il utilise des signals pour son état ou appelle `ChangeDetectorRef.markForCheck()`.
Si le composant ne sert que de wrapper de test et n'est jamais utilisé dans une application,
il est acceptable de lui appliquer `ChangeDetectionStrategy.Eager` et d'appeler `fixture.changeDetectorRef.markForCheck()`.

IMPORTANT: `fixture.changeDetectorRef` appartient à la host view de la fixture, donc son `markForCheck()`
ne rafraîchit pas le template d'un composant `OnPush`. Les composants de votre application ne devraient pas en avoir besoin
dans les tests. Si un test en a besoin, c'est généralement que le composant n'est pas entièrement réactif ou
[compatible zoneless](#requirements-for-zoneless-compatibility).

### Vérification en mode debug pour garantir la détection des mises à jour {#debug-mode-check-to-ensure-updates-are-detected}

IMPORTANT: `provideCheckNoChangesConfig` est en [developer preview](reference/releases#developer-preview).

Angular fournit aussi un outil supplémentaire pour vérifier qu'une application effectue
ses mises à jour d'état d'une manière compatible zoneless. `provideCheckNoChangesConfig({exhaustive: true, interval: <milliseconds>})`
peut être utilisé pour vérifier périodiquement qu'aucun binding n'a été mis à jour
sans notification. Angular lève `ExpressionChangedAfterItHasBeenCheckedError`
s'il existe un binding mis à jour qui n'aurait pas été rafraîchi par la détection de changements
zoneless.
