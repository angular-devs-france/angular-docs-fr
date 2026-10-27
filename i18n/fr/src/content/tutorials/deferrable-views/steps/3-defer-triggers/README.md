<!-- source: 66be85fd711e4d6ac008ac3c056cac42e719eef8 -->
# Les déclencheurs de defer

Si les options par défaut de `@defer` sont déjà très utiles pour charger en lazy loading des parties de vos composants, vous pouvez quand même vouloir personnaliser davantage l'expérience de chargement différé.

Par défaut, le contenu différé se charge lorsque le navigateur est inactif. Vous pouvez toutefois personnaliser le moment de ce chargement en spécifiant un **déclencheur**. Cela vous permet de choisir le comportement de chargement le mieux adapté à votre composant.

Les vues différables proposent deux types de déclencheurs de chargement :

| Déclencheur | Description                                                                                                                                                                                                    |
| ------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `on`    | Une condition de déclenchement qui utilise un déclencheur de la liste des déclencheurs intégrés.<br/>Par exemple : `@defer (on viewport)`                                                                                                |
| `when`  | Une condition sous forme d'expression, évaluée pour savoir si elle est truthy. Quand l'expression est truthy, le placeholder est remplacé par le contenu chargé en différé.<br/>Par exemple : `@defer (when customizedCondition)` |

Si la condition `when` est évaluée à `false`, le bloc `defer` ne revient pas au placeholder. Le remplacement est une opération unique.

Vous pouvez définir plusieurs déclencheurs d'événements à la fois : ces déclencheurs sont évalués comme des conditions OU.

- Ex. : `@defer (on viewport; on timer(2s))`
- Ex. : `@defer (on viewport; when customizedCondition)`

Dans cette activité, vous allez apprendre à utiliser les déclencheurs pour spécifier la condition de chargement des vues différables.

<hr>

<docs-workflow>

<docs-step title="Ajouter le déclencheur `on hover` {#add-on-hover-trigger}">
Dans votre `app.ts`, ajoutez un déclencheur `on hover` au bloc `@defer`.

```angular-html {highlight:[1]}
@defer (on hover) {
  <article-comments />
} @placeholder (minimum 1s) {
  <p>Placeholder for comments</p>
} @loading (minimum 1s; after 500ms) {
  <p>Loading comments...</p>
} @error {
  <p>Failed to load comments</p>
}
```

Désormais, la page n'affiche la section des commentaires que lorsque vous survolez son placeholder.
</docs-step>

<docs-step title="Ajouter un bouton « Show all comments » {#add-a-show-all-comments-button}">
Modifiez ensuite le template pour inclure un bouton avec le libellé « Show all comments ». Ajoutez au bouton une variable de template nommée `#showComments`.

```angular-html {highlight:[1]}
<button type="button" #showComments>Show all comments</button>

@defer (on hover) {
  <article-comments />
} @placeholder (minimum 1s) {
  <p>Placeholder for comments</p>
} @loading (minimum 1s; after 500ms) {
  <p>Loading comments...</p>
} @error {
  <p>Failed to load comments</p>
}
```

NOTE: pour plus d'informations sur les [variables de template, consultez la documentation](/guide/templates/variables#declaring-a-template-reference-variable).

</docs-step>

<docs-step title="Ajouter le déclencheur `on interaction` {#add-on-interaction-trigger}">
Modifiez le bloc `@defer` du template pour utiliser le déclencheur `on interaction`. Passez la variable de template `showComments` en paramètre de `interaction`.

```angular-html {highlight:[3]}
<button type="button" #showComments>Show all comments</button>

@defer (on hover; on interaction(showComments)) {
  <article-comments />
} @placeholder (minimum 1s) {
  <p>Placeholder for comments</p>
} @loading (minimum 1s; after 500ms) {
  <p>Loading comments...</p>
} @error {
  <p>Failed to load comments</p>
}
```

Avec ces modifications, la page attend que l'une des conditions suivantes soit remplie avant d'afficher la section des commentaires :

- L'utilisateur survole le placeholder de la section des commentaires
- L'utilisateur clique sur le bouton « Show all comments »

Vous pouvez recharger la page pour essayer différents déclencheurs d'affichage de la section des commentaires.
</docs-step>
</docs-workflow>

Pour en savoir plus, consultez la documentation [Chargement différé avec `@defer`](/guide/templates/defer).
Continuez à apprendre pour découvrir d'autres fonctionnalités d'Angular.
