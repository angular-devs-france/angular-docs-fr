<!-- source: 3cc5621c9e475f55ab9d8b9e36c9211ef41cc532 -->
# Grid

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/grid/" title="Pattern ARIA Grid"/>
  <docs-pill href="/api?query=grid#angular_aria_grid" title="Référence de l'API Grid"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Une grid permet aux utilisateurs de naviguer dans des données ou des éléments interactifs à deux dimensions à l'aide des touches fléchées, Home, End et Page Up/Down. Les grids conviennent aux tableaux de données, aux calendriers, aux feuilles de calcul et aux mises en page qui regroupent des éléments interactifs liés.

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/overview/basic/app/app.ts">
  <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/overview/basic/app/app.ts"/>
  <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/overview/basic/app/app.html"/>
  <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/overview/basic/app/app.css"/>
</docs-code-multifile>

## Utilisation {#usage}

Les grids conviennent bien aux données ou aux éléments interactifs organisés en lignes et en colonnes, lorsque les utilisateurs ont besoin de naviguer au clavier dans plusieurs directions.

**Utilisez une grid pour :**

- Construire des tableaux de données interactifs avec des cellules modifiables ou sélectionnables
- Créer des calendriers ou des sélecteurs de date
- Implémenter des interfaces de type feuille de calcul
- Regrouper des éléments interactifs (boutons, cases à cocher) afin de réduire le nombre d'arrêts de tabulation sur une page
- Construire des interfaces qui nécessitent une navigation au clavier à deux dimensions

**Évitez les grids pour :**

- Afficher de simples tableaux en lecture seule (utilisez plutôt l'élément HTML sémantique `<table>`)
- Afficher des listes à une seule colonne (utilisez plutôt une [listbox](guide/aria/listbox))
- Afficher des données hiérarchiques (utilisez plutôt un [tree](guide/aria/tree))
- Construire des formulaires sans mise en page tabulaire (utilisez des contrôles de formulaire standards)

## Fonctionnalités {#features}

- **Navigation à deux dimensions** - Les touches fléchées déplacent le focus entre les cellules dans toutes les directions
- **Modes de focus** - Choisissez entre les stratégies de focus roving tabindex ou activedescendant
- **Prise en charge de la sélection** - Sélection de cellules facultative, en mode simple ou multiple
- **Comportement de bouclage** - Configurez la façon dont la navigation boucle aux bords de la grid (continuous, loop ou nowrap)
- **Sélection de plage** - Sélectionnez plusieurs cellules avec les touches de modification ou par glissement
- **États désactivés** - Désactivez la grid entière ou des cellules individuelles
- **Prise en charge du RTL** - Navigation automatique pour les langues de droite à gauche

## Exemples {#examples}

### Grid de tableau de données {#data-table-grid}

Utilisez une grid pour les tableaux interactifs dans lesquels les utilisateurs doivent naviguer entre les cellules avec les touches fléchées. Cet exemple montre un tableau de données simple avec navigation au clavier.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/table/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/table/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/table/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/table/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/table/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/table/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/table/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/table/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Appliquez la directive `ngGrid` à l'élément du tableau, `ngGridRow` à chaque ligne et `ngGridCell` à chaque cellule.

### Grid de calendrier {#calendar-grid}

Les calendriers sont un cas d'usage courant des grids. Cet exemple montre une vue mensuelle dans laquelle les utilisateurs naviguent entre les dates avec les touches fléchées.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/calendar/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/calendar/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/calendar/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/calendar/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/calendar/material/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/calendar/material/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/calendar/material/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/calendar/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/calendar/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/calendar/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/calendar/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/calendar/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Les utilisateurs peuvent activer une date en appuyant sur Entrée ou Espace lorsque le focus est sur une cellule.

### Grid de mise en page {#layout-grid}

Utilisez une grid de mise en page pour regrouper des éléments interactifs et réduire le nombre d'arrêts de tabulation. Cet exemple montre une grid de boutons en forme de pastilles.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/pill-list/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/pill-list/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/pill-list/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/pill-list/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/pill-list/material/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/pill-list/material/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/pill-list/material/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/pill-list/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/grid/src/pill-list/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/grid/src/pill-list/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/grid/src/pill-list/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/grid/src/pill-list/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Au lieu de parcourir chaque bouton avec Tab, les utilisateurs naviguent avec les touches fléchées et un seul bouton reçoit le focus de tabulation.

### Modes de sélection et de focus {#selection-and-focus-modes}

Activez la sélection avec `[enableSelection]="true"` et configurez la façon dont le focus et la sélection interagissent.

```angular-html
<table
  ngGrid
  [enableSelection]="true"
  [selectionMode]="'explicit'"
  [multi]="true"
  [focusMode]="'roving'"
>
  <tr ngGridRow>
    <td ngGridCell>Cell 1</td>
    <td ngGridCell>Cell 2</td>
  </tr>
</table>
```

**Modes de sélection :**

- `follow` : la cellule qui a le focus est automatiquement sélectionnée
- `explicit` : les utilisateurs sélectionnent les cellules avec Espace ou par un clic

**Modes de focus :**

- `roving` : le focus se déplace sur les cellules à l'aide de `tabindex` (préférable pour les grids simples)
- `activedescendant` : le focus reste sur le conteneur de la grid, et `aria-activedescendant` indique la cellule active (préférable pour le défilement virtuel)

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants grid.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {GridHarness} from '@angular/aria/grid/testing';
import {MyGridComponent} from './my-grid'; // Your component

describe('MyGridComponent', () => {
  let fixture: ComponentFixture<MyGridComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyGridComponent],
    });

    fixture = TestBed.createComponent(MyGridComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should read cell values and focus cells', async () => {
    const grid = await loader.getHarness(GridHarness);

    // Get all cells text in a 2D array organized by rows
    const cellTexts = await grid.getCellTextByIndex();
    expect(cellTexts).toEqual([
      ['Cell 1.1', 'Cell 1.2'],
      ['Cell 2.1', 'Cell 2.2'],
    ]);

    // Get a specific cell by text
    const cells = await grid.getCells({text: 'Cell 1.1'});
    expect(cells.length).toBe(1);
    const cell = cells[0];

    // Verify cell state
    expect(await cell.isSelected()).toBe(true);
    expect(await cell.isActive()).toBe(true);

    // Focus the cell
    await cell.focus();
    expect(await cell.isFocused()).toBe(true);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références suivantes :

- [`Grid`](/api/aria/grid/Grid)
- [`GridRow`](/api/aria/grid/GridRow)
- [`GridCell`](/api/aria/grid/GridCell)
- [`GridCellWidget`](/api/aria/grid/GridCellWidget)
