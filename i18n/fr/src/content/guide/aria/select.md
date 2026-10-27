<!-- source: 5da7e6685be7abb7c5ebd5e73c090a82d3e672bd -->
# Select

## Vue d'ensemble {#overview}

Un pattern qui combine une combobox et une listbox pour créer des listes déroulantes à sélection unique, avec navigation au clavier et prise en charge des lecteurs d'écran.

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

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/icons/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

Le pattern select convient le mieux lorsque les utilisateurs doivent choisir une seule valeur parmi un ensemble d'options familier.

Envisagez ce pattern lorsque :

- **La liste d'options est fixe** (moins de 20 éléments) - Les utilisateurs peuvent parcourir et choisir sans filtrer
- **Les options sont familières** - Les utilisateurs reconnaissent les choix sans avoir besoin de chercher
- **Les formulaires ont besoin de champs standard** - Sélection d'un pays, d'une région, d'une catégorie ou d'un statut
- **Paramètres et configuration** - Menus déroulants pour des préférences ou des options
- **Libellés d'options clairs** - Chaque choix porte un nom distinct et facile à parcourir

Évitez ce pattern lorsque :

- **La liste compte plus de 20 éléments** - Utilisez le [pattern Autocomplete](guide/aria/autocomplete) pour un meilleur filtrage
- **Les utilisateurs doivent rechercher parmi les options** - [Autocomplete](guide/aria/autocomplete) fournit une saisie de texte et un filtrage
- **Une sélection multiple est nécessaire** - Utilisez plutôt le [pattern Multiselect](guide/aria/multiselect)
- **Il existe très peu d'options (2-3)** - Des boutons radio offrent une meilleure visibilité de tous les choix

## Fonctionnalités {#features}

Le pattern select combine les directives [Combobox](guide/aria/combobox) et [Listbox](guide/aria/listbox) pour fournir une liste déroulante entièrement accessible avec :

- **Navigation au clavier** - Parcourez les options avec les touches fléchées, sélectionnez avec Entrée, fermez avec Échap
- **Prise en charge des lecteurs d'écran** - Attributs ARIA intégrés pour les technologies d'assistance
- **Affichage personnalisé** - Affichez les valeurs sélectionnées avec des icônes, une mise en forme ou du contenu riche
- **Réactivité à base de signals** - Gestion d'état réactive avec les signals Angular
- **Positionnement intelligent** - CDK Overlay gère les bords du viewport et le défilement
- **Prise en charge du texte bidirectionnel** - Gère automatiquement les langues de droite à gauche (RTL)

## Exemples {#examples}

### Select de base {#basic-select}

Les utilisateurs ont besoin d'une liste déroulante standard pour choisir parmi une liste de valeurs. Une combobox associée à une listbox offre l'expérience de select habituelle avec une prise en charge complète de l'accessibilité.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

La saisie de texte est empêchée en appliquant la directive `ngCombobox` directement sur un host element non interactif (comme un `div` ou un `button`) plutôt que sur un `<input>`. Les utilisateurs interagissent avec la liste déroulante à l'aide des touches fléchées et d'Entrée, exactement comme avec un élément select natif.

### Select avec affichage personnalisé {#select-with-custom-display}

Les options ont souvent besoin d'indicateurs visuels comme des icônes ou des badges pour aider les utilisateurs à identifier rapidement les choix. Des templates personnalisés dans les options permettent une mise en forme riche tout en préservant l'accessibilité.

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

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/icons/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/icons/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Chaque option affiche une icône à côté du libellé. La valeur sélectionnée se met à jour pour montrer l'icône et le texte de l'option choisie, ce qui fournit un retour visuel clair.

### Select désactivé {#disabled-select}

Un select peut être désactivé pour empêcher toute interaction de l'utilisateur lorsque certaines conditions du formulaire ne sont pas remplies. L'état désactivé fournit un retour visuel et empêche l'interaction au clavier.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/disabled/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/disabled/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/disabled/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/disabled/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/disabled/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/disabled/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/disabled/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/disabled/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/select/src/disabled/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/select/src/disabled/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/select/src/disabled/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/select/src/disabled/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Lorsqu'il est désactivé, le select affiche un état visuel désactivé et bloque toute interaction de l'utilisateur. Les lecteurs d'écran annoncent l'état désactivé aux utilisateurs de technologies d'assistance.

## Tests {#testing}

Le pattern select peut être testé en combinant `ComboboxHarness` et `ListboxHarness`, issus de `@angular/aria/combobox/testing` et `@angular/aria/listbox/testing`.
Voici un exemple d'utilisation des harnais pour tester un composant select :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {ComboboxHarness} from '@angular/aria/combobox/testing';
import {ListboxHarness} from '@angular/aria/listbox/testing';
import {MySelectComponent} from './my-select'; // Your component

describe('MySelectComponent', () => {
  let fixture: ComponentFixture<MySelectComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MySelectComponent],
    });

    fixture = TestBed.createComponent(MySelectComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should allow selecting an option', async () => {
    // Load the combobox harness (which acts as the select trigger)
    const select = await loader.getHarness(ComboboxHarness);

    // Verify it is closed initially
    expect(await select.isOpen()).toBe(false);

    // Open the dropdown
    await select.open();
    expect(await select.isOpen()).toBe(true);

    // Get the listbox harness from the popup
    const listbox = await select.getPopupWidget(ListboxHarness);
    const options = await listbox.getOptions();
    expect(options.length).toBe(3);

    // Click the second option
    await options[1].click();

    // Verify the dropdown closed and the value updated
    expect(await select.isOpen()).toBe(false);
    expect(await (await select.host()).text()).toContain('Option 2');
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

### Positionnement {#positioning}

Le pattern select s'intègre à [CDK Overlay](https://material.angular.dev/cdk/overlay/overview) pour un positionnement intelligent. Utilisez `cdkConnectedOverlay` pour gérer automatiquement les bords du viewport et le défilement.
