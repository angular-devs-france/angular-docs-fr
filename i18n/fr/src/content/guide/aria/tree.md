<!-- source: c0d91516886d762089700f70bc13aa99796a8061 -->
# Tree

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/treeview/" title="Pattern ARIA Tree"/>
  <docs-pill href="/api/aria/tree/Tree" title="Référence de l'API Tree"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Un tree affiche des données hiérarchiques dont les éléments peuvent se déplier pour révéler leurs enfants ou se replier pour les masquer. Les utilisateurs naviguent avec les touches fléchées, déplient et replient les nœuds et, éventuellement, sélectionnent des éléments dans des scénarios de navigation ou de sélection de données.

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.ts">
  <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.ts"/>
  <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.html"/>
  <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.css"/>
</docs-code-multifile>

## Utilisation {#usage}

Les trees conviennent bien pour afficher des données hiérarchiques dans lesquelles les utilisateurs doivent parcourir des structures imbriquées.

**Utilisez des trees lorsque vous :**

- Construisez une navigation dans un système de fichiers
- Affichez des hiérarchies de dossiers et de documents
- Créez des structures de menus imbriqués
- Affichez des organigrammes
- Parcourez des données hiérarchiques
- Implémentez une navigation de site avec des sections imbriquées

**Évitez les trees lorsque vous :**

- Affichez des listes plates (utilisez plutôt [Listbox](guide/aria/listbox))
- Affichez des tableaux de données (utilisez plutôt [Grid](guide/aria/grid))
- Créez des listes déroulantes simples (utilisez plutôt [Select](guide/aria/select))
- Construisez un fil d'Ariane (utilisez des patterns breadcrumb)

## Fonctionnalités {#features}

- **Navigation hiérarchique** - Structure arborescente imbriquée, avec dépliage et repliage
- **Modes de sélection** - Sélection unique ou multiple, avec un comportement explicite ou qui suit le focus
- **La sélection suit le focus** - Sélection automatique facultative lorsque le focus change
- **Navigation au clavier** - Touches fléchées, Début, Fin et recherche par saisie (type-ahead)
- **Déplier/replier** - Flèches Droite/Gauche ou Entrée pour basculer les nœuds parents
- **Éléments désactivés** - Désactivez des nœuds spécifiques avec gestion du focus
- **Modes de focus** - Stratégies de focus roving tabindex ou activedescendant
- **Prise en charge RTL** - Navigation pour les langues de droite à gauche

## Exemples {#examples}

### Tree de navigation {#navigation-tree}

Utilisez un tree pour la navigation lorsque le clic sur les éléments déclenche des actions plutôt que de les sélectionner.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/nav/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/nav/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/nav/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/nav/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Définissez `[nav]="true"` pour activer le mode navigation. Celui-ci utilise `aria-current` pour indiquer la page courante au lieu de la sélection.

### Sélection unique {#single-selection}

Activez la sélection unique pour les scénarios où les utilisateurs choisissent un seul élément du tree.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/single-select/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/single-select/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/single-select/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/single-select/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/single-select/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Laissez `[multi]="false"` (la valeur par défaut) pour une sélection unique. Les utilisateurs appuient sur Espace pour sélectionner l'élément qui a le focus.

### Sélection multiple {#multi-selection}

Permettez aux utilisateurs de sélectionner plusieurs éléments du tree.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/multi-select/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/multi-select/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/multi-select/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/multi-select/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/multi-select/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/multi-select/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/multi-select/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/multi-select/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Définissez `[multi]="true"` sur le tree. Les utilisateurs sélectionnent les éléments un par un avec Espace, ou des plages avec Maj+touches fléchées.

### La sélection suit le focus {#selection-follows-focus}

Lorsque la sélection suit le focus, l'élément qui a le focus est automatiquement sélectionné. Cela simplifie l'interaction dans les scénarios de navigation.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/single-select-follow-focus/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Définissez `[selectionMode]="'follow'"` sur le tree. La sélection se met à jour automatiquement à mesure que les utilisateurs naviguent avec les touches fléchées.

### Éléments de tree désactivés {#disabled-tree-items}

Désactivez des nœuds spécifiques du tree pour empêcher toute interaction. Vous contrôlez si les éléments désactivés peuvent recevoir le focus.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/disabled-focusable/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/disabled-focusable/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/disabled-focusable/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/disabled-focusable/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tree/src/disabled-focusable/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/tree/src/disabled-focusable/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/tree/src/disabled-focusable/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/tree/src/disabled-focusable/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Lorsque `[softDisabled]="true"` est défini sur le tree, les éléments désactivés peuvent recevoir le focus mais ne peuvent être ni activés ni sélectionnés. Lorsque `[softDisabled]="false"`, les éléments désactivés sont ignorés lors de la navigation au clavier.

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants de tree.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {TreeHarness} from '@angular/aria/tree/testing';
import {MyTreeComponent} from './my-tree'; // Your component

describe('MyTreeComponent', () => {
  let fixture: ComponentFixture<MyTreeComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyTreeComponent],
    });

    fixture = TestBed.createComponent(MyTreeComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should navigate and expand tree items', async () => {
    const tree = await loader.getHarness(TreeHarness);

    // Get top-level structure representation
    expect(await tree.getTreeStructure()).toEqual({
      children: [{text: 'public'}, {text: 'src'}, {text: 'package.json'}],
    });

    // Get all items (currently visible)
    const items = await tree.getItems();
    expect(items.length).toBe(3);

    // Expand the first item ('public')
    expect(await items[0].isExpanded()).toBe(false);
    await items[0].click();
    expect(await items[0].isExpanded()).toBe(true);

    // Verifying tree structure updates after expansion
    expect(await tree.getTreeStructure()).toEqual({
      children: [
        {
          text: 'public',
          children: [{text: 'index.html'}, {text: 'styles.css'}],
        },
        {text: 'src'},
        {text: 'package.json'},
      ],
    });
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références d'API suivantes :

- [`Tree`](/api/aria/tree/Tree)
- [`TreeItem`](/api/aria/tree/TreeItem)
- [`TreeItemGroup`](/api/aria/tree/TreeItemGroup)
