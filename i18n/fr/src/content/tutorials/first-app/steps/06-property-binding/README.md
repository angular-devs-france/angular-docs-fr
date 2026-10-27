<!-- source: 17d28361f0beb51f0ec5193381986d7c949ecb06 -->
# Ajouter un property binding au template d'un composant

Cette leçon du tutoriel montre comment ajouter un property binding à un template et l'utiliser pour transmettre des données dynamiques aux composants.

<docs-video src="https://www.youtube.com/embed/eM3zi_n7lNs?si=AsiczpWnMz5HhJqB&amp;start=599"/>

## Ce que vous allez apprendre {#what-youll-learn}

- Votre application comporte des bindings de données dans le template de `Home`.
- Votre application envoie des données de `Home` vers `HousingLocation`.

## Aperçu conceptuel des inputs {#conceptual-preview-of-inputs}

Dans cette leçon, vous allez poursuivre le partage de données du composant parent vers le composant enfant en liant des données à ces propriétés dans le template à l'aide du property binding.

Le property binding vous permet de connecter une variable à un `Input` dans un template Angular. Les données sont ensuite liées dynamiquement à l'`Input`.

Pour une explication plus approfondie, consultez le guide [Lier des propriétés et des attributs dynamiques](/guide/templates/binding#binding-dynamic-properties-and-attributes).

<docs-workflow>

<docs-step title="Mettre à jour le template de `Home` {#update-the-home-template}">
Cette étape ajoute un property binding à la balise `<app-housing-location>`.

Dans l'éditeur de code :

1.  Allez dans `src/app/home/home.ts`
1.  Dans la propriété template du décorateur `@Component`, mettez à jour le code pour qu'il corresponde au code ci-dessous :
    <docs-code language="angular-ts" header="Ajouter le property binding housingLocation" path="adev/src/content/tutorials/first-app/steps/07-dynamic-template-values/src/app/home/home.ts" visibleLines="[15,17]"/>

    Lorsque nous ajoutons un property binding à la balise d'un composant, nous utilisons la syntaxe `[attribute] = "value"` pour indiquer à Angular que la valeur affectée doit être traitée comme une propriété de la classe du composant et non comme une chaîne de caractères.

    La valeur à droite est le nom de la propriété de `Home`.

</docs-step>

<docs-step title="Vérifier que le code fonctionne toujours {#confirm-the-code-still-works}">
1.  Enregistrez vos modifications et vérifiez que l'application ne présente aucune erreur.
1.  Corrigez les éventuelles erreurs avant de passer à l'étape suivante.
</docs-step>

</docs-workflow>

SUMMARY: Dans cette leçon, vous avez ajouté un nouveau property binding et transmis une référence à une propriété de classe. `HousingLocation` a désormais accès à des données qu'il peut utiliser pour personnaliser l'affichage du composant.

Pour en savoir plus sur les sujets abordés dans cette leçon, consultez :

<docs-pill-row>
  <docs-pill href="/guide/templates/binding#binding-dynamic-properties-and-attributes" title="Property binding"/>
</docs-pill-row>
