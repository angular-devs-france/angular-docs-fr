<!-- source: 1519a9914fe565da51fc576852623ec3e54c05f1 -->
# Résoudre la pollution de la zone

**Zone.js** est un mécanisme de signalement qu'Angular utilise pour détecter quand l'état d'une application a pu changer. Il capture les opérations asynchrones comme `setTimeout`, les requêtes réseau et les event listeners. Angular planifie la détection de changements en fonction des notifications de Zone.js.

Dans certains cas, les [tâches](https://developer.mozilla.org/docs/Web/API/HTML_DOM_API/Microtask_guide#tasks) ou [microtâches](https://developer.mozilla.org/docs/Web/API/HTML_DOM_API/Microtask_guide#microtasks) planifiées ne modifient pas le modèle de données, ce qui rend l'exécution de la détection de changements inutile. Exemples courants :

- `requestAnimationFrame`, `setTimeout` ou `setInterval`
- La planification de tâches ou de microtâches par des libraries tierces

Cette section explique comment identifier ces situations et comment exécuter du code en dehors de la zone Angular pour éviter des appels inutiles à la détection de changements.

## Identifier les appels inutiles à la détection de changements {#identifying-unnecessary-change-detection-calls}

Vous pouvez détecter les appels inutiles à la détection de changements avec Angular DevTools. Ils apparaissent souvent sous forme de barres consécutives dans la timeline du profiler, avec pour source `setTimeout`, `setInterval`, `requestAnimationFrame` ou un event handler. Lorsque votre application fait peu appel à ces API, le déclenchement de la détection de changements est généralement dû à une library tierce.

<img alt="Aperçu du profiler d'Angular DevTools montrant une pollution de la zone" src="assets/images/best-practices/runtime-performance/zone-pollution.png">

Dans l'image ci-dessus, on voit une série d'appels à la détection de changements déclenchés par des event handlers associés à un élément. C'est un problème courant avec les composants tiers, non natifs d'Angular, qui ne modifient pas le comportement par défaut de `NgZone`.

## Exécuter des tâches en dehors de `NgZone` {#run-tasks-outside-ngzone}

Dans ces cas-là, vous pouvez indiquer à Angular de ne pas déclencher la détection de changements pour les tâches planifiées par un morceau de code donné, à l'aide de [NgZone](/api/core/NgZone).

```ts {header:"Run outside of the Zone" , linenums}
import { Component, NgZone, OnInit, inject } from '@angular/core';

@Component(...)
class App implements OnInit {
  private ngZone = inject(NgZone);

  ngOnInit() {
    this.ngZone.runOutsideAngular(() => setInterval(pollForUpdates, 500));
  }
}
```

L'extrait précédent indique à Angular d'appeler `setInterval` en dehors de la zone Angular et de ne pas exécuter la détection de changements après l'exécution de `pollForUpdates`.

Les libraries tierces déclenchent souvent des cycles de détection de changements inutiles lorsque leurs API sont invoquées dans la zone Angular. Ce phénomène touche particulièrement les libraries qui mettent en place des event listeners ou lancent d'autres tâches (comme des timers, des requêtes XHR, etc.). Évitez ces cycles supplémentaires en appelant les API de la library en dehors de la zone Angular :

```ts {header:"Move the plot initialization outside of the Zone" , linenums}
import { Component, NgZone, OnInit, inject } from '@angular/core';
import * as Plotly from 'plotly.js-dist-min';

@Component(...)
class App implements OnInit {
  private ngZone = inject(NgZone);

  ngOnInit() {
    this.ngZone.runOutsideAngular(() => {
      Plotly.newPlot('chart', data);
    });
  }
}
```

Exécuter `Plotly.newPlot('chart', data);` dans `runOutsideAngular` indique au framework qu'il ne doit pas exécuter la détection de changements après l'exécution des tâches planifiées par la logique d'initialisation.

Par exemple, si `Plotly.newPlot('chart', data)` ajoute des event listeners à un élément du DOM, Angular n'exécute pas la détection de changements après l'exécution de leurs handlers.

Mais il arrive que vous deviez écouter des événements émis par des API tierces. Dans ce cas, n'oubliez pas que ces event listeners s'exécuteront eux aussi en dehors de la zone Angular si la logique d'initialisation y a été exécutée :

```ts {header:"Check whether the handler is called outside of the Zone" , linenums}
import { Component, NgZone, OnInit, output, inject } from '@angular/core';
import * as Plotly from 'plotly.js-dist-min';

@Component(...)
class App implements OnInit {
  private ngZone = inject(NgZone);

  plotlyClick = output<Plotly.PlotMouseEvent>();

  ngOnInit() {
    this.ngZone.runOutsideAngular(() => {
      this.createPlotly();
    });
  }

  private async createPlotly() {
    const plotly = await Plotly.newPlot('chart', data);

    plotly.on('plotly_click', (event: Plotly.PlotMouseEvent) => {
      // This handler will be called outside of the Angular zone because
      // the initialization logic is also called outside of the zone. To check
      // whether we're in the Angular zone, we can call the following:
      console.log(NgZone.isInAngularZone());
      this.plotlyClick.emit(event);
    });
  }
}
```

Si vous devez émettre des événements vers des composants parents et exécuter une logique précise de mise à jour de la vue, envisagez de revenir dans la zone Angular pour indiquer au framework d'exécuter la détection de changements, ou exécutez la détection de changements manuellement :

```ts {header:"Re-enter the Angular zone when dispatching event" , linenums}
import { Component, NgZone, OnInit, output, inject } from '@angular/core';
import * as Plotly from 'plotly.js-dist-min';

@Component(...)
class App implements OnInit {
  private ngZone = inject(NgZone);

  plotlyClick = output<Plotly.PlotMouseEvent>();

  ngOnInit() {
    this.ngZone.runOutsideAngular(() => {
      this.createPlotly();
    });
  }

  private async createPlotly() {
    const plotly = await Plotly.newPlot('chart', data);

    plotly.on('plotly_click', (event: Plotly.PlotMouseEvent) => {
      this.ngZone.run(() => {
        this.plotlyClick.emit(event);
      });
    });
  }
}
```

Il peut aussi arriver que des événements soient émis en dehors de la zone Angular. N'oubliez pas que déclencher la détection de changements (par exemple manuellement) peut entraîner la création ou la mise à jour de vues en dehors de la zone Angular.
