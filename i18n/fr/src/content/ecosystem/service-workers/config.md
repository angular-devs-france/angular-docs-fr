<!-- source: a65c9628a09e318d5dc69f037f7abed913db6c9e -->
# Fichier de configuration du service worker

Cette page décrit les propriétés du fichier de configuration du service worker.

## Modifier la configuration {#modifying-the-configuration}

Le fichier de configuration JSON `ngsw-config.json` indique quels fichiers et quelles URL de données le service worker d'Angular doit mettre en cache, et comment il doit mettre à jour les fichiers et données en cache.
L'[Angular CLI](tools/cli) traite ce fichier de configuration pendant `ng build`.

Tous les chemins de fichiers doivent commencer par `/`, qui correspond au répertoire de déploiement — généralement `dist/<project-name>` dans les projets CLI.

Sauf indication contraire, les patterns utilisent un format glob **limité\*** qui sera converti en interne en expression régulière :

| Formats glob  | Détails                                                                                                |
| :----------- | :----------------------------------------------------------------------------------------------------- |
| `**`         | Correspond à 0 ou plusieurs segments de chemin                                                         |
| `*`          | Correspond à 0 ou plusieurs caractères, à l'exception de `/`                                           |
| `?`          | Correspond à exactement un caractère, à l'exception de `/`                                             |
| Préfixe `!`  | Marque le pattern comme négatif : seuls les fichiers qui ne correspondent pas au pattern sont inclus   |

<docs-callout important title="Les caractères spéciaux doivent être échappés">
Attention : lors de la conversion interne du glob en expression régulière, certains caractères ayant une signification particulière dans une expression régulière ne sont pas échappés, et le pattern n'est pas non plus encadré par `^`/`$`.

`$` est un caractère spécial en regex qui correspond à la fin de la chaîne ; il ne sera pas automatiquement échappé lors de la conversion du pattern glob en expression régulière.

Si vous voulez faire correspondre littéralement le caractère `$`, vous devez l'échapper vous-même (avec `\\$`). Par exemple, le pattern glob `/foo/bar/$value` produit une expression qui ne peut jamais correspondre, car il est impossible qu'une chaîne contienne des caractères après sa fin.

Le pattern ne sera pas automatiquement encadré par `^` et `$` lors de sa conversion en expression régulière. Les patterns correspondront donc partiellement aux URL des requêtes.

Si vous voulez que vos patterns correspondent au début et/ou à la fin des URL, vous pouvez ajouter `^`/`$` vous-même. Par exemple, le pattern glob `/foo/bar/*.js` correspondra à la fois aux fichiers `.js` et `.json`. Si vous voulez uniquement les fichiers `.js`, utilisez `/foo/bar/*.js$`.
</docs-callout>

Exemples de patterns :

| Patterns     | Détails                                       |
| :----------- | :------------------------------------ |
| `/**/*.html` | Désigne tous les fichiers HTML                |
| `/*.html`    | Désigne uniquement les fichiers HTML à la racine |
| `!/**/*.map` | Exclut toutes les sourcemaps                  |

## Propriétés de configuration du service worker {#service-worker-configuration-properties}

Les sections suivantes décrivent chaque propriété du fichier de configuration.

### `appData`

Cette section vous permet de transmettre toutes les données que vous souhaitez pour décrire cette version précise de l'application.
Le service `SwUpdate` inclut ces données dans les notifications de mise à jour.
De nombreuses applications utilisent cette section pour fournir des informations supplémentaires destinées à l'affichage de popups dans l'interface, qui informent les utilisateurs de la mise à jour disponible.

### `index`

Indique le fichier qui sert de page d'index pour répondre aux requêtes de navigation.
Il s'agit généralement de `/index.html`.

### `assetGroups`

Les _assets_ sont des ressources qui font partie de la version de l'application et qui sont mises à jour avec elle.
Ils peuvent inclure des ressources chargées depuis l'origine de la page ainsi que des ressources tierces chargées depuis des CDN et d'autres URL externes.
Comme toutes ces URL externes ne sont pas forcément connues au moment du build, il est possible d'utiliser des patterns d'URL.

HELPFUL: Pour que le service worker puisse gérer des ressources chargées depuis d'autres origines, assurez-vous que [CORS](https://developer.mozilla.org/docs/Web/HTTP/CORS) est correctement configuré sur le serveur de chaque origine.

Ce champ contient un tableau de groupes d'assets, chacun définissant un ensemble de ressources et la politique selon laquelle elles sont mises en cache.

```ts
{
  "assetGroups": [
    {
      …
    },
    {
      …
    }
  ]
}
```

HELPFUL: Lorsque le ServiceWorker traite une requête, il examine les groupes d'assets dans l'ordre où ils apparaissent dans `ngsw-config.json`.
Le premier groupe d'assets qui correspond à la ressource demandée traite la requête.

Il est recommandé de placer les groupes d'assets les plus spécifiques en haut de la liste.
Par exemple, un groupe d'assets qui correspond à `/foo.js` doit apparaître avant un groupe qui correspond à `*.js`.

Chaque groupe d'assets définit à la fois un groupe de ressources et une politique qui les régit.
Cette politique détermine quand les ressources sont récupérées et ce qui se passe lorsque des changements sont détectés.

Les groupes d'assets suivent l'interface TypeScript présentée ici :

```ts
interface AssetGroup {
  name: string;
  installMode?: 'prefetch' | 'lazy';
  updateMode?: 'prefetch' | 'lazy';
  resources: {
    files?: string[];
    urls?: string[];
  };
  cacheQueryOptions?: {
    ignoreSearch?: boolean;
  };
}
```

Chaque `AssetGroup` est défini par les propriétés de groupe d'assets suivantes.

#### `name`

Un `name` est obligatoire.
Il identifie ce groupe d'assets précis d'une version de la configuration à l'autre.

#### `installMode`

L'`installMode` détermine comment ces ressources sont initialement mises en cache.
L'`installMode` peut prendre l'une des deux valeurs suivantes :

| Valeurs    | Détails                                                                                                                                                                                                                                                                                                                                                                                       |
| :--------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `prefetch` | Indique au service worker d'Angular de récupérer chacune des ressources listées pendant qu'il met en cache la version actuelle de l'application. Ce mode consomme beaucoup de bande passante, mais garantit que les ressources sont disponibles dès qu'elles sont demandées, même si le navigateur est actuellement hors ligne.                                                                  |
| `lazy`     | Ne met aucune ressource en cache à l'avance. Le service worker d'Angular ne met en cache que les ressources pour lesquelles il reçoit des requêtes. Il s'agit d'un mode de mise en cache à la demande. Les ressources qui ne sont jamais demandées ne sont pas mises en cache. C'est utile par exemple pour des images en différentes résolutions : le service worker ne met en cache que les assets adaptés à l'écran et à l'orientation concernés. |

La valeur par défaut est `prefetch`.

#### `updateMode`

Pour les ressources déjà en cache, l'`updateMode` détermine le comportement de mise en cache lorsqu'une nouvelle version de l'application est découverte.
Toutes les ressources du groupe qui ont changé depuis la version précédente sont mises à jour conformément à `updateMode`.

| Valeurs    | Détails                                                                                                                                                                                                                                  |
| :--------- | :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `prefetch` | Indique au service worker de télécharger et de mettre en cache immédiatement les ressources modifiées.                                                                                                                                   |
| `lazy`     | Indique au service worker de ne pas mettre ces ressources en cache. Il les traite plutôt comme non demandées et attend qu'elles soient de nouveau demandées pour les mettre à jour. Un `updateMode` à `lazy` n'est valide que si l'`installMode` est également `lazy`. |

La valeur par défaut est celle de `installMode`.

#### `resources`

Cette section décrit les ressources à mettre en cache, réparties dans les groupes suivants :

| Groupes de ressources | Détails                                                                                                                                                                                                                                                                                                                                                                                                       |
| :-------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `files`         | Liste des patterns qui correspondent à des fichiers du répertoire de distribution. Il peut s'agir de fichiers uniques ou de patterns de type glob qui correspondent à plusieurs fichiers.                                                                                                                                                                                                                       |
| `urls`          | Inclut à la fois des URL et des patterns d'URL évalués à l'exécution. Ces ressources ne sont pas récupérées directement et n'ont pas de hash de contenu, mais elles sont mises en cache selon leurs en-têtes HTTP. C'est surtout utile pour les CDN, comme le service Google Fonts. <br /> _(Les patterns glob négatifs ne sont pas pris en charge et `?` est pris au sens littéral ; autrement dit, il ne correspond à aucun autre caractère que `?`.)_ |

#### `cacheQueryOptions`

Ces options servent à modifier le comportement de correspondance des requêtes.
Elles sont transmises à la fonction `Cache#match` du navigateur.
Consultez [MDN](https://developer.mozilla.org/docs/Web/API/Cache/match) pour plus de détails.
Actuellement, seules les options suivantes sont prises en charge :

| Options        | Détails                                       |
| :------------- | :-------------------------------------------- |
| `ignoreSearch` | Ignore les query parameters. Valeur par défaut : `false`. |

### `dataGroups`

Contrairement aux ressources de type asset, les requêtes de données ne sont pas versionnées avec l'application.
Elles sont mises en cache selon des politiques configurées manuellement, plus adaptées à des situations comme les requêtes d'API et les autres dépendances de données.

Ce champ contient un tableau de groupes de données, chacun définissant un ensemble de ressources de données et la politique selon laquelle elles sont mises en cache.

```json
{
  "dataGroups": [
    {
      …
    },
    {
      …
    }
  ]
}
```

HELPFUL: Lorsque le ServiceWorker traite une requête, il examine les groupes de données dans l'ordre où ils apparaissent dans `ngsw-config.json`.
Le premier groupe de données qui correspond à la ressource demandée traite la requête.

Il est recommandé de placer les groupes de données les plus spécifiques en haut de la liste.
Par exemple, un groupe de données qui correspond à `/api/foo.json` doit apparaître avant un groupe qui correspond à `/api/*.json`.

Les groupes de données suivent cette interface TypeScript :

```ts
export interface DataGroup {
  name: string;
  urls: string[];
  version?: number;
  cacheConfig: {
    maxSize: number;
    maxAge: string;
    timeout?: string;
    refreshAhead?: string;
    strategy?: 'freshness' | 'performance';
  };
  cacheQueryOptions?: {
    ignoreSearch?: boolean;
  };
}
```

Chaque `DataGroup` est défini par les propriétés de groupe de données suivantes.

#### `name` {#datagroups-name}

Comme pour `assetGroups`, chaque groupe de données possède un `name` qui l'identifie de manière unique.

#### `urls`

Une liste de patterns d'URL.
Les URL qui correspondent à ces patterns sont mises en cache selon la politique de ce groupe de données.
Seules les requêtes qui ne modifient pas de données (GET et HEAD) sont mises en cache.

- Les patterns glob négatifs ne sont pas pris en charge
- `?` est pris au sens littéral ; autrement dit, il correspond _uniquement_ au caractère `?`

#### `version`

Il arrive que des API changent de format de manière non rétrocompatible.
Une nouvelle version de l'application peut ne pas être compatible avec l'ancien format de l'API, et donc ne pas être compatible avec les ressources de cette API déjà en cache.

`version` fournit un mécanisme pour indiquer que les ressources mises en cache ont été mises à jour de manière non rétrocompatible, et que les anciennes entrées du cache — celles des versions précédentes — doivent être supprimées.

`version` est un champ entier dont la valeur par défaut est `1`.

#### `cacheConfig`

Les propriétés suivantes définissent la politique selon laquelle les requêtes correspondantes sont mises en cache.

##### `maxSize`

Le nombre maximal d'entrées, ou de réponses, dans le cache.

CRITICAL: Des caches sans limite peuvent grossir indéfiniment et finir par dépasser les quotas de stockage, ce qui entraîne leur éviction.

##### `maxAge`

Le paramètre `maxAge` indique combien de temps les réponses peuvent rester dans le cache avant d'être considérées comme invalides et évincées. `maxAge` est une chaîne de durée qui utilise les suffixes d'unité suivants :

| Suffixes | Détails      |
| :------- | :----------- |
| `d`      | Jours        |
| `h`      | Heures       |
| `m`      | Minutes      |
| `s`      | Secondes     |
| `u`      | Millisecondes |

Par exemple, la chaîne `3d12h` met le contenu en cache pendant trois jours et demi au maximum.

##### `timeout`

Cette chaîne de durée indique le timeout réseau.
Le timeout réseau est le temps pendant lequel le service worker d'Angular attend une réponse du réseau avant d'utiliser une réponse en cache, s'il est configuré pour le faire.
`timeout` est une chaîne de durée qui utilise les suffixes d'unité suivants :

| Suffixes | Détails      |
| :------- | :----------- |
| `d`      | Jours        |
| `h`      | Heures       |
| `m`      | Minutes      |
| `s`      | Secondes     |
| `u`      | Millisecondes |

Par exemple, la chaîne `5s30u` correspond à un timeout réseau de cinq secondes et 30 millisecondes.

##### `refreshAhead`

Cette chaîne de durée indique combien de temps avant l'expiration d'une ressource en cache le service worker d'Angular doit tenter de manière proactive d'actualiser la ressource depuis le réseau.
La durée `refreshAhead` est une configuration facultative qui détermine combien de temps avant l'expiration d'une réponse en cache le service worker doit lancer une requête pour actualiser la ressource depuis le réseau.

| Suffixes | Détails      |
| :------- | :----------- |
| `d`      | Jours        |
| `h`      | Heures       |
| `m`      | Minutes      |
| `s`      | Secondes     |
| `u`      | Millisecondes |

Par exemple, la chaîne `1h30m` correspond à une heure et 30 minutes avant l'heure d'expiration.

##### `strategy`

Le service worker d'Angular peut utiliser l'une des deux stratégies de mise en cache suivantes pour les ressources de données.

| Stratégies de mise en cache | Détails                                                                                                                                                                                                                                                                                                                                                   |
| :----------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `performance`      | La stratégie par défaut, qui privilégie des réponses aussi rapides que possible. Si une ressource existe dans le cache, la version en cache est utilisée et aucune requête réseau n'est effectuée. Cela tolère un certain degré d'obsolescence, selon `maxAge`, en échange de meilleures performances. Elle convient aux ressources qui changent rarement, par exemple les avatars des utilisateurs. |
| `freshness`        | Privilégie la fraîcheur des données en récupérant de préférence les données demandées depuis le réseau. Ce n'est que si le réseau dépasse le timeout, défini par `timeout`, que la requête se rabat sur le cache. C'est utile pour les ressources qui changent fréquemment, par exemple les soldes de comptes.                                                                              |

HELPFUL: Vous pouvez aussi émuler une troisième stratégie, [staleWhileRevalidate](https://developers.google.com/web/fundamentals/instant-and-offline/offline-cookbook/#stale-while-revalidate), qui renvoie les données en cache si elles sont disponibles, mais récupère aussi des données fraîches depuis le réseau en arrière-plan pour la fois suivante.
Pour utiliser cette stratégie, définissez `strategy` à `freshness` et `timeout` à `0u` dans `cacheConfig`.

Concrètement, cela revient à :

1. Essayer d'abord de récupérer les données depuis le réseau.
2. Si la requête réseau ne se termine pas immédiatement, c'est-à-dire après un timeout de 0&nbsp;ms, ignorer l'âge du cache et se rabattre sur la valeur en cache.
3. Une fois la requête réseau terminée, mettre à jour le cache pour les requêtes futures.
4. Si la ressource n'existe pas dans le cache, attendre malgré tout la requête réseau.

##### `cacheOpaqueResponses`

Indique si le service worker d'Angular doit mettre en cache les réponses opaques ou non.

Si elle n'est pas précisée, la valeur par défaut dépend de la stratégie configurée pour le groupe de données :

| Stratégies                             | Détails                                                                                                                                                                                                                                                                                                                     |
| :------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Groupes avec la stratégie `freshness`   | La valeur par défaut est `true` et le service worker met en cache les réponses opaques. Ces groupes demandent les données à chaque fois et ne se rabattent sur la réponse en cache que hors ligne ou sur un réseau lent. Il importe donc peu que le service worker mette en cache une réponse d'erreur.                                         |
| Groupes avec la stratégie `performance` | La valeur par défaut est `false` et le service worker ne met pas en cache les réponses opaques. Ces groupes continueraient de renvoyer une réponse en cache jusqu'à l'expiration de `maxAge`, même si l'erreur était due à un problème temporaire de réseau ou de serveur. Il serait donc problématique que le service worker mette en cache une réponse d'erreur. |

<docs-callout title="Remarque sur les réponses opaques">

Si vous ne connaissez pas ce concept, une [réponse opaque](https://fetch.spec.whatwg.org#concept-filtered-response-opaque) est un type de réponse particulier renvoyé lorsqu'on demande une ressource située sur une autre origine qui ne renvoie pas d'en-têtes CORS.
L'une des caractéristiques d'une réponse opaque est que le service worker n'est pas autorisé à lire son statut : il ne peut donc pas vérifier si la requête a réussi ou non.
Consultez [Introduction à `fetch()`](https://developers.google.com/web/updates/2015/03/introduction-to-fetch#response_types) pour plus de détails.

Si vous ne pouvez pas mettre en place CORS — par exemple si vous ne contrôlez pas l'origine —, privilégiez la stratégie `freshness` pour les ressources qui produisent des réponses opaques.

</docs-callout>

#### `cacheQueryOptions` {#datagroups-cachequeryoptions}

Consultez [assetGroups](#assetgroups) pour plus de détails.

### `navigationUrls`

Cette section facultative vous permet de spécifier une liste personnalisée d'URL qui seront redirigées vers le fichier d'index.

#### Gérer les requêtes de navigation {#handling-navigation-requests}

Le ServiceWorker redirige les requêtes de navigation qui ne correspondent à aucun groupe `asset` ou `data` vers le [fichier d'index](#index) indiqué.
Une requête est considérée comme une requête de navigation si :

- Sa [méthode](https://developer.mozilla.org/docs/Web/API/Request/method) est `GET`
- Son [mode](https://developer.mozilla.org/docs/Web/API/Request/mode) est `navigate`
- Elle accepte une réponse `text/html`, d'après la valeur de l'en-tête `Accept`
- Son URL respecte les critères suivants :
  - L'URL ne doit pas contenir d'extension de fichier (c'est-à-dire un `.`) dans le dernier segment du chemin
  - L'URL ne doit pas contenir `__`

HELPFUL: Pour configurer si les requêtes de navigation sont transmises au réseau ou non, consultez les sections [navigationRequestStrategy](#navigationrequeststrategy) et [applicationMaxAge](#applicationmaxage).

#### Faire correspondre les URL des requêtes de navigation {#matching-navigation-request-urls}

Si ces critères par défaut conviennent dans la plupart des cas, il est parfois souhaitable de configurer des règles différentes.
Par exemple, vous pouvez vouloir ignorer certaines routes, comme celles qui ne font pas partie de l'application Angular, et les transmettre au serveur.

Ce champ contient un tableau d'URL et de patterns d'URL [de type glob](#modifying-the-configuration) évalués à l'exécution.
Il peut contenir à la fois des patterns négatifs (c'est-à-dire commençant par `!`) et des patterns et URL non négatifs.

Seules les requêtes dont l'URL correspond à _au moins un_ des URL/patterns non négatifs et à _aucun_ des patterns négatifs sont considérées comme des requêtes de navigation.
La query string de l'URL est ignorée lors de la correspondance.

Si le champ est omis, sa valeur par défaut est :

```ts
[
  '/**', // Include all URLs.
  '!/**/*.*', // Exclude URLs to files (containing a file extension in the last segment).
  '!/**/*__*', // Exclude URLs containing `__` in the last segment.
  '!/**/*__*/**', // Exclude URLs containing `__` in any other segment.
];
```

### `navigationRequestStrategy`

Cette propriété facultative vous permet de configurer la façon dont le service worker gère les requêtes de navigation :

```json
{
  "navigationRequestStrategy": "freshness"
}
```

| Valeurs possibles | Détails                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| :-------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `'performance'` | Le paramètre par défaut. Sert le [fichier d'index](#index) indiqué, qui est généralement en cache.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `'freshness'`   | Transmet les requêtes au réseau et se rabat sur le comportement `performance` hors ligne. Cette valeur est utile lorsque le serveur redirige les requêtes de navigation ailleurs à l'aide d'un code de statut de redirection HTTP `3xx`. Voici quelques raisons d'utiliser cette valeur : <ul> <li> Rediriger vers un site d'authentification lorsque l'authentification n'est pas gérée par l'application </li> <li> Rediriger des URL précises pour ne pas casser les liens et favoris existants après une refonte du site </li> <li> Rediriger vers un autre site, comme une page d'état du serveur, pendant qu'une page est temporairement indisponible </li> </ul> |

IMPORTANT: La stratégie `freshness` entraîne généralement davantage de requêtes envoyées au serveur, ce qui peut augmenter la latence des réponses. Il est recommandé d'utiliser la stratégie performance par défaut chaque fois que possible.

### `applicationMaxAge`

Cette propriété facultative vous permet de configurer combien de temps le service worker met en cache les requêtes. Pendant la durée `maxAge`, les fichiers sont servis depuis le cache. Au-delà, toutes les requêtes sont servies uniquement depuis le réseau, y compris les requêtes d'assets et de données.
