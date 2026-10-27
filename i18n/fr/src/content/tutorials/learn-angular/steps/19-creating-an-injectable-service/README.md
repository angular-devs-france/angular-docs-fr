<!-- source: b21652754ccfaf923aa45c718738df73c05ed8db -->
# Créer un service injectable

L'injection de dépendances (DI) est l'une des fonctionnalités les plus puissantes du framework Angular. Voyez l'injection de dépendances comme la capacité d'Angular à _fournir_ à l'exécution les ressources dont votre application a besoin. Une dépendance peut être un service ou une autre ressource.

NOTE: Pour en savoir plus, consultez [l'injection de dépendances dans le guide des Fondamentaux](/essentials/dependency-injection).

Dans cette activité, vous allez apprendre à créer un service `injectable`.

<hr>

Un service peut notamment servir d'intermédiaire pour interagir avec des données et des API. Pour rendre un service réutilisable, gardez la logique dans le service et partagez-le dans toute l'application là où c'est nécessaire.

Pour qu'une classe puisse être injectée par le système de DI, utilisez le décorateur `@Service`. Par exemple :

```ts {highlight:[1]}
@Service()
class UserService {
  // methods to retrieve and return data
}
```

Le décorateur `@Service` marque la classe comme un service et indique au système de DI que `UserService` est accessible partout dans votre application. Par défaut, Angular fournit le service à toute votre application : vous n'avez donc aucune configuration supplémentaire à écrire.

NOTE: Par défaut, `@Service` fournit la classe au niveau de l'injector racine. Si vous voulez la fournir manuellement, par exemple pour limiter sa portée à une route ou à un composant précis, définissez `autoProvided: false`. Pour en savoir plus, consultez le [guide Créer et utiliser des services](guide/di/creating-and-using-services#using-the-service-vs-injectable-decorator).

À vous de jouer :

<docs-workflow>

<docs-step title="Ajouter le décorateur `@Service` {#add-the-service-decorator}">
Mettez à jour le code de `car.service.ts` en ajoutant le décorateur `@Service()` à la classe `CarService`.

TIP: Inspirez-vous de l'exemple ci-dessus pour trouver la bonne syntaxe.

</docs-step>

</docs-workflow>

Bien joué 👍 ce service est maintenant `injectable` et peut entrer dans la danse. Maintenant que le service est `injectable`, essayons de l'injecter dans un composant 👉
