<!-- source: 74ba7736397a4fcb7cf0e521322a51e06f9e9341 -->
# Traitement en arrière-plan avec les web workers

Les [web workers](https://developer.mozilla.org/docs/Web/API/Web_Workers_API) vous permettent d'exécuter des calculs gourmands en CPU dans un thread d'arrière-plan, ce qui libère le thread principal pour mettre à jour l'interface utilisateur.
Les applications qui effectuent beaucoup de calculs, comme la génération de dessins de conception assistée par ordinateur \(CAO\) ou des calculs géométriques lourds, peuvent utiliser des web workers pour améliorer leurs performances.

HELPFUL: L'Angular CLI ne peut pas elle-même s'exécuter dans un web worker.

## Ajouter un web worker {#adding-a-web-worker}

Pour ajouter un web worker à un projet existant, utilisez la commande `ng generate` de l'Angular CLI.

```shell
ng generate web-worker <location>
```

Vous pouvez ajouter un web worker n'importe où dans votre application.
Par exemple, pour ajouter un web worker au composant racine, `src/app/app.component.ts`, exécutez la commande suivante.

```shell
ng generate web-worker app
```

La commande effectue les actions suivantes.

1. Elle configure votre projet pour utiliser les web workers, si ce n'est pas déjà le cas.
1. Elle ajoute le code de base suivant à `src/app/app.worker.ts` pour recevoir des messages.

   ```ts {header:"src/app/app.worker.ts"}
   addEventListener('message', ({data}) => {
     const response = `worker response to ${data}`;
     postMessage(response);
   });
   ```

1. Elle ajoute le code de base suivant à `src/app/app.component.ts` pour utiliser le worker.

   ```ts {header:"src/app/app.component.ts"}
   if (typeof Worker !== 'undefined') {
     // Create a new
     const worker = new Worker(new URL('./app.worker', import.meta.url));
     worker.onmessage = ({data}) => {
       console.log(`page got message: ${data}`);
     };
     worker.postMessage('hello');
   } else {
     // Web workers are not supported in this environment.
     // You should add a fallback so that your program still executes correctly.
   }
   ```

Après avoir créé ce code de base initial, vous devez refactoriser votre code pour utiliser le web worker en envoyant des messages vers et depuis le worker.

IMPORTANT: Certains environnements ou plateformes, comme `@angular/platform-server` utilisé pour le [rendu côté serveur](guide/ssr), ne prennent pas en charge les web workers.

Pour garantir que votre application fonctionne dans ces environnements, vous devez fournir un mécanisme de repli qui effectue les calculs que le worker effectuerait autrement.
