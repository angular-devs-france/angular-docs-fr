<!-- source: 67a8cd6e304dc0a9c05764198f02cc0adc115c29 -->
# Styliser les composants

TIP: Ce guide suppose que vous avez déjà lu le [guide des Fondamentaux](essentials). Commencez par celui-ci si vous découvrez Angular.

Les composants peuvent, de façon facultative, inclure des styles CSS qui s'appliquent au DOM de ce composant :

```angular-ts {highlight:[4]}
@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
  styles: `
    img {
      border-radius: 50%;
    }
  `,
})
export class ProfilePhoto {}
```

Vous pouvez aussi choisir d'écrire vos styles dans des fichiers distincts :

```angular-ts {highlight:[4]}
@Component({
  selector: 'profile-photo',
  templateUrl: 'profile-photo.html',
  styleUrl: 'profile-photo.css',
})
export class ProfilePhoto {}
```

Quand Angular compile votre composant, ces styles sont émis avec la sortie JavaScript de votre
composant. Cela signifie que les styles de composant participent au système de modules JavaScript. Quand vous
rendez un composant Angular, le framework inclut automatiquement les styles qui lui sont associés, même lors du
lazy loading d'un composant.

Angular fonctionne avec n'importe quel outil qui produit du CSS,
notamment [Sass](https://sass-lang.com), [Less](https://lesscss.org)
et [Stylus](https://stylus-lang.com).

## Portée des styles {#style-scoping}

Chaque composant possède un paramètre d'**encapsulation des vues** qui détermine comment le framework limite la portée des
styles d'un composant. Il existe quatre modes d'encapsulation des vues : `Emulated`, `ShadowDom`, `ExperimentalIsolatedShadowDom` et `None`.
Vous pouvez spécifier le mode dans le décorateur `@Component` :

```angular-ts {highlight:[3]}
@Component({
  ...,
  encapsulation: ViewEncapsulation.None,
})
export class ProfilePhoto { }
```

### ViewEncapsulation.Emulated {#viewencapsulationemulated}

Par défaut, Angular utilise l'encapsulation émulée pour que les styles d'un composant ne s'appliquent qu'aux éléments
définis dans le template de ce composant. Dans ce mode, le framework génère un attribut HTML unique
pour chaque instance de composant, ajoute cet attribut aux éléments du template du composant et
insère cet attribut dans les selectors CSS définis dans les styles de votre composant.

Ce mode garantit que les styles d'un composant ne débordent pas et n'affectent pas les autres composants. En revanche,
les styles globaux définis en dehors d'un composant peuvent tout de même affecter les éléments à l'intérieur d'un composant qui utilise
l'encapsulation émulée.

En mode émulé, Angular prend en charge
la pseudo-classe [`:host`](https://developer.mozilla.org/docs/Web/CSS/:host).
Bien que la pseudo-classe [`:host-context()`](https://developer.mozilla.org/docs/Web/CSS/:host-context)
soit dépréciée dans les navigateurs modernes, le compilateur d'Angular la prend entièrement en charge. Les deux pseudo-classes
peuvent être utilisées sans dépendre du
[Shadow DOM](https://developer.mozilla.org/docs/Web/Web_Components/Using_shadow_DOM) natif.
Pendant la compilation, le framework transforme ces pseudo-classes en attributs, si bien qu'elles ne
suivent pas les règles de ces pseudo-classes natives à l'exécution (par exemple la compatibilité des navigateurs ou la spécificité). Le mode
d'encapsulation émulée d'Angular ne prend en charge aucune autre pseudo-classe liée au Shadow DOM, comme
`::shadow` ou `::part`.

#### `::ng-deep` {#ng-deep}

Le mode d'encapsulation émulée d'Angular prend en charge une pseudo-classe personnalisée, `::ng-deep`.
**L'équipe Angular déconseille fortement toute nouvelle utilisation de `::ng-deep`**. Ces API sont conservées
uniquement pour la rétrocompatibilité.

Quand un selector contient `::ng-deep`, Angular cesse d'appliquer les limites de l'encapsulation des vues à partir de ce point du selector. Toute partie du selector qui suit `::ng-deep` peut correspondre à des éléments situés en dehors du template du composant.

Par exemple :

- un selector de règle CSS comme `p a`, avec l'encapsulation émulée, correspondra aux éléments `<a>` descendants d'un élément `<p>`,
  tous deux situés dans le template du composant lui-même.

- Un selector comme `::ng-deep p a` correspondra aux éléments `<a>` situés n'importe où dans l'application, descendants d'un élément `<p>` situé n'importe où dans l'application.

  Il se comporte donc en pratique comme un style global.

- Dans `p ::ng-deep a`, Angular exige que l'élément `<p>` provienne du template du composant lui-même, mais l'élément `<a>` peut se trouver n'importe où dans l'application.

  En pratique, l'élément `<a>` peut donc se trouver dans le template du composant, ou dans n'importe lequel de ses contenus projetés ou enfants.

- Avec `:host ::ng-deep p a`, les éléments `<a>` et `<p>` doivent tous deux être des descendants du host element du composant.

  Ils peuvent provenir du template du composant ou des vues de ses composants enfants, mais pas d'ailleurs dans l'application.

### ViewEncapsulation.ShadowDom {#viewencapsulationshadowdom}

Ce mode limite la portée des styles à l'intérieur d'un composant en
utilisant [l'API standard du web Shadow DOM](https://developer.mozilla.org/docs/Web/Web_Components/Using_shadow_DOM).
Quand vous activez ce mode, Angular attache une shadow root au host element du composant et rend
le template et les styles du composant dans le shadow tree correspondant.

Les styles à l'intérieur du shadow tree ne peuvent pas affecter les éléments situés en dehors de ce shadow tree.

Activer l'encapsulation `ShadowDom` a toutefois des conséquences qui vont au-delà de la portée des styles. Rendre le
composant dans un shadow tree affecte la propagation des événements, l'interaction
avec [l'API `<slot>`](https://developer.mozilla.org/docs/Web/Web_Components/Using_templates_and_slots)
et la façon dont les outils de développement du navigateur affichent les éléments. Comprenez toujours toutes les implications de l'utilisation du
Shadow DOM dans votre application avant d'activer cette option.

### ViewEncapsulation.ExperimentalIsolatedShadowDom {#viewencapsulationexperimentalisolatedshadowdom}

Se comporte comme ci-dessus, sauf que ce mode garantit strictement que _seuls_ les styles de ce composant s'appliquent aux éléments du
template du composant. Les styles globaux ne peuvent pas affecter les éléments d'un shadow tree et les styles à l'intérieur du
shadow tree ne peuvent pas affecter les éléments situés en dehors de ce shadow tree.

### ViewEncapsulation.None {#viewencapsulationnone}

Ce mode désactive toute encapsulation des styles pour le composant. Tous les styles associés au
composant se comportent comme des styles globaux.

NOTE: Dans les modes `Emulated` et `ShadowDom`, Angular ne garantit pas à 100 % que les styles de votre composant remplaceront toujours les styles provenant de l'extérieur.
On suppose que ces styles ont la même spécificité que les styles de votre composant en cas de conflit.

## Définir des styles dans les templates {#defining-styles-in-templates}

Vous pouvez utiliser l'élément `<style>` dans le template d'un composant pour définir des styles supplémentaires. Le
mode d'encapsulation des vues du composant s'applique aux styles définis de cette façon.

Angular ne prend pas en charge les bindings à l'intérieur des éléments style.

## Référencer des fichiers de style externes {#referencing-external-style-files}

Les templates de composant peuvent
utiliser [l'élément `<link>`](https://developer.mozilla.org/docs/Web/HTML/Element/link) pour
référencer des fichiers CSS. De plus, votre CSS peut
utiliser [la règle `@import`](https://developer.mozilla.org/docs/Web/CSS/@import) pour référencer
des fichiers CSS. Angular traite ces références comme des styles _externes_. Les styles externes ne sont pas affectés par
l'encapsulation des vues émulée.

## Appliquer un namespace aux propriétés personnalisées CSS {#namespacing-css-custom-properties}

Angular peut ajouter un préfixe aux propriétés personnalisées CSS (aussi appelées variables CSS) que les
styles de votre composant déclarent et lisent. Les propriétés personnalisées sont héritées le long de l'arbre du DOM, donc quand quelque chose
en dehors de votre application définit une propriété personnalisée comme `--primary-color` sur un élément
ancêtre, vos composants lisent cette valeur. Cela compte quand votre application partage une page avec une autre
application ou avec du balisage que vous ne contrôlez pas. Angular n'applique pas de namespace aux propriétés personnalisées tant que
vous ne le lui demandez pas, et une application qui possède sa page n'a pas besoin de namespace.

Pour limiter la portée des propriétés personnalisées des styles de vos composants à votre application, ajoutez
[`provideCssVarNamespacing`](api/platform-browser/provideCssVarNamespacing) aux providers de votre
application. Il utilise l'[`APP_ID`](api/core/APP_ID) de l'application comme namespace :

```ts {header: "app.config.ts"}
import {APP_ID, ApplicationConfig} from '@angular/core';
import {provideCssVarNamespacing} from '@angular/platform-browser';

export const appConfig: ApplicationConfig = {
  providers: [{provide: APP_ID, useValue: 'my-app'}, provideCssVarNamespacing()],
};
```

Angular préfixe les propriétés personnalisées des styles de vos composants avec ce namespace suivi d'un
tiret bas, si bien que `--primary-color` devient `--my-app_primary-color`. Le préfixe s'applique aux
déclarations, aux références `var()`, aux règles `@property` et aux style bindings comme
`[style.--primary-color]`, y compris les style bindings qu'un composant déclare dans son objet `host`.

`APP_ID` vaut `ng` sauf si vous le définissez, donc donnez à chaque application de la page son propre `APP_ID`.
Sinon, les applications partagent un préfixe et entrent de nouveau en conflit. Pour utiliser un namespace différent de
l'identifiant de l'application, passez-le à `provideCssVarNamespacing`. Angular ajoute lui-même le tiret bas :
`provideCssVarNamespacing('my-app_')` produit `--my-app__primary-color`.

Angular applique le namespace aux styles qu'il compile dans un composant : les `styles` et le `styleUrl` du
composant, les styles que vous [écrivez dans un élément `<style>`](#defining-styles-in-templates) de son
template, les feuilles de style que son template référence avec un `<link rel="stylesheet">` relatif, et
les styles d'un composant qui utilise `ViewEncapsulation.None`. Angular n'applique pas de namespace à une feuille de style que le navigateur
charge à l'exécution, comme une feuille de style globale listée dans votre configuration de build ou un
[style externe](#referencing-external-style-files) que votre build n'intègre pas.

Le namespace s'applique à chaque composant compilé par Angular, y compris les composants des libraries
que vous installez. Quand les styles d'une library lisent une propriété personnalisée définie par une feuille de style globale, comme
les propriétés d'un thème, la référence ne correspond plus, le navigateur utilise la valeur de repli de `var()`
s'il y en a une ou, à défaut,
[la valeur héritée ou initiale de la propriété](https://www.w3.org/TR/css-variables-1/#invalid-variables),
et aucune erreur n'est signalée. Avant d'activer le namespace dans une
application existante, passez en revue les propriétés personnalisées partagées entre vos feuilles de style globales et
vos composants.

IMPORTANT: Angular ne réécrit les noms des propriétés personnalisées que dans les styles et les bindings qu'il compile.
Partout ailleurs, le nom que vous écrivez est conservé, et rien ne signale l'incohérence.

Angular ne réécrit pas le nom dans :

- Les attributs de style statiques, comme `style="--primary-color: red"`, y compris l'entrée `style` de l'objet
  `host` d'un composant.
- Les style bindings sous forme d'objet ou de chaîne, comme `[style]="{'--primary-color': color}"`,
  `[style]="'--primary-color: red'"` et `ngStyle`, y compris l'entrée `[style]` de l'objet `host`
  d'un composant.
- Les noms de propriétés personnalisées à l'intérieur de la valeur d'un binding, comme
  `[style.border-color]="'var(--primary-color)'"`.
- Les appels à `Renderer2.setStyle`.

Chacun de ces cas produit une propriété que vos styles avec namespace ne lisent plus. Appliquez vous-même le namespace à ces
noms, comme décrit dans
[Utiliser les propriétés avec namespace en TypeScript](#using-namespaced-properties-in-typescript).

### Exclure des propriétés du namespace {#opting-out-of-namespacing}

Pour déclarer ou lire une propriété personnalisée à laquelle Angular n'applique pas de namespace, comme une propriété définie dans une
feuille de style globale, préfixez son nom par `--global--`. Angular retire `--global` et laisse inchangés
le `--` restant et le reste du nom :

```css
:host {
  /* Declares --accent-color and reads --brand-color, not --my-app_brand-color. */
  --global--accent-color: navy;
  color: var(--global--brand-color);
}
```

Écrivez deux tirets après `global`. Un seul tiret, comme dans `--global-brand-color`, ne suffit pas à exclure la propriété,
et Angular applique le namespace à ce nom comme à n'importe quel autre. Évitez les noms qui commencent par `--global-` suivi
d'autre chose qu'un second tiret ; une future version majeure prévoit de les rejeter au moment du build.
Angular retire `--global` dans toutes les applications, y compris celles qui ne configurent jamais
de namespace.

### Utiliser les propriétés avec namespace en TypeScript {#using-namespaced-properties-in-typescript}

Préférez un style binding comme `[style.--primary-color]`, auquel Angular applique le namespace pour vous. Quand
vous passez plutôt par une API du DOM, passez le nom que vous avez écrit dans vos styles à
[`CssVarNamespacer`](api/platform-browser/CssVarNamespacer), y compris le `--` initial :

```angular-ts {header: "profile-photo.ts"}
import {Component, ElementRef, inject} from '@angular/core';
import {CssVarNamespacer} from '@angular/platform-browser';

@Component({
  selector: 'profile-photo',
  template: `<img src="profile-photo.jpg" alt="Your profile photo" />`,
  styles: `
    img {
      border: 2px solid var(--primary-color);
    }
  `,
})
export class ProfilePhoto {
  private readonly host: HTMLElement = inject(ElementRef).nativeElement;
  private readonly cssVarNamespacer = inject(CssVarNamespacer);

  setPrimaryColor(color: string): void {
    this.host.style.setProperty(this.cssVarNamespacer.namespace('--primary-color'), color);
  }
}
```

Utilisez le nom simple pour une propriété que vous avez déclarée avec `--global--`, car Angular ne leur applique jamais
de namespace. Pour tout le reste, `namespace` renvoie le nom inchangé quand une application ne configure
aucun namespace, si bien qu'une library peut l'appeler pour les propriétés personnalisées que ses propres styles déclarent.
