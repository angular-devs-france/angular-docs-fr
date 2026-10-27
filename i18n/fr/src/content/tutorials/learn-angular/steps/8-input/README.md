<!-- source: f5a40baab7069ffd2870b2906cb9d0d71bedb2df -->
# Les propriétés input des composants

En développement d'applications, vous avez parfois besoin d'envoyer des données dans un composant. Ces données peuvent servir à personnaliser un composant ou, par exemple, à transmettre des informations d'un composant parent à un composant enfant.

Angular utilise un concept appelé `input`. Il est similaire aux `props` d'autres frameworks. Pour créer une propriété `input`, utilisez la fonction `input()`.

NOTE: Pour en savoir plus, consultez [la réception de données avec les propriétés input dans le guide des inputs](/guide/components/inputs).

Dans cette activité, vous allez apprendre à utiliser la fonction `input()` pour envoyer des informations aux composants.

<hr>

Pour créer une propriété `input`, utilisez la fonction `input()` pour initialiser une propriété d'une classe de composant :

```ts {header:"user.ts"}
class User {
  occupation = input<string>();
}
```

Quand vous êtes prêt à transmettre une valeur via un `input`, vous pouvez définir les valeurs dans les templates avec la syntaxe d'attribut. Voici un exemple :

```angular-ts {header:"app.ts", highlight:[3]}
@Component({
  ...
  template: `<app-user occupation="Angular Developer"></app-user>`
})
export class App {}
```

La fonction `input` renvoie un `InputSignal`. Vous pouvez lire la valeur en appelant le signal.

```angular-ts {header:"user.ts"}
@Component({
  ...
  template: `<p>The user's occupation is {{occupation()}}</p>`
})
```

<docs-workflow>

<docs-step title="Définir une propriété `input()` {#define-an-input-property}">
Mettez à jour le code de `user.ts` pour définir sur `User` une propriété `input` nommée `name` et spécifiez le type `string`. Pour l'instant, ne définissez pas de valeur initiale et appelez `input()` sans argument. Pensez à mettre à jour le template pour appeler et interpoler la propriété `name` à la fin de la phrase.
</docs-step>

<docs-step title="Transmettre une valeur à la propriété `input` {#pass-a-value-to-the-input-property}">
Mettez à jour le code de `app.ts` pour transmettre la propriété `name` avec la valeur `"Simran"`.
<br>

Une fois le code correctement mis à jour, l'application affiche `The user's name is Simran`.
</docs-step>

</docs-workflow>

C'est très bien, mais ce n'est qu'un seul sens de la communication entre composants. Et si vous voulez envoyer des informations et des données d'un composant enfant vers un composant parent ? Consultez la leçon suivante pour le découvrir.

P.-S. : vous vous débrouillez très bien, continuez 🎉
