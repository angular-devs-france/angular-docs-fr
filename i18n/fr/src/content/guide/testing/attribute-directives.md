<!-- source: 7bb4e4ff366d246433f030fcf152c73e1df8d4a2 -->
# Tester les directives d'attribute

Une _directive d'attribute_ modifie le comportement d'un élément, d'un composant ou d'une autre directive.
Son nom reflète la manière dont la directive est appliquée : comme un attribut sur un host element.

## Tester la directive `Highlight` {#testing-the-highlight-directive}

La directive `Highlight` de l'application d'exemple définit la couleur d'arrière-plan d'un élément à partir d'une couleur liée par binding ou d'une couleur par défaut \(gris clair\).
Elle définit aussi une propriété personnalisée de l'élément \(`customProperty`\) à `true`, sans autre raison que de montrer qu'elle en est capable.

```ts
import {Directive, inject, input} from '@angular/core';

/**
 * Set backgroundColor for the attached element to highlight color
 * and set the element's customProperty attribute to true
 */
@Directive({
  selector: '[highlight]',
  host: {
    '[style.backgroundColor]': 'bgColor() || defaultColor',
  },
})
export class Highlight {
  readonly defaultColor = 'rgb(211, 211, 211)'; // lightgray

  readonly bgColor = input('', {alias: 'highlight'});
}
```

Elle est utilisée partout dans l'application, le plus simplement sans doute dans le composant `About` :

```ts
@Component({
  imports: [Twain, Highlight],
  template: `
    <h2 highlight="skyblue">About</h2>
    <h3>Quote of the day:</h3>
    <twain-quote />
  `,
})
export class About {}
```

Tester l'usage spécifique de la directive `Highlight` dans le composant `About` ne demande que les techniques présentées dans la section [« Tests de composants imbriqués »](guide/testing/components-scenarios#nested-component-tests) de [Scénarios de test de composants](guide/testing/components-scenarios).

```ts
let fixture: ComponentFixture<About>;

beforeEach(async () => {
  TestBed.configureTestingModule({
    providers: [TwainService, UserService],
    schemas: [CUSTOM_ELEMENTS_SCHEMA],
  });
  fixture = TestBed.createComponent(About);
  await fixture.whenStable();
});

it('should have skyblue <h2>', () => {
  const h2: HTMLElement = fixture.nativeElement.querySelector('h2');
  const bgColor = h2.style.backgroundColor;
  expect(bgColor).toBe('skyblue');
});
```

Cependant, tester un seul cas d'usage a peu de chances d'explorer l'ensemble des capacités d'une directive.
Trouver et tester tous les composants qui utilisent la directive est fastidieux, fragile, et n'a guère plus de chances d'offrir une couverture complète.

Les _tests de la classe seule_ peuvent être utiles, mais les directives d'attribute comme celle-ci ont tendance à manipuler le DOM.
Les tests unitaires isolés ne touchent pas au DOM et n'inspirent donc pas confiance dans l'efficacité de la directive.

Une meilleure solution consiste à créer un composant de test artificiel qui montre toutes les manières d'appliquer la directive.

```angular-ts
@Component({
  imports: [Highlight],
  template: `
    <h2 highlight="yellow">Something Yellow</h2>
    <h2 highlight>The Default (Gray)</h2>
    <h2>No Highlight</h2>
    <input #box [highlight]="box.value" value="cyan" />
  `,
})
class Test {}
```

<img alt="Spec de HighlightDirective en action" src="assets/images/guide/testing/highlight-directive-spec.png">

HELPFUL: Le cas de l'`<input>` lie la directive `Highlight` au nom d'une couleur saisi dans le champ de saisie.
La valeur initiale est le mot « cyan », qui doit être la couleur d'arrière-plan du champ de saisie.

Voici quelques tests de ce composant :

```ts
let fixture: ComponentFixture<Test>;
let des: DebugElement[]; // the three elements w/ the directive

beforeEach(async () => {
  fixture = TestBed.createComponent(Test);
  await fixture.whenStable();

  // all elements with an attached Highlight
  des = fixture.debugElement.queryAll(By.directive(Highlight));
});

// color tests
it('should have three highlighted elements', () => {
  expect(des.length).toBe(3);
});

it('should color 1st <h2> background "yellow"', () => {
  const bgColor = des[0].nativeElement.style.backgroundColor;
  expect(bgColor).toBe('yellow');
});

it('should color 2nd <h2> background w/ default color', () => {
  const dir = des[1].injector.get(Highlight);
  const bgColor = des[1].nativeElement.style.backgroundColor;
  expect(bgColor).toBe(dir.defaultColor);
});

it('should bind <input> background to value color', async () => {
  // easier to work with nativeElement
  const input = des[2].nativeElement as HTMLInputElement;
  expect(input.style.backgroundColor, 'initial backgroundColor').toBe('cyan');

  input.value = 'green';

  // Dispatch a DOM event so that Angular responds to the input value change.
  input.dispatchEvent(new Event('input'));
  await fixture.whenStable();

  expect(input.style.backgroundColor, 'changed backgroundColor').toBe('green');
});

it('bare <h2> should not have a backgroundColor', () => {
  // the h2 without the Highlight directive
  const bareH2 = fixture.debugElement.query(By.css('h2:not([highlight])'));

  expect(bareH2.styles.backgroundColor).toBeUndefined();
});
```

Quelques techniques méritent d'être soulignées :

- Le prédicat `By.directive` est un excellent moyen de récupérer les éléments qui portent cette directive _lorsque leur type d'élément est inconnu_
- La [pseudo-classe `:not`](https://developer.mozilla.org/docs/Web/CSS/:not) dans `By.css('h2:not([highlight])')` aide à trouver les éléments `<h2>` qui _ne portent pas_ la directive.
  `By.css('*:not([highlight])')` trouve _n'importe quel_ élément qui ne porte pas la directive.

- `DebugElement.styles` donne accès aux styles de l'élément même en l'absence d'un vrai navigateur, grâce à l'abstraction `DebugElement`.
  N'hésitez pas pour autant à exploiter `nativeElement` lorsque cela vous semble plus simple ou plus clair que l'abstraction.

- Angular ajoute une directive à l'injector de l'élément auquel elle est appliquée.
  Le test de la couleur par défaut utilise l'injector du deuxième `<h2>` pour obtenir son instance de `Highlight` et sa `defaultColor`.

- `DebugElement.properties` donne accès à la propriété personnalisée artificielle définie par la directive

## Tester une directive de manière isolée {#testing-a-directive-in-isolation}

Une directive peut être créée seule avec `TestBed.createDirective()`, qui renvoie une `DirectiveFixture`. Vous pouvez aussi l'afficher via le template d'un composant.
La directive `Highlight` peut être testée de cette manière, en utilisant un input d'un composant de test local pour contrôler la directive.

```ts
@Component({
  imports: [Highlight],
  template: `<p [highlight]="color()">{{ color() }}</p>`,
})
class Test {
  readonly color = input('');
}

describe('Highlight', () => {
  let fixture: ComponentFixture<Test>;

  beforeEach(async () => {
    fixture = TestBed.createComponent(Test);
    await fixture.whenStable();
  });

  it('should use the specified color once an input is provided', async () => {
    fixture.componentRef.setInput('color', 'blue');
    await fixture.whenStable();

    const p = fixture.nativeElement.querySelector('p');
    expect(p.style.backgroundColor).toBe('blue');
  });
});
```
