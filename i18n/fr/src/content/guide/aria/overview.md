<!-- source: ff581e81003e3ecfad71d90bffc5a36d49c5eb28 -->
<docs-decorative-header title="Angular Aria" imgSrc="adev/src/assets/images/aria.svg"> <!-- markdownlint-disable-line -->
</docs-decorative-header>

## Qu'est-ce qu'Angular Aria ? {#what-is-angular-aria}

Créer des composants accessibles semble simple, mais les implémenter conformément aux [règles d'accessibilité du W3C](https://www.w3.org/TR/wcag/) demande un effort important et une expertise en accessibilité.

Angular Aria est une collection de directives headless et accessibles qui implémentent les [patterns WAI-ARIA](https://www.w3.org/WAI/ARIA/apg/patterns/) courants. Les directives gèrent les interactions au clavier, les attributs ARIA, la gestion du focus et la prise en charge des lecteurs d'écran. Il vous suffit de fournir la structure HTML, les styles CSS et la logique métier !

## Installation {#installation}

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install @angular/aria
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add @angular/aria
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add @angular/aria
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add @angular/aria
  </docs-code>
</docs-code-multifile>

## Démonstration {#showcase}

Prenons par exemple un menu de toolbar. Même s'il peut sembler n'être qu'une « simple » rangée de boutons liés à une logique spécifique, la navigation au clavier et les lecteurs d'écran ajoutent beaucoup de complexité inattendue pour qui n'est pas familier de l'accessibilité.

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

  <docs-tab label="Retro">
    <docs-code-multifile preview hideCode path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.ts">
      <docs-code header="app.ts" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.ts"/>
      <docs-code header="app.html" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.html"/>
      <docs-code header="app.css" path="adev/src/content/examples/aria/toolbar/src/basic/retro/app/app.css"/>
    </docs-code-multifile>
  </docs-tab>
</docs-tab-group>

Dans ce seul scénario, les développeurs doivent prendre en compte :

- **La navigation au clavier**. Les utilisateurs doivent ouvrir le menu avec Entrée ou Espace, parcourir les options avec les touches fléchées, sélectionner avec Entrée et fermer avec Échap.
- **Les lecteurs d'écran** doivent annoncer l'état du menu, le nombre d'options et l'option qui a le focus.
- **La gestion du focus** doit se déplacer logiquement entre le déclencheur et les éléments du menu.
- **Les langues de droite à gauche** nécessitent de pouvoir naviguer en sens inverse.

## Qu'est-ce qui est inclus dans Angular Aria ? {#whats-included}

Angular Aria inclut des directives avec une documentation complète, des exemples fonctionnels et des références d'API pour les patterns interactifs courants :

### Recherche et sélection {#search-and-selection}

| Composant | Description |
| --------------------------------------- | -------------------------------------------------------------- |
| [Autocomplete](guide/aria/autocomplete) | Champ de saisie avec des suggestions filtrées qui apparaissent pendant la frappe |
| [Listbox](guide/aria/listbox) | Listes d'options à sélection simple ou multiple avec navigation au clavier |
| [Select](guide/aria/select) | Pattern de liste déroulante à sélection simple avec navigation au clavier |
| [Multiselect](guide/aria/multiselect) | Pattern de liste déroulante à sélection multiple avec affichage compact |
| [Combobox](guide/aria/combobox) | Directive primitive qui coordonne un champ de saisie avec une popup |

### Navigation et appels à l'action {#navigation-and-call-to-actions}

| Composant | Description |
| ----------------------------- | ---------------------------------------------------------- |
| [Menu](guide/aria/menu) | Menus déroulants avec sous-menus imbriqués et raccourcis clavier |
| [Menubar](guide/aria/menubar) | Barre de navigation horizontale pour des menus d'application persistants |
| [Toolbar](guide/aria/toolbar) | Ensembles de contrôles groupés avec une navigation au clavier logique |

### Organisation du contenu {#content-organization}

| Composant | Description |
| --------------------------------- | ---------------------------------------------------------------------- |
| [Accordion](guide/aria/accordion) | Panneaux de contenu repliables qui peuvent se déplier individuellement ou de manière exclusive |
| [Tabs](guide/aria/tabs) | Interfaces à onglets avec des modes d'activation automatique ou manuelle |
| [Tree](guide/aria/tree) | Listes hiérarchiques avec possibilité de déplier et replier |
| [Grid](guide/aria/grid) | Affichage de données en deux dimensions avec navigation au clavier cellule par cellule |

## Quand faire appel à Angular Aria {#when-to-use-angular-aria}

Angular Aria convient bien lorsque vous avez besoin de composants interactifs accessibles, conformes aux WCAG, avec des styles personnalisés. Par exemple :

- **Créer un design system** - Votre équipe maintient une library de composants avec des standards visuels spécifiques qui nécessitent des implémentations accessibles
- **Libraries de composants d'entreprise** - Vous créez des composants réutilisables pour plusieurs applications au sein d'une organisation
- **Exigences de marque personnalisées** - L'interface doit correspondre à des spécifications de design précises que les libraries de composants pré-stylisés ne peuvent pas facilement satisfaire

## Quand ne pas faire appel à Angular Aria {#when-not-to-use-angular-aria}

Angular Aria ne convient pas forcément à tous les scénarios :

- **Composants pré-stylisés** - Si vous avez besoin de composants qui ont un rendu complet sans styles personnalisés, utilisez plutôt Angular Material
- **Formulaires simples** - Les contrôles de formulaire HTML natifs comme `<button>` et `<input type="radio">` offrent une accessibilité intégrée pour les cas d'usage simples
- **Prototypage rapide** - Lorsque vous validez rapidement des concepts, les libraries de composants pré-stylisés réduisent le temps de développement initial

## Prochaines étapes {#next-steps}

Choisissez un composant dans la navigation latérale ou dans la [liste ci-dessus](#whats-included), ou commencez par [Toolbar](guide/aria/toolbar) pour voir un exemple complet du fonctionnement des directives Angular Aria !
