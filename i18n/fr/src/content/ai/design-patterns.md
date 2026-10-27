<!-- source: 8a4ce4d670a6c623f022cf794b7da4f971925db5 -->
# Design patterns pour les SDK d'IA et les API de signals

Interagir avec des API d'IA et de grands modèles de langage (LLM) pose des défis particuliers, comme la gestion des opérations asynchrones, le traitement des données en streaming et la conception d'une expérience utilisateur réactive face à des requêtes réseau potentiellement lentes ou peu fiables. Les [signals](guide/signals) d'Angular et l'API [`resource`](guide/signals/resource) fournissent des outils puissants pour résoudre ces problèmes avec élégance.

## Déclencher des requêtes avec les signals {#triggering-requests-with-signals}

Un pattern courant, lorsque vous travaillez avec des prompts fournis par l'utilisateur, consiste à séparer la saisie en direct de l'utilisateur de la valeur soumise qui déclenche l'appel à l'API.

1. Stockez la saisie brute de l'utilisateur dans un signal pendant qu'il tape
2. Quand l'utilisateur soumet (par ex. en cliquant sur un bouton), mettez à jour un second signal avec le contenu du premier.
3. Utilisez le second signal dans le champ **`params`** de votre `resource`.

Cette configuration garantit que la fonction **`loader`** de la resource ne s'exécute que lorsque l'utilisateur soumet explicitement son prompt, et non à chaque frappe. Vous pouvez utiliser des paramètres de signal supplémentaires, comme un `sessionId` ou un `userId` (utiles pour créer des sessions LLM persistantes), dans le champ `loader`. Ainsi, la requête utilise toujours les valeurs actuelles de ces paramètres sans redéclencher la fonction asynchrone définie dans le champ `loader`.

De nombreux SDK d'IA fournissent des méthodes utilitaires pour effectuer des appels d'API. Par exemple, la library cliente de Genkit expose une méthode `runFlow` pour appeler des flows Genkit, que vous pouvez appeler depuis le `loader` d'une resource. Pour les autres API, vous pouvez utiliser [`httpResource`](guide/signals/resource#reactive-data-fetching-with-httpresource).

L'exemple suivant montre une `resource` qui récupère des parties d'une histoire générée par l'IA. Le `loader` n'est déclenché que lorsque le signal `storyInput` change.

```ts
// A resource that fetches three parts of an AI generated story
storyResource = resource({
  // The default value to use before the first request or on error
  defaultValue: DEFAULT_STORY,
  // The loader is re-triggered when this signal changes
  params: () => this.storyInput(),
  // The async function to fetch data
  loader: ({params}): Promise<StoryData> => {
    // The params value is the current value of the storyInput signal
    const url = this.endpoint();
    return runFlow({
      url,
      input: {
        userInput: params,
        sessionId: this.storyService.sessionId(), // Read from another signal
      },
    });
  },
});
```

## Préparer les données des LLM pour les templates {#preparing-llm-data-for-templates}

Vous pouvez configurer les API de LLM pour qu'elles renvoient des données structurées. Typer fortement votre `resource` pour correspondre à la sortie attendue du LLM offre une meilleure sûreté de typage et une meilleure autocomplétion dans l'éditeur.

Pour gérer un état dérivé d'une resource, utilisez un signal `computed` ou `linkedSignal`. Comme `linkedSignal` [donne accès aux valeurs précédentes](guide/signals/linked-signal), il peut servir dans de nombreux cas d'usage liés à l'IA, notamment pour

- construire un historique de conversation
- préserver ou personnaliser les données affichées par les templates pendant que les LLM génèrent du contenu

Dans l'exemple ci-dessous, `storyParts` est un `linkedSignal` qui ajoute les dernières parties de l'histoire renvoyées par `storyResource` au tableau existant des parties de l'histoire.

```ts
storyParts = linkedSignal<string[], string[]>({
  // The source signal that triggers the computation
  source: () => this.storyResource.value().storyParts,
  // The computation function
  computation: (newStoryParts, previous) => {
    // Get the previous value of this linkedSignal, or an empty array
    const existingStoryParts = previous?.value || [];
    // Return a new array with the old and new parts
    return [...existingStoryParts, ...newStoryParts];
  },
});
```

## Performances et expérience utilisateur {#performance-and-user-experience}

Les API de LLM peuvent être plus lentes et plus sujettes aux erreurs que des API conventionnelles, plus déterministes. Vous pouvez utiliser plusieurs fonctionnalités d'Angular pour construire une interface performante et conviviale.

- **Chargement ciblé :** placez la `resource` dans le composant qui utilise directement les données. Cela aide à limiter les cycles de détection de changements (en particulier dans les applications zoneless) et évite de bloquer d'autres parties de votre application. Si les données doivent être partagées entre plusieurs composants, fournissez la `resource` depuis un service.
- **SSR et hydratation :** utilisez le rendu côté serveur (SSR) avec l'hydratation incrémentale pour afficher rapidement le contenu initial de la page. Vous pouvez afficher un placeholder pour le contenu généré par l'IA et différer la récupération des données jusqu'à ce que le composant soit hydraté sur le client.
- **État de chargement :** utilisez le [statut](guide/signals/resource#resource-status) `LOADING` de la `resource` pour afficher un indicateur, comme un spinner, pendant que la requête est en cours. Ce statut couvre à la fois les chargements initiaux et les rechargements.
- **Gestion des erreurs et nouvelles tentatives :** utilisez la méthode [**`reload()`**](guide/signals/resource#reloading) de la `resource` pour permettre simplement aux utilisateurs de relancer les requêtes échouées, ce qui peut être plus fréquent lorsque vous dépendez de contenu généré par l'IA.

L'exemple suivant montre comment créer une interface réactive qui affiche dynamiquement une image générée par l'IA, avec gestion du chargement et possibilité de réessayer.

```angular-html
<!-- Display a loading spinner while the LLM generates the image -->
@if (imgResource.isLoading()) {
  <div class="img-placeholder">
    <mat-spinner [diameter]="50" />
  </div>
  <!-- Dynamically populates the src attribute with the generated image URL -->
} @else if (imgResource.hasValue()) {
  <img [src]="imgResource.value()" />
  <!-- Provides a retry option if the request fails  -->
} @else {
  <div class="img-placeholder" (click)="imgResource.reload()">
    <mat-icon fontIcon="refresh" />
    <p>Failed to load image. Click to retry.</p>
  </div>
}
```

## Patterns d'IA en pratique : réponses de chat en streaming {#ai-patterns-in-action-streaming-chat-responses}

Les interfaces affichent souvent de manière incrémentale les résultats partiels des API basées sur des LLM, au fur et à mesure que les données de la réponse arrivent. L'API resource d'Angular permet de recevoir les réponses en streaming pour prendre en charge ce type de pattern. La propriété `stream` de `resource` accepte une fonction asynchrone que vous pouvez utiliser pour appliquer des mises à jour à la valeur d'un signal au fil du temps. Le signal mis à jour représente les données reçues en streaming.

```ts
characters = resource({
  stream: async () => {
    const data = signal<ResourceStreamItem<string>>({value: ''});
    // Calls a Genkit streaming flow using the streamFlow method
    // exposed by the Genkit client SDK
    const response = streamFlow({
      url: '/streamCharacters',
      input: 10,
    });

    (async () => {
      for await (const chunk of response.stream) {
        data.update((prev) => {
          if ('value' in prev) {
            return {value: `${prev.value} ${chunk}`};
          } else {
            return {error: chunk as unknown as Error};
          }
        });
      }
    })();

    return data;
  },
});
```

Le membre `characters` est mis à jour de manière asynchrone et peut être affiché dans le template.

```angular-html
@if (characters.isLoading()) {
  <p>Loading...</p>
} @else if (characters.hasValue()) {
  <p>{{ characters.value() }}</p>
} @else {
  <p>{{ characters.error() }}</p>
}
```

Côté serveur, dans `server.ts` par exemple, l'endpoint défini envoie au client les données à streamer. Le code suivant utilise Gemini avec le framework Genkit, mais cette technique s'applique à d'autres API qui prennent en charge les réponses en streaming des LLM :

```ts
import {startFlowServer} from '@genkit-ai/express';
import {genkit} from 'genkit/beta';
import {googleAI, gemini20Flash} from '@genkit-ai/googleai';

const ai = genkit({plugins: [googleAI()]});

export const streamCharacters = ai.defineFlow(
  {
    name: 'streamCharacters',
    inputSchema: z.number(),
    outputSchema: z.string(),
    streamSchema: z.string(),
  },
  async (count, {sendChunk}) => {
    const {response, stream} = ai.generateStream({
      model: gemini20Flash,
      config: {
        temperature: 1,
      },
      prompt: `Generate ${count} different RPG game characters.`,
    });

    (async () => {
      for await (const chunk of stream) {
        sendChunk(chunk.content[0].text!);
      }
    })();

    return (await response).text;
  },
);

startFlowServer({
  flows: [streamCharacters],
});
```
