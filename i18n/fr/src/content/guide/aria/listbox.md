<!-- source: f322d8aafa592eee3c51d57e44fd5d61baf88b5d -->
# Listbox

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/listbox/" title="Pattern Listbox"/>
  <docs-pill href="/api?query=listbox#angular_aria_listbox" title="Référence de l'API Listbox"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Une directive qui affiche une liste d'options parmi lesquelles les utilisateurs peuvent choisir, avec la navigation au clavier, la sélection simple ou multiple et la prise en charge des lecteurs d'écran.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/basic/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/basic/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/basic/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/listbox/src/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/basic/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/basic/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/basic/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/listbox/src/basic/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/listbox/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

## Utilisation {#usage}

La listbox est une directive de base utilisée par les patterns [Select](guide/aria/select), [Multiselect](guide/aria/multiselect) et [Autocomplete](guide/aria/autocomplete). Pour la plupart des besoins de liste déroulante, utilisez plutôt ces patterns documentés.

Envisagez d'utiliser directement une listbox pour :

- **Construire des composants de sélection personnalisés** - Créer des interfaces spécialisées avec un comportement spécifique
- **Des listes de sélection visibles** - Afficher des éléments sélectionnables directement sur la page (pas dans des listes déroulantes)
- **Des patterns d'intégration personnalisés** - S'intégrer à des exigences particulières de popup ou de mise en page

Évitez la listbox quand :

- **Vous avez besoin de menus de navigation** - Utilisez la directive [Menu](guide/aria/menu) pour les actions et les commandes

## Fonctionnalités {#features}

La listbox d'Angular fournit une implémentation de liste entièrement accessible avec :

- **Navigation au clavier** - Parcourez les options avec les touches fléchées, sélectionnez avec Entrée ou Espace
- **Prise en charge des lecteurs d'écran** - Attributs ARIA intégrés, dont role="listbox"
- **Sélection simple ou multiple** - L'attribute `multi` contrôle le mode de sélection
- **Horizontale ou verticale** - L'attribute `orientation` définit la direction de la mise en page
- **Recherche par saisie (type-ahead)** - Tapez des caractères pour atteindre les options correspondantes
- **Réactivité à base de signals** - Gestion d'état réactive avec les signals d'Angular

## Exemples {#examples}

### Listbox simple {#basic-listbox}

Les applications ont parfois besoin de listes sélectionnables visibles directement sur la page plutôt que cachées dans une liste déroulante. Une listbox standalone fournit la navigation au clavier et la sélection pour ces interfaces de liste visibles.

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/basic/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/basic/app/app.ts" />
  <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/basic/app/app.html" />
</docs-code-multifile>

Le signal model `value` fournit un two-way binding vers les éléments sélectionnés. Avec `selectionMode="explicit"`, les utilisateurs appuient sur Espace ou Entrée pour sélectionner des options. Pour les patterns de liste déroulante qui combinent une listbox avec une combobox et un positionnement en overlay, consultez le pattern [Select](guide/aria/select).

### Listbox horizontale {#horizontal-listbox}

Les listes fonctionnent parfois mieux à l'horizontale, par exemple pour des interfaces de type toolbar ou des sélections de type onglets. L'attribute `orientation` modifie à la fois la mise en page et la direction de la navigation au clavier.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/horizontal/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/horizontal/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/horizontal/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/listbox/src/horizontal/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/horizontal/material/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/horizontal/material/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/horizontal/material/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/listbox/src/horizontal/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>

  <docs-tab label="Rétro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/horizontal/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/horizontal/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/horizontal/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/listbox/src/horizontal/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Avec `orientation="horizontal"`, les touches fléchées gauche et droite permettent de naviguer entre les options au lieu des touches haut et bas. La listbox gère automatiquement les langues de droite à gauche (RTL) en inversant la direction de navigation.

### Modes de sélection {#selection-modes}

La listbox prend en charge deux modes de sélection qui déterminent à quel moment les éléments sont sélectionnés.

Le mode `'follow'` sélectionne automatiquement l'élément qui a le focus, ce qui accélère l'interaction lorsque la sélection change fréquemment. Le mode `'explicit'` exige Espace ou Entrée pour confirmer la sélection, ce qui évite les changements accidentels pendant la navigation. Les patterns de liste déroulante utilisent généralement le mode `'follow'` pour la sélection simple.

#### Explicit {#explicit}

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/modes/app/explicit/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/modes/app/explicit/app.ts" />
  <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/modes/app/explicit/app.html" />
</docs-code-multifile>

#### Follow {#follow}

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/listbox/src/modes/app/follow/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/aria/listbox/src/modes/app/follow/app.ts" />
  <docs-code header="app.html" path="adev/src/content/examples/aria/listbox/src/modes/app/follow/app.html" />
</docs-code-multifile>

| Mode         | Description                                                                                            |
| ------------ | ------------------------------------------------------------------------------------------------------ |
| `'follow'`   | Sélectionne automatiquement l'élément qui a le focus, ce qui accélère l'interaction lorsque la sélection change fréquemment |
| `'explicit'` | Exige Espace ou Entrée pour confirmer la sélection, ce qui évite les changements accidentels pendant la navigation           |

TIP: Les patterns de liste déroulante utilisent généralement le mode `'follow'` pour la sélection simple.

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants listbox.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {ListboxHarness} from '@angular/aria/listbox/testing';
import {MyListboxComponent} from './my-listbox'; // Your component

describe('MyListboxComponent', () => {
  let fixture: ComponentFixture<MyListboxComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyListboxComponent],
    });

    fixture = TestBed.createComponent(MyListboxComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should allow selecting options', async () => {
    const listbox = await loader.getHarness(ListboxHarness);

    // Verify listbox properties
    expect(await listbox.isMulti()).toBe(true);

    // Get all options
    const options = await listbox.getOptions();
    expect(options.length).toBe(2);

    // Click an option
    await options[0].click();

    // Verify option is selected
    expect(await options[0].isSelected()).toBe(true);

    // Filter options by text
    const bananaOption = await listbox.getOptions({text: 'Banana'});
    expect(bananaOption.length).toBe(1);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références suivantes :

- [`Listbox`](/api/aria/listbox/Listbox)
- [`Option`](/api/aria/listbox/Option)

### Patterns associés {#related-patterns}

La listbox est utilisée par ces patterns de liste déroulante documentés :

- [Select](guide/aria/select) - Pattern de liste déroulante à sélection simple utilisant une combobox en lecture seule + une listbox
- [Multiselect](guide/aria/multiselect) - Pattern de liste déroulante à sélection multiple utilisant une combobox en lecture seule + une listbox avec `multi`
- [Autocomplete](guide/aria/autocomplete) - Pattern de liste déroulante filtrable utilisant une combobox + une listbox

Pour des patterns de liste déroulante complets avec déclencheur, popup et positionnement en overlay, consultez ces guides de patterns plutôt que d'utiliser la listbox seule.
