<!-- source: 440df8883a4efd7b552fae23ba1b74f6640cb751 -->
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

[CliBuild]: cli/build 'ng build | CLI | Angular'
[GuideDeployment]: tools/cli/deployment 'Deployment | Angular'
[GuideWorkspaceConfig]: reference/configs/workspace-config 'Angular workspace configuration | Angular'
