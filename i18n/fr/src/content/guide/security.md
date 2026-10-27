<!-- source: dbd1ed9142e94c0659bb2f4c4568507fcb70f2a5 -->
# Sécurité

Cette page décrit les protections intégrées d'Angular contre les vulnérabilités et attaques courantes des applications web, comme les attaques de cross-site scripting.
Elle ne traite pas de la sécurité au niveau applicatif, comme l'authentification et l'autorisation.

Pour plus d'informations sur les attaques et les mesures de protection décrites ci-dessous, consultez le [guide de l'Open Web Application Security Project (OWASP)](https://www.owasp.org/index.php/Category:OWASP_Guide_Project).

<a id="report-issues"></a>

<docs-callout title="Signaler des vulnérabilités">

Angular fait partie du [programme de récompenses pour les vulnérabilités des logiciels open source](https://bughunters.google.com/about/rules/6521337925468160/google-open-source-software-vulnerability-reward-program-rules) de Google. Pour les vulnérabilités d'Angular, soumettez votre rapport sur [https://bughunters.google.com](https://bughunters.google.com/report).

Pour plus d'informations sur la manière dont Google traite les problèmes de sécurité, consultez [la philosophie de sécurité de Google](https://www.google.com/about/appsecurity).

</docs-callout>

## Bonnes pratiques {#best-practices}

Voici quelques bonnes pratiques pour garantir la sécurité de votre application Angular.

1. **Restez à jour avec les dernières versions des libraries Angular** - Les libraries Angular reçoivent des mises à jour régulières, et ces mises à jour peuvent corriger des failles de sécurité découvertes dans des versions précédentes. Consultez le [change log](https://github.com/angular/angular/blob/main/CHANGELOG.md) d'Angular pour les mises à jour liées à la sécurité.
2. **Ne modifiez pas votre copie d'Angular** - Les versions privées et personnalisées d'Angular ont tendance à prendre du retard sur la version actuelle et peuvent ne pas inclure d'importants correctifs et améliorations de sécurité. Partagez plutôt vos améliorations d'Angular avec la communauté en ouvrant une pull request.
3. **Évitez les API Angular signalées dans la documentation comme « _Security Risk_ »** - Pour plus d'informations, consultez la section [Faire confiance aux valeurs sûres](#trusting-safe-values) de cette page.

## Prévenir le cross-site scripting (XSS) {#preventing-cross-site-scripting-xss}

Le [cross-site scripting (XSS)](https://en.wikipedia.org/wiki/Cross-site_scripting) permet à des attaquants d'injecter du code malveillant dans des pages web.
Ce code peut alors, par exemple, voler des données utilisateur et des identifiants de connexion, ou effectuer des actions en se faisant passer pour l'utilisateur.
C'est l'une des attaques les plus courantes sur le web.

Pour bloquer les attaques XSS, vous devez empêcher le code malveillant d'entrer dans le Document Object Model (DOM).
Par exemple, si des attaquants parviennent à vous faire insérer une balise `<script>` dans le DOM, ils peuvent exécuter du code arbitraire sur votre site web.
L'attaque ne se limite pas aux balises `<script>` : de nombreux éléments et propriétés du DOM permettent l'exécution de code, par exemple `<img alt="" onerror="...">` et `<a href="javascript:...">`.
Si des données contrôlées par un attaquant entrent dans le DOM, attendez-vous à des vulnérabilités de sécurité.

### Le modèle de sécurité d'Angular contre le cross-site scripting {#angulars-cross-site-scripting-security-model}

Pour bloquer systématiquement les failles XSS, Angular considère par défaut toutes les valeurs comme non fiables.
Lorsqu'une valeur est insérée dans le DOM depuis un binding de template ou une interpolation, Angular assainit et échappe les valeurs non fiables.
Si une valeur a déjà été assainie en dehors d'Angular et est considérée comme sûre, indiquez-le à Angular en marquant la [valeur comme étant de confiance](#trusting-safe-values).

Contrairement aux valeurs destinées au rendu, les templates Angular sont considérés par défaut comme dignes de confiance et doivent être traités comme du code exécutable.
Ne créez jamais de templates en concaténant des saisies utilisateur et de la syntaxe de template.
Cela permettrait à des attaquants d'[injecter du code arbitraire](https://en.wikipedia.org/wiki/Code_injection) dans votre application.
Pour éviter ces vulnérabilités, utilisez toujours le [compilateur de templates Ahead-Of-Time (AOT)](#use-the-aot-template-compiler) par défaut dans vos déploiements de production.

Une couche de protection supplémentaire peut être apportée par l'utilisation de la Content Security Policy et des Trusted Types.
Ces fonctionnalités de la plateforme web opèrent au niveau du DOM, qui est l'endroit le plus efficace pour prévenir les problèmes XSS. À ce niveau, elles ne peuvent pas être contournées par d'autres API de plus bas niveau.
C'est pourquoi il est fortement recommandé de tirer parti de ces fonctionnalités. Pour cela, configurez la [Content Security Policy](#content-security-policy) de l'application et activez l'[application obligatoire des Trusted Types](#enforcing-trusted-types).

### Assainissement et contextes de sécurité {#sanitization-and-security-contexts}

L'_assainissement_ consiste à inspecter une valeur non fiable pour la transformer en une valeur qu'il est sûr d'insérer dans le DOM.
Dans de nombreux cas, l'assainissement ne modifie pas du tout la valeur.
L'assainissement dépend d'un contexte.
Par exemple, une valeur inoffensive en CSS est potentiellement dangereuse dans une URL.

Angular définit les contextes de sécurité suivants :

| Contextes de sécurité | Détails                                                                           |
| :---------------- | :-------------------------------------------------------------------------------- |
| HTML              | Utilisé lorsqu'une valeur est interprétée comme du HTML, par exemple lors d'un binding sur `innerHtml`. |
| Style             | Utilisé lors d'un binding de CSS sur la propriété `style`.                                  |
| URL               | Utilisé pour les propriétés d'URL, comme `<a href>`.                                      |
| Resource URL      | Une URL chargée et exécutée comme du code, par exemple dans `<script src>`.        |

Angular assainit les valeurs non fiables pour le HTML et les URL. Il est impossible d'assainir les URL de ressource, car elles contiennent du code arbitraire.
En mode développement, Angular affiche un avertissement dans la console lorsqu'il doit modifier une valeur pendant l'assainissement.

### Exemple d'assainissement {#sanitization-example}

Le template suivant lie la valeur de `htmlSnippet`. Une fois en l'interpolant dans le contenu d'un élément, et une fois en la liant à la propriété `innerHTML` d'un élément :

<docs-code header="inner-html-binding.component.html" path="adev/src/content/examples/security/src/app/inner-html-binding.component.html"/>

Le contenu interpolé est toujours échappé : le HTML n'est pas interprété et le navigateur affiche les chevrons dans le contenu textuel de l'élément.

Pour que le HTML soit interprété, liez-le à une propriété HTML comme `innerHTML`.
Sachez que lier à `innerHTML` une valeur qu'un attaquant pourrait contrôler provoque normalement une vulnérabilité XSS.
Par exemple, on pourrait exécuter du JavaScript de la manière suivante :

<docs-code header="inner-html-binding.component.ts (class)" path="adev/src/content/examples/security/src/app/inner-html-binding.component.ts" region="class"/>

Angular reconnaît la valeur comme non sûre et l'assainit automatiquement, ce qui supprime l'élément `script` tout en conservant le contenu sûr, comme l'élément `<b>`.

<img alt="Capture d'écran montrant des valeurs HTML interpolées et liées" src="assets/images/guide/security/binding-inner-html.png#small">

### Utilisation directe des API du DOM et appels explicites d'assainissement {#direct-use-of-the-dom-apis-and-explicit-sanitization-calls}

À moins d'imposer les Trusted Types, les API du DOM intégrées au navigateur ne vous protègent pas automatiquement des vulnérabilités de sécurité.
Par exemple, `document`, le nœud accessible via `ElementRef` et de nombreuses API tierces contiennent des méthodes non sûres.
De même, si vous interagissez avec d'autres libraries qui manipulent le DOM, vous ne bénéficierez probablement pas du même assainissement automatique qu'avec les interpolations Angular.
Évitez d'interagir directement avec le DOM et utilisez plutôt les templates Angular lorsque c'est possible.

Dans les cas où c'est inévitable, utilisez les fonctions d'assainissement intégrées d'Angular.
Assainissez les valeurs non fiables avec la méthode [DomSanitizer.sanitize](api/platform-browser/DomSanitizer#sanitize) et le `SecurityContext` approprié.
Cette fonction accepte aussi les valeurs marquées comme étant de confiance à l'aide des fonctions `bypassSecurityTrust`, et ne les assainit pas, comme [décrit ci-dessous](#trusting-safe-values).

### Faire confiance aux valeurs sûres {#trusting-safe-values}

Il arrive que des applications aient réellement besoin d'inclure du code exécutable, d'afficher une `<iframe>` depuis une URL donnée ou de construire des URL potentiellement dangereuses.
Pour empêcher l'assainissement automatique dans ces situations, indiquez à Angular que vous avez inspecté une valeur, vérifié comment elle a été créée et vous êtes assuré qu'elle est sûre.
Soyez _prudent_.
Si vous faites confiance à une valeur potentiellement malveillante, vous introduisez une vulnérabilité de sécurité dans votre application.
En cas de doute, faites appel à un expert en revue de sécurité.

Pour marquer une valeur comme étant de confiance, injectez `DomSanitizer` et appelez l'une des méthodes suivantes :

- `bypassSecurityTrustHtml`
- `bypassSecurityTrustScript`
- `bypassSecurityTrustStyle`
- `bypassSecurityTrustUrl`
- `bypassSecurityTrustResourceUrl`

N'oubliez pas que la sûreté d'une valeur dépend du contexte : choisissez donc le bon contexte pour l'usage prévu de la valeur.
Imaginez que le template suivant doive lier une URL à un appel `javascript:alert(...)` :

<docs-code header="bypass-security.component.html (URL)" path="adev/src/content/examples/security/src/app/bypass-security.component.html" region="URL"/>

Normalement, Angular assainit automatiquement l'URL, désactive le code dangereux et, en mode développement, consigne cette action dans la console.
Pour l'éviter, marquez la valeur de l'URL comme une URL de confiance avec l'appel `bypassSecurityTrustUrl` :

<docs-code header="bypass-security.component.ts (trust-url)" path="adev/src/content/examples/security/src/app/bypass-security.component.ts" region="trust-url"/>

<img alt="Capture d'écran montrant une boîte d'alerte créée à partir d'une URL de confiance" src="assets/images/guide/security/bypass-security-component.png#medium">

Si vous devez convertir une saisie utilisateur en valeur de confiance, utilisez une méthode du composant.
Le template suivant permet aux utilisateurs de saisir un identifiant de vidéo YouTube et de charger la vidéo correspondante dans une `<iframe>`.
L'attribut `<iframe src>` relève du contexte de sécurité Resource URL, car une source non fiable peut, par exemple, introduire en douce des téléchargements de fichiers que des utilisateurs non avertis pourraient exécuter.
Pour l'éviter, appelez une méthode du composant afin de construire une URL de vidéo de confiance, ce qui amène Angular à autoriser le binding sur `<iframe src>` :

<docs-code header="bypass-security.component.html (iframe)" path="adev/src/content/examples/security/src/app/bypass-security.component.html" region="iframe"/>

<docs-code header="bypass-security.component.ts (trust-video-url)" path="adev/src/content/examples/security/src/app/bypass-security.component.ts" region="trust-video-url"/>

### Politique de sécurité du contenu (Content Security Policy) {#content-security-policy}

La Content Security Policy \(CSP\) est une technique de défense en profondeur pour prévenir le XSS.
Pour activer la CSP, configurez votre serveur web pour qu'il renvoie un en-tête HTTP `Content-Security-Policy` approprié.
Pour en savoir plus sur la Content Security Policy, consultez le [guide Web Fundamentals](https://developers.google.com/web/fundamentals/security/csp) sur le site Google Developers.

La politique minimale requise pour une toute nouvelle application Angular est la suivante :

```txt
default-src 'self'; style-src 'self' 'nonce-randomNonceGoesHere'; script-src 'self' 'nonce-randomNonceGoesHere';
```

Lorsqu'il sert votre application Angular, le serveur doit inclure un nonce généré aléatoirement dans l'en-tête HTTP de chaque requête.
Vous devez fournir ce nonce à Angular pour que le framework puisse effectuer le rendu des éléments `<style>`.
Vous pouvez définir le nonce pour Angular de l'une des manières suivantes :

1. Définissez l'attribut `ngCspNonce` sur l'élément racine de l'application, sous la forme `<app ngCspNonce="randomNonceGoesHere"></app>`. Utilisez cette approche si vous disposez d'un templating côté serveur capable d'ajouter le nonce à la fois à l'en-tête et à `index.html` lors de la construction de la réponse.
1. Fournissez le nonce avec le jeton d'injection `CSP_NONCE`. Utilisez cette approche si vous avez accès au nonce à l'exécution et souhaitez pouvoir mettre en cache `index.html`.

```ts
import {CSP_NONCE} from '@angular/core';
import {bootstrapApplication} from '@angular/platform-browser';
import {AppComponent} from './app/app.component';

bootstrapApplication(AppComponent, {
  providers: [
    {
      provide: CSP_NONCE,
      useValue: globalThis.myRandomNonceValue,
    },
  ],
});
```

<docs-callout title="Nonces uniques">

Assurez-vous toujours que les nonces que vous fournissez sont <strong>uniques pour chaque requête</strong> et qu'ils ne sont ni prévisibles ni devinables.
Si un attaquant peut prédire les futurs nonces, il peut contourner les protections offertes par la CSP.

Il est généralement déconseillé de générer un nonce sur le serveur d'origine lorsque vous utilisez un CDN, car les réponses sont fréquemment mises en cache. Si le serveur génère un nonce et que le CDN met en cache cette réponse HTML, chaque visiteur suivant reçoit la même valeur « unique », ce qui permet à un attaquant de découvrir cette valeur statique et de contourner les protections de la CSP.

Pour préserver l'intégrité « à usage unique » d'un nonce, celui-ci devrait idéalement être généré au niveau de la couche Edge (par exemple, le CDN) juste avant que le contenu ne soit livré à l'utilisateur.

</docs-callout>

NOTE: Si vous souhaitez [intégrer en ligne le CSS critique](/tools/cli/build#critical-css-inlining) de votre application, vous ne pouvez pas utiliser le jeton `CSP_NONCE` ; privilégiez alors l'option `security.autoCsp` dans la [configuration du workspace](reference/configs/workspace-config#extra-build-and-test-options) ou définissez l'attribut `ngCspNonce` sur l'élément racine de l'application.

Si vous ne pouvez pas générer de nonces dans votre projet, vous pouvez autoriser les styles inline en ajoutant `'unsafe-inline'` à la section `style-src` de l'en-tête CSP.

| Sections                                         | Détails                                                                                                                                                                                                                                                                                                                                                  |
| :----------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `default-src 'self';`                            | Permet à la page de charger toutes les ressources dont elle a besoin depuis la même origine.                                                                                                                                                                                                                                                                                 |
| `style-src 'self' 'nonce-randomNonceGoesHere';`  | Permet à la page de charger les styles globaux depuis la même origine \(`'self'`\) et les styles insérés par Angular avec le `nonce-randomNonceGoesHere`.                                                                                                                                                                                                             |
| `script-src 'self' 'nonce-randomNonceGoesHere';` | Permet à la page de charger du JavaScript depuis la même origine \(`'self'`\) et les scripts insérés par Angular CLI avec le `nonce-randomNonceGoesHere`. Requis uniquement lorsque vous activez l'intégration en ligne du CSS critique ou l'intégrité des sous-ressources (qui injecte une import map pour valider les chunks importés dynamiquement), qui ajoutent toutes deux des scripts inline à `index.html`. |

Angular lui-même n'a besoin que de ces paramètres pour fonctionner correctement.
À mesure que votre projet grandit, vous devrez peut-être étendre vos paramètres CSP pour prendre en charge des fonctionnalités supplémentaires propres à votre application.

#### Hébergement statique sans nonce par réponse {#static-hosting-without-per-response-nonces}

Si votre environnement d'hébergement ou votre CDN peut remplacer un jeton de substitution dans le HTML mis en cache au niveau de l'edge (par exemple avec SSI, ESI ou une edge function), vous pouvez générer `index.html` avec un placeholder dans `ngCspNonce` (comme `<app ngCspNonce="__CSP_NONCE__"></app>`) et le remplacer par un nonce unique à chaque réponse.

Si vous déployez votre application sur un hébergeur statique qui sert `index.html` tel quel, sans transformation à l'edge, ne codez pas en dur un nonce statique. Utilisez plutôt l'une des approches suivantes :

##### Hacher les scripts inline avec `autoCsp` {#hash-inline-scripts-with-autocsp}

Définissez l'option `security.autoCsp` à `true` dans la [configuration du workspace](reference/configs/workspace-config#extra-build-and-test-options).
Au moment du build, Angular CLI calcule un hash de chaque script inline de `index.html`, y compris les scripts ajoutés par l'intégration en ligne du CSS critique et par l'intégrité des sous-ressources.
La CLI remplace les éléments `<script src>` par un script loader haché et ajoute une balise `<meta>` au début de `<head>` :

```txt
script-src 'strict-dynamic' 'sha256-...' https: 'unsafe-inline'; object-src 'none'; base-uri 'self';
```

Comme les hashes ne dépendent que du contenu des scripts, `index.html` reste valide pour chaque visiteur et peut être mis en cache.
Les navigateurs qui prennent en charge les hashes et `'strict-dynamic'` ignorent les sources de repli `https:` et `'unsafe-inline'`.

La politique générée par `autoCsp` présente les limites suivantes :

- Elle ne couvre que les scripts. Vous devez configurer `style-src` séparément.
- Les navigateurs ignorent certaines directives, comme `frame-ancestors`, `report-uri` et `sandbox`, lorsqu'elles apparaissent dans une balise `<meta>`. Envoyez ces directives dans un en-tête HTTP `Content-Security-Policy`.
- Lorsqu'une page a plusieurs politiques, le navigateur les applique toutes. Si vous envoyez aussi un en-tête CSP, omettez à la fois `script-src` et `default-src` de cet en-tête afin qu'ils ne bloquent pas les scripts inline hachés.
- Vous ne pouvez pas utiliser `autoCsp` avec le rendu côté serveur (SSR).

##### Éviter les scripts inline {#avoid-inline-scripts}

Angular CLI n'ajoute des scripts inline à `index.html` que pour l'intégration en ligne du CSS critique et pour l'intégrité des sous-ressources (qui émet une import map transportant les métadonnées d'intégrité des imports dynamiques de modules).
Si vous définissez `optimization.styles.inlineCritical` à `false` et laissez `subresourceIntegrity` désactivé, `index.html` ne contient aucun script inline. Vous pouvez alors configurer un en-tête `Content-Security-Policy` avec `script-src 'self'`.

NOTE: Désactiver l'intégration en ligne du CSS critique peut ralentir le rendu initial de votre application, et désactiver l'intégrité des sous-ressources supprime les vérifications d'intégrité des scripts.

##### Configurer les styles pour l'hébergement statique {#configure-styles-for-static-hosting}

Angular insère des éléments `<style>` pour les styles des composants à l'exécution, et l'intégration en ligne du CSS critique ajoute un élément `<style>` à `index.html`.
Ni `autoCsp` ni la désactivation des scripts inline ne couvrent ces styles. Sans nonce par réponse, autorisez-les en ajoutant `'unsafe-inline'` à `style-src`.
Par exemple, si votre build évite les scripts inline, configurez votre hébergeur pour qu'il envoie l'en-tête suivant :

```txt
default-src 'self'; style-src 'self' 'unsafe-inline';
```

Si vous utilisez `autoCsp`, envoyez `style-src 'self' 'unsafe-inline'` dans l'en-tête HTTP, sans `default-src` ni `script-src`.

Le code de votre application et les libraries tierces peuvent nécessiter des directives supplémentaires dans les deux cas.

### Rendre obligatoire l'usage des Trusted Types {#enforcing-trusted-types}

Il est recommandé d'utiliser les [Trusted Types](https://w3c.github.io/trusted-types/dist/spec/) pour aider à protéger vos applications contre les attaques de cross-site scripting.
Les Trusted Types sont une fonctionnalité de la [plateforme web](https://en.wikipedia.org/wiki/Web_platform) qui peut vous aider à prévenir les attaques de cross-site scripting en imposant des pratiques de codage plus sûres.
Les Trusted Types peuvent aussi faciliter l'audit du code de l'application.

<docs-callout title="Trusted Types">

Les Trusted Types ne sont peut-être pas encore disponibles dans tous les navigateurs ciblés par votre application.
Si votre application compatible Trusted Types s'exécute dans un navigateur qui ne les prend pas en charge, les fonctionnalités de l'application sont préservées. Votre application reste protégée contre le XSS grâce au DomSanitizer d'Angular.
Consultez [caniuse.com/trusted-types](https://caniuse.com/trusted-types) pour connaître la prise en charge actuelle par les navigateurs.

</docs-callout>

Pour imposer les Trusted Types dans votre application, vous devez configurer le serveur web de votre application pour qu'il émette des en-têtes HTTP avec l'une des politiques Angular suivantes :

| Politiques                 | Détail                                                                                                                                                                                                                                                                                     |
| :----------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `angular`                | Cette politique est utilisée dans du code interne à Angular ayant fait l'objet d'une revue de sécurité, et elle est requise pour qu'Angular fonctionne lorsque les Trusted Types sont imposés. Toutes les valeurs de template inline et tout contenu assaini par Angular sont considérés comme sûrs par cette politique.                                          |
| `angular#bundler`        | Cette politique est utilisée par le bundler d'Angular CLI lors de la création des fichiers de chunks chargés en lazy loading.                                                                                                                                                                                                             |
| `angular#unsafe-bypass`  | Cette politique est utilisée par les applications qui utilisent l'une des méthodes du [DomSanitizer](api/platform-browser/DomSanitizer) d'Angular contournant la sécurité, comme `bypassSecurityTrustHtml`. Toute application qui utilise ces méthodes doit activer cette politique.                                  |
| `angular#unsafe-jit`     | Cette politique est utilisée par le [compilateur Just-In-Time (JIT)](api/core/Compiler). Vous devez activer cette politique si votre application interagit directement avec le compilateur JIT ou s'exécute en mode JIT à l'aide de [platform browser dynamic](api/platform-browser-dynamic/platformBrowserDynamic). |
| `angular#unsafe-upgrade` | Cette politique est utilisée par le package [@angular/upgrade](api/upgrade/static/UpgradeModule). Vous devez activer cette politique si votre application est une application hybride AngularJS.                                                                                                                           |

Vous devez configurer les en-têtes HTTP pour les Trusted Types aux emplacements suivants :

- L'infrastructure de production qui sert l'application
- Angular CLI \(`ng serve`\), à l'aide de la propriété `headers` du fichier `angular.json`, pour le développement local et les tests de bout en bout
- Karma \(`ng test`\), à l'aide de la propriété `customHeaders` du fichier `karma.config.js`, pour les tests unitaires

Voici un exemple d'en-tête configuré spécifiquement pour les Trusted Types et Angular :

```html
Content-Security-Policy: trusted-types angular; require-trusted-types-for 'script';
```

Un exemple d'en-tête configuré spécifiquement pour les Trusted Types et les applications Angular qui utilisent l'une des méthodes du [DomSanitizer](api/platform-browser/DomSanitizer) d'Angular contournant la sécurité :

```html
Content-Security-Policy: trusted-types angular angular#unsafe-bypass; require-trusted-types-for
'script';
```

Voici un exemple d'en-tête configuré spécifiquement pour les Trusted Types et les applications Angular utilisant le JIT :

```html
Content-Security-Policy: trusted-types angular angular#unsafe-jit; require-trusted-types-for
'script';
```

Voici un exemple d'en-tête configuré spécifiquement pour les Trusted Types et les applications Angular qui chargent des modules en lazy loading :

```html
Content-Security-Policy: trusted-types angular angular#bundler; require-trusted-types-for 'script';
```

<docs-callout title="Contributions de la communauté">

Pour en savoir plus sur le dépannage des configurations Trusted Types, la ressource suivante peut être utile :

[Prevent DOM-based cross-site scripting vulnerabilities with Trusted Types](https://web.dev/trusted-types/#how-to-use-trusted-types)

</docs-callout>

### Utiliser le compilateur de templates AOT {#use-the-aot-template-compiler}

Le compilateur de templates AOT empêche toute une catégorie de vulnérabilités appelées injections de template, et améliore grandement les performances de l'application.
Le compilateur de templates AOT est le compilateur utilisé par défaut par les applications Angular CLI, et vous devez l'utiliser dans tous vos déploiements de production.

Une alternative au compilateur AOT est le compilateur JIT, qui compile les templates en code de template exécutable dans le navigateur, à l'exécution.
Angular fait confiance au code des templates : générer dynamiquement des templates et les compiler, en particulier des templates contenant des données utilisateur, contourne donc les protections intégrées d'Angular. C'est un anti-pattern de sécurité.
Pour savoir comment construire dynamiquement des formulaires de manière sûre, consultez le guide [Formulaires dynamiques](guide/forms/dynamic-forms).

### Protection XSS côté serveur {#server-side-xss-protection}

Le HTML construit sur le serveur est vulnérable aux attaques par injection.
Injecter du code de template dans une application Angular revient à injecter du code exécutable dans l'application :
cela donne à l'attaquant le contrôle total de l'application.
Pour l'éviter, utilisez sur le serveur un langage de templating qui échappe automatiquement les valeurs afin de prévenir les vulnérabilités XSS.
Ne créez pas de templates Angular côté serveur à l'aide d'un langage de templating. Cela comporte un risque élevé d'introduire des vulnérabilités d'injection de template.

## Vulnérabilités au niveau HTTP {#http-level-vulnerabilities}

Angular intègre une prise en charge permettant d'aider à prévenir deux vulnérabilités HTTP courantes : la falsification de requête intersites \(CSRF ou XSRF\) et l'inclusion de script intersites \(XSSI\).
Ces deux vulnérabilités doivent être traitées principalement côté serveur, mais Angular fournit des utilitaires qui facilitent l'intégration côté client.

### Falsification de requête intersites {#cross-site-request-forgery}

Dans une falsification de requête intersites \(CSRF ou XSRF\), un attaquant incite l'utilisateur à visiter une autre page web \(comme `evil.com`\) contenant du code malveillant. Cette page web envoie secrètement une requête malveillante au serveur web de l'application \(comme `example-bank.com`\).

Supposons que l'utilisateur soit connecté à l'application sur `example-bank.com`.
L'utilisateur ouvre un e-mail et clique sur un lien vers `evil.com`, qui s'ouvre dans un nouvel onglet.

La page `evil.com` envoie immédiatement une requête malveillante à `example-bank.com`.
Il s'agit peut-être d'une requête de virement depuis le compte de l'utilisateur vers celui de l'attaquant.
Le navigateur envoie automatiquement avec cette requête les cookies de `example-bank.com`, y compris le cookie d'authentification.

Si le serveur de `example-bank.com` ne dispose pas de protection XSRF, il ne peut pas distinguer une requête légitime provenant de l'application de la requête falsifiée provenant de `evil.com`.

Pour l'éviter, l'application doit s'assurer qu'une requête utilisateur provient bien de la véritable application, et non d'un autre site.
Le serveur et le client doivent coopérer pour déjouer cette attaque.

Dans une technique anti-XSRF courante, le serveur de l'application envoie un jeton d'authentification créé aléatoirement dans un cookie.
Le code client lit le cookie et ajoute un en-tête de requête personnalisé contenant le jeton à toutes les requêtes suivantes.
Le serveur compare la valeur du cookie reçu à celle de l'en-tête de requête et rejette la requête si les valeurs sont absentes ou ne correspondent pas.

Cette technique est efficace parce que tous les navigateurs implémentent la _same-origin policy_ (politique de même origine).
Seul le code du site web sur lequel les cookies sont définis peut lire les cookies de ce site et définir des en-têtes personnalisés sur les requêtes envoyées à ce site.
Cela signifie que seule votre application peut lire ce jeton de cookie et définir l'en-tête personnalisé.
Le code malveillant de `evil.com` ne le peut pas.

### Sécurité XSRF/CSRF de `HttpClient` {#httpclient-xsrf-csrf-security}

`HttpClient` prend en charge un [mécanisme courant](https://en.wikipedia.org/wiki/Cross-site_request_forgery#Cookie-to-header_token) utilisé pour prévenir les attaques XSRF. Lors de l'exécution de requêtes HTTP, un intercepteur lit un jeton depuis un cookie, par défaut `XSRF-TOKEN`, et le définit comme en-tête HTTP, `X-XSRF-TOKEN`. Comme seul le code qui s'exécute sur votre domaine peut lire le cookie, le backend peut être certain que la requête HTTP provient de votre application cliente et non d'un attaquant.

Par défaut, un intercepteur envoie cet en-tête sur toutes les requêtes modifiantes (comme `POST`) vers des URL relatives et de même origine, mais pas sur les requêtes `GET` ou `HEAD`.

<docs-callout helpful title="Pourquoi ne pas protéger les requêtes GET ?">
La protection CSRF n'est nécessaire que pour les requêtes susceptibles de modifier l'état du backend. Par nature, les attaques CSRF franchissent les frontières de domaine, et la [same-origin policy](https://developer.mozilla.org/docs/Web/Security/Same-origin_policy) du web empêche une page malveillante de récupérer les résultats de requêtes `GET` authentifiées.
</docs-callout>

Pour en tirer parti, votre serveur doit définir un jeton dans un cookie de session lisible par JavaScript, appelé `XSRF-TOKEN`, soit au chargement de la page, soit lors de la première requête GET. Lors des requêtes suivantes, le serveur peut vérifier que le cookie correspond à l'en-tête HTTP `X-XSRF-TOKEN`, et donc être sûr que seul du code s'exécutant sur votre domaine a pu envoyer la requête. Le jeton doit être unique pour chaque utilisateur et vérifiable par le serveur ; cela empêche le client de fabriquer ses propres jetons. Pour plus de sécurité, définissez le jeton comme un condensé du cookie d'authentification de votre site, avec un sel.

Pour éviter les collisions dans les environnements où plusieurs applications Angular partagent le même domaine ou sous-domaine, donnez à chaque application un nom de cookie unique.

<docs-callout important title="HttpClient ne prend en charge que la partie client du mécanisme de protection XSRF">
  Votre service backend doit être configuré pour définir le cookie de votre page et pour vérifier que l'en-tête est présent sur toutes les requêtes concernées. Sans cela, la protection par défaut d'Angular est inefficace.
</docs-callout>

### Configurer des noms de cookie et d'en-tête personnalisés {#configure-custom-cookie-header-names}

Si votre service backend utilise des noms différents pour le cookie ou l'en-tête du jeton XSRF, utilisez `withXsrfConfiguration` pour remplacer les valeurs par défaut.

Ajoutez-le à l'appel `provideHttpClient` comme suit :

```ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withXsrfConfiguration({
        cookieName: 'CUSTOM_XSRF_TOKEN',
        headerName: 'X-Custom-Xsrf-Header',
      }),
    ),
  ],
};
```

### Désactiver la protection XSRF {#disabling-xsrf-protection}

Si le mécanisme de protection XSRF intégré ne fonctionne pas pour votre application, vous pouvez le désactiver avec la fonctionnalité `withNoXsrfProtection` :

```ts
export const appConfig: ApplicationConfig = {
  providers: [provideHttpClient(withNoXsrfProtection())],
};
```

Pour plus d'informations sur la CSRF selon l'Open Web Application Security Project \(OWASP\), consultez [Cross-Site Request Forgery (CSRF)](https://owasp.org/www-community/attacks/csrf) et [Cross-Site Request Forgery (CSRF) Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html).
L'article de l'université Stanford [Robust Defenses for Cross-Site Request Forgery](https://seclab.stanford.edu/websec/csrf/csrf.pdf) est une source riche en détails.

Voir aussi la [conférence de Dave Smith sur le XSRF à AngularConnect 2016](https://www.youtube.com/watch?v=9inczw6qtpY 'Cross Site Request Funkery Securing Your Angular Apps From Evil Doers').

### Inclusion de script intersites (XSSI) {#cross-site-script-inclusion-xssi}

L'inclusion de script intersites, également appelée vulnérabilité JSON, peut permettre au site web d'un attaquant de lire les données d'une API JSON.
L'attaque fonctionne sur les anciens navigateurs en remplaçant les constructeurs d'objets JavaScript intégrés, puis en incluant une URL d'API à l'aide d'une balise `<script>`.

Cette attaque ne réussit que si le JSON renvoyé est exécutable en tant que JavaScript.
Les serveurs peuvent prévenir une attaque en préfixant toutes les réponses JSON pour les rendre non exécutables, par convention avec la chaîne bien connue `")]}',\n"`.

La library `HttpClient` d'Angular reconnaît cette convention et supprime automatiquement la chaîne `")]}',\n"` de toutes les réponses avant de poursuivre l'analyse.

Pour plus d'informations, consultez la section XSSI de cet [article du blog Google sur la sécurité web](https://security.googleblog.com/2011/05/website-security-for-webmasters.html).

## Prévenir la falsification de requête côté serveur (SSRF) {#preventing-server-side-request-forgery-ssrf}

Angular inclut une validation stricte des en-têtes `Host`, `Forwarded`, `X-Forwarded-Host`, `X-Forwarded-Proto`, `X-Forwarded-Prefix` et `X-Forwarded-Port` dans le pipeline de traitement des requêtes, afin de prévenir la [falsification de requête côté serveur (SSRF)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/SSRF) basée sur les en-têtes.

Les règles de validation sont les suivantes :

- `Host`, `X-Forwarded-Host` et le paramètre `host` de l'en-tête `Forwarded` sont validés par rapport à une liste d'autorisation stricte et ne peuvent pas contenir de séparateurs de chemin.
- L'en-tête `X-Forwarded-Port` doit être numérique.
- L'en-tête `X-Forwarded-Proto` et le paramètre `proto` de l'en-tête `Forwarded` doivent valoir `http` ou `https`.
- L'en-tête `X-Forwarded-Prefix` doit commencer par `/` et ne contenir que des caractères alphanumériques, des tirets et des underscores, séparés par des barres obliques simples.
- Par défaut, l'en-tête `Forwarded` et tous les en-têtes `X-Forwarded-*` sont considérés comme non fiables et sont supprimés de la requête. Pour les conserver, ils doivent être explicitement autorisés via la configuration de `trustProxyHeaders`.

Les en-têtes invalides déclenchent un log d'erreur, et les en-têtes de proxy non autorisés sont supprimés de la requête. Les requêtes avec des noms d'hôte non reconnus aboutissent à une réponse `400 Bad Request`.

NOTE: La plupart des fournisseurs cloud et des fournisseurs de CDN effectuent automatiquement la validation de ces en-têtes avant même qu'une requête n'atteigne l'origine de l'application. Ce filtrage inhérent réduit considérablement la surface d'attaque en pratique.

### Configurer les hôtes autorisés {#configuring-allowed-hosts}

Pour autoriser des noms d'hôte spécifiques, vous devez les ajouter à la liste d'autorisation. C'est essentiel pour garantir que votre application fonctionne correctement et de manière sûre une fois déployée. Les motifs prennent en charge les wildcards pour une correspondance flexible des noms d'hôte.

Vous pouvez configurer l'option `allowedHosts` dans votre `angular.json` :

```json {hideCopy}
{
  // ...
  "projects": {
    "your-project-name": {
      // ...
      "architect": {
        "build": {
          "builder": "@angular/build:application",
          "options": {
            "security": {
              "allowedHosts": [
                "example.com",
                "*.example.com" // allows all subdomains of example.com
              ]
            }
            // ... other options
          }
        }
      }
    }
  }
}
```

Vous pouvez aussi configurer `allowedHosts` lors de l'initialisation du moteur de l'application :

```typescript
const appEngine = new AngularAppEngine({
  allowedHosts: ['example.com', '*.trusted-example.com'],
});

const nodeAppEngine = new AngularNodeAppEngine({
  allowedHosts: ['example.com', '*.trusted-example.com'],
});
```

Pour la variante Node.js `AngularNodeAppEngine`, vous pouvez aussi fournir la variable d'environnement `NG_ALLOWED_HOSTS` (liste séparée par des virgules) pour autoriser des hôtes.

```bash {hideDollar}
export NG_ALLOWED_HOSTS="example.com,*.trusted-example.com"
```

IMPORTANT: Vous pouvez utiliser `*` comme valeur dans `allowedHosts` pour autoriser tous les noms d'hôte, mais c'est généralement déconseillé et cela présente un risque de sécurité. Accepter n'importe quel en-tête host peut exposer votre application à des attaques par injection d'en-tête host et à des attaques de [falsification de requête côté serveur (SSRF)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/SSRF). Cette configuration ne doit être utilisée que lorsque la validation des en-têtes `Host` et `X-Forwarded-Host` est effectuée dans une autre couche, comme un load balancer ou un reverse proxy. Pour une meilleure sécurité, nous recommandons d'utiliser une liste explicite d'hôtes autorisés chaque fois que possible. Consultez [GHSA-x288-3778-4hhx](https://github.com/angular/angular-cli/security/advisories/GHSA-x288-3778-4hhx) pour plus de détails.

### Configurer les en-têtes de proxy de confiance {#configuring-trusted-proxy-headers}

Par défaut, Angular ignore l'en-tête standard `Forwarded` et tous les en-têtes `X-Forwarded-*`. Si votre application se trouve derrière un reverse proxy de confiance (comme un load balancer) qui définit ces en-têtes, vous pouvez configurer Angular pour qu'il leur fasse confiance.

Si l'en-tête `Forwarded` est de confiance, ses paramètres `host` et `proto` sont extraits et prennent le pas sur les en-têtes `x-forwarded-host` et `x-forwarded-proto` correspondants.

Vous pouvez configurer `trustProxyHeaders` lors de l'initialisation du moteur de l'application :

```typescript
const appEngine = new AngularAppEngine({
  trustProxyHeaders: ['forwarded'], // Trust the standard Forwarded header
});

const appEngine = new AngularAppEngine({
  trustProxyHeaders: ['x-forwarded-host', 'x-forwarded-proto'], // Trust non-standard headers
});

const nodeAppEngine = new AngularNodeAppEngine({
  trustProxyHeaders: true, // Trust standard Forwarded and all X-Forwarded-* headers
});
```

Pour la variante Node.js `AngularNodeAppEngine`, vous pouvez aussi fournir la variable d'environnement `NG_TRUST_PROXY_HEADERS` (avec comme valeur une liste d'en-têtes séparés par des virgules) pour autoriser l'utilisation de ces en-têtes.

```bash {hideDollar}
export NG_TRUST_PROXY_HEADERS="X-FORWARDED-HOST,X-FORWARDED-PREFIX"
```

IMPORTANT: N'activez `trustProxyHeaders` que si votre application se trouve derrière un proxy de confiance qui valide strictement ou remplace ces en-têtes. Sinon, des attaquants peuvent usurper ces en-têtes pour provoquer des attaques de [falsification de requête côté serveur (SSRF)](https://developer.mozilla.org/en-US/docs/Web/Security/Attacks/SSRF).

## Auditer les applications Angular {#auditing-angular-applications}

Les applications Angular doivent suivre les mêmes principes de sécurité que les applications web classiques, et doivent être auditées comme telles.
Les API propres à Angular qui doivent être auditées lors d'une revue de sécurité, comme les méthodes [_bypassSecurityTrust_](#trusting-safe-values), sont signalées dans la documentation comme sensibles du point de vue de la sécurité.
