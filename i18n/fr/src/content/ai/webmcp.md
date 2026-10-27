<!-- source: 684bc4e1ae5b4969df309f2ff6182a6871901a03 -->
# WebMCP

WebMCP est une [proposition de standard web](https://github.com/webmachinelearning/webmcp/) qui permet aux applications web d'exposer des outils structurés directement aux agents IA qui s'exécutent nativement dans le navigateur. Les outils définis par une application permettent aux assistants IA d'interagir directement avec elle, ce qui fournit des capacités supplémentaires à l'agent et réduit le besoin d'interactions avec le DOM.

Par exemple, une application d'inscription de nouveaux utilisateurs peut fournir un outil WebMCP permettant à l'agent IA du navigateur de créer directement l'utilisateur, plutôt que d'obliger l'agent à parcourir une interface d'assistant complexe via des interactions avec le DOM.

Angular fournit un support expérimental de WebMCP, qui vous permet d'enregistrer facilement des outils liés au cycle de vie de l'injection de dépendances de votre application et de transformer automatiquement vos Signal Forms en outils prêts pour l'IA.

IMPORTANT: La spécification WebMCP en est à un stade très précoce et évolue fréquemment. C'est pourquoi le support de WebMCP dans Angular est actuellement [**expérimental**](reference/releases#experimental). Les API peuvent changer, même en dehors des versions majeures.

## Fournir des outils pour l'application {#provide-tools-for-the-application}

Utilisez [`provideExperimentalWebMcpTools`](api/core/provideExperimentalWebMcpTools) dans la configuration de votre application pour enregistrer des outils pendant toute la durée de vie de l'application. Les outils fournis de cette façon sont automatiquement enregistrés à l'initialisation de l'application et désenregistrés à sa destruction.

Le callback `execute` est invoqué dans le contexte d'injection de l'`Injector` associé, ce qui signifie que vous pouvez [`inject`](api/core/inject) directement des services.

```ts {header:"main.ts"}
import {Service, inject, provideExperimentalWebMcpTools} from '@angular/core';
import {bootstrapApplication} from '@angular/platform-browser';
import {AppRoot} from './app-root';

@Service()
class Greeter {
  sayHello(): string {
    return 'Hello agent!';
  }
}

bootstrapApplication(AppRoot, {
  providers: [
    provideExperimentalWebMcpTools([
      {
        name: 'greet',
        description: 'Greets the agent.',
        inputSchema: {type: 'object', properties: {}},
        execute: () => {
          const greeter = inject(Greeter);

          return {content: [{type: 'text', text: greeter.sayHello()}]};
        },
      },
    ]),
  ],
});
```

### Définir les paramètres d'un outil {#define-tool-parameters}

Lorsqu'un outil a besoin d'entrées fournies par l'assistant IA, définissez les arguments attendus dans `inputSchema` avec la syntaxe [JSON Schema](https://json-schema.org/). Angular infère automatiquement les types des paramètres passés à votre callback `execute` à partir de la définition du schéma.

```ts {header:"main.ts"}
import {provideExperimentalWebMcpTools} from '@angular/core';
import {bootstrapApplication} from '@angular/platform-browser';
import {AppRoot} from './app-root';

bootstrapApplication(AppRoot, {
  providers: [
    provideExperimentalWebMcpTools([
      {
        name: 'searchCatalog',
        description: 'Searches the store catalog for products matching a query.',
        inputSchema: {
          type: 'object',
          properties: {
            query: {
              type: 'string',
              description: 'The search keywords.',
            },
            maxResults: {
              type: 'number',
              description: 'Maximum number of results to return.',
            },
          },
          required: ['query'],
          additionalProperties: false,
        },
        execute: ({query, maxResults}) => {
          // Type of `query` is inferred as `string`.
          // Type of `maxResults` is inferred as `number | undefined`.

          // Consider validating this at runtime, since inputs may not be validated to match the schema.
          if (typeof query !== 'string') throw new Error(`Bad query: ${query}`);
          if (typeof maxResults !== 'number' && maxResults !== undefined)
            throw new Error(`Bad maxResults: ${maxResults}`);

          const limit = maxResults ?? 5;
          return {
            content: [{type: 'text', text: `Returning up to ${limit} results for "${query}".`}],
          };
        },
      },
    ]),
  ],
});
```

TIP: Utilisez `required: ['param1', 'param2', ...]` pour retirer `undefined` des types de ces paramètres, et `additionalProperties: false` pour restreindre le type de l'objet d'arguments à ces seuls paramètres.

## Fournir des outils pour une route {#provide-tools-for-a-route}

Dans des applications complexes, vous pouvez vouloir que certains outils ne soient disponibles que lorsque l'utilisateur consulte certaines routes. Pour cela, fournissez les outils directement dans les définitions de routes.

```ts {header:"routes.ts"}
import {provideExperimentalWebMcpTools} from '@angular/core';
import {Routes} from '@angular/router';

export const routes: Routes = [
  {
    path: 'dashboard',
    loadComponent: () => import('./dashboard').then((m) => m.Dashboard),
    providers: [
      provideExperimentalWebMcpTools([
        {
          name: 'exportDashboardReports',
          description: 'Exports the current dashboard analytics.',
          inputSchema: {type: 'object', properties: {}},
          execute: () => ({
            content: [{type: 'text', text: 'Dashboard export successfully triggered.'}],
          }),
        },
      ]),
    ],
  },
];
```

NOTE: Lorsque vous enregistrez des outils sur une route particulière, envisagez de configurer le router pour utiliser [`withAutoCleanupInjectors`](api/router/withAutoCleanupInjectors) afin que les outils soient automatiquement _désenregistrés_ quand l'utilisateur quitte la route. Sans cette option, les outils WebMCP déclarés sur des routes restent accessibles aux agents IA même après que l'utilisateur a navigué vers une autre route.

```ts {header:"app.config.ts"}
import {ApplicationConfig} from '@angular/core';
import {provideRouter, withAutoCleanupInjectors} from '@angular/router';
import {routes} from './routes';

export const appConfig: ApplicationConfig = {
  providers: [provideRouter(routes, withAutoCleanupInjectors())],
};
```

## Fournir des outils dans des services {#provide-tools-within-services}

Pour les cas d'usage dynamiques, la fonction [`declareExperimentalWebMcpTool`](api/core/declareExperimentalWebMcpTool) enregistre un outil directement dans un contexte d'injection et le désenregistre automatiquement quand ce contexte est détruit.

```ts {header:"counter.ts"}
import {Service, declareExperimentalWebMcpTool, signal, inject} from '@angular/core';

@Service()
export class Counter {
  readonly count = signal(0);

  constructor() {
    declareExperimentalWebMcpTool({
      name: 'getCounter',
      description: 'Reads the global counter.',
      inputSchema: {type: 'object', properties: {}},
      execute: () => ({
        content: [{type: 'text', text: `The count is: ${this.count()}.`}],
      }),
    });
  }
}
```

Même si `declareExperimentalWebMcpTool` fonctionne dans n'importe quel contexte d'injection, faites attention aux [collisions de noms](#name-collisions) et privilégiez son utilisation dans des services root.

## Outils implicites dans les Signal Forms {#implicit-tools-in-signal-forms}

Vous pouvez créer implicitement un outil WebMCP à partir d'un [Signal Form](essentials/signal-forms) Angular existant avec une configuration minimale. Angular convertit les models de vos formulaires en outils WebMCP riches, ce qui prend en charge efficacement des formulaires très dynamiques sans que vous ayez à écrire manuellement des schémas JSON ou des event handlers.

### Activer la fonctionnalité WebMCP pour les formulaires {#enable-the-webmcp-forms-feature}

Commencez par ajouter [`provideExperimentalWebMcpForms`](api/forms/signals/provideExperimentalWebMcpForms) aux providers racine de votre application :

```ts {header:"main.ts"}
import {bootstrapApplication} from '@angular/platform-browser';
import {provideExperimentalWebMcpForms} from '@angular/forms/signals';
import {AppRoot} from './app-root';

bootstrapApplication(AppRoot, {
  providers: [provideExperimentalWebMcpForms()],
});
```

### Activer l'outil sur un Signal Form {#opt-in-a-signal-form}

Ensuite, lorsque vous définissez un Signal Form avec [`form`](api/forms/signals/form), passez l'option de configuration `experimentalWebMcpTool` pour activer un outil WebMCP implicite. Angular inspecte le model de données de votre formulaire et génère automatiquement un schéma JSON pour les agents IA connectés.

```ts {header:"user-registration.ts"}
import {Component, signal} from '@angular/core';
import {form, required, minLength} from '@angular/forms/signals';

@Component({
  selector: 'app-user-registration',
  templateUrl: './user-registration.html',
})
export class UserRegistration {
  private readonly model = signal({
    firstName: '',
    lastName: '',
    age: 0,
    hobbies: ['Web Development'],
  });

  readonly userForm = form(
    this.model,
    (f) => {
      required(f.firstName, {message: 'First name is mandatory.'});
      required(f.lastName, {message: 'Last name is mandatory.'});
    },
    {
      // Implicitly registers a WebMCP tool named `registerUser` with parameters derived from `model`.
      experimentalWebMcpTool: {
        name: 'registerUser',
        description: 'Registers a new user.',
      },
      submission: {
        action: async (formValue) => {
          console.log('Submitting user:', formValue().value());
          // ...
        },
      },
    },
  );
}
```

Dans cet exemple, Angular génère un outil WebMCP avec un schéma JSON qui :

1. inclut `firstName`, `lastName`, `age` et `hobbies` comme paramètres, inférés à partir de la valeur initiale du signal `model`.
2. définit `firstName` et `lastName` comme champs _obligatoires_, comme l'infère le validateur [`required`](api/forms/signals/required).
3. définit `hobbies` comme un tableau de chaînes, ce qui permet à l'agent de fournir un nombre arbitraire de hobbies.

Au-delà de l'inférence du schéma d'entrée, Angular connecte aussi l'outil WebMCP à la logique de validation et au gestionnaire de soumission du formulaire. L'agent observe ainsi toutes les erreurs de validation déclenchées par ses entrées et tous les échecs survenus pendant la soumission, ce qui lui permet de se corriger et éventuellement de réessayer.

NOTE: Par défaut, les validateurs asynchrones ne bloquent pas la soumission : l'action de soumission s'exécute avant qu'ils ne se terminent, donc leurs erreurs ne sont pas signalées à l'agent. Gérez plutôt ces vérifications dans l'action de soumission.

#### Contraintes {#constraints}

Angular infère le schéma WebMCP à partir de la valeur initiale du model de votre formulaire. Cela nécessite :

- Des valeurs initiales concrètes (`''`, `0`, `false`) : Angular ne peut pas inférer les types de données à partir de `null` ou `undefined`.
- Des tableaux non vides (`['Hello!']`) : Angular ne peut pas inférer les types de données à partir d'un tableau vide et a besoin d'au moins une valeur initiale.

## Bonnes pratiques {#best-practices}

Gardez à l'esprit les bonnes pratiques suivantes :

### Collisions de noms {#name-collisions}

WebMCP exige que chaque outil ait un nom unique et lève une erreur si le même nom d'outil est enregistré plusieurs fois. Ainsi, appeler `declareExperimentalWebMcpTool` ou `provideExperimentalWebMcpTools` dans un contexte où ils peuvent être enregistrés plusieurs fois (comme le constructeur d'un composant) peut provoquer des erreurs à l'exécution.

Privilégiez, autant que possible, le placement des outils dans les providers de l'application, les providers de routes ou les services root. Lorsque vous placez des outils sur un composant, y compris les [outils implicites dans les Signal Forms](#implicit-tools-in-signal-forms), assurez-vous que ce composant n'est jamais affiché plus d'_une seule_ fois à la fois sur la page.

### Valider les entrées des outils {#validate-tool-inputs}

Angular ne fournit aucune validation implicite garantissant que les entrées fournies par un agent correspondent réellement au schéma JSON défini. Envisagez de valider explicitement les arguments de la fonction `execute` avant de les utiliser, pour plus de fiabilité.

### Tests {#testing}

Envisagez d'utiliser une implémentation mock de WebMCP comme [`@mcp-b/webmcp-polyfill`](https://www.npmjs.com/package/@mcp-b/webmcp-polyfill) pour tester efficacement vos outils de manière unitaire.
