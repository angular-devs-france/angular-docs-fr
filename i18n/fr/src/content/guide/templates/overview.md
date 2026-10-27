<!-- source: b1901c6dcb67be5e1e15763b4b3ea8af68948b1f -->
<docs-decorative-header title="Syntaxe de template" imgSrc="adev/src/assets/images/templates.svg"> <!-- markdownlint-disable-line -->
Dans Angular, un template est un morceau de HTML.
Utilisez une syntaxe spéciale dans un template pour tirer parti de nombreuses fonctionnalités d'Angular.
</docs-decorative-header>

TIP: Consultez les [Fondamentaux](essentials/templates) d'Angular avant de vous plonger dans ce guide complet.

Chaque composant Angular possède un **template** qui définit le [DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) que le composant rend sur la page. Grâce aux templates, Angular peut automatiquement maintenir votre page à jour quand les données changent.

Les templates se trouvent généralement soit dans la propriété `template` d'un fichier `*.ts`, soit dans le fichier `*.html`. Pour en savoir plus, consultez le [guide approfondi sur les composants](/guide/components).

## Comment fonctionnent les templates ? {#how-do-templates-work}

Les templates reposent sur la syntaxe [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML), avec des fonctionnalités supplémentaires comme des fonctions de template intégrées, le data binding, l'écoute d'événements, des variables, et plus encore.

Angular compile les templates en JavaScript afin de se construire une représentation interne de votre application. L'un des avantages de cette approche réside dans les optimisations de rendu intégrées qu'Angular applique automatiquement à votre application.

### Différences avec le HTML standard {#differences-from-standard-html}

Voici quelques différences entre les templates et la syntaxe HTML standard :

- Les commentaires présents dans le code source du template ne sont pas inclus dans le rendu final
- Les éléments de composants et de directives peuvent être auto-fermants (par exemple, `<UserProfile />`)
- Les attributs comportant certains caractères (comme `[]`, `()`, etc.) ont une signification particulière pour Angular. Consultez la [documentation sur les bindings](guide/templates/binding) et la [documentation sur l'ajout d'event listeners](guide/templates/event-listeners) pour plus d'informations.
- Le caractère `@` a une signification particulière pour Angular et sert à ajouter un comportement dynamique aux templates, comme le [control flow](guide/templates/control-flow). Vous pouvez inclure un caractère `@` littéral en l'échappant sous forme d'entité HTML (`&commat;` ou `&#64;`).
- Angular ignore et réduit les caractères d'espacement inutiles. Consultez [les espaces dans les templates](guide/templates/whitespace) pour plus de détails.
- Angular peut ajouter des nœuds de commentaire à une page comme placeholders pour du contenu dynamique, mais les développeurs peuvent les ignorer.

Par ailleurs, bien que la majeure partie de la syntaxe HTML soit une syntaxe de template valide, Angular ne prend pas en charge l'élément `<script>` dans les templates. Pour plus d'informations, consultez la page [Sécurité](best-practices/security).

## Et ensuite ? {#whats-next}

Les sujets suivants pourraient également vous intéresser :

| Sujets                                                                      | Détails                                                                                 |
| :-------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------- |
| [Lier du texte, des propriétés et des attributs dynamiques](guide/templates/binding) | Liez des données dynamiques à du texte, des propriétés et des attributs.     |
| [Ajouter des event listeners](guide/templates/event-listeners)              | Réagissez aux événements dans vos templates.                                            |
| [Two-way binding](guide/templates/two-way-binding)                          | Lie une valeur et propage ses changements simultanément.                                |
| [Control flow](guide/templates/control-flow)                                | Affichez, masquez et répétez des éléments de manière conditionnelle.                    |
| [Pipes](guide/templates/pipes)                                              | Transformez des données de manière déclarative.                                         |
| [Insérer du contenu enfant avec ng-content](guide/templates/ng-content)     | Contrôlez la manière dont les composants rendent le contenu.                            |
| [Créer des fragments de template avec ng-template](guide/templates/ng-template) | Déclarez un fragment de template.                                                   |
| [Regrouper des éléments avec ng-container](guide/templates/ng-container)    | Regroupez plusieurs éléments ou marquez un emplacement de rendu.                        |
| [Variables dans les templates](guide/templates/variables)                   | Découvrez les déclarations de variables.                                                |
| [Chargement différé avec @defer](guide/templates/defer)                     | Créez des vues différables avec `@defer`.                                               |
| [Syntaxe des expressions](guide/templates/expression-syntax)                | Découvrez les similitudes et les différences entre les expressions Angular et le JavaScript standard. |
| [Espaces dans les templates](guide/templates/whitespace)                    | Découvrez comment Angular gère les espaces.                                             |
