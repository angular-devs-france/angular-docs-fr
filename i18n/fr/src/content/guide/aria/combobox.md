<!-- source: 09841bccb0aa863e5e0ba5615509ab511a6e41e7 -->
# Combobox

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/combobox/" title="Pattern ARIA Combobox"/>
  <docs-pill href="/api?query=combobox#angular_aria_combobox" title="Référence de l'API Combobox"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Une directive qui coordonne un élément déclencheur (comme un champ de saisie, un bouton ou une `div`) avec une popup, et qui fournit la directive primitive des patterns autocomplete, select et multiselect.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/manual/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/manual/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/manual/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/manual/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/manual/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/manual/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/manual/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/manual/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/manual/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/manual/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/manual/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/manual/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

La combobox est la directive primitive qui coordonne un élément déclencheur interactif (comme un champ de saisie, un bouton ou une `div`) avec une popup. Elle sert de base aux patterns autocomplete, select et multiselect. Envisagez d'utiliser directement la combobox pour :

- **Créer des patterns autocomplete personnalisés** - Concevoir un comportement de filtrage ou de suggestion spécialisé
- **Créer des composants de sélection personnalisés** - Développer des listes déroulantes aux exigences particulières
- **Coordonner un champ de saisie avec une popup** - Associer un champ de saisie à un contenu listbox, tree ou dialog
- **Implémenter un filtrage personnalisé** - Filtrer et orchestrer les options correspondantes dans votre propre code

Utilisez plutôt les patterns documentés lorsque :

- Vous avez besoin d'une autocomplete standard avec filtrage - Consultez le [pattern Autocomplete](guide/aria/autocomplete) pour des exemples prêts à l'emploi
- Vous avez besoin de listes déroulantes à sélection simple - Consultez le [pattern Select](guide/aria/select) pour une implémentation complète de liste déroulante
- Vous avez besoin de listes déroulantes à sélection multiple - Consultez le [pattern Multiselect](guide/aria/multiselect) pour une sélection multiple avec affichage compact

NOTE: Les guides [Autocomplete](guide/aria/autocomplete), [Select](guide/aria/select) et [Multiselect](guide/aria/multiselect) présentent des patterns documentés qui combinent cette directive avec [Listbox](guide/aria/listbox) pour des cas d'usage spécifiques.

## Fonctionnalités {#features}

La combobox d'Angular fournit un système de coordination entre champ de saisie et popup entièrement accessible, avec :

- **Élément déclencheur avec popup** - Coordonne l'élément déclencheur avec le contenu de la popup
- **Coordination flexible** - S'intègre parfaitement aux mises en page standard (listbox, tree, grid ou dialog)
- **Navigation au clavier** - Gestion des touches fléchées, d'Entrée et d'Échap
- **Prise en charge des lecteurs d'écran** - Attributs ARIA intégrés, notamment `role="combobox"` et aria-expanded
- **Gestion de la popup** - Affichage et masquage automatiques selon les interactions de l'utilisateur
- **Réactivité basée sur les signals** - Gestion d'état réactive avec les signals d'Angular

## Exemples {#examples}

### Autocomplete {#autocomplete}

Un champ de saisie accessible qui filtre et suggère des options pendant la frappe, pour aider les utilisateurs à trouver et sélectionner des valeurs dans une liste.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Le filtrage est géré dans votre propre code en mettant à jour un signal qui filtre la liste d'options de manière réactive. Les utilisateurs naviguent avec les touches fléchées et sélectionnent avec Entrée ou un clic. Cela offre un contrôle total et une flexibilité maximale pour une logique de sélection personnalisée. Consultez le [guide Autocomplete](guide/aria/autocomplete) pour des patterns de filtrage et des exemples complets.

### Mode en lecture seule {#readonly-mode}

Un pattern qui combine une combobox en lecture seule avec une listbox pour créer des listes déroulantes à sélection simple, avec navigation au clavier et prise en charge des lecteurs d'écran.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/icons/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/icons/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/icons/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/icons/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/icons/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/icons/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/icons/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/icons/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/icons/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Pour ouvrir une liste déroulante sans saisie de texte, vous pouvez utiliser un bouton comme déclencheur host, ou appliquer l'attribut HTML natif `readonly` au champ de saisie déclencheur. La popup s'ouvre au clic ou avec les touches fléchées.

Cette configuration sert de base aux patterns [Select](guide/aria/select) et [Multiselect](guide/aria/multiselect). Consultez ces guides pour des implémentations complètes de listes déroulantes avec déclencheurs et positionnement de l'overlay.

### Grid de sélecteur de date {#datepicker-grid}

La combobox peut se coordonner avec une grid à deux dimensions pour créer des sélecteurs de date accessibles. Les utilisateurs parcourent les dates dans le tableau de la grid du calendrier avec les touches fléchées directionnelles et confirment la sélection par un clic, Entrée ou la barre d'espace.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/combobox/src/datepicker/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/combobox/src/datepicker/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/combobox/src/datepicker/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/combobox/src/datepicker/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/combobox/src/datepicker/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/combobox/src/datepicker/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/combobox/src/datepicker/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/combobox/src/datepicker/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/combobox/src/datepicker/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/combobox/src/datepicker/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/combobox/src/datepicker/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/combobox/src/datepicker/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

### Popup dialog {#dialog-popup}

Les popups dialog combinent le déclencheur de la combobox avec des mises en page de dialog standard et des pièges à focus (comme `cdkTrapFocus` du CDK). Utilisez des popups dialog lorsque l'overlay nécessite un comportement modal ou une interaction avec l'arrière-plan (backdrop).

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/combobox/src/dialog/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/combobox/src/dialog/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/combobox/src/dialog/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/combobox/src/dialog/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/combobox/src/dialog/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/combobox/src/dialog/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/combobox/src/dialog/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/combobox/src/dialog/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/combobox/src/dialog/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/combobox/src/dialog/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/combobox/src/dialog/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/combobox/src/dialog/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Tests {#testing}

Angular Aria fournit un `ComboboxHarness` pour tester les composants combobox.
Voici un exemple d'utilisation du harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {ComboboxHarness} from '@angular/aria/combobox/testing';
import {MyComboboxComponent} from './my-combobox'; // Your component

describe('MyComboboxComponent', () => {
  let fixture: ComponentFixture<MyComboboxComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyComboboxComponent],
    });

    fixture = TestBed.createComponent(MyComboboxComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should allow opening and closing the popup', async () => {
    const combobox = await loader.getHarness(ComboboxHarness);

    // Verify initial state
    expect(await combobox.isOpen()).toBe(false);

    // Open the popup
    await combobox.open();
    expect(await combobox.isOpen()).toBe(true);

    // Close the popup
    await combobox.close();
    expect(await combobox.isOpen()).toBe(false);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références d'API suivantes :

- [`Combobox`](/api/aria/combobox/Combobox)
- [`ComboboxPopup`](/api/aria/combobox/ComboboxPopup)
- [`ComboboxWidget`](/api/aria/combobox/ComboboxWidget)

### Patterns et directives associés {#related-patterns-and-directives}

La combobox est la directive primitive de ces patterns documentés :

- [Autocomplete](guide/aria/autocomplete) - Pattern de filtrage et de suggestions (coordonne la saisie dans le champ avec la liste d'options)
- [Select](guide/aria/select) - Pattern de liste déroulante à sélection simple (appliqué directement sur des boutons déclencheurs non modifiables)
- [Multiselect](guide/aria/multiselect) - Pattern de sélection multiple (appliqué sur des déclencheurs non modifiables avec une Listbox multi-sélection)

La combobox se combine généralement avec :

- [Listbox](guide/aria/listbox) - Contenu de popup le plus courant
- [Tree](guide/aria/tree) - Contenu de popup hiérarchique (consultez le guide Tree pour des exemples)
