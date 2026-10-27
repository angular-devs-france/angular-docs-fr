<!-- source: 1cd3e8ae3a57ae3c523d1cbce9cbf4dda5363a3f -->
# Menubar

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/menubar/" title="Pattern ARIA Menubar"/>
  <docs-pill href="/api/aria/menu/MenuBar" title="Référence de l'API Menubar"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

La menubar est une barre de navigation horizontale qui donne un accès permanent aux menus de l'application. Les menubars organisent les commandes en catégories logiques comme Fichier, Édition et Affichage, ce qui aide les utilisateurs à découvrir et à exécuter les fonctionnalités de l'application au clavier ou à la souris.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

Les menubars conviennent bien pour organiser les commandes d'une application dans une navigation permanente et facile à découvrir.

**Utilisez une menubar pour :**

- Construire des barres de commandes d'application (comme Fichier, Édition, Affichage, Insertion, Format)
- Créer une navigation permanente qui reste visible dans toute l'interface
- Organiser les commandes en catégories logiques de premier niveau
- Obtenir une navigation horizontale dans les menus avec prise en charge du clavier
- Construire des interfaces d'application de type bureau

**Évitez les menubars pour :**

- Construire des menus déroulants pour des actions individuelles (utilisez plutôt un [menu avec déclencheur](guide/aria/menu))
- Créer des menus contextuels (utilisez le pattern du guide [Menu](guide/aria/menu))
- De simples listes d'actions standalone (utilisez plutôt un [menu](guide/aria/menu))
- Les interfaces mobiles où l'espace horizontal est limité
- Une navigation qui relève d'un pattern de barre latérale ou d'en-tête

## Fonctionnalités {#features}

- **Navigation horizontale** - Les touches fléchées gauche/droite permettent de passer d'une catégorie de premier niveau à l'autre
- **Visibilité permanente** - Toujours visible, ni modale ni masquable
- **Ouverture au survol** - Les sous-menus s'ouvrent au survol après une première interaction au clavier ou au clic
- **Sous-menus imbriqués** - Prise en charge de plusieurs niveaux de profondeur de menu
- **Navigation au clavier** - Touches fléchées, Entrée/Espace, Échap et recherche par saisie (typeahead)
- **États désactivés** - Désactivez la menubar entière ou des éléments individuels
- **Prise en charge du RTL** - Navigation automatique pour les langues de droite à gauche

## Exemples {#examples}

### Menubar simple {#basic-menubar}

Une menubar donne un accès permanent aux commandes de l'application, organisées en catégories de premier niveau. Les utilisateurs naviguent entre les catégories avec les flèches gauche/droite et ouvrent les menus avec Entrée ou la flèche bas.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Appuyez sur la flèche droite pour passer de Fichier à Édition puis à Affichage. Appuyez sur Entrée ou sur la flèche bas pour ouvrir un menu, puis parcourez les éléments du sous-menu avec les flèches haut/bas.

### Éléments de menubar désactivés {#disabled-menubar-items}

Désactivez des éléments de menu spécifiques ou la menubar entière pour empêcher toute interaction. Contrôlez si les éléments désactivés peuvent recevoir le focus clavier avec l'input `softDisabled`.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/disabled/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/disabled/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/disabled/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/disabled/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/disabled/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/disabled/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/disabled/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/disabled/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/disabled/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/disabled/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/disabled/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/disabled/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Avec `[softDisabled]="true"` sur la menubar, les éléments désactivés peuvent recevoir le focus mais ne peuvent pas être activés. Avec `[softDisabled]="false"`, les éléments désactivés sont ignorés pendant la navigation au clavier.

### Prise en charge du RTL {#rtl-support}

Les menubars s'adaptent automatiquement aux langues de droite à gauche (RTL). La navigation avec les touches fléchées s'inverse et les sous-menus se positionnent sur le côté gauche.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/rtl/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/rtl/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/rtl/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/rtl/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/rtl/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/rtl/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/rtl/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/rtl/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/menubar/src/rtl/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/menubar/src/rtl/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/menubar/src/rtl/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/menubar/src/rtl/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

L'attribut `dir="rtl"` active le mode RTL. La flèche gauche déplace vers la droite et la flèche droite vers la gauche, ce qui préserve une navigation naturelle pour les utilisateurs de langues RTL.

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants menubar.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {MenuHarness} from '@angular/aria/menu/testing';
import {MyMenubarComponent} from './my-menubar'; // Your component

describe('MyMenubarComponent', () => {
  let fixture: ComponentFixture<MyMenubarComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyMenubarComponent],
    });

    fixture = TestBed.createComponent(MyMenubarComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should interact with menubar items', async () => {
    // Load the menubar harness (which is a MenuHarness with selector '[ngMenuBar]')
    const menubar = await loader.getHarness(MenuHarness.with({selector: '[ngMenuBar]'}));

    // Menubars are persistent and always "open"
    expect(await menubar.isOpen()).toBe(true);
    expect(await menubar.isMenuBar()).toBe(true);

    // Get top-level items
    const items = await menubar.getItems();
    expect(items.length).toBe(2);
    expect(await items[0].getText()).toBe('File');
    expect(await items[1].getText()).toBe('Edit');

    // Click an item to open its dropdown menu
    await items[0].click();

    const fileMenu = await items[0].getSubmenu();
    expect(fileMenu).toBeTruthy();
    expect(await fileMenu!.isOpen()).toBe(true);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références suivantes :

- [`MenuBar`](/api/aria/menu/MenuBar)
- [`MenuItem`](/api/aria/menu/MenuItem)
- [`MenuTrigger`](/api/aria/menu/MenuTrigger)
- [`Menu`](/api/aria/menu/Menu)
