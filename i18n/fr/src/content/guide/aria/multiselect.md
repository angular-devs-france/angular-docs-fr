<!-- source: 950254c4d667f08239a35b350e0492ee55341353 -->
# Multiselect

## Vue d'ensemble {#overview}

Le pattern multiselect combine un déclencheur de combobox en lecture seule et une popup de listbox à sélection multiple pour créer des listes déroulantes à sélection multiple hautement accessibles, avec navigation au clavier et prise en charge des lecteurs d'écran.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/icons/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/icons/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/icons/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/icons/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

Le pattern multiselect convient le mieux lorsque les utilisateurs doivent choisir plusieurs éléments liés parmi un ensemble d'options familier.

Envisagez ce pattern lorsque :

- **Les utilisateurs ont besoin de plusieurs sélections** - Tags, catégories, filtres ou libellés pour lesquels plusieurs choix s'appliquent
- **La liste d'options est fixe** (moins de 20 éléments) - Les utilisateurs peuvent parcourir les options sans recherche
- **Filtrage de contenu** - Plusieurs critères peuvent être actifs simultanément
- **Attribution de propriétés** - Libellés, permissions ou fonctionnalités pour lesquels plusieurs valeurs ont du sens
- **Choix liés** - Options qui fonctionnent logiquement ensemble (comme la sélection de plusieurs membres d'une équipe)

Évitez ce pattern lorsque :

- **Une seule sélection est nécessaire** - Utilisez le [pattern Select](guide/aria/select) pour des listes déroulantes plus simples à choix unique
- **La liste compte plus de 20 éléments et nécessite une recherche** - Utilisez le [pattern Autocomplete](guide/aria/autocomplete) avec la sélection multiple
- **La plupart des options, voire toutes, seront sélectionnées** - Un pattern de liste de cases à cocher offre une meilleure visibilité
- **Les choix sont des options binaires indépendantes** - Des cases à cocher individuelles communiquent les choix plus clairement

## Fonctionnalités {#features}

Le pattern multiselect combine les directives [Combobox](guide/aria/combobox) et [Listbox](guide/aria/listbox) pour fournir une liste déroulante entièrement accessible avec :

- **Navigation au clavier** - Parcourez les options avec les touches fléchées, basculez avec Espace, fermez avec Échap
- **Prise en charge des lecteurs d'écran** - Attributs ARIA intégrés, dont aria-multiselectable
- **Affichage du nombre de sélections** - Affiche un format compact « Élément + 2 de plus » pour les sélections multiples
- **Réactivité à base de signals** - Gestion d'état réactive avec les signals Angular
- **Positionnement intelligent** - CDK Overlay gère les bords du viewport et le défilement
- **Sélection persistante** - Les options sélectionnées restent visibles avec une coche après la sélection

## Exemples {#examples}

### Multiselect de base {#basic-multiselect}

Les utilisateurs doivent sélectionner plusieurs éléments dans une liste d'options. Une combobox en lecture seule associée à une listbox à sélection multiple fournit la fonctionnalité de multiselect habituelle avec une prise en charge complète de l'accessibilité.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

L'attribute `multi` sur `ngListbox` active la sélection multiple. Appuyez sur Espace pour basculer les options : la popup reste ouverte pour d'autres sélections. L'affichage montre le premier élément sélectionné suivi du nombre de sélections restantes.

### Multiselect avec affichage personnalisé {#multiselect-with-custom-display}

Les options ont souvent besoin d'indicateurs visuels comme des icônes ou des couleurs pour aider les utilisateurs à identifier les choix. Des templates personnalisés dans les options permettent une mise en forme riche, tandis que la valeur affichée montre un résumé compact.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/icons/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/icons/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/icons/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/icons/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/icons/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/icons/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Chaque option affiche une icône à côté de son libellé. La valeur affichée se met à jour pour montrer l'icône et le texte de la première sélection, suivis du nombre de sélections supplémentaires. Les options sélectionnées affichent une coche pour un retour visuel clair.

### Sélection contrôlée {#controlled-selection}

Les formulaires doivent parfois limiter le nombre de sélections ou valider les choix des utilisateurs. Le contrôle programmatique de la sélection permet d'appliquer ces contraintes tout en préservant l'accessibilité.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/limited/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/limited/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/limited/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/limited/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/limited/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/limited/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/limited/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/limited/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/multiselect/src/limited/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/multiselect/src/limited/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/multiselect/src/limited/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/multiselect/src/limited/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Cet exemple limite la sélection à deux éléments. Lorsque la limite est atteinte, les options non sélectionnées sont désactivées pour empêcher d'autres sélections, et l'affichage de la combobox se met à jour pour refléter les choix.

## Tests {#testing}

Le pattern multiselect peut être testé en combinant `ComboboxHarness` et `ListboxHarness`, issus de `@angular/aria/combobox/testing` et `@angular/aria/listbox/testing`.
Voici un exemple d'utilisation des harnais pour tester un composant multiselect :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {ComboboxHarness} from '@angular/aria/combobox/testing';
import {ListboxHarness} from '@angular/aria/listbox/testing';
import {MyMultiselectComponent} from './my-multiselect'; // Your component

describe('MyMultiselectComponent', () => {
  let fixture: ComponentFixture<MyMultiselectComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyMultiselectComponent],
    });

    fixture = TestBed.createComponent(MyMultiselectComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should allow selecting multiple options', async () => {
    const select = await loader.getHarness(ComboboxHarness);

    // Open the dropdown
    await select.open();

    // Get the listbox harness from the popup
    const listbox = await select.getPopupWidget(ListboxHarness);
    expect(await listbox.isMulti()).toBe(true);

    const options = await listbox.getOptions();

    // Select first and second options
    await options[0].click();
    await options[1].click();

    // Verify both options are selected
    expect(await options[0].isSelected()).toBe(true);
    expect(await options[1].isSelected()).toBe(true);

    // Close the dropdown
    await select.close();

    // Verify value is updated (e.g., comma separated list or count)
    expect(await (await select.host()).text()).toContain('Option 1, Option 2');
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références d'API suivantes :

- [`Combobox`](/api/aria/combobox/Combobox)
- [`ComboboxPopup`](/api/aria/combobox/ComboboxPopup)
- [`ComboboxWidget`](/api/aria/combobox/ComboboxWidget)
- [`Listbox`](/api/aria/listbox/Listbox)
- [`Option`](/api/aria/listbox/Option)
