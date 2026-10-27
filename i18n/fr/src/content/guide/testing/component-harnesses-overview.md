<!-- source: 48d4f27109d868ddf937ec0015129ecaf44a7733 -->
# Vue d'ensemble des harnais de composant

Un <strong>harnais de composant</strong> est une classe qui permet aux tests d'interagir avec les composants comme le ferait un utilisateur final, via une API prise en charge. Vous pouvez créer des harnais de test pour n'importe quel composant, du petit widget réutilisable à la page complète.

Les harnais offrent plusieurs avantages :

- Ils rendent les tests moins fragiles en les isolant des détails d'implémentation d'un composant, comme sa structure DOM
- Ils rendent les tests plus lisibles et plus faciles à maintenir
- Ils peuvent être utilisés dans plusieurs environnements de test

```ts
// Example of test with a harness for a component called MyButtonComponent
it('should load button with exact text', async () => {
  const button = await loader.getHarness(MyButtonComponentHarness);
  expect(await button.getText()).toBe('Confirm');
});
```

Les harnais de composant sont particulièrement utiles pour les widgets d'interface partagés. Les développeurs écrivent souvent des tests qui dépendent de détails d'implémentation privés des widgets, comme la structure DOM et les classes CSS. Ces dépendances rendent les tests fragiles et difficiles à maintenir. Les harnais offrent une alternative : une API prise en charge qui interagit avec le widget de la même manière qu'un utilisateur final. Les changements d'implémentation d'un widget risquent désormais moins de casser les tests des utilisateurs. Par exemple, [Angular Material](https://material.angular.dev/components/categories) fournit un harnais de test pour chaque composant de la library.

Les harnais de composant prennent en charge plusieurs environnements de test. Vous pouvez utiliser la même implémentation de harnais dans les tests unitaires comme dans les tests de bout en bout. Les auteurs de tests n'ont qu'une seule API à apprendre et les auteurs de composants n'ont pas à maintenir des implémentations distinctes pour les tests unitaires et de bout en bout.

De nombreux développeurs se rangent dans l'une des catégories suivantes : auteurs de tests, auteurs de harnais de composant et auteurs d'environnements de harnais. Utilisez le tableau ci-dessous pour trouver la section la plus pertinente de ce guide selon ces catégories :

| Type de développeur | Description | Section pertinente |
| :-------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | :----------------------------------------------------------------------------------------------------------- |
| Auteurs de tests | Développeurs qui utilisent des harnais de composant écrits par quelqu'un d'autre pour tester leur application. Par exemple, un développeur d'application qui utilise un composant de menu tiers et doit interagir avec ce menu dans un test unitaire. | [Utiliser des harnais de composant dans les tests](guide/testing/using-component-harnesses) |
| Auteurs de harnais de composant | Développeurs qui maintiennent des composants Angular réutilisables et veulent créer un harnais de test que leurs utilisateurs pourront utiliser dans leurs tests. Par exemple, l'auteur d'une library tierce de composants Angular ou un développeur qui maintient un ensemble de composants communs pour une grande application Angular. | [Créer des harnais de composant pour vos composants](guide/testing/creating-component-harnesses) |
| Auteurs d'environnements de harnais | Développeurs qui veulent permettre l'utilisation des harnais de composant dans d'autres environnements de test. Pour en savoir plus sur les environnements de test pris en charge nativement, consultez [environnements et loaders de harnais de test](guide/testing/using-component-harnesses#test-harness-environments-and-loaders). | [Ajouter la prise en charge d'autres environnements de test](guide/testing/component-harnesses-testing-environments) |

Pour la référence complète de l'API, consultez la [page de référence de l'API des harnais de composant du CDK Angular](/api#angular_cdk_testing).
