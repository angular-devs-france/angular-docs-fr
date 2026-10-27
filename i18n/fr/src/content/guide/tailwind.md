<!-- source: 36cd034d9f5a761ca7147c3e26303a133abb600c -->
# Utiliser Tailwind CSS avec Angular

[Tailwind CSS](https://tailwindcss.com/) est un framework CSS « utility-first » qui permet de construire des sites web modernes sans jamais quitter votre HTML. Ce guide vous accompagne dans la mise en place de Tailwind CSS dans votre projet Angular.

## Configuration automatisée avec `ng add` {#automated-setup-with-ng-add}

Angular CLI offre un moyen simple d'intégrer Tailwind CSS à votre projet grâce à la commande `ng add`. Cette commande installe automatiquement les packages nécessaires, configure Tailwind CSS et met à jour les paramètres de build de votre projet.

Commencez par vous placer dans le répertoire racine de votre projet Angular dans un terminal, puis exécutez la commande suivante :

```shell
ng add tailwindcss
```

Cette commande effectue les actions suivantes :

- Installe `tailwindcss` et ses dépendances peer.
- Configure le projet pour utiliser Tailwind CSS.
- Ajoute l'instruction `@import` de Tailwind CSS à vos styles.

Après avoir exécuté `ng add tailwindcss`, vous pouvez immédiatement utiliser les classes utilitaires de Tailwind dans les templates de vos composants.

## Configuration manuelle (méthode alternative) {#manual-setup-alternative-method}

Si vous préférez configurer Tailwind CSS manuellement, suivez ces étapes :

### 1. Créer un projet Angular {#1-create-an-angular-project}

Commencez par créer un nouveau projet Angular si vous n'en avez pas déjà un.

```shell
ng new my-project
cd my-project
```

### 2. Installer Tailwind CSS {#2-install-tailwind-css}

Ensuite, ouvrez un terminal dans le répertoire racine de votre projet Angular et exécutez la commande suivante pour installer Tailwind CSS et ses dépendances peer :

<docs-code-multifile>
  <docs-code header="npm" language="shell">
    npm install tailwindcss @tailwindcss/postcss postcss
  </docs-code>
  <docs-code header="yarn" language="shell">
    yarn add tailwindcss @tailwindcss/postcss postcss
  </docs-code>
  <docs-code header="pnpm" language="shell">
    pnpm add tailwindcss @tailwindcss/postcss postcss
  </docs-code>
  <docs-code header="bun" language="shell">
    bun add tailwindcss @tailwindcss/postcss postcss
  </docs-code>
</docs-code-multifile>

### 3. Configurer les plugins PostCSS {#3-configure-postcss-plugins}

Ensuite, ajoutez un fichier `.postcssrc.json` à la racine du projet.
Ajoutez le plugin `@tailwindcss/postcss` à votre configuration PostCSS.

```json {header: '.postcssrc.json'}
{
  "plugins": {
    "@tailwindcss/postcss": {}
  }
}
```

### 4. Importer Tailwind CSS {#4-import-tailwind-css}

Ajoutez à `./src/styles.css` un `@import` qui importe Tailwind CSS.

```css {header: "src/styles.css"}
@import 'tailwindcss';
```

Si vous utilisez SCSS, ajoutez `@use` à `./src/styles.scss`.

```scss {header: "src/styles.scss"}
@use 'tailwindcss';
```

### 5. Commencer à utiliser Tailwind dans votre projet {#5-start-using-tailwind-in-your-project}

Vous pouvez maintenant utiliser les classes utilitaires de Tailwind dans les templates de vos composants pour styliser votre application. Lancez votre build avec `ng serve` : vous devriez voir le titre stylisé.

Par exemple, vous pouvez ajouter le code suivant à votre fichier `app.html` :

```html
<h1 class="text-3xl font-bold underline">Hello world!</h1>
```

## Ressources supplémentaires {#additional-resources}

- [Documentation de Tailwind CSS](https://tailwindcss.com/docs)
