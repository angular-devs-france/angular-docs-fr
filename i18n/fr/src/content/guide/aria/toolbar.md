<!-- source: 03640eb4fa4aeaaae5dd79c269a6734445b76a04 -->
# Toolbar

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/toolbar/" title="Pattern ARIA Toolbar"/>
  <docs-pill href="/api/aria/toolbar/Toolbar" title="Référence de l'API Toolbar"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Un conteneur qui regroupe des contrôles et des actions liés avec navigation au clavier, couramment utilisé pour la mise en forme de texte, les barres d'outils et les panneaux de commandes.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

Une toolbar convient le mieux pour regrouper des contrôles liés auxquels les utilisateurs accèdent fréquemment. Envisagez une toolbar lorsque :

- **Plusieurs actions liées** - Vous avez plusieurs contrôles qui remplissent des fonctions liées (comme des boutons de mise en forme de texte)
- **L'efficacité au clavier compte** - Les utilisateurs profitent d'une navigation rapide au clavier avec les touches fléchées
- **Contrôles groupés** - Vous devez organiser les contrôles en sections logiques avec des séparateurs
- **Accès fréquent** - Les contrôles sont utilisés de façon répétée au sein d'un flux de travail

Évitez une toolbar lorsque :

- Un simple groupe de boutons suffit - Pour seulement 2 ou 3 actions sans lien entre elles, des boutons individuels fonctionnent mieux
- Les contrôles ne sont pas liés - Une toolbar implique un regroupement logique ; des contrôles sans lien entre eux déroutent les utilisateurs
- Navigation imbriquée complexe - Les hiérarchies profondes sont mieux servies par des menus ou des composants de navigation

## Fonctionnalités {#features}

La toolbar d'Angular fournit une implémentation de toolbar entièrement accessible avec :

- **Navigation au clavier** - Parcourez les widgets avec les touches fléchées, Début et Fin grâce au roving tabindex, tandis que les contrôles conservent leur activation native
- **Prise en charge des lecteurs d'écran** - Attributs ARIA intégrés pour les technologies d'assistance
- **Groupes de widgets** - Organisez des widgets liés, comme des groupes de boutons radio ou de boutons bascule
- **Orientation flexible** - Dispositions horizontales ou verticales avec navigation au clavier automatique
- **Réactivité à base de signals** - Gestion d'état réactive avec les signals Angular
- **Prise en charge du texte bidirectionnel** - Gère automatiquement les langues de droite à gauche (RTL)
- **Focus configurable** - Choisissez entre une navigation qui boucle ou des arrêts nets aux extrémités

## Exemples {#examples}

### Toolbar horizontale de base {#basic-horizontal-toolbar}

Les toolbars horizontales organisent les contrôles de gauche à droite, comme c'est courant dans les éditeurs de texte et les outils de design. Les touches fléchées permettent de passer d'un widget à l'autre, en gardant le focus dans la toolbar jusqu'à ce que l'utilisateur appuie sur Tab pour passer à l'élément suivant de la page.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

### Toolbar verticale {#vertical-toolbar}

Les toolbars verticales empilent les contrôles de haut en bas, ce qui est utile pour les panneaux latéraux ou les palettes de commandes verticales. Les touches fléchées haut et bas permettent de passer d'un widget à l'autre.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/vertical/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/vertical/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/vertical/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/vertical/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/vertical/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/vertical/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/vertical/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/vertical/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/vertical/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/vertical/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/vertical/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/vertical/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

### Groupes de widgets {#widget-groups}

Les groupes de widgets organisent des contrôles liés qui fonctionnent ensemble, comme des options d'alignement de texte ou des bascules de mise en forme. Les groupes conservent la navigation par roving tabindex tout en présentant la structure sémantique appropriée aux technologies d'assistance.

Dans les exemples ci-dessus, les boutons d'alignement sont enveloppés dans `ngToolbarWidgetGroup` avec `role="radiogroup"`. La sélection est découplée du conteneur de la toolbar, ce qui vous permet de gérer l'état avec des signals Angular ou des directives personnalisées :

```angular-html
<!-- Mutually exclusive radio group -->
<div ngToolbarWidgetGroup role="radiogroup" aria-label="Alignment">
  <button
    ngToolbarWidget
    role="radio"
    type="button"
    [attr.aria-checked]="alignment() === 'left'"
    (click)="alignment.set('left')"
  >
    Left
  </button>
  <button
    ngToolbarWidget
    role="radio"
    type="button"
    [attr.aria-checked]="alignment() === 'center'"
    (click)="alignment.set('center')"
  >
    Center
  </button>
  <button
    ngToolbarWidget
    role="radio"
    type="button"
    [attr.aria-checked]="alignment() === 'right'"
    (click)="alignment.set('right')"
  >
    Right
  </button>
</div>

<!-- Toggle button group -->
<div class="group" role="group" aria-label="Text styling">
  <button ngToolbarWidget type="button" [attr.aria-pressed]="bold()" (click)="bold.set(!bold())">
    Bold
  </button>
  <button
    ngToolbarWidget
    type="button"
    [attr.aria-pressed]="italic()"
    (click)="italic.set(!italic())"
  >
    Italic
  </button>
</div>
```

### Widgets désactivés {#disabled-widgets}

Les toolbars prennent en charge deux modes de désactivation :

1. Les widgets **soft-disabled** restent focusables mais indiquent visuellement qu'ils ne sont pas disponibles
2. Les widgets **hard-disabled** sont entièrement retirés de la navigation au clavier.

Par défaut, `softDisabled` vaut `true`, ce qui permet aux widgets désactivés de recevoir encore le focus. Pour activer le mode hard-disabled, définissez `[softDisabled]="false"` sur la toolbar.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/disabled/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/disabled/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/disabled/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/disabled/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/disabled/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/disabled/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/disabled/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/disabled/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/disabled/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/disabled/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/disabled/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/disabled/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

### Prise en charge de la lecture de droite à gauche (RTL) {#right-to-left-rtl-support}

Les toolbars prennent automatiquement en charge les langues de droite à gauche. Enveloppez la toolbar dans un conteneur avec `dir="rtl"` pour inverser la disposition et le sens de la navigation au clavier. La navigation avec les touches fléchées s'ajuste automatiquement : la flèche gauche passe au widget suivant, la flèche droite au précédent.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/rtl/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/rtl/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/rtl/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/rtl/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/rtl/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/rtl/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/rtl/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/rtl/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/rtl/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/rtl/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/rtl/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/rtl/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants de toolbar.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {ToolbarHarness} from '@angular/aria/toolbar/testing';
import {MyToolbarComponent} from './my-toolbar'; // Your component

describe('MyToolbarComponent', () => {
  let fixture: ComponentFixture<MyToolbarComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyToolbarComponent],
    });

    fixture = TestBed.createComponent(MyToolbarComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should have widgets and update toggle state on click', async () => {
    // Load the toolbar harness
    const toolbar = await loader.getHarness(ToolbarHarness);

    // Get all widgets
    const widgets = await toolbar.getWidgets();
    expect(widgets.length).toBe(3);

    // Click the first widget
    await widgets[0].click();

    // Verify pressed state updated via click handler
    expect(await widgets[0].isSelected()).toBe(true);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références d'API suivantes :

- [`Toolbar`](/api/aria/toolbar/Toolbar)
- [`ToolbarWidget`](/api/aria/toolbar/ToolbarWidget)
- [`ToolbarWidgetGroup`](/api/aria/toolbar/ToolbarWidgetGroup)
