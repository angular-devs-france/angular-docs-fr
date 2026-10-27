<!-- source: b135069985978cab88b36e94b8cf04be6282d9b9 -->
# Calculs lents

À chaque cycle de détection de changements, Angular, de manière synchrone :

- Évalue toutes les expressions de template de tous les composants, sauf indication contraire, selon la stratégie de détection de chaque composant
- Exécute les lifecycle hooks `ngDoCheck`, `ngAfterContentChecked`, `ngAfterViewChecked` et `ngOnChanges`.
  Un seul calcul lent dans un template ou un lifecycle hook peut ralentir l'ensemble du processus de détection de changements, car Angular exécute les calculs séquentiellement.

## Identifier les calculs lents {#identifying-slow-computations}

Vous pouvez identifier les calculs coûteux avec le profiler d'Angular DevTools. Dans la timeline de performances, cliquez sur une barre pour afficher un cycle de détection de changements particulier. Cela affiche un graphique à barres qui montre le temps passé par le framework dans la détection de changements pour chaque composant. Lorsque vous cliquez sur un composant, vous pouvez voir combien de temps Angular a passé à évaluer son template et ses lifecycle hooks.

<img alt="Aperçu du profiler d'Angular DevTools montrant un calcul lent" src="assets/images/best-practices/runtime-performance/slow-computations.png">

Par exemple, dans la capture d'écran précédente, le deuxième cycle de détection de changements enregistré est sélectionné. Angular a passé plus de 573 ms sur ce cycle, la majeure partie du temps dans `EmployeeListComponent`. Dans le panneau de détails, vous pouvez voir qu'Angular a passé plus de 297 ms à évaluer le template de `EmployeeListComponent`.

## Optimiser les calculs lents {#optimizing-slow-computations}

Voici plusieurs techniques pour éliminer les calculs lents :

- **Optimiser l'algorithme sous-jacent**. C'est l'approche recommandée. Si vous pouvez accélérer l'algorithme à l'origine du problème, vous accélérez l'ensemble du mécanisme de détection de changements.
- **Mettre en cache avec des pipes purs**. Vous pouvez déplacer le calcul coûteux dans un [pipe](guide/templates/pipes) pur. Angular ne réévalue un pipe pur que s'il détecte que ses entrées ont changé depuis le dernier appel.
- **Utiliser la mémoïsation**. La [mémoïsation](https://en.wikipedia.org/wiki/Memoization) est une technique similaire aux pipes purs, à la différence que les pipes purs ne conservent que le dernier résultat du calcul, alors que la mémoïsation peut stocker plusieurs résultats.
- **Éviter les repaints/reflows dans les lifecycle hooks**. Certaines [opérations](https://web.dev/avoid-large-complex-layouts-and-layout-thrashing/) obligent le navigateur à recalculer de manière synchrone la mise en page de la page ou à la redessiner. Comme les reflows et les repaints sont généralement lents, il vaut mieux éviter de les effectuer à chaque cycle de détection de changements.

Les pipes purs et la mémoïsation présentent des compromis différents. Les pipes purs sont un concept intégré à Angular, alors que la mémoïsation est une pratique générale du génie logiciel pour mettre en cache les résultats de fonctions. Le surcoût mémoire de la mémoïsation peut être important si vous invoquez fréquemment le calcul coûteux avec des arguments différents.
