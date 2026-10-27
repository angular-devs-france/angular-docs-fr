<!-- source: a0caa5d7037caf6f253096cd147855e8e40cbfac -->
# Bases du test de composants

Un composant, contrairement à toutes les autres parties d'une application Angular, combine un template HTML et une classe TypeScript.
Le composant, c'est véritablement le template et la classe _qui fonctionnent ensemble_.
Pour tester correctement un composant, vous devez vérifier qu'ils fonctionnent ensemble comme prévu.

De tels tests nécessitent de créer le host element du composant dans le DOM du navigateur, comme le fait Angular, et d'examiner l'interaction de la classe du composant avec le DOM telle que la décrit son template.

Le `TestBed` d'Angular facilite ce type de test, comme vous le verrez dans les sections suivantes.
Mais dans de nombreux cas, _tester la classe du composant seule_, sans faire intervenir le DOM, permet de valider une grande partie du comportement du composant de manière simple et plus évidente.

## Tester le DOM d'un composant {#component-dom-testing}

Un composant ne se résume pas à sa classe.
Un composant interagit avec le DOM et avec d'autres composants.
Les classes seules ne peuvent pas vous dire si le composant va s'afficher correctement, réagir aux saisies et aux gestes de l'utilisateur, ou s'intégrer à ses composants parent et enfants.

- `Lightswitch.clicked()` est-il lié à quelque chose qui permette à l'utilisateur de l'invoquer ?
- `Lightswitch.message` est-il affiché ?
- L'utilisateur peut-il réellement sélectionner le héros affiché par le composant `DashboardHero` ?
- Le nom du héros s'affiche-t-il comme prévu \(par exemple en majuscules\) ?
- Le message de bienvenue est-il affiché par le template du composant `Welcome` ?

Ces questions ne posent peut-être pas de difficulté pour les composants simples présentés précédemment.
Mais de nombreux composants ont des interactions complexes avec les éléments du DOM décrits dans leurs templates, ce qui fait apparaître et disparaître du HTML à mesure que l'état du composant change.

Pour répondre à ce genre de questions, vous devez créer les éléments du DOM associés aux composants, examiner le DOM pour confirmer que l'état du composant s'affiche correctement aux moments appropriés, et simuler l'interaction de l'utilisateur avec l'écran pour déterminer si ces interactions font que le composant se comporte comme prévu.

Pour écrire ce genre de tests, vous utiliserez des fonctionnalités supplémentaires du `TestBed` ainsi que d'autres utilitaires de test.

### Tests générés par la CLI {#cli-generated-tests}

Par défaut, la CLI crée pour vous un fichier de test initial quand vous lui demandez de générer un nouveau composant.

Par exemple, la commande CLI suivante génère un composant `Banner` dans le dossier `app/banner` \(avec un template et des styles inline\) :

```shell
ng generate component banner --inline-template --inline-style
```

Elle génère aussi un fichier de test initial pour le composant, `banner.spec.ts`, qui ressemble à ceci :

```ts
import {ComponentFixture, TestBed} from '@angular/core/testing';
import {Banner} from './banner';

describe('Banner', () => {
  let component: Banner;
  let fixture: ComponentFixture<Banner>;

  beforeEach(async () => {
    TestBed.configureTestingModule({});

    fixture = TestBed.createComponent(Banner);
    component = fixture.componentInstance;
    await fixture.whenStable();
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

### Réduire la configuration {#reduce-the-setup}

Seules les trois dernières lignes de ce fichier testent réellement le composant, et elles se contentent de vérifier qu'Angular peut créer le composant.

Le reste du fichier est du code de configuration standard qui anticipe des tests plus avancés, qui _pourraient_ devenir nécessaires si le composant évolue vers quelque chose de conséquent.

Vous découvrirez ces fonctionnalités de test avancées dans les sections suivantes.
Pour l'instant, vous pouvez réduire radicalement ce fichier de test à une taille plus raisonnable :

```ts
describe('Banner (minimal)', () => {
  it('should create', () => {
    const fixture = TestBed.createComponent(Banner);
    const component = fixture.componentInstance;
    expect(component).toBeDefined();
  });
});
```

Plus tard, vous appellerez `TestBed.configureTestingModule()` avec des imports, des providers et d'autres déclarations selon vos besoins de test.
Des méthodes `override` facultatives permettent d'affiner davantage certains aspects de la configuration.

NOTE: `TestBed.compileComponents` n'est nécessaire que lorsque des blocs `@defer` sont utilisés dans les composants testés.

### `createComponent()` {#createcomponent}

Après avoir configuré `TestBed`, appelez sa méthode `createComponent()`.

```ts
const fixture = TestBed.createComponent(Banner);
```

`TestBed.createComponent()` crée une instance du composant `Banner`, ajoute un élément correspondant au DOM du test runner et renvoie une [`ComponentFixture`](#componentfixture).

IMPORTANT: Ne reconfigurez pas `TestBed` après avoir appelé `createComponent`.

La méthode `createComponent` fige la définition actuelle de `TestBed`, qui n'accepte alors plus aucune configuration.

Vous ne pouvez plus appeler aucune méthode de configuration de `TestBed`, ni `configureTestingModule()`, ni aucune des méthodes `override...`.
Si vous essayez, `TestBed` lève une erreur.

### `ComponentFixture` {#componentfixture}

La [`ComponentFixture`](api/core/testing/ComponentFixture) est un harnais de test qui permet d'interagir avec le composant créé et l'élément correspondant.

Accédez à l'instance du composant via la fixture et confirmez qu'elle existe avec une attente :

```ts
const component = fixture.componentInstance;
expect(component).toBeDefined();
```

### `beforeEach()` {#beforeeach}

Vous ajouterez d'autres tests à mesure que ce composant évoluera.
Plutôt que de dupliquer la configuration de `TestBed` pour chaque test, refactorisez pour extraire la configuration dans un `beforeEach()` et quelques variables auxiliaires :

```ts
describe('Banner (with beforeEach)', () => {
  let component: Banner;
  let fixture: ComponentFixture<Banner>;

  beforeEach(async () => {
    fixture = TestBed.createComponent(Banner);
    component = fixture.componentInstance;

    await fixture.whenStable(); // necessary to wait for the initial rendering
  });

  it('should create', () => {
    expect(component).toBeDefined();
  });
});
```

HELPFUL: En attendant le rendu initial dans le `beforeEach` avec `await fixture.whenStable`, les tests individuels peuvent être synchrones.

Ajoutez maintenant un test qui récupère l'élément du composant depuis `fixture.nativeElement` et recherche le texte attendu.

```ts
it('should contain "banner works!"', () => {
  const bannerElement: HTMLElement = fixture.nativeElement;
  expect(bannerElement.textContent).toContain('banner works!');
});
```

### Créer une fonction `setup` {#create-a-setup-function}

Comme alternative à `beforeEach`, vous pouvez aussi créer une fonction de configuration que vous appellerez dans chaque test.
Une fonction de configuration a l'avantage d'être personnalisable via des paramètres.

Voici un exemple de ce à quoi pourrait ressembler une fonction de configuration :

```ts
function setup(providers?: Provider[]): ComponentFixture<Banner> {
  TestBed.configureTestingModule({providers});
  return TestBed.createComponent(Banner);
}
```

### `nativeElement` {#nativeelement}

La valeur de `ComponentFixture.nativeElement` est de type `any`.
Vous rencontrerez plus tard `DebugElement.nativeElement`, qui est lui aussi de type `any`.

Angular ne peut pas savoir à la compilation quel type d'élément HTML est le `nativeElement`, ni même s'il s'agit d'un élément HTML.
L'application peut s'exécuter sur une _plateforme autre qu'un navigateur_, comme le serveur ou un environnement Node, où l'élément peut avoir une API réduite, voire ne pas exister du tout.

Les tests de ce guide sont conçus pour s'exécuter dans un navigateur, donc une valeur `nativeElement` sera toujours un `HTMLElement` ou l'une de ses classes dérivées.

Sachant qu'il s'agit d'une forme d'`HTMLElement`, utilisez le `querySelector` HTML standard pour explorer plus profondément l'arbre des éléments.

Voici un autre test qui appelle `HTMLElement.querySelector` pour récupérer l'élément paragraphe et y rechercher le texte de la bannière :

```ts
it('should have <p> with "banner works!"', () => {
  const bannerElement: HTMLElement = fixture.nativeElement;
  const p = bannerElement.querySelector('p')!;
  expect(p.textContent).toEqual('banner works!');
});
```

### `DebugElement` {#debugelement}

La _fixture_ d'Angular fournit directement l'élément du composant via `fixture.nativeElement`.

```ts
const bannerElement: HTMLElement = fixture.nativeElement;
```

Il s'agit en réalité d'une méthode de commodité, implémentée sous la forme `fixture.debugElement.nativeElement`.

```ts
const bannerDe: DebugElement = fixture.debugElement;
const bannerEl: HTMLElement = bannerDe.nativeElement;
```

Ce chemin détourné vers l'élément a une bonne raison d'être.

Les propriétés du `nativeElement` dépendent de l'environnement d'exécution.
Vous pourriez exécuter ces tests sur une plateforme _autre qu'un navigateur_, qui n'a pas de DOM ou dont l'émulation du DOM ne prend pas en charge l'intégralité de l'API `HTMLElement`.

Angular s'appuie sur l'abstraction `DebugElement` pour fonctionner de manière sûre sur _toutes les plateformes prises en charge_.
Au lieu de créer un arbre d'éléments HTML, Angular crée un arbre de `DebugElement` qui enveloppe les _éléments natifs_ de la plateforme d'exécution.
La propriété `nativeElement` extrait l'élément enveloppé par le `DebugElement` et renvoie l'objet élément propre à la plateforme.

Comme les tests d'exemple de ce guide sont conçus pour s'exécuter uniquement dans un navigateur, un `nativeElement` est toujours, dans ces tests, un `HTMLElement` dont vous pouvez explorer les méthodes et propriétés familières dans un test.

Voici le test précédent, réimplémenté avec `fixture.debugElement.nativeElement` :

```ts
it('should find the <p> with fixture.debugElement.nativeElement', () => {
  const bannerDe: DebugElement = fixture.debugElement;
  const bannerEl: HTMLElement = bannerDe.nativeElement;
  const p = bannerEl.querySelector('p')!;
  expect(p.textContent).toEqual('banner works!');
});
```

Le `DebugElement` possède d'autres méthodes et propriétés utiles dans les tests, comme vous le verrez ailleurs dans ce guide.

Vous importez le symbole `DebugElement` depuis la library core d'Angular.

```ts
import {DebugElement} from '@angular/core';
```

### `By.css()` {#bycss}

Bien que les tests de ce guide s'exécutent tous dans le navigateur, certaines applications peuvent s'exécuter sur une plateforme différente, au moins une partie du temps.

Par exemple, le composant peut d'abord être rendu sur le serveur dans le cadre d'une stratégie visant à accélérer le lancement de l'application sur les appareils mal connectés.
Le renderer côté serveur peut ne pas prendre en charge l'intégralité de l'API des éléments HTML.
S'il ne prend pas en charge `querySelector`, le test précédent pourrait échouer.

Le `DebugElement` propose des méthodes de query qui fonctionnent sur toutes les plateformes prises en charge.
Ces méthodes de query prennent une fonction _prédicat_ qui renvoie `true` quand un nœud de l'arbre de `DebugElement` correspond aux critères de sélection.

Vous créez un _prédicat_ à l'aide d'une classe `By` importée depuis une library propre à la plateforme d'exécution.
Voici l'import de `By` pour la plateforme navigateur :

```ts
import {By} from '@angular/platform-browser';
```

L'exemple suivant réimplémente le test précédent avec `DebugElement.query()` et la méthode `By.css` du navigateur.

```ts
it('should find the <p> with fixture.debugElement.query(By.css)', () => {
  const bannerDe: DebugElement = fixture.debugElement;
  const paragraphDe = bannerDe.query(By.css('p'));
  const p: HTMLElement = paragraphDe.nativeElement;
  expect(p.textContent).toEqual('banner works!');
});
```

Quelques observations notables :

- La méthode statique `By.css()` sélectionne les nœuds `DebugElement` avec un [selector CSS standard](https://developer.mozilla.org/docs/Learn/CSS/Building_blocks/Selectors 'CSS selectors').
- La query renvoie un `DebugElement` pour le paragraphe.
- Vous devez extraire l'élément enveloppé dans ce résultat pour obtenir l'élément paragraphe.

Quand vous filtrez par selector CSS et ne testez que les propriétés de l'_élément natif_ d'un navigateur, l'approche `By.css` peut être excessive.

Il est souvent plus simple et plus clair de filtrer avec une méthode standard de `HTMLElement` comme `querySelector()` ou `querySelectorAll()`.
