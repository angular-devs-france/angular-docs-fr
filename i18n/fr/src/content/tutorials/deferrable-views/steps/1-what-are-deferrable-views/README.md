<!-- source: 7d1dc52881c03b95c90f1481b17e7efcdef71061 -->
# Que sont les vues différables ?

Une page Angular entièrement rendue peut contenir de nombreux composants, directives et pipes différents. Si certaines parties de la page doivent être affichées immédiatement à l'utilisateur, d'autres peuvent attendre pour s'afficher plus tard.
Les _vues différables_ d'Angular, avec la syntaxe `@defer`, peuvent vous aider à accélérer votre application en indiquant à Angular d'attendre avant de télécharger le JavaScript des parties de la page qui n'ont pas besoin d'être affichées tout de suite.

Dans cette activité, vous allez apprendre à utiliser les vues différables pour charger en différé une section du template de votre composant.

<hr>

<docs-workflow>

<docs-step title="Ajouter un bloc `@defer` à une section d'un template {#add-a-defer-block-to-a-section-of-a-template}">
Dans votre `app.ts`, enveloppez le composant `article-comments` dans un bloc `@defer` pour le charger en différé.

```angular-html
@defer {
  <article-comments />
}
```

Par défaut, `@defer` charge le composant `article-comments` lorsque le navigateur est inactif.

Dans la console de développement de votre navigateur, vous pouvez voir que le fichier de chunk lazy `article-comments-component` est chargé séparément (les noms de fichiers exacts peuvent changer d'une exécution à l'autre) :

```markdown
Initial chunk files | Names | Raw size
chunk-NNSQHFIE.js | - | 769.00 kB |
main.js | main | 229.25 kB |

Lazy chunk files | Names | Raw size
chunk-T5UYXUSI.js | article-comments-component | 1.84 kB |
```

</docs-step>
</docs-workflow>

Beau travail ! Vous avez appris les bases des vues différables.
