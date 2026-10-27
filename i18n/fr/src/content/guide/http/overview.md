<!-- source: 6bbbd312a698dd5b6dd990f0e38396eb0cb625e4 -->
# Comprendre la communication avec les services backend via HTTP

La plupart des applications front-end doivent communiquer avec un serveur via le protocole HTTP pour télécharger ou envoyer des données et accéder à d'autres services backend. Angular fournit une API HTTP cliente pour les applications Angular, la classe de service `HttpClient` dans `@angular/common/http`.

## Fonctionnalités du service client HTTP {#http-client-service-features}

Le service client HTTP offre les fonctionnalités principales suivantes :

- La possibilité de demander des [valeurs de réponse typées](guide/http/making-requests#fetching-json-data)
- Une [gestion des erreurs](guide/http/making-requests#handling-request-failure) simplifiée
- L'[interception](guide/http/interceptors) des requêtes et des réponses
- Des [utilitaires de test](guide/http/testing) robustes

## Et ensuite ? {#whats-next}

<docs-pill-row>
  <docs-pill href="guide/http/setup" title="Configurer HttpClient"/>
  <docs-pill href="guide/http/making-requests" title="Effectuer des requêtes HTTP"/>
</docs-pill-row>
