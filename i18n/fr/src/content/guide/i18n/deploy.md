<!-- source: 601120cfd251db1d1ba5fd007151b9868798ed3b -->
# Déployer plusieurs locales

Si `myapp` est le répertoire qui contient les fichiers distribuables de votre projet, vous mettez généralement à disposition différentes versions pour différentes locales dans des répertoires de locale.
Par exemple, votre version française se trouve dans le répertoire `myapp/fr` et la version espagnole dans le répertoire `myapp/es`.

La balise HTML `base` avec l'attribut `href` spécifie l'URI de base, ou URL, pour les liens relatifs.
Si vous définissez l'option `"localize"` dans le fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig] à `true` ou à un tableau d'identifiants de locale, la CLI ajuste le `href` de base pour chaque version de l'application.
Pour ajuster le `href` de base pour chaque version de l'application, la CLI utilise le `"subPath"` configuré comme segment d'URL pour la locale.
Si vous ne définissez pas `"subPath"`, sa valeur par défaut est l'identifiant de locale.
Spécifiez le `"subPath"` de chaque locale dans votre fichier de configuration de build du workspace [`angular.json`][GuideWorkspaceConfig].
L'exemple suivant montre `"subPath"` défini sur une chaîne vide.

<docs-code header="angular.json" path="adev/src/content/examples/i18n/angular.json" region="i18n-subPath"/>

## Configurer un serveur {#configure-a-server}

Un déploiement typique de plusieurs langues sert chaque langue depuis un sous-répertoire différent.
Les utilisateurs sont redirigés vers la langue préférée définie dans le navigateur grâce à l'en-tête HTTP `Accept-Language`.
Si l'utilisateur n'a pas défini de langue préférée, ou si la langue préférée n'est pas disponible, le serveur se rabat sur la langue par défaut.
Pour changer de langue, changez votre emplacement actuel pour un autre sous-répertoire.
Le changement de sous-répertoire se fait souvent au moyen d'un menu implémenté dans l'application.

Pour en savoir plus sur le déploiement d'applications sur un serveur distant, consultez [Déploiement][GuideDeployment].

IMPORTANT: Si vous utilisez le [rendu côté serveur](guide/ssr) avec `outputMode` défini sur `server`, Angular gère automatiquement la redirection de manière dynamique en fonction de l'en-tête HTTP `Accept-Language`. Cela simplifie le déploiement en supprimant le besoin d'ajustements manuels du serveur ou de la configuration.

### Exemple Nginx {#nginx-example}

L'exemple suivant montre une configuration Nginx.

<docs-code path="adev/src/content/examples/i18n/doc-files/nginx.conf" language="nginx"/>

### Exemple Apache {#apache-example}

L'exemple suivant montre une configuration Apache.

<docs-code path="adev/src/content/examples/i18n/doc-files/apache2.conf" language="apache"/>

## Prérendu {#prerendering}

Si vous définissez `outputMode` à `static`, la CLI prérend au moment du build, pour chaque locale, les routes que vos routes serveur configurent avec `RenderMode.Prerender`, et ne produit aucun serveur.
Pour plus d'informations, consultez [Générer une application entièrement statique](guide/ssr#generate-a-fully-static-application).
Comme aucun serveur ne choisit la langue, c'est votre configuration d'hébergement qui gère les redirections et les pages introuvables.

### Organisation de la sortie {#output-layout}

La CLI écrit chaque locale dans son propre répertoire sous `browser/`.
Chaque répertoire contient le HTML prérendu, les bundles et une copie des fichiers de `public/` pour cette locale.
L'attribut `lang` et le `<base href>` des pages prérendues correspondent à la locale.

L'organisation suivante suppose que la locale source est servie depuis la racine, comme décrit dans la section suivante.

```text
browser/
├── index.html
├── about/index.html
├── tr/
│   ├── index.html
│   └── about/index.html
└── es/
    ├── index.html
    └── about/index.html
```

Les pages à la racine utilisent `lang="en"` et `<base href="/">`. Les pages de `tr/` utilisent `lang="tr"` et `<base href="/tr/">`, et les pages de `es/` utilisent `lang="es"` et `<base href="/es/">`.

### Servir la locale source depuis la racine {#serve-the-source-locale-from-the-root}

Pour servir la locale source depuis `/` et imbriquer les autres locales en dessous, utilisez la forme objet de `sourceLocale` et définissez `"subPath"` à une chaîne vide.

```json
{
  "i18n": {
    "sourceLocale": {
      "code": "en",
      "subPath": ""
    },
    "locales": {
      "tr": "src/locale/messages.tr.xlf",
      "es": "src/locale/messages.es.xlf"
    }
  }
}
```

Sans le `"subPath"` vide, la CLI place la locale source dans son propre répertoire nommé d'après le code de la locale, par exemple `/en/`.

### Rediriger les utilisateurs vers leur langue {#redirect-users-to-their-language}

La sortie statique ne redirige pas en fonction de l'en-tête HTTP `Accept-Language`.
Configurez votre hébergeur pour rediriger en fonction de cet en-tête.
Les exemples [Nginx](#nginx-example) et [Apache](#apache-example) supposent que chaque locale, y compris celle par défaut, se trouve dans son propre sous-répertoire.
Si vous servez la locale source depuis la racine, adaptez-les pour ne rediriger `/` que vers les autres locales.

### Pages introuvables {#not-found-pages}

La CLI ne prérend pas de page pour la route joker `**`.
Pour obtenir une page introuvable, déclarez une route concrète telle que `404` et assurez-vous que vos routes serveur la prérendent.
Chaque locale obtient alors sa propre page, par exemple `/404/index.html` et `/tr/404/index.html`.
Configurez votre hébergeur pour servir la page correspondant au chemin de locale de l'URL manquante, par exemple avec `error_page` dans Nginx ou `ErrorDocument` dans Apache.

[CliBuild]: cli/build 'ng build | CLI | Angular'
[GuideDeployment]: tools/cli/deployment 'Deployment | Angular'
[GuideWorkspaceConfig]: reference/configs/workspace-config 'Angular workspace configuration | Angular'
