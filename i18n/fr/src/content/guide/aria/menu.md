<!-- source: 5b82654a87494822b3bb3ee6016417ce3f8dab10 -->
# Menu

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/menubar/" title="Pattern ARIA Menu"/>
  <docs-pill href="/api/aria/menu/Menu" title="Référence de l'API Menu"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Un menu propose une liste d'actions ou d'options aux utilisateurs et apparaît généralement en réponse à un clic sur un bouton ou à un clic droit. Les menus prennent en charge la navigation au clavier avec les touches fléchées, les sous-menus, les cases à cocher, les boutons radio et les éléments désactivés.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

Les menus conviennent bien pour présenter des listes d'actions ou de commandes parmi lesquelles les utilisateurs peuvent choisir.

**Utilisez un menu pour :**

- Construire des menus de commandes d'application (Fichier, Édition, Affichage)
- Créer des menus contextuels (actions au clic droit)
- Afficher des listes d'actions déroulantes
- Implémenter des listes déroulantes dans une toolbar
- Organiser des paramètres ou des options

**Évitez les menus pour :**

- Construire la navigation d'un site (utilisez plutôt des landmarks de navigation)
- Créer des listes de sélection dans un formulaire (utilisez le composant [Select](guide/aria/select))
- Basculer entre des panneaux de contenu (utilisez des [tabs](guide/aria/tabs))
- Afficher du contenu repliable (utilisez un [accordion](guide/aria/accordion))

## Fonctionnalités {#features}

- **Navigation au clavier** - Touches fléchées, Home/End et recherche par caractère pour une navigation efficace
- **Sous-menus** - Prise en charge des menus imbriqués avec positionnement automatique
- **Types de menus** - Menus standalone, menus avec déclencheur et menubars
- **Cases à cocher et boutons radio** - Éléments de menu à bascule et de sélection
- **Éléments désactivés** - États désactivés souples ou stricts avec gestion du focus
- **Fermeture automatique** - Fermeture configurable lors de la sélection
- **Prise en charge du RTL** - Navigation pour les langues de droite à gauche

## Exemples {#examples}

### Menu avec déclencheur {#menu-with-trigger}

Créez un menu déroulant en associant un bouton déclencheur à un menu. Le déclencheur ouvre et ferme le menu.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Le menu se ferme automatiquement lorsque l'utilisateur sélectionne un élément ou appuie sur Échap.

### Menu contextuel {#context-menu}

Les menus contextuels apparaissent à la position du curseur lorsque les utilisateurs font un clic droit sur un élément.

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-context/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-context/app/app.ts"/>
  <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-context/app/app.html"/>
</docs-code-multifile>

Positionnez le menu à l'aide des coordonnées de l'événement `contextmenu`.

### Menu standalone {#standalone-menu}

Un menu standalone ne nécessite pas de déclencheur et reste visible dans l'interface.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-standalone/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-standalone/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-standalone/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-standalone/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-standalone/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-standalone/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-standalone/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-standalone/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-standalone/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-standalone/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-standalone/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-standalone/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Les menus standalone conviennent bien aux listes d'actions ou à une navigation toujours visibles.

### Éléments de menu désactivés {#disabled-menu-items}

Désactivez des éléments de menu spécifiques avec l'input `disabled`. Contrôlez le comportement du focus avec `softDisabled`.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menu/src/menu-trigger-disabled/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Avec `[softDisabled]="true"`, les éléments désactivés peuvent recevoir le focus mais ne peuvent pas être activés. Avec `[softDisabled]="false"`, les éléments désactivés sont ignorés pendant la navigation au clavier.

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants menu.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {MenuHarness, MenuItemHarness} from '@angular/aria/menu/testing';
import {MyMenuComponent} from './my-menu'; // Your component

describe('MyMenuComponent', () => {
  let fixture: ComponentFixture<MyMenuComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyMenuComponent],
    });

    fixture = TestBed.createComponent(MyMenuComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should open menu and click item', async () => {
    // Load the menu harness by its trigger text
    const menu = await loader.getHarness(MenuHarness.with({triggerText: 'Open Menu'}));

    // Verify initial state
    expect(await menu.isOpen()).toBe(false);

    // Open the menu
    await menu.open();
    expect(await menu.isOpen()).toBe(true);

    // Get items
    const items = await menu.getItems();
    expect(items.length).toBe(3);
    expect(await items[0].getText()).toBe('Item 1');

    // Click first item
    await items[0].click();

    // Menu should close after selection (depending on your implementation)
    expect(await menu.isOpen()).toBe(false);
  });

  it('should interact with submenus', async () => {
    const menu = await loader.getHarness(MenuHarness.with({triggerText: 'Open Menu'}));
    await menu.open();

    // Get the item that triggers a submenu
    const subItem = await loader.getHarness(MenuItemHarness.with({text: 'Submenu'}));
    expect(await subItem.hasSubmenu()).toBe(true);

    // Open submenu
    await subItem.click();
    const submenu = await subItem.getSubmenu();
    expect(submenu).toBeTruthy();
    expect(await submenu!.isOpen()).toBe(true);

    // Interact with submenu items
    const subItems = await submenu!.getItems();
    expect(subItems.length).toBe(1);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références suivantes :

- [`Menu`](/api/aria/menu/Menu)
- [`MenuBar`](/api/aria/menu/MenuBar)
- [`MenuItem`](/api/aria/menu/MenuItem)
- [`MenuTrigger`](/api/aria/menu/MenuTrigger)
- [`MenuContent`](/api/aria/menu/MenuContent)
