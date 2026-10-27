<!-- source: a68e1739a62b109f3ebff6d9a729861175331e0d -->
# Le pattern app shell

Le [pattern app shell](https://developer.chrome.com/blog/app-shell) est une façon de rendre une partie de votre application à l'aide d'une route au moment du build.
Il peut améliorer l'expérience utilisateur en affichant rapidement une page rendue statiquement (un skeleton commun à toutes les pages) pendant que le navigateur télécharge la version client complète, puis bascule automatiquement vers celle-ci une fois le code chargé.

Les utilisateurs obtiennent ainsi un premier affichage significatif de votre application, qui apparaît rapidement, car le navigateur peut rendre le HTML et le CSS sans avoir à initialiser le moindre JavaScript.

<docs-workflow>
<docs-step title="Préparer l'application {#prepare-the-application}">
Pour cela, utilisez la commande Angular CLI suivante :

```shell
ng new my-app
```

Pour une application existante, vous devez ajouter manuellement le `Router` et définir un `<router-outlet>` dans votre application.
</docs-step>
<docs-step title="Créer l'app shell {#create-the-application-shell}">
Utilisez l'Angular CLI pour créer automatiquement l'app shell.

```shell
ng generate app-shell
```

Pour plus d'informations sur cette commande, consultez [la commande app shell](cli/generate/app-shell).

La commande met à jour le code de l'application et ajoute des fichiers supplémentaires à la structure du projet.

```text
src
├── app
│ ├── app.config.server.ts # server application configuration
│ └── app-shell # app-shell component
│   ├── app-shell.component.html
│   ├── app-shell.component.scss
│   ├── app-shell.component.spec.ts
│   └── app-shell.component.ts
└── main.server.ts # main server application bootstrapping
```

</docs-step>
<docs-step title="Vérifier que l'application est compilée avec le contenu de l'app shell {#verify-the-application-is-built-with-the-shell-content}">

```shell
ng build --configuration=development
```

Ou, pour utiliser la configuration de production :

```shell
ng build
```

Pour vérifier le résultat du build, ouvrez <code class="no-auto-link">dist/my-app/browser/index.html</code>.
Recherchez le texte par défaut `app-shell works!`, qui indique que la route de l'app shell a été rendue dans le résultat.
</docs-step>
</docs-workflow>
