<!-- source: 185cef66099200e54b1361017fb895eea7f3f302 -->
# Les propriétés output des composants

Quand vous travaillez avec des composants, il peut être nécessaire de notifier d'autres composants que quelque chose s'est produit. Par exemple, un bouton a été cliqué, un élément a été ajouté à une liste ou retiré, ou une autre mise à jour importante a eu lieu. Dans ce cas, les composants doivent communiquer avec leurs composants parents.

Angular utilise la fonction `output()` pour permettre ce type de comportement.

NOTE: Pour en savoir plus, consultez [les événements personnalisés dans le guide des outputs](/guide/components/outputs).

Dans cette activité, vous allez apprendre à utiliser la fonction `output()` pour communiquer avec les composants.

<hr />

Pour créer le chemin de communication d'un composant enfant vers un composant parent, utilisez la fonction `output` pour initialiser une propriété de classe.

```ts {header:"child.ts"}
@Component({...})
class Child {
  incrementCountEvent = output<number>();
}
```

Le composant peut maintenant générer des événements que le composant parent peut écouter. Déclenchez des événements en appelant la méthode `emit` :

```ts {header:"child.ts"}
class Child {
  ...

  onClick() {
    this.count++;
    this.incrementCountEvent.emit(this.count);
  }
}
```

La fonction emit génère un événement du même type que celui défini par l'`output`.

À vous d'essayer. Complétez le code en suivant ces tâches :

<docs-workflow>

<docs-step title="Ajouter une propriété `output()` {#add-an-output-property}">
Mettez à jour `child.ts` en ajoutant une propriété output nommée `addItemEvent` ; pensez à définir le type de l'output à `string`.
</docs-step>

<docs-step title="Compléter la méthode `addItem` {#complete-additem-method}">
Dans `child.ts`, mettez à jour la méthode `addItem` ; utilisez le code suivant comme logique :

```ts {header:"child.ts", highlight:[2]}
addItem() {
  this.addItemEvent.emit('🐢');
}
```

</docs-step>

<docs-step title="Mettre à jour le template de `App` {#update-the-app-template}">
Dans `app.ts`, mettez à jour le template pour écouter l'événement émis en ajoutant le code suivant :

```angular-html
<app-child (addItemEvent)="addItem($event)" />
```

Désormais, le bouton « Add Item » ajoute un nouvel élément à la liste à chaque clic.

</docs-step>

</docs-workflow>

Bravo, vous avez maintenant terminé les fondamentaux des composants : impressionnant 👏

Continuez votre apprentissage pour découvrir d'autres fonctionnalités formidables d'Angular.
