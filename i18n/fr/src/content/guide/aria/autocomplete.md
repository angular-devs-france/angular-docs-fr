<!-- source: c9afdf9ec3224ff0e4e261c19f86e9737c518119 -->
# Autocomplete

## Vue d'ensemble {#overview}

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

## Utilisation {#usage}

L'autocomplete est idéal lorsque les utilisateurs doivent choisir parmi un grand nombre d'options et que taper est plus rapide que faire défiler. Envisagez l'autocomplete lorsque :

- **La liste d'options est longue** (plus de 20 éléments) - Taper affine les choix plus vite que de faire défiler une liste déroulante
- **Les utilisateurs savent ce qu'ils cherchent** - Ils peuvent taper une partie de la valeur attendue (comme un nom d'État, un produit ou un nom d'utilisateur)
- **Les options suivent des schémas prévisibles** - Les utilisateurs peuvent deviner des correspondances partielles (comme des codes pays, des domaines d'e-mail ou des catégories)
- **La rapidité compte** - Les formulaires bénéficient d'une sélection rapide sans navigation fastidieuse

Évitez l'autocomplete lorsque :

- La liste compte moins de 10 options - Une liste déroulante classique ou un groupe de boutons radio offre une meilleure visibilité
- Les utilisateurs doivent parcourir les options - Si la découverte est importante, affichez toutes les options d'emblée
- Les options ne sont pas familières - Les utilisateurs ne peuvent pas taper ce dont ils ignorent la présence dans la liste

## Fonctionnalités {#features}

L'autocomplete d'Angular fournit une implémentation de combobox entièrement accessible avec :

- **Navigation au clavier** - Parcourez les options avec les touches fléchées, sélectionnez avec Entrée, fermez avec Échap
- **Prise en charge des lecteurs d'écran** - Attributs ARIA intégrés pour les technologies d'assistance
- **Comportement de mise en surbrillance dynamique** - Prise en charge intégrée des suggestions de sélection en ligne
- **Réactivité basée sur les signals** - Gestion d'état réactive avec les signals d'Angular
- **Intégration de l'API Popover** - Exploite l'API Popover native du HTML pour un positionnement optimal
- **Prise en charge du texte bidirectionnel** - Gère automatiquement les langues de droite à gauche (RTL)

## Exemples {#examples}

### Mode de sélection automatique {#auto-select-mode}

Les utilisateurs qui tapent un texte partiel attendent une confirmation immédiate que leur saisie correspond à une option disponible. Le mode de sélection automatique met à jour la valeur du champ pour qu'elle corresponde à la première option filtrée pendant la frappe, ce qui réduit le nombre de frappes nécessaires et indique instantanément que la recherche est sur la bonne voie.

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

### Mode de sélection manuelle {#manual-selection-mode}

Le mode de sélection manuelle conserve le texte tapé inchangé pendant que les utilisateurs parcourent la liste de suggestions, ce qui évite la confusion due aux mises à jour automatiques. Le champ ne change que lorsque les utilisateurs confirment explicitement leur choix avec Entrée ou un clic.

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

### Mode de mise en surbrillance {#highlight-mode}

Le mode de mise en surbrillance permet à l'utilisateur de parcourir les options avec les touches fléchées sans modifier la valeur du champ pendant la navigation, jusqu'à ce qu'il sélectionne explicitement une nouvelle option avec Entrée ou un clic.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/highlight/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/highlight/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/highlight/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/highlight/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/highlight/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/highlight/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/highlight/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/highlight/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/highlight/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/highlight/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/highlight/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/highlight/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

### Intégration dans un formulaire Signal Forms {#signal-forms-integration}

Angular Aria s'intègre parfaitement avec l'API [Signal Forms](guide/forms/signals/overview), basée sur les signals. Vous pouvez encapsuler des champs complexes dans des composants de contrôle personnalisé réutilisables qui implémentent `FormValueControl`.

L'exemple suivant montre un composant de sélection de pays (country selector) qui implémente `FormValueControl<string>`, lié au formulaire parent avec `[formField]` et protégé par des règles de validation de schéma.

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/autocomplete/src/signal-forms/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/aria/autocomplete/src/signal-forms/app/app.ts"/>
  <docs-code header="app.html" path="adev/src/content/examples/aria/autocomplete/src/signal-forms/app/app.html"/>
  <docs-code header="country-selector.ts" path="adev/src/content/examples/aria/autocomplete/src/signal-forms/app/country-selector.ts"/>
  <docs-code header="country-selector.html" path="adev/src/content/examples/aria/autocomplete/src/signal-forms/app/country-selector.html"/>
  <docs-code header="country-selector.css" path="adev/src/content/examples/aria/autocomplete/src/signal-forms/app/country-selector.css"/>
  <docs-code header="app.css" path="adev/src/content/examples/aria/autocomplete/src/signal-forms/app/app.css"/>
</docs-code-multifile>

## Tests {#testing}

Le pattern autocomplete peut être testé en combinant `ComboboxHarness` et `ListboxHarness` depuis `@angular/aria/combobox/testing` et `@angular/aria/listbox/testing`.
Voici un exemple d'utilisation des harnais pour tester un composant autocomplete :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {ComboboxHarness} from '@angular/aria/combobox/testing';
import {ListboxHarness} from '@angular/aria/listbox/testing';
import {MyAutocompleteComponent} from './my-autocomplete'; // Your component

describe('MyAutocompleteComponent', () => {
  let fixture: ComponentFixture<MyAutocompleteComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyAutocompleteComponent],
    });

    fixture = TestBed.createComponent(MyAutocompleteComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should filter options based on input', async () => {
    const combobox = await loader.getHarness(ComboboxHarness);

    // Type in the input to trigger filtering
    await combobox.setValue('ap');
    expect(await combobox.isOpen()).toBe(true);

    // Get the listbox harness from the popup
    const listbox = await combobox.getPopupWidget(ListboxHarness);
    const options = await listbox.getOptions();

    // Verify options are filtered (e.g., 'Apple', 'Apricot')
    expect(options.length).toBe(2);
    expect(await options[0].getText()).toBe('Apple');

    // Select the first option
    await options[0].click();

    // Verify the input value is updated and popup is closed
    expect(await combobox.isOpen()).toBe(false);
    expect(await combobox.getValue()).toBe('Apple');
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
