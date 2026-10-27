<!-- source: 101524fe744b0ca0d70f8d8151a0237716bddfcf -->
# Les espaces dans les templates

Par défaut, les templates Angular ne conservent pas les espaces que le framework juge inutiles. Cela se produit généralement dans deux situations : les espaces entre les éléments, et les espaces réductibles à l'intérieur du texte.

## Espaces entre les éléments {#whitespace-between-elements}

La plupart des développeurs préfèrent formater leurs templates avec des retours à la ligne et de l'indentation pour les rendre lisibles :

```html
<section>
  <h3>User profile</h3>
  <label>
    User name
    <input />
  </label>
</section>
```

Ce template contient des espaces entre tous les éléments. L'extrait suivant montre le même HTML où chaque caractère d'espacement est remplacé par le caractère dièse (`#`) pour mettre en évidence la quantité d'espaces présents :

<!-- prettier-ignore -->
```html
<!-- Total Whitespace: 20 -->
<section>###<h3>User profile</h3>###<label>#####User name#####<input>###</label>#</section>
```

Conserver les espaces tels qu'ils sont écrits dans le template produirait de nombreux [nœuds texte](https://developer.mozilla.org/en-US/docs/Web/API/Text) inutiles et alourdirait le rendu de la page. En ignorant ces espaces entre les éléments, Angular effectue moins de travail lors du rendu du template sur la page, ce qui améliore les performances globales.

## Espaces réductibles à l'intérieur du texte {#collapsible-whitespace-inside-text}

Lorsque votre navigateur web rend du HTML sur une page, il réduit plusieurs caractères d'espacement consécutifs à un seul caractère :

<!-- prettier-ignore -->
```html
<!-- What it looks like in the template -->
<p>Hello         world</p>
```

Dans cet exemple, le navigateur n'affiche qu'une seule espace entre « Hello » et « world ».

```angular-html
<!-- What shows up in the browser -->
<p>Hello world</p>
```

Consultez [How whitespace is handled by HTML, CSS, and in the DOM](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model/Whitespace) pour en savoir plus sur ce fonctionnement.

Angular évite d'envoyer ces caractères d'espacement inutiles au navigateur dès le départ en les réduisant à un seul caractère lors de la compilation du template.

## Conserver les espaces {#preserving-whitespace}

Vous pouvez demander à Angular de conserver les espaces d'un template en spécifiant `preserveWhitespaces: true` dans le décorateur `@Component` de ce template.

```angular-ts
@Component({
  /* ... */,
  preserveWhitespaces: true,
  template: `
    <p>Hello         world</p>
  `
})
```

Évitez de définir cette option sauf en cas de nécessité absolue. Conserver les espaces peut amener Angular à produire beaucoup plus de nœuds lors du rendu, ce qui ralentit votre application.

Vous pouvez également utiliser une entité HTML spéciale propre à Angular, `&ngsp;`. Cette entité produit un unique caractère espace qui est conservé dans la sortie compilée.
