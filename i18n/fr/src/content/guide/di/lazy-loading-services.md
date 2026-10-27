<!-- source: 25b1d708a3b2b933ee7cb5f708e767147be9d69d -->
# Lazy loading des services

IMPORTANT: Pour que le lazy loading fonctionne, le service que vous chargez doit être fourni automatiquement. Décorez-le avec `@Injectable({providedIn: 'root'})` ou avec [`@Service()`](guide/di/creating-and-using-services#using-the-service-vs-injectable-decorator). Sans fourniture automatique, Angular n'a aucun moyen de construire le service après son chargement.

La fonction `injectAsync` d'Angular vous permet de charger un service à la demande, uniquement quand il est réellement nécessaire. C'est utile quand un service dépend d'une library volumineuse ou d'une fonctionnalité rarement utilisée, et que vous ne voulez pas en payer le coût lors du chargement initial de la page.

Quand vous utilisez `injectAsync`, votre bundler place le code du service dans un chunk JavaScript séparé, téléchargé la première fois que vous demandez l'instance. Une fois chargé, Angular résout le service via le système de DI habituel : il peut donc toujours dépendre d'autres injectables et se comporte comme n'importe quel autre singleton.

## Injecter un service en lazy loading {#lazily-injecting-a-service}

Imaginez un `ReportExporter` qui dépend d'une library de tableur volumineuse. La plupart des utilisateurs ouvrent le rapport ; seuls quelques-uns cliquent sur **Export**. Chargez l'exportateur à la demande :

```angular-ts
import {Component, injectAsync} from '@angular/core';

@Component({
  selector: 'app-report',
  template: `<button (click)="export()">Export</button>`,
})
export class Report {
  private exporter = injectAsync(() => import('./report-exporter').then((m) => m.ReportExporter));

  async export() {
    const exporter = await this.exporter();
    exporter.export();
  }
}
```

Le premier appel à `this.exporter()` déclenche l'import dynamique et résout le service depuis la DI. Les appels suivants réutilisent la même promise : le chunk n'est donc récupéré qu'une seule fois.

Si le service chargé en lazy loading est l'[export par défaut](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_the_default_export), passez directement l'import dynamique : Angular extrait le `default` pour vous :

```ts {header: "report-exporter.ts"}
@Service()
export default class ReportExporter {
  /* … */
}
```

```ts {header: "report.ts"}
private exporter = injectAsync(() => import('./report-exporter'));
```

## Précharger la dépendance {#prefetching-the-dependency}

Par défaut, le chunk chargé en lazy loading n'est récupéré que lorsque vous appelez la fonction renvoyée. Vous pouvez lancer le téléchargement plus tôt en passant un déclencheur `prefetch` dans les options. Un déclencheur est n'importe quelle fonction qui renvoie une `Promise` ; quand elle est résolue, Angular lance le loader.

Angular fournit `onIdle`, un déclencheur intégré qui attend que le navigateur soit inactif :

```ts
import {Component, injectAsync, onIdle} from '@angular/core';

@Component({/* … */})
export class Report {
  private exporter = injectAsync(() => import('./report-exporter').then((m) => m.ReportExporter), {
    prefetch: onIdle,
  });
}
```

Vous pouvez aussi configurer `onIdle` avec un temps d'attente maximal, pour que le préchargement ait toujours lieu dans un délai connu, même sur des pages chargées :

```ts
injectAsync(loader, {prefetch: () => onIdle({timeout: 1_000})});
```

NOTE: Le préchargement est opportuniste. Si l'utilisateur appelle la fonctionnalité avant que le préchargement ne se déclenche, Angular charge tout de même la dépendance immédiatement et résout votre `await` dès qu'elle est prête.

## Fournir un déclencheur de préchargement personnalisé {#provide-a-custom-prefetch-trigger}

Un `PrefetchTrigger` est simplement une fonction qui renvoie une promise ; le loader s'exécute dès que la promise est résolue. Utilisez-le pour aligner le préchargement sur vos propres événements, comme un survol ou un tick d'un scheduler :

```ts
import {PrefetchTrigger} from '@angular/core';

export function onHover(target: HTMLElement): PrefetchTrigger {
  return () =>
    new Promise<void>((resolve) => {
      target.addEventListener('pointerenter', () => resolve(), {once: true});
    });
}
```
