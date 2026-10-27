<!-- source: 00e164d9d6ebccd339d1c04f64f34c5056706728 -->
# Les blocs @loading, @error et @placeholder

Les vues différables vous permettent de définir le contenu à afficher selon les différents états de chargement.

| Bloc           | Description                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `@placeholder` | Par défaut, les blocs defer n'affichent aucun contenu avant d'être déclenchés. Le `@placeholder` est un bloc optionnel qui déclare le contenu à afficher avant le chargement du contenu différé. Angular remplace le placeholder par le contenu différé une fois le chargement terminé. Bien que ce bloc soit optionnel, l'équipe Angular recommande de toujours inclure un placeholder. [En savoir plus dans la documentation complète sur les vues différables](guide/templates/defer) |
| `@loading`     | Ce bloc optionnel vous permet de déclarer le contenu à afficher pendant le chargement des dépendances différées.                                                                                                                                                                                                                                                                                                                                      |
| `@error`       | Ce bloc vous permet de déclarer le contenu affiché si le chargement différé échoue.                                                                                                                                                                                                                                                                                                                                                                  |

Le contenu de tous les sous-blocs ci-dessus est chargé immédiatement. De plus, certaines fonctionnalités nécessitent un bloc `@placeholder`.

Dans cette activité, vous allez apprendre à utiliser les blocs `@loading`, `@error` et `@placeholder` pour gérer les états des vues différables.

<hr>

<docs-workflow>

<docs-step title="Ajouter un bloc `@placeholder` {#add-placeholder-block}">
Dans votre `app.ts`, ajoutez un bloc `@placeholder` au bloc `@defer`.

```angular-html {highlight:[3,4,5]}
@defer {
  <article-comments />
} @placeholder {
  <p>Placeholder for comments</p>
}
```

</docs-step>

<docs-step title="Configurer le bloc `@placeholder` {#configure-the-placeholder-block}">
Le bloc `@placeholder` accepte un paramètre optionnel pour spécifier la durée `minimum` pendant laquelle ce placeholder doit être affiché. Ce paramètre `minimum` s'exprime en millisecondes (ms) ou en secondes (s). Ce paramètre sert à éviter un clignotement rapide du contenu du placeholder lorsque les dépendances différées sont récupérées rapidement.

```angular-html {highlight:[3,4,5]}
@defer {
  <article-comments />
} @placeholder (minimum 1s) {
  <p>Placeholder for comments</p>
}
```

</docs-step>

<docs-step title="Ajouter un bloc `@loading` {#add-loading-block}">
Ajoutez ensuite un bloc `@loading` au template du composant.

Le bloc `@loading` accepte deux paramètres optionnels :

- `minimum` : la durée pendant laquelle ce bloc doit être affiché
- `after` : le temps d'attente après le début du chargement avant d'afficher le template de chargement

Les deux paramètres s'expriment en millisecondes (ms) ou en secondes (s).

Modifiez `app.ts` pour inclure un bloc `@loading` avec un paramètre minimum de `1s` ainsi qu'un paramètre after de valeur 500ms dans le bloc @loading.

```angular-html {highlight:[5,6,7]}
@defer {
  <article-comments />
} @placeholder (minimum 1s) {
  <p>Placeholder for comments</p>
} @loading (minimum 1s; after 500ms) {
  <p>Loading comments...</p>
}
```

NOTE: cet exemple utilise deux paramètres, séparés par le caractère « ; ».

</docs-step>

<docs-step title="Ajouter un bloc `@error` {#add-error-block}">
Enfin, ajoutez un bloc `@error` au bloc `@defer`.

```angular-html {highlight:[7,8,9]}
@defer {
  <article-comments />
} @placeholder (minimum 1s) {
  <p>Placeholder for comments</p>
} @loading (minimum 1s; after 500ms) {
  <p>Loading comments...</p>
} @error {
  <p>Failed to load comments</p>
}
```

</docs-step>
</docs-workflow>

Félicitations ! Vous comprenez maintenant bien les vues différables. Continuez sur cette lancée : découvrons ensuite les déclencheurs.
