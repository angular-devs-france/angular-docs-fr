<!-- source: d070fa8517c0727d46fdab71f7c1e9d7efa81644 -->
# Accordion

<docs-pill-row>
  <docs-pill href="https://www.w3.org/WAI/ARIA/apg/patterns/accordion/" title="Pattern ARIA Accordion"/>
  <docs-pill href="/api?query=accordion#angular_aria_accordion" title="Référence de l'API Accordion"/>
</docs-pill-row>

## Vue d'ensemble {#overview}

Un accordion organise du contenu associé en sections dépliables et repliables, ce qui réduit le défilement de la page et aide les utilisateurs à se concentrer sur les informations pertinentes. Chaque section possède un bouton déclencheur et un panneau de contenu. Cliquer sur un déclencheur bascule la visibilité du panneau associé.

<docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.ts">
  <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.ts"/>
  <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.html"/>
  <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.css"/>
</docs-code-multifile>

## Utilisation {#usage}

Les accordions conviennent bien pour organiser du contenu en groupes logiques lorsque les utilisateurs ont généralement besoin de consulter une section à la fois.

**Utilisez des accordions pour :**

- Afficher des FAQ avec plusieurs questions et réponses
- Organiser de longs formulaires en sections faciles à gérer
- Réduire le défilement sur les pages riches en contenu
- Révéler progressivement des informations associées

**Évitez les accordions pour :**

- Créer des menus de navigation (utilisez plutôt le composant [Menu](guide/aria/menu))
- Créer des interfaces à onglets (utilisez plutôt le composant [Tabs](guide/aria/tabs))
- Afficher une seule section repliable (utilisez plutôt un pattern disclosure)
- Les cas où les utilisateurs doivent voir plusieurs sections simultanément (envisagez une autre mise en page)

## Fonctionnalités {#features}

- **Modes de dépliage** - Contrôlez si un seul ou plusieurs panneaux peuvent être ouverts en même temps
- **Navigation au clavier** - Naviguez entre les déclencheurs avec les touches fléchées, Début et Fin
- **Rendu lazy** - Le contenu n'est créé que lorsqu'un panneau se déplie, ce qui améliore les performances du chargement initial
- **États désactivés** - Désactivez tout le groupe ou des déclencheurs individuels
- **Gestion du focus** - Contrôlez si les éléments désactivés peuvent recevoir le focus clavier
- **Contrôle programmatique** - Dépliez, repliez ou basculez les panneaux depuis le code de votre composant
- **Prise en charge RTL** - Prise en charge automatique des langues de droite à gauche

## Exemples {#examples}

### Mode de dépliage unique {#single-expansion-mode}

Définissez `[multiExpandable]="false"` pour n'autoriser qu'un seul panneau ouvert à la fois. Ouvrir un nouveau panneau ferme automatiquement le panneau ouvert précédemment.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/single-expansion/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/single-expansion/material/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/single-expansion/material/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/single-expansion/material/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/single-expansion/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/single-expansion/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/single-expansion/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/single-expansion/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/single-expansion/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Ce mode convient bien aux FAQ ou aux situations où vous voulez que les utilisateurs se concentrent sur une réponse à la fois.

### Mode de dépliage multiple {#multiple-expansion-mode}

Définissez `[multiExpandable]="true"` pour permettre l'ouverture simultanée de plusieurs panneaux. Les utilisateurs peuvent déplier autant de panneaux que nécessaire sans fermer les autres.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/multi-expansion/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/multi-expansion/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/multi-expansion/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/multi-expansion/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/multi-expansion/material/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/multi-expansion/material/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/multi-expansion/material/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/multi-expansion/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/multi-expansion/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/multi-expansion/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/multi-expansion/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/multi-expansion/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Ce mode est utile pour les sections de formulaire ou lorsque les utilisateurs doivent comparer le contenu de plusieurs panneaux.

NOTE: L'input `multiExpandable` vaut `true` par défaut. Définissez-le explicitement à `false` si vous voulez un comportement de dépliage unique.

### Éléments d'accordion désactivés {#disabled-accordion-items}

Désactivez des déclencheurs spécifiques avec l'input `disabled`. Contrôlez le comportement des éléments désactivés pendant la navigation au clavier avec l'input `softDisabled` du groupe d'accordion.

<docs-tab-group>
  <docs-tab label="Basique">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/disabled-focusable/basic/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/basic/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/basic/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/basic/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Material">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/disabled-focusable/material/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/material/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/material/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/material/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/accordion/src/disabled-focusable/retro/app/app.ts">
      <docs-code header="TS" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/retro/app/app.ts"/>
      <docs-code header="HTML" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/retro/app/app.html"/>
      <docs-code header="CSS" path="adev/src/content/examples/aria/accordion/src/disabled-focusable/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Lorsque `[softDisabled]="true"` (valeur par défaut), les éléments désactivés peuvent recevoir le focus mais ne peuvent pas être activés. Lorsque `[softDisabled]="false"`, les éléments désactivés sont entièrement ignorés pendant la navigation au clavier.

### Rendu lazy du contenu {#lazy-content-rendering}

Utilisez la directive `ngAccordionContent` sur un `ng-template` pour différer le rendu du contenu jusqu'au dépliage du panneau. Cela améliore les performances des accordions au contenu lourd, comme des images, des graphiques ou des composants complexes.

```angular-html
<div ngAccordionGroup>
  <div>
    <button ngAccordionTrigger [panel]="panel1">Trigger Text</button>
    <div ngAccordionPanel #panel1="ngAccordionPanel">
      <ng-template ngAccordionContent>
        <!-- This content only renders when the panel opens -->
        <img src="large-image.jpg" alt="Description" />
        <app-expensive-component />
      </ng-template>
    </div>
  </div>
</div>
```

Par défaut, le contenu est retiré du DOM lorsque le panneau se replie. Définissez `[preserveContent]="true"` pour le conserver, par exemple lorsque le panneau contient une vidéo qui ne doit pas être réinitialisée au repli.

## Tests {#testing}

Angular Aria fournit des harnais de composant pour tester les composants accordion.
Voici un exemple d'utilisation des harnais dans un test de composant :

```typescript
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {HarnessLoader} from '@angular/cdk/testing';
import {TestbedHarnessEnvironment} from '@angular/cdk/testing/testbed';
import {AccordionGroupHarness} from '@angular/aria/accordion/testing';
import {MyAccordionComponent} from './my-accordion'; // Your component

describe('MyAccordionComponent', () => {
  let fixture: ComponentFixture<MyAccordionComponent>;
  let loader: HarnessLoader;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [MyAccordionComponent],
    });

    fixture = TestBed.createComponent(MyAccordionComponent);
    await fixture.whenStable();
    loader = TestbedHarnessEnvironment.loader(fixture);
  });

  it('should allow expanding panels', async () => {
    // Load the accordion group harness
    const group = await loader.getHarness(AccordionGroupHarness);

    // Get all individual accordions (items) in the group
    const accordions = await group.getAccordions();
    expect(accordions.length).toBe(3);

    // Verify initial state (first expanded, others collapsed)
    expect(await accordions[0].isExpanded()).toBe(true);
    expect(await accordions[1].isExpanded()).toBe(false);

    // Expand the second panel
    await accordions[1].expand();

    // Verify updated state
    expect(await accordions[1].isExpanded()).toBe(true);
    // If multiExpandable is false, the first one should now be collapsed
    expect(await accordions[0].isExpanded()).toBe(false);
  });
});
```

## Référence de l'API {#api-reference}

Pour une documentation détaillée de l'API, consultez les références d'API suivantes :

- [`AccordionGroup`](/api/aria/accordion/AccordionGroup)
- [`AccordionTrigger`](/api/aria/accordion/AccordionTrigger)
- [`AccordionPanel`](/api/aria/accordion/AccordionPanel)
- [`AccordionContent`](/api/aria/accordion/AccordionContent)
