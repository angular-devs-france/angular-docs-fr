<!-- source: 877a6d2f9728a172d19a47b84ddec1c20c6176ec -->
# Tabs

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/tabs/" title="Pattern ARIA Tabs"/>
  <docs-pill href="/api/aria/tabs/Tabs" title="Référence de l'API Tabs"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Les tabs (onglets) affichent des sections de contenu superposées dont un seul panneau est visible à la fois. Les utilisateurs passent d'un panneau à l'autre en cliquant sur les boutons d'onglet ou en parcourant la liste d'onglets avec les touches fléchées.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

Les tabs conviennent bien pour organiser du contenu lié en sections distinctes, lorsque les utilisateurs passent d'une vue ou d'une catégorie à l'autre.

**Utilisez des tabs lorsque vous :**

- Organisez du contenu lié en sections distinctes
- Créez des panneaux de paramètres comportant plusieurs catégories
- Construisez une documentation couvrant plusieurs sujets
- Implémentez des tableaux de bord avec différentes vues
- Affichez du contenu entre lequel les utilisateurs doivent changer de contexte

**Évitez les tabs lorsque vous :**

- Construisez des formulaires séquentiels ou des assistants (utilisez un pattern stepper)
- Naviguez entre des pages (utilisez la navigation du router)
- Affichez une seule section de contenu (les tabs sont inutiles)
- Avez plus de 7 à 8 onglets (envisagez une autre mise en page)

## Fonctionnalités {#features}

- **Modes de sélection** - Les onglets s'activent automatiquement au focus ou nécessitent une activation manuelle
- **Navigation au clavier** - Touches fléchées, Début et Fin pour naviguer efficacement entre les onglets
- **Orientation** - Disposition horizontale ou verticale de la liste d'onglets
- **Contenu lazy** - Les panneaux d'onglet ne sont rendus qu'une fois activés
- **Onglets désactivés** - Désactivez des onglets individuels avec gestion du focus
- **Modes de focus** - Stratégies de focus roving tabindex ou activedescendant
- **Prise en charge RTL** - Navigation pour les langues de droite à gauche

## Exemples {#examples}

### La sélection suit le focus {#selection-follows-focus}

Lorsque la sélection suit le focus, les onglets s'activent immédiatement à mesure que vous naviguez avec les touches fléchées. Cela fournit un retour instantané et convient bien aux contenus légers.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/selection-follows-focus/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Définissez `[selectionMode]="'follow'"` sur la liste d'onglets pour activer ce comportement.

### Activation manuelle {#manual-activation}

Avec l'activation manuelle, les touches fléchées déplacent le focus entre les onglets sans changer l'onglet sélectionné. Les utilisateurs appuient sur Espace ou Entrée pour activer l'onglet qui a le focus.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/explicit-selection/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/explicit-selection/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/explicit-selection/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/explicit-selection/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/explicit-selection/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/explicit-selection/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/explicit-selection/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/explicit-selection/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/explicit-selection/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/explicit-selection/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/explicit-selection/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/explicit-selection/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Utilisez `[selectionMode]="'explicit'"` pour les panneaux au contenu lourd afin d'éviter des rendus inutiles.

### Onglets verticaux {#vertical-tabs}

Disposez les onglets verticalement pour des interfaces comme des panneaux de paramètres ou des barres latérales de navigation.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/vertical/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/vertical/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/vertical/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/vertical/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/vertical/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/vertical/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/vertical/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/vertical/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/vertical/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/vertical/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/vertical/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/vertical/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Définissez `[orientation]="'vertical'"` sur la liste d'onglets. La navigation passe alors aux touches fléchées Haut/Bas.

### Rendu lazy du contenu {#lazy-content-rendering}

Utilisez la directive `ngTabContent` sur un `ng-template` pour différer le rendu des panneaux d'onglet jusqu'à ce qu'ils soient affichés.

```angular-html
<div ngTabs>
  <ul ngTabList [(selectedTab)]="selectedTab">
    <li ngTab value="tab1">Tab 1</li>
    <li ngTab value="tab2">Tab 2</li>
  </ul>

  <div ngTabPanel value="tab1">
    <ng-template ngTabContent>
      <!-- This content only renders when Tab 1 is shown -->
      <app-heavy-component />
    </ng-template>
  </div>

  <div ngTabPanel value="tab2">
    <ng-template ngTabContent>
      <!-- This content only renders when Tab 2 is shown -->
      <app-another-component />
    </ng-template>
  </div>
</div>
```

Par défaut, le contenu est retiré du DOM lorsque le panneau est masqué. Définissez `[preserveContent]="true"` pour le conserver, par exemple lorsque le panneau contient une vidéo qui ne doit pas être réinitialisée lors du changement d'onglet.

### Onglets désactivés {#disabled-tabs}

Désactivez certains onglets pour empêcher toute interaction de l'utilisateur. Vous contrôlez si les onglets désactivés peuvent recevoir le focus clavier.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/disabled/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/disabled/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/disabled/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/disabled/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/disabled/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/disabled/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/disabled/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/disabled/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/tabs/src/disabled/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/tabs/src/disabled/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/tabs/src/disabled/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/tabs/src/disabled/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Lorsque `[softDisabled]="true"` est défini sur la liste d'onglets, les onglets désactivés peuvent recevoir le focus mais ne peuvent pas être activés. Lorsque `[softDisabled]="false"`, les onglets désactivés sont ignorés lors de la navigation au clavier.

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants de tabs.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {ComponentHarness, HarnessLoader} from '@angular/cdk/testing';
import {TabsHarness} from '@angular/aria/tabs/testing';
import {MyTabsComponent} from './my-tabs'; // Your component

// A simple harness to help query content inside the tab panel
class TestContentHarness extends ComponentHarness {
  static hostSelector = '.test-content';
  async getText(): Promise<string> {
    return (await this.host()).text();
  }
}

describe('MyTabsComponent', () => {
  let fixture: ComponentFixture<MyTabsComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyTabsComponent],
    });

    fixture = TestBed.createComponent(MyTabsComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should switch tabs and scope panel queries', async () => {
    const tabs = await loader.getHarness(TabsHarness);

    // Get all tabs
    const tabItems = await tabs.getTabs();
    expect(tabItems.length).toBe(3);

    // Verify initial selection
    expect(await tabItems[0].isSelected()).toBe(true);
    expect(await tabItems[1].isSelected()).toBe(false);

    // Query content inside the active tab panel
    // TabHarness automatically scopes queries to its associated panel
    const content = await tabItems[0].getHarness(TestContentHarness);
    expect(await content.getText()).toBe('Content 1');

    // Switch to the second tab
    await tabItems[1].select();

    // Verify selection updated
    expect(await tabItems[0].isSelected()).toBe(false);
    expect(await tabItems[1].isSelected()).toBe(true);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références d'API suivantes :

- [`Tabs`](/api/aria/tabs/Tabs)
- [`TabList`](/api/aria/tabs/TabList)
- [`Tab`](/api/aria/tabs/Tab)
- [`TabPanel`](/api/aria/tabs/TabPanel)
- [`TabContent`](/api/aria/tabs/TabContent)
