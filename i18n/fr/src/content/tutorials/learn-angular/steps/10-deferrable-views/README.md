<!-- source: 11e6f260aa577871e032f000bc59c01b4cf8044e -->
# Vues différables

En développement d'applications, vous vous retrouvez parfois avec de nombreux composants à référencer dans votre application, mais certains d'entre eux n'ont pas besoin d'être chargés tout de suite, pour diverses raisons.

Ils se trouvent peut-être sous la ligne de flottaison, ou ce sont des composants lourds avec lesquels l'utilisateur n'interagit que plus tard. Dans ce cas, nous pouvons charger certaines de ces ressources plus tard grâce aux vues différables.

NOTE: Pour en savoir plus, consultez [le chargement différé avec @defer dans le guide approfondi](/guide/templates/defer).

Dans cette activité, vous allez apprendre à utiliser les vues différables pour différer le chargement d'une section du template de votre composant.

<hr>

<docs-workflow>

<docs-step title="Ajouter un bloc `@defer` autour du composant de commentaires {#add-a-defer-block-around-the-comments-component}">

Dans votre application, la page d'article de blog contient un composant de commentaires après le détail de l'article.

Entourez le composant de commentaires d'un bloc `@defer` pour différer son chargement.

```angular-html
@defer {
  <comments />
}
```

Le code ci-dessus montre comment utiliser un bloc `@defer` simple. Par défaut, `@defer` charge le composant `comments` lorsque le navigateur est inactif.

</docs-step>

<docs-step title="Ajouter un placeholder {#add-a-placeholder}">

Ajoutez un bloc `@placeholder` au bloc `@defer`. Le bloc `@placeholder` contient le HTML qui s'affiche avant le début du chargement différé. Le contenu des blocs `@placeholder` est chargé immédiatement.

```angular-html {highlight:[3,4,5]}
@defer {
  <comments />
} @placeholder {
  <p>Future comments</p>
}
```

</docs-step>

<docs-step title="Ajouter un bloc de chargement {#add-a-loading-block}">

Ajoutez un bloc `@loading` au bloc `@defer`. Le bloc `@loading` contient le HTML qui s'affiche _pendant_ que le contenu différé est en cours de récupération, avant la fin de celle-ci. Le contenu des blocs `@loading` est chargé immédiatement.

```angular-html {highlight:[5,6,7]}
@defer {
  <comments />
} @placeholder {
  <p>Future comments</p>
} @loading {
  <p>Loading comments...</p>
}
```

</docs-step>

<docs-step title="Ajouter une durée minimale {#add-a-minimum-duration}">

Les sections `@placeholder` et `@loading` disposent de paramètres optionnels pour éviter un effet de clignotement quand le chargement est rapide. `@placeholder` accepte `minimum`, et `@loading` accepte `minimum` et `after`. Ajoutez une durée `minimum` au bloc `@loading` pour qu'il soit affiché pendant au moins 2 secondes.

```angular-html {highlight:[5]}
@defer {
  <comments />
} @placeholder {
  <p>Future comments</p>
} @loading (minimum 2s) {
  <p>Loading comments...</p>
}
```

</docs-step>

<docs-step title="Ajouter un déclencheur viewport {#add-a-viewport-trigger}">

Les vues différables proposent plusieurs options de déclencheur. Ajoutez un déclencheur viewport pour que le contenu se charge en différé dès qu'il entre dans le viewport.

```angular-html {highlight:[1]}
@defer (on viewport) {
  <comments />
} @placeholder {
  <p>Future comments</p>
} @loading (minimum 2s) {
  <p>Loading comments...</p>
}
```

</docs-step>

<docs-step title="Ajouter du contenu {#add-content}">

Un déclencheur viewport est idéal pour différer du contenu situé suffisamment bas dans la page pour qu'il faille défiler pour le voir. Ajoutons donc du contenu à notre article de blog. Vous pouvez écrire le vôtre, ou copier le contenu ci-dessous et le placer dans l'élément `<article>`.

```html {highlight:[1]}
<article>
  <p>
    Angular is my favorite framework, and this is why. Angular has the coolest deferrable view
    feature that makes defer loading content the easiest and most ergonomic it could possibly be.
    The Angular community is also filled with amazing contributors and experts that create excellent
    content. The community is welcoming and friendly, and it really is the best community out there.
  </p>
  <p>
    I can't express enough how much I enjoy working with Angular. It offers the best developer
    experience I've ever had. I love that the Angular team puts their developers first and takes
    care to make us very happy. They genuinely want Angular to be the best framework it can be, and
    they're doing such an amazing job at it, too. This statement comes from my heart and is not at
    all copied and pasted. In fact, I think I'll say these exact same things again a few times.
  </p>
  <p>
    Angular is my favorite framework, and this is why. Angular has the coolest deferrable view
    feature that makes defer loading content the easiest and most ergonomic it could possibly be.
    The Angular community is also filled with amazing contributors and experts that create excellent
    content. The community is welcoming and friendly, and it really is the best community out there.
  </p>
  <p>
    I can't express enough how much I enjoy working with Angular. It offers the best developer
    experience I've ever had. I love that the Angular team puts their developers first and takes
    care to make us very happy. They genuinely want Angular to be the best framework it can be, and
    they're doing such an amazing job at it, too. This statement comes from my heart and is not at
    all copied and pasted. In fact, I think I'll say these exact same things again a few times.
  </p>
  <p>
    Angular is my favorite framework, and this is why. Angular has the coolest deferrable view
    feature that makes defer loading content the easiest and most ergonomic it could possibly be.
    The Angular community is also filled with amazing contributors and experts that create excellent
    content. The community is welcoming and friendly, and it really is the best community out there.
  </p>
  <p>
    I can't express enough how much I enjoy working with Angular. It offers the best developer
    experience I've ever had. I love that the Angular team puts their developers first and takes
    care to make us very happy. They genuinely want Angular to be the best framework it can be, and
    they're doing such an amazing job at it, too. This statement comes from my heart and is not at
    all copied and pasted.
  </p>
</article>
```

Une fois ce code ajouté, faites défiler la page vers le bas pour voir le contenu différé se charger dès qu'il entre dans le viewport.

</docs-step>

</docs-workflow>

Dans cette activité, vous avez appris à utiliser les vues différables dans vos applications. Beau travail. 🙌

Vous pouvez en faire bien plus avec elles, comme utiliser différents déclencheurs, le préchargement et les blocs `@error`.

Pour en savoir plus, consultez la [documentation sur les vues différables](/guide/templates/defer).
