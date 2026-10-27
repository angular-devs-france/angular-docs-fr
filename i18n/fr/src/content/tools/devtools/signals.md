<!-- source: cdc1a53ff2c0b45d26dc3bd2bfcdd7eb266519e5 -->
# Déboguer le graphe de signals

L'onglet **Components** peut afficher un graphe de signals en temps réel pour le composant ou la directive sélectionné.
Le graphe montre comment les signals, les computeds, les effects et les autres nœuds réactifs de ce composant dépendent les uns des autres, ce qui facilite la compréhension et le débogage des flux de données réactifs.

<img src="assets/images/guide/devtools/signal-graph.png" alt="Une capture d'écran de l'onglet 'Components' d'Angular DevTools avec le graphe de signals ouvert pour le composant sélectionné. Le graphe montre un nœud signal relié par des arêtes à trois nœuds computed lus par le template du composant, et les propriétés du composant sont listées dans une barre latérale à droite.">

## Ouvrir le graphe de signals {#open-the-signal-graph}

Dans l'onglet **Components**, sélectionnez un composant ou une directive.
Dans l'en-tête du panneau des propriétés, cliquez sur **Show Signal Graph**.
Le graphe de signals s'ouvre à côté de l'arbre de composants.

Vous pouvez aussi accéder directement à un signal précis : dans la vue des propriétés, cliquez sur le bouton du graphe de signals à côté de la propriété ou, lorsque la propriété propose plusieurs actions, ouvrez son menu d'actions et choisissez **Show in signal graph**.

Le bouton **Show Signal Graph** apparaît automatiquement lorsque l'application inspectée prend en charge l'API du graphe de signals : il n'y a aucun paramètre à activer.

## Lire le graphe {#read-the-graph}

Les nœuds du graphe représentent les nœuds réactifs qui participent au composant sélectionné :

- **Signal** : un signal modifiable créé avec `signal()`.
- **Computed** : une valeur dérivée créée avec `computed()`.
- **Effect** : un effet de bord créé avec `effect()` ou `afterRenderEffect()`.
- **Linked signal** : un signal modifiable créé avec `linkedSignal()`.
- **Template** : le template du composant, qui lit des signals lors de son rendu.
- **Resource** : les signals appartenant à une `resource()` ou à une `rxResource()` sont regroupés dans un cluster repliable. Cliquez sur **Expand** sur le nœud du cluster pour voir les nœuds individuels qu'il contient.

Les lignes entre les nœuds représentent les dépendances réactives : chaque nœud est relié aux producteurs qu'il lit (en amont) et aux dépendants qui le lisent (en aval).

TIP: Les nœuds sont libellés avec le `debugName` de leur nœud réactif. Donner un `debugName` à vos signals, à vos computeds et surtout à vos effects, qui ne sont généralement pas assignés à un membre de classe, rend le graphe beaucoup plus lisible :

```ts
effect(
  () => {
    // ...
  },
  {debugName: 'name in graph'},
);
```

## Rechercher dans le graphe {#search-the-graph}

Cliquez sur l'icône de recherche dans la barre d'outils du graphe pour trouver un nœud par son nom.
Angular DevTools met en évidence chaque correspondance et vous permet de les parcourir une par une.

## Inspecter un nœud {#inspect-a-node}

Cliquez sur un nœud pour ouvrir son panneau de détails, qui affiche la valeur actuelle du nœud sous forme d'arbre dépliable et propose les actions suivantes :

- **View source** : accédez à la définition du nœud dans l'onglet Sources (Chrome) ou l'onglet Debugger (Firefox). L'action est désactivée lorsque l'emplacement dans le code source n'est pas disponible.
- **Set breakpoint** : mettez l'exécution en pause lorsque le signal est mis à jour. Les points d'arrêt ne sont disponibles que dans les navigateurs basés sur Chromium.
- **Watch signal** : journalisez les mises à jour et les invalidations du signal dans la console du navigateur. Cliquez de nouveau sur l'action pour arrêter la surveillance.
- **Highlight upstream / downstream** : tracez les chemins des dépendances du nœud ou de ses dépendants.

Si le composant n'utilise aucun signal, Angular DevTools affiche le message « No signals in this component » à la place du graphe.

## Versions prises en charge {#version-support}

Le graphe de signals nécessite une application compilée avec Angular 19 ou une version ultérieure, exécutée en mode développement.
