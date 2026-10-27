<!-- source: 85a6ad3d80cbfce40b65513a9b3138e533f415b1 -->
# Scénarios de test de composants

Ce guide explore des cas d'usage courants du test de composants.

## Binding de composant {#component-binding}

Dans l'application d'exemple, le composant `Banner` présente un titre statique dans le template HTML.

Après quelques modifications, le composant `Banner` présente un titre dynamique grâce à un binding sur la propriété `title` du composant, comme ceci.

```angular-ts {header: "banner.ts"}
import {Component, signal} from '@angular/core';

@Component({
  selector: 'app-banner',
  template: '<h1>{{ title() }}</h1>',
  styles: ['h1 { color: green; font-size: 350%}'],
})
export class Banner {
  title = signal('Test Tour of Heroes');
}
```

Aussi minimal soit-il, vous décidez d'ajouter un test pour confirmer que le composant affiche réellement le bon contenu là où vous l'attendez.

### Récupérer le `<h1>` {#query-for-the-h1}

Vous allez écrire une série de tests qui inspectent la valeur de l'élément `<h1>` qui encapsule le binding d'interpolation de la propriété _title_.

Vous modifiez le `beforeEach` pour trouver cet élément avec un `querySelector` HTML standard et l'assigner à la variable `h1`.

```ts {header: "banner.component.spec.ts"}
let component: Banner;
let fixture: ComponentFixture<Banner>;
let h1: HTMLElement;

beforeEach(() => {
  fixture = TestBed.createComponent(Banner);
  component = fixture.componentInstance; // Banner test instance
  h1 = fixture.nativeElement.querySelector('h1');
});
```

### `createComponent()` ne lie pas les données {#createcomponent-does-not-bind-data}

Pour votre premier test, vous aimeriez vérifier que l'écran affiche le `title` par défaut.
Votre premier réflexe est d'écrire un test qui inspecte immédiatement le `<h1>`, comme ceci :

```ts
it('should display original title', () => {
  expect(h1.textContent).toContain(component.title());
});
```

_Ce test échoue_ avec le message :

```shell {hideCopy}
expected '' to contain 'Test Tour of Heroes'.
```

Le binding a lieu quand Angular effectue la **détection de changements**.

En production, la détection de changements se déclenche automatiquement quand Angular crée un composant ou quand l'utilisateur saisit une touche, par exemple.

`TestBed.createComponent` ne déclenche pas la détection de changements de manière synchrone ; un fait confirmé dans le test révisé :

```ts
it('no title in the DOM after createComponent()', () => {
  expect(h1.textContent).toEqual('');
});
```

### `whenStable()` {#whenstable}

Vous pouvez demander au `TestBed` d'attendre l'exécution de la détection de changements avec `await fixture.whenStable()`.
Ce n'est qu'alors que le `<h1>` contient le titre attendu.

```ts
it('should display original title', async () => {
  await fixture.whenStable();
  expect(h1.textContent).toContain(component.title());
});
```

Le report de la détection de changements est intentionnel et utile.
Il donne au testeur l'occasion d'inspecter et de modifier l'état du composant _avant qu'Angular n'initie le data binding et n'appelle les [lifecycle hooks](guide/components/lifecycle)_.

Voici un autre test qui modifie la propriété `title` du composant _avant_ d'appeler `fixture.whenStable()`.

```ts
it('should display a different test title', async () => {
  component.title.set('Test Title');
  await fixture.whenStable();
  expect(h1.textContent).toContain('Test Title');
});
```

### Lier des signals aux inputs {#binding-signals-to-inputs}

Pour refléter les changements des inputs et écouter les outputs, vous pouvez lier dynamiquement des signals aux inputs et des fonctions aux outputs.

```ts
import {inputBinding, outputBinding} from '@angular/core';
import {TestBed} from '@angular/core/testing';

const fixture = TestBed.createComponent(ValueDisplay, {
  bindings: [
    inputBinding('value', value),
    outputBinding('valueChange', () =>  (/* ... */) ),
  ],
});
```

### Modifier la valeur d'un champ avec `dispatchEvent()` {#change-an-input-value-with-dispatchevent}

Pour simuler une saisie de l'utilisateur, trouvez l'élément input et définissez sa propriété `value`.

Mais il existe une étape intermédiaire essentielle.

Angular ne sait pas que vous avez défini la propriété `value` de l'élément input.
Il ne lira pas cette propriété tant que vous n'aurez pas déclenché l'événement `input` de l'élément en appelant `dispatchEvent()`.

L'exemple suivant, un composant qui utilise le `TitleCasePipe`, illustre la bonne séquence.

```ts
it('should convert hero name to Title Case', async () => {
  const hostElement = fixture.nativeElement;
  const nameInput: HTMLInputElement = hostElement.querySelector('input')!;
  const nameDisplay: HTMLElement = hostElement.querySelector('span')!;

  // simulate user entering a new name into the input box
  nameInput.value = 'quick BROWN  fOx';

  // Dispatch a DOM event so that Angular learns of input value change.
  nameInput.dispatchEvent(new Event('input'));

  // Wait for Angular to update the display binding through the title pipe
  await fixture.whenStable();

  expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
});
```

## Composant avec une dépendance {#component-with-a-dependency}

Les composants ont souvent des dépendances envers des services.

Le composant `Welcome` affiche un message de bienvenue à l'utilisateur connecté.
Il sait qui est l'utilisateur grâce à une propriété du service `UserAuthentication` injecté :

```angular-ts
import {Component, inject, OnInit, signal} from '@angular/core';
import {UserAuthentication} from '../model/user.authentication';

@Component({
  selector: 'app-welcome',
  template: '<h3 class="welcome"><i>{{ welcome() }}</i></h3>',
})
export class Welcome {
  private userAuth = inject(UserAuthentication);
  welcome = signal(
    this.userAuth.isLoggedIn() ? `Welcome, ${this.userAuth.user().name}` : 'Please log in.',
  );
}
```

Le composant `Welcome` contient une logique de décision qui interagit avec le service, une logique qui rend ce composant digne d'être testé.

### Fournir des doublures de test pour les services {#provide-service-test-doubles}

Un _composant sous test_ n'a pas besoin de recevoir les vrais services.

Injecter le vrai `UserAuthentication` pourrait être difficile.
Le vrai service pourrait demander à l'utilisateur ses identifiants de connexion et tenter de joindre un serveur d'authentification.
Ces comportements peuvent être difficiles à intercepter. Gardez à l'esprit que l'utilisation de doublures de test fait que le test se comporte différemment de la production : utilisez-les donc avec parcimonie.

### Récupérer les services injectés {#get-injected-services}

Les tests ont besoin d'accéder au `UserAuthentication` injecté dans le composant `Welcome`.

Angular dispose d'un système d'injection hiérarchique.
Il peut y avoir des injectors à plusieurs niveaux, depuis l'injector racine créé par le `TestBed` jusqu'au bas de l'arbre de composants.

La manière la plus sûre de récupérer le service injecté, celle qui **_fonctionne toujours_**,
consiste à **le récupérer depuis l'injector du _composant sous test_**.
L'injector du composant est une propriété du `DebugElement` de la fixture.

```ts
// UserAuthentication actually injected into the component
userAuth = fixture.debugElement.injector.get(UserAuthentication);
```

HELPFUL: Ce n'est _généralement_ pas nécessaire. Les services sont souvent fournis à la racine ou via les surcharges du TestBed, et peuvent être récupérés plus facilement avec `TestBed.inject()` (voir ci-dessous).

### `TestBed.inject()` {#testbedinject}

C'est plus facile à retenir et moins verbeux que de récupérer un service via le `DebugElement` de la fixture.

Dans cette suite de tests, le _seul_ provider de `UserAuthentication` est le module de test racine : vous pouvez donc appeler `TestBed.inject()` sans risque, comme suit :

```ts
userAuth = TestBed.inject(UserAuthentication);
```

HELPFUL: Pour un cas d'usage dans lequel `TestBed.inject()` ne fonctionne pas, consultez la section [_Surcharger les providers d'un composant_](#override-component-providers), qui explique quand et pourquoi vous devez plutôt récupérer le service depuis l'injector du composant.

### Configuration finale et tests {#final-setup-and-tests}

Voici le `beforeEach()` complet, avec `TestBed.inject()` :

```ts
let fixture: ComponentFixture<Welcome>;
let comp: Welcome;
let userAuth: UserAuthentication; // the TestBed injected service
let el: HTMLElement; // the DOM element with the welcome message

beforeEach(() => {
  fixture = TestBed.createComponent(Welcome);
  comp = fixture.componentInstance;

  // UserAuthentication from the root injector
  userAuth = TestBed.inject(UserAuthentication);

  //  get the "welcome" element by CSS selector (e.g., by class name)
  el = fixture.nativeElement.querySelector('.welcome');
});
```

Et voici quelques tests :

```ts
it('should welcome the user', async () => {
  await fixture.whenStable();
  const content = el.textContent;

  expect(content, '"Welcome ..."').toContain('Welcome');
  expect(content, 'expected name').toContain('Test User');
});

it('should welcome "Bubba"', async () => {
  userAuth.user.set({name: 'Bubba'}); // welcome message hasn't been shown yet
  await fixture.whenStable();

  expect(el.textContent).toContain('Bubba');
});

it('should request login if not logged in', async () => {
  userAuth.isLoggedIn.set(false); // welcome message hasn't been shown yet
  await fixture.whenStable();
  const content = el.textContent;

  expect(content, 'not welcomed').not.toContain('Welcome');
  expect(content, '"log in"').toMatch(/log in/i);
});
```

Le premier est un test de bon sens (sanity test) ; il confirme que `UserAuthentication` est appelé et fonctionne.

HELPFUL: Le 2e argument de `expect` \(par exemple, `'expected name'`\) est un libellé d'échec optionnel.
Si l'attente échoue, Vitest ajoute ce libellé au message d'échec de l'attente.
Dans une spec contenant plusieurs attentes, il peut aider à comprendre ce qui s'est mal passé et quelle attente a échoué.

Les tests restants confirment la logique du composant quand le service renvoie différentes valeurs.
Le deuxième test valide l'effet d'un changement du nom de l'utilisateur.
Le troisième test vérifie que le composant affiche le bon message quand aucun utilisateur n'est connecté.

## Composant avec un service asynchrone {#component-with-async-service}

Dans cet exemple, le template du composant `About` héberge un composant `Twain`.
Le composant `Twain` affiche des citations de Mark Twain.

```angular-html
<p class="twain">
  <i>{{ quote | async }}</i>
</p>
<button type="button" (click)="getQuote()">Next quote</button>
@if (errorMessage()) {
  <p class="error">{{ errorMessage() }}</p>
}
```

HELPFUL: La valeur de la propriété `quote` du composant passe par un `AsyncPipe`.
Cela signifie que la propriété renvoie soit une `Promise`, soit un `Observable`.

Dans cet exemple, la méthode `TwainQuotes.getQuote()` vous indique que la propriété `quote` renvoie un `Observable`.

```ts
getQuote() {
  this.errorMessage.set('');
  this.quote = this.twainQuotes.getQuote().pipe(
    startWith('...'),
    catchError((err: any) => {
      this.errorMessage.set(err.message || err.toString());
      return of('...'); // reset message to placeholder
    }),
  );
}
```

Le composant `Twain` récupère des citations depuis un `TwainQuotes` injecté.
Le composant démarre l'`Observable` renvoyé avec une valeur de placeholder \(`'...'`\), avant que le service ne puisse renvoyer sa première citation.

Le `catchError` intercepte les erreurs du service, prépare un message d'erreur et renvoie la valeur de placeholder sur le canal de succès.

Ce sont autant de fonctionnalités que vous voudrez tester.

### Tester en mockant les requêtes HTTP avec `HttpTestingController`. {#testing-by-mocking-http-requests-with-the-httptestingcontroller}

Lorsque vous testez un composant, seule l'API publique du service devrait compter.
En général, les tests eux-mêmes ne devraient pas effectuer d'appels vers des serveurs distants.
Ils devraient émuler ces appels.

Si votre service asynchrone s'appuie sur `HttpClient` pour charger des données distantes, il est recommandé de renvoyer des mocks de réponses au niveau HTTP avec `HttpTestingController`.

Pour plus de détails sur le mock de `HttpBackend`, consultez le [guide dédié](guide/http/testing).

### Tester en fournissant une implémentation stub d'un service. {#testing-by-providing-a-stubbed-implementation-of-a-service}

Quand il n'est pas possible de mocker les requêtes asynchrones au niveau HTTP, une alternative consiste à tirer parti des spies.

La configuration de ce fichier `app/twain/twain-quotes.spec.ts` montre une façon de procéder :

```ts {header: "twain.spec.ts"}
class TwainQuotesStub implements TwainQuotes {
  private testQuote = 'Test Quote';

  getQuote() {
    return of(this.testQuote);
  }

  // ... Implement everything to conform to the API
}

beforeEach(async () => {
  TestBed.configureTestingModule({
    providers: [{provide: TwainQuotes, useClass: TwainQuotesStub}],
  });

  fixture = TestBed.createComponent(Twain);
  component = fixture.componentInstance;
  await fixture.whenStable();
  quoteEl = fixture.nativeElement.querySelector('.twain');
});
```

Concentrez-vous sur la façon dont l'implémentation stub remplace l'implémentation d'origine.

```ts
TestBed.configureTestingModule({
  providers: [{provide: TwainQuotes, useClass: TwainQuotesStub}],
});
```

Le stub est conçu de telle sorte que tout composant ou service qui l'injecte recevra l'implémentation stub.
Cela signifie que tout appel à `getQuote` reçoit un observable contenant une citation de test.

Contrairement à la vraie méthode `getQuote()`, ce spy contourne le serveur et renvoie un observable synchrone dont la valeur est disponible immédiatement.

### Test asynchrone avec les fake timers de Vitest {#async-test-with-a-vitest-fake-timers}

Pour mocker des fonctions asynchrones comme `setTimeout` ou des `Promise`, vous pouvez tirer parti des fake timers de Vitest pour contrôler le moment où elles se déclenchent.

```ts
it('should display error when TwainQuotes service fails', async () => {
  class TwainQuotesStub implements TwainQuotes {
    getQuote() {
      return defer(() => {
        return new Promise<string>((_, reject) => {
          setTimeout(() => reject('TwainService test failure'));
        });
      });
    }

    // ... Implement everything to conform to the API
  }

  TestBed.configureTestingModule({
    providers: [{provide: TwainQuotes, useClass: TwainQuotesStub}],
  });

  vi.useFakeTimers(); // setting up the fake timers
  const fixture = TestBed.createComponent(TwainComponent);

  // rendering isn't async, we need to flush
  await vi.runAllTimersAsync();

  await expect(fixture.nativeElement.querySelector('.error')!.textContent).toMatch(/test failure/);
  expect(fixture.nativeElement.querySelector('.twain')!.textContent).toBe('...');

  vi.useRealTimers(); // resets to regular async execution
});
```

### Autres tests asynchrones {#more-async-tests}

Comme le service stub renvoie des observables asynchrones, la plupart de vos tests devront également être asynchrones.

Voici un test qui illustre le flux de données auquel vous vous attendriez en conditions réelles.

```ts
class MockTwainQuotes implements TwainQuotes {
  private subject = new Subject<string>();

  getQuote() {
    return this.subject.asObservable();
  }

  emit(val: string) {
    this.subject.next(val);
  }
}

it('should show quote after getQuote (success)', async () => {
  vi.useFakeTimers();

  TestBed.configureTestingModule({
    providers: [{provide: TwainQuotes, useClass: MockTwainQuotes}],
  });

  const fixture = TestBed.createComponent(TwainComponent);
  const twainQuotes = TestBed.inject(TwainQuotes) as MockTwainQuotes;
  await vi.runAllTimersAsync(); // render before the quote is received

  const quoteEl = fixture.nativeElement.querySelector('.twain');
  expect(quoteEl.textContent).toBe('...');

  twainQuotes.emit('Twain Quote'); // emits the quote
  await vi.runAllTimersAsync(); // render with the quote received

  expect(quoteEl.textContent).toBe('Twain Quote');
  expect(fixture.nativeElement.querySelector('.error')).toBeNull();

  vi.useRealTimers();
});
```

Remarquez que l'élément de citation affiche la valeur de placeholder \(`'...'`\) lors du premier rendu.
La première citation n'est pas encore arrivée.

Vous pouvez ensuite vérifier que l'élément de citation affiche le texte attendu.

### Tests asynchrones avec `zone.js` et `fakeAsync` {#async-tests-with-zonejs-and-fakeasync}

La fonction utilitaire `fakeAsync` est une autre horloge simulée qui s'appuie sur le patch des API asynchrones par `zone.js`. Elle était couramment utilisée pour les tests dans les applications basées sur `zone.js`. L'utilisation de `fakeAsync` n'est plus recommandée.

TIP: Privilégiez les stratégies natives de test asynchrone ou d'autres fake timers (aussi appelés horloges simulées, ou mock clocks), comme ceux de Vitest ou de Jasmine.

IMPORTANT: `fakeAsync` nécessite `zone.js`. Pour l'utiliser avec le test runner Vitest, ajoutez le polyfill `zone.js/plugins/vitest-patch` comme décrit dans [le guide de migration vers Vitest](guide/testing/migrating-to-vitest#zonejs-vitest-patch).

## Composant avec des inputs et des outputs {#component-with-inputs-and-outputs}

Un composant avec des inputs et des outputs apparaît généralement dans le template de la vue d'un composant hôte.
Le host utilise un property binding pour définir la propriété d'input et un event binding pour écouter les événements émis par la propriété d'output.

L'objectif du test est de vérifier que ces bindings fonctionnent comme prévu.
Les tests doivent définir les valeurs des inputs et écouter les événements des outputs.

Le composant `DashboardHero` est un petit exemple de composant dans ce rôle.
Il affiche un héros individuel fourni par le composant `Dashboard`.
Cliquer sur ce héros indique au composant `Dashboard` que l'utilisateur a sélectionné le héros.

Le composant `DashboardHero` est intégré dans le template du composant `Dashboard` comme ceci :

```angular-html
@for (hero of heroes; track hero) {
  <dashboard-hero class="col-1-4" [hero]="hero" (selected)="gotoDetail($event)" />
}
```

Le composant `DashboardHero` apparaît dans un bloc `@for`, qui définit la propriété d'input `hero` de chaque composant avec la valeur de l'itération et écoute l'événement `selected` du composant.

Voici la définition complète du composant :

```angular-ts
@Component({
  selector: 'dashboard-hero',
  imports: [UpperCasePipe],
  template: `
    <button type="button" (click)="click()" class="hero">
      {{ hero().name | uppercase }}
    </button>
  `,
})
export class DashboardHero {
  readonly hero = input.required<Hero>();
  readonly selected = output<Hero>();

  click() {
    this.selected.emit(this.hero());
  }
}
```

Même si tester un composant aussi simple a peu de valeur intrinsèque, il est utile de savoir comment faire.
Utilisez l'une de ces approches :

- Le tester tel qu'il est utilisé par le composant `Dashboard`
- Le tester en tant que composant standalone
- Le tester tel qu'il est utilisé par un substitut du composant `Dashboard`

L'objectif immédiat est de tester le composant `DashboardHero`, et non le composant `Dashboard` : essayez donc les deuxième et troisième options.

### Tester le composant `DashboardHero` de manière isolée {#test-the-dashboardhero-component-standalone}

Voici l'essentiel de la configuration du fichier de spec.

```ts
let fixture: ComponentFixture<DashboardHero>;
let comp: DashboardHero;
let heroDe: DebugElement;
let heroEl: HTMLElement;
let expectedHero: Hero;

beforeEach(async () => {
  fixture = TestBed.createComponent(DashboardHero);
  comp = fixture.componentInstance;

  // find the hero's DebugElement and element
  heroDe = fixture.debugElement.query(By.css('.hero'));
  heroEl = heroDe.nativeElement;

  // mock the hero supplied by the parent component
  expectedHero = {id: 42, name: 'Test Name'};

  // simulate the parent setting the input property with that hero
  fixture.componentRef.setInput('hero', expectedHero);

  // wait for initial data binding
  await fixture.whenStable();
});
```

Remarquez comment le code de configuration assigne un héros de test \(`expectedHero`\) à la propriété `hero` du composant, en émulant la façon dont le `Dashboard` la définirait avec le property binding dans sa boucle de répétition.

Le test suivant vérifie que le nom du héros est propagé au template grâce à un binding.

```ts
it('should display hero name in uppercase', () => {
  const expectedPipedName = expectedHero.name.toUpperCase();
  expect(heroEl.textContent).toContain(expectedPipedName);
});
```

Comme le template fait passer le nom du héros par le `UpperCasePipe` d'Angular, le test doit comparer la valeur de l'élément avec le nom en majuscules.

### Cliquer {#clicking}

Cliquer sur le héros doit émettre un événement `selected` que le composant hôte \(vraisemblablement `Dashboard`\) peut écouter :

```ts
it('should raise selected event when clicked (triggerEventHandler)', () => {
  let selectedHero: Hero | undefined;
  comp.selected.subscribe((hero: Hero) => (selectedHero = hero));

  heroDe.triggerEventHandler('click');
  expect(selectedHero).toBe(expectedHero);
});
```

La propriété `selected` du composant est un `OutputEmitterRef`, dont la méthode `subscribe` notifie les abonnés de manière synchrone.
Le test s'y abonne _explicitement_, tout comme le composant hôte le fait _implicitement_.

Si le composant se comporte comme prévu, cliquer sur l'élément du héros doit indiquer à la propriété `selected` du composant d'émettre l'objet `hero`.

Le test détecte cet événement grâce à son abonnement à `selected`.

### `triggerEventHandler` {#triggereventhandler}

Le `heroDe` du test précédent est un `DebugElement` qui représente le `<button>` du héros.

Il possède des propriétés et méthodes Angular qui abstraient l'interaction avec l'élément natif.
Ce test appelle `DebugElement.triggerEventHandler` avec le nom d'événement « click ».
L'event binding « click » réagit en appelant `DashboardHero.click()`.

`DebugElement.triggerEventHandler` d'Angular peut déclencher _n'importe quel événement lié par binding_ à partir de son _nom d'événement_.
Le second paramètre est l'objet événement transmis au handler.

Le test a déclenché un événement « click ».

```ts
heroDe.triggerEventHandler('click');
```

Dans ce cas, le test suppose à juste titre que l'event handler exécuté, la méthode `click()` du composant, ne se soucie pas de l'objet événement.

HELPFUL: D'autres handlers sont moins tolérants.
Par exemple, la directive `RouterLink` attend un objet avec une propriété `button` qui identifie quel bouton de la souris a éventuellement été pressé lors du clic.
La directive `RouterLink` lève une erreur si l'objet événement est absent.

### Cliquer sur l'élément {#click-the-element}

L'alternative de test suivante appelle la propre méthode `click()` de l'élément natif, ce qui convient parfaitement pour _ce composant_.

```ts
it('should raise selected event when clicked (element.click)', () => {
  let selectedHero: Hero | undefined;
  comp.selected.subscribe((hero: Hero) => (selectedHero = hero));

  heroEl.click();
  expect(selectedHero).toBe(expectedHero);
});
```

### Fonction utilitaire `click()` {#click-helper}

Cliquer sur un bouton, un lien ou n'importe quel élément HTML est une tâche de test courante.

Rendez-la cohérente et simple en encapsulant le processus de _déclenchement du clic_ dans une fonction utilitaire comme la fonction `click()` suivante :

```ts
/** Button events to pass to `DebugElement.triggerEventHandler` for RouterLink event handler */
export const ButtonClickEvents = {
  left: {button: 0},
  right: {button: 2},
};

/** Simulate element click. Defaults to mouse left-button click event. */
export function click(
  el: DebugElement | HTMLElement,
  eventObj: any = ButtonClickEvents.left,
): void {
  if (el instanceof HTMLElement) {
    el.click();
  } else {
    el.triggerEventHandler('click', eventObj);
  }
}
```

Le premier paramètre est l'_élément sur lequel cliquer_.
Si vous le souhaitez, passez un objet événement personnalisé comme second paramètre.
La valeur par défaut est un [objet d'événement souris du bouton gauche](https://developer.mozilla.org/docs/Web/API/MouseEvent/button) partiel, accepté par de nombreux handlers, dont la directive `RouterLink`.

IMPORTANT: La fonction utilitaire `click()` ne fait **pas** partie des utilitaires de test d'Angular.
C'est une fonction définie dans _le code d'exemple de ce guide_.
Tous les tests d'exemple l'utilisent.
Si elle vous plaît, ajoutez-la à votre propre collection de fonctions utilitaires.

Voici le test précédent, réécrit avec la fonction utilitaire click.

```ts
it('should raise selected event when clicked (click helper with DebugElement)', () => {
  let selectedHero: Hero | undefined;
  comp.selected.subscribe((hero: Hero) => (selectedHero = hero));

  click(heroDe); // click helper with DebugElement

  expect(selectedHero).toBe(expectedHero);
});
```

## Composant dans un test host {#component-inside-a-test-host}

Les tests précédents jouaient eux-mêmes le rôle du composant hôte `Dashboard`.
Mais le composant `DashboardHero` fonctionne-t-il correctement lorsqu'il est correctement lié par binding à un composant hôte ?

```angular-ts
@Component({
  imports: [DashboardHero],
  template: ` <dashboard-hero [hero]="hero" (selected)="onSelected($event)" />`,
})
class TestHost {
  hero: Hero = {id: 42, name: 'Test Name'};
  selectedHero: Hero | undefined;

  onSelected(hero: Hero) {
    this.selectedHero = hero;
  }
}
```

Le test host définit la propriété d'input `hero` du composant avec son héros de test.
Il lie l'événement `selected` du composant à son handler `onSelected`, qui enregistre le héros émis dans sa propriété `selectedHero`.

Plus tard, les tests pourront vérifier `selectedHero` pour s'assurer que l'événement `DashboardHero.selected` a émis le héros attendu.

La configuration des tests `test-host` est similaire à celle des tests isolés :

```ts
beforeEach(async () => {
  // create TestHost instead of DashboardHero
  fixture = TestBed.createComponent(TestHost);
  testHost = fixture.componentInstance;
  heroEl = fixture.nativeElement.querySelector('.hero');

  await fixture.whenStable();
});
```

Cette configuration du module de test présente deux différences importantes :

- Elle _crée_ le composant `TestHost` au lieu du `DashboardHero`
- Le composant `TestHost` définit `DashboardHero.hero` avec un binding

`createComponent` renvoie une `fixture` qui contient une instance de `TestHost` au lieu d'une instance de `DashboardHero`.

Créer le `TestHost` a pour effet de bord de créer un `DashboardHero`, car ce dernier apparaît dans le template du premier.
La query de l'élément du héros \(`heroEl`\) le trouve toujours dans le DOM de test, bien qu'à une profondeur plus grande qu'auparavant dans l'arbre des éléments.

Les tests eux-mêmes sont presque identiques à la version isolée :

```ts
it('should display hero name', () => {
  const expectedPipedName = testHost.hero.name.toUpperCase();
  expect(heroEl.textContent).toContain(expectedPipedName);
});

it('should raise selected event when clicked', () => {
  click(heroEl);
  // selected hero should be the same data bound hero
  expect(testHost.selectedHero).toBe(testHost.hero);
});
```

Seul le test de l'événement selected diffère.
Il confirme que le héros `DashboardHero` sélectionné remonte bien jusqu'au composant hôte via l'event binding.

## Composant de routing {#routing-component}

Un _composant de routing_ est un composant qui demande au `Router` de naviguer vers un autre composant.
Le composant `Dashboard` est un _composant de routing_, car l'utilisateur peut naviguer vers le composant `HeroDetail` en cliquant sur l'un des _boutons de héros_ du tableau de bord.

Angular fournit des utilitaires de test pour réduire le code répétitif et tester plus efficacement le code qui dépend de `HttpClient`. La fonction `provideRouter` peut aussi être utilisée directement dans le module de test.

```ts
beforeEach(async () => {
  TestBed.configureTestingModule({
    providers: [
      provideRouter([{path: '**', component: Dashboard}]),
      provideHttpClientTesting(),
      HeroService,
    ],
  });
  harness = await RouterTestingHarness.create();
  comp = await harness.navigateByUrl('/', Dashboard);
  TestBed.inject(HttpTestingController).expectOne('api/heroes').flush(getTestHeroes());
});
```

Le test suivant clique sur le héros affiché et confirme que la navigation mène à l'URL attendue.

```ts
it('should tell navigate when hero clicked', async () => {
  // get first <dashboard-hero> DebugElement
  const heroDe = harness.routeDebugElement!.query(By.css('dashboard-hero'));
  heroDe.triggerEventHandler('selected', comp.heroes[0]);

  // expecting to navigate to id of the component's first hero
  const id = comp.heroes[0].id;
  expect(TestBed.inject(Router).url, 'should nav to HeroDetail for first hero').toEqual(
    `/heroes/${id}`,
  );
});
```

## Composants routés {#routed-components}

Un _composant routé_ est la destination d'une navigation du `Router`.
Il peut être plus délicat à tester, en particulier quand la route vers le composant _inclut des paramètres_.
`HeroDetail` est un _composant routé_ qui est la destination d'une telle route.

Quand un utilisateur clique sur un héros du _Dashboard_, le `Dashboard` demande au `Router` de naviguer vers `heroes/:id`.
`:id` est un paramètre de route dont la valeur est l'`id` du héros à modifier.

Le `Router` fait correspondre cette URL à une route vers `HeroDetail`.
Il crée un objet `ActivatedRoute` avec les informations de routing et l'injecte dans une nouvelle instance de `HeroDetail`.

Voici les services injectés dans `HeroDetail` :

```ts
private heroDetailService = inject(HeroDetailService);
private route = inject(ActivatedRoute);
private router = inject(Router);
```

Le composant `HeroDetail` a besoin du paramètre `id` pour récupérer le héros correspondant avec `HeroDetailService`.
Le composant doit obtenir l'`id` depuis la propriété `ActivatedRoute.paramMap`, qui est un `Observable`.

Il ne peut pas simplement référencer la propriété `id` de `ActivatedRoute.paramMap`.
Le composant doit _s'abonner_ à l'observable `ActivatedRoute.paramMap` et être prêt à ce que l'`id` change au cours de sa vie.

```ts
constructor() {
  // get hero when `id` param changes
  this.route.paramMap.subscribe((pmap) => this.getHero(pmap.get('id')));
}
```

Les tests peuvent explorer la façon dont `HeroDetail` réagit à différentes valeurs du paramètre `id` en naviguant vers différentes routes.

## Tests de composants imbriqués {#nested-component-tests}

Les templates de composants contiennent souvent des composants imbriqués, dont les templates peuvent eux-mêmes contenir d'autres composants.

L'arbre de composants peut être très profond, et parfois les composants imbriqués ne jouent aucun rôle dans le test du composant situé en haut de l'arbre.

Le composant `App`, par exemple, affiche une barre de navigation avec des liens et leurs directives `RouterLink`.

```angular-html
<app-banner />
<app-welcome />

<nav>
  <a routerLink="/dashboard">Dashboard</a>
  <a routerLink="/heroes">Heroes</a>
  <a routerLink="/about">About</a>
</nav>

<router-outlet />
```

Pour valider les liens mais pas la navigation, vous n'avez pas besoin que le `Router` navigue, ni du `<router-outlet>` pour indiquer où le `Router` insère les _composants routés_.

Les composants `Banner` et `Welcome` \(indiqués par `<app-banner>` et `<app-welcome>`\) ne sont pas pertinents non plus.

Pourtant, tout test qui crée le composant `App` dans le DOM crée aussi des instances de ces trois composants et, si vous laissez faire, vous devrez configurer le `TestBed` pour les créer.

Si vous omettez de les déclarer, le compilateur Angular ne reconnaîtra pas les balises `<app-banner>`, `<app-welcome>` et `<router-outlet>` du template de `App` et lèvera une erreur.

Si vous déclarez les vrais composants, vous devrez aussi déclarer _leurs_ composants imbriqués et fournir _tous_ les services injectés dans _n'importe quel_ composant de l'arbre.

Cette section décrit deux techniques pour réduire la configuration au minimum.
Utilisez-les, seules ou combinées, pour rester concentré sur le test du composant principal.

### Remplacer les composants inutiles par des stubs {#stubbing-unneeded-components}

Dans la première technique, vous créez et déclarez des versions stub des composants et de la directive qui jouent peu ou pas de rôle dans les tests.

```ts
@Component({selector: 'app-banner', template: ''})
class BannerStub {}

@Component({selector: 'router-outlet', template: ''})
class RouterOutletStub {}

@Component({selector: 'app-welcome', template: ''})
class WelcomeStub {}
```

Les selectors des stubs correspondent aux selectors des vrais composants correspondants.
Mais leurs templates et leurs classes sont vides.

Déclarez-les ensuite en surchargeant les `imports` de votre composant avec `TestBed.overrideComponent`.

```ts
let comp: App;
let fixture: ComponentFixture<App>;

beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [provideRouter([]), UserAuthentication],
  }).overrideComponent(App, {
    set: {
      imports: [RouterLink, BannerStub, RouterOutletStub, WelcomeStub],
    },
  });

  fixture = TestBed.createComponent(App);
  comp = fixture.componentInstance;
});
```

HELPFUL: Dans cet exemple, la clé `set` remplace tous les imports existants de votre composant : veillez à importer toutes les dépendances, et pas seulement les stubs. Vous pouvez aussi utiliser les clés `remove`/`add` pour retirer et ajouter des imports de manière sélective.

### `NO_ERRORS_SCHEMA` {#noerrorsschema}

Dans la seconde approche, ajoutez `NO_ERRORS_SCHEMA` aux surcharges de métadonnées de votre composant.

```ts
beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [provideRouter([]), UserAuthentication],
  }).overrideComponent(App, {
    set: {
      imports: [], // resets all imports
      schemas: [NO_ERRORS_SCHEMA],
    },
  });
});
```

`NO_ERRORS_SCHEMA` indique au compilateur Angular d'ignorer les éléments et attributs non reconnus.

Le compilateur reconnaît l'élément `<app-root>` et l'attribut `routerLink`, car vous avez déclaré un composant `App` et un `RouterLink` correspondants dans la configuration du `TestBed`.

Mais le compilateur ne lèvera pas d'erreur lorsqu'il rencontrera `<app-banner>`, `<app-welcome>` ou `<router-outlet>`.
Il les rend simplement comme des balises vides, et le navigateur les ignore.

Vous n'avez plus besoin des composants stub.

### Utiliser les deux techniques ensemble {#use-both-techniques-together}

Ce sont des techniques de _test superficiel de composants_ (Shallow Component Testing), ainsi nommées parce qu'elles réduisent la surface visuelle du composant aux seuls éléments de son template qui comptent pour les tests.

L'approche `NO_ERRORS_SCHEMA` est la plus simple des deux, mais n'en abusez pas.

`NO_ERRORS_SCHEMA` empêche aussi le compilateur de vous signaler les composants et attributs manquants que vous avez omis par inadvertance ou mal orthographiés.
Vous pourriez perdre des heures à traquer des bugs fantômes que le compilateur aurait détectés en un instant.

L'approche par _composants stub_ présente un autre avantage.
Même si les stubs de _cet_ exemple étaient vides, vous pourriez leur donner des templates et des classes simplifiés si vos tests doivent interagir avec eux d'une manière ou d'une autre.

En pratique, vous combinerez les deux techniques dans la même configuration, comme dans cet exemple.

```ts
beforeEach(() => {
  TestBed.configureTestingModule({
    providers: [provideRouter([]), UserAuthentication],
  }).overrideComponent(App, {
    remove: {imports: [RouterOutlet, Welcome]},
    set: {schemas: [NO_ERRORS_SCHEMA]},
  });
});
```

Le compilateur Angular crée le `BannerStub` pour l'élément `<app-banner>` et applique le `RouterLink` aux liens possédant l'attribut `routerLink`, mais il ignore les balises `<app-welcome>` et `<router-outlet>`.

### `By.directive` et directives injectées {#bydirective-and-injected-directives}

Un peu plus de configuration déclenche le data binding initial et récupère des références vers les liens de navigation :

```ts
beforeEach(async () => {
  await fixture.whenStable();

  // find DebugElements with an attached RouterLinkStubDirective
  linkDes = fixture.debugElement.queryAll(By.directive(RouterLink));

  // get attached link directive instances
  // using each DebugElement's injector
  routerLinks = linkDes.map((de) => de.injector.get(RouterLink));
});
```

Trois points méritent une attention particulière :

- Localisez les éléments de lien auxquels une directive est attachée avec `By.directive`
- La query renvoie des wrappers `DebugElement` autour des éléments correspondants
- Chaque `DebugElement` expose un injector de dépendances contenant l'instance spécifique de la directive attachée à cet élément

Les liens du composant `App` à valider sont les suivants :

```angular-html
<nav>
  <a routerLink="/dashboard">Dashboard</a>
  <a routerLink="/heroes">Heroes</a>
  <a routerLink="/about">About</a>
</nav>
```

Voici quelques tests qui confirment que ces liens sont reliés aux directives `routerLink` comme prévu :

```ts
it('can get RouterLinks from template', () => {
  expect(routerLinks.length, 'should have 3 routerLinks').toBe(3);
  expect(routerLinks[0].href).toBe('/dashboard');
  expect(routerLinks[1].href).toBe('/heroes');
  expect(routerLinks[2].href).toBe('/about');
});

it('can click Heroes link in template', async () => {
  const heroesLinkDe = linkDes[1]; // heroes link DebugElement

  TestBed.inject(Router).resetConfig([{path: '**', children: []}]);
  heroesLinkDe.triggerEventHandler('click', {button: 0});

  await fixture.whenStable();

  expect(TestBed.inject(Router).url).toBe('/heroes');
});
```

## Utiliser un objet `page` {#use-a-page-object}

Le composant `HeroDetail` est une vue simple avec un titre, deux champs de héros et deux boutons.

Mais même ce formulaire simple présente une bonne dose de complexité dans son template.

```angular-html
@if (hero) {
  <div>
    <h2>
      <span>{{ hero.name | titlecase }}</span> Details
    </h2>
    <div><span>id: </span>{{ hero.id }}</div>
    <div>
      <label for="name">name: </label>
      <input id="name" [(ngModel)]="hero.name" placeholder="name" />
    </div>
    <button type="button" (click)="save()">Save</button>
    <button type="button" (click)="cancel()">Cancel</button>
  </div>
}
```

Les tests qui exercent le composant ont besoin…

- D'attendre l'arrivée d'un héros avant que les éléments n'apparaissent dans le DOM
- D'une référence vers le texte du titre
- D'une référence vers le champ de saisie du nom, pour l'inspecter et le définir
- De références vers les deux boutons, pour pouvoir cliquer dessus

Même un petit formulaire comme celui-ci peut produire un fouillis de configuration conditionnelle alambiquée et de sélections d'éléments CSS.

Maîtrisez cette complexité avec une classe `Page` qui gère l'accès aux propriétés du composant et encapsule la logique qui les définit.

Voici une telle classe `Page` pour `hero-detail.component.spec.ts`

```ts
class Page {
  // getter properties wait to query the DOM until called.
  get buttons() {
    return this.queryAll<HTMLButtonElement>('button');
  }
  get saveBtn() {
    return this.buttons[0];
  }
  get cancelBtn() {
    return this.buttons[1];
  }
  get nameDisplay() {
    return this.query<HTMLElement>('span');
  }
  get nameInput() {
    return this.query<HTMLInputElement>('input');
  }

  //// query helpers ////
  private query<T>(selector: string): T {
    return harness.routeNativeElement!.querySelector(selector)! as T;
  }

  private queryAll<T>(selector: string): T[] {
    return harness.routeNativeElement!.querySelectorAll(selector) as any as T[];
  }
}
```

Désormais, les points d'accroche importants pour manipuler et inspecter le composant sont bien organisés et accessibles depuis une instance de `Page`.

Une méthode `createComponent` crée un objet `page` et complète les éléments manquants une fois le `hero` arrivé.

```ts
async function createComponent(id: number) {
  harness = await RouterTestingHarness.create();
  component = await harness.navigateByUrl(`/heroes/${id}`, HeroDetail);
  page = new Page();

  const request = TestBed.inject(HttpTestingController).expectOne(`api/heroes/?id=${id}`);
  const hero = getTestHeroes().find((h) => h.id === Number(id));
  request.flush(hero ? [hero] : []);
  await harness.fixture.whenStable();
}
```

Voici quelques autres tests du composant `HeroDetail` pour appuyer ce point.

```ts
it("should display that hero's name", () => {
  expect(page.nameDisplay.textContent).toBe(expectedHero.name);
});

it('should navigate when click cancel', () => {
  click(page.cancelBtn);
  expect(TestBed.inject(Router).url).toEqual(`/heroes/${expectedHero.id}`);
});

it('should save when click save but not navigate immediately', () => {
  click(page.saveBtn);
  expect(TestBed.inject(HttpTestingController).expectOne({method: 'PUT', url: 'api/heroes'}));
  expect(TestBed.inject(Router).url).toEqual('/heroes/41');
});

it('should navigate when click save and save resolves', async () => {
  click(page.saveBtn);
  await harness.fixture.whenStable();
  expect(TestBed.inject(Router).url).toEqual('/heroes/41');
});

it('should convert hero name to Title Case', async () => {
  // get the name's input and display elements from the DOM
  const hostElement: HTMLElement = harness.routeNativeElement!;
  const nameInput: HTMLInputElement = hostElement.querySelector('input')!;
  const nameDisplay: HTMLElement = hostElement.querySelector('span')!;

  // simulate user entering a new name into the input box
  nameInput.value = 'quick BROWN  fOx';

  // Dispatch a DOM event so that Angular learns of input value change.
  nameInput.dispatchEvent(new Event('input'));

  // Wait for Angular to update the display binding through the title pipe
  await harness.fixture.whenStable();

  expect(nameDisplay.textContent).toBe('Quick Brown  Fox');
});
```

## Surcharger les providers d'un composant {#override-component-providers}

`HeroDetail` fournit son propre `HeroDetailService`.

```ts
@Component({
  /* ... */
  providers: [HeroDetailService],
})
export class HeroDetail {
  private heroDetailService = inject(HeroDetailService);
  private route = inject(ActivatedRoute);
  private router = inject(Router);
}
```

Il n'est pas possible de remplacer par un stub le `HeroDetailService` du composant dans les `providers` de `TestBed.configureTestingModule`.
Ce sont des providers pour le _module de test_, pas pour le composant.
Ils préparent l'injector de dépendances au _niveau de la fixture_.

Angular crée le composant avec son _propre_ injector, qui est un _enfant_ de l'injector de la fixture.
Il enregistre les providers du composant \(ici, le `HeroDetailService`\) auprès de l'injector enfant.

Un test ne peut pas atteindre les services de l'injector enfant depuis l'injector de la fixture.
Et `TestBed.configureTestingModule` ne peut pas non plus les configurer.

Angular a donc créé de nouvelles instances du vrai `HeroDetailService` depuis le début !

HELPFUL: Ces tests pourraient échouer ou dépasser le délai d'attente si le `HeroDetailService` effectuait ses propres appels XHR vers un serveur distant.
Il se peut qu'il n'y ait aucun serveur distant à appeler.

Heureusement, le `HeroDetailService` délègue la responsabilité de l'accès aux données distantes à un `HeroService` injecté.

```ts
@Service()
export class HeroDetailService {
  private heroService = inject(HeroService);
}
```

La configuration de test précédente remplace le vrai `HeroService` par un `TestHeroService` qui intercepte les requêtes vers le serveur et simule leurs réponses.

Et si vous n'avez pas cette chance ?
Et s'il est difficile de simuler le `HeroService` ?
Et si `HeroDetailService` effectue ses propres requêtes vers le serveur ?

La méthode `TestBed.overrideComponent` peut remplacer les `providers` du composant par des _doublures de test_ faciles à gérer, comme le montre la variante de configuration suivante :

```ts
beforeEach(async () => {
  await TestBed.configureTestingModule({
    providers: [
      provideRouter([
        {path: 'heroes', component: HeroList},
        {path: 'heroes/:id', component: HeroDetail},
      ]),
      // HeroDetailService at this level is IRRELEVANT!
      {provide: HeroDetailService, useValue: {}},
    ],
  }).overrideComponent(HeroDetail, {
    set: {providers: [{provide: HeroDetailService, useClass: HeroDetailServiceSpy}]},
  });
});
```

Remarquez que `TestBed.configureTestingModule` ne fournit plus de faux `HeroService`, car il n'est [pas nécessaire](#provide-a-spy-stub-herodetailservicespy).

### La méthode `overrideComponent` {#the-overridecomponent-method}

Concentrez-vous sur la méthode `overrideComponent`.

```ts
.overrideComponent(HeroDetail, {
  set: {providers: [{provide: HeroDetailService, useClass: HeroDetailServiceSpy}]},
});
```

Elle prend deux arguments : le type du composant à surcharger \(`HeroDetail`\) et un objet de métadonnées de surcharge.
L'[objet de métadonnées de surcharge](/guide/testing/utility-apis#testbed-class-summary) est un générique défini comme suit :

```ts
type MetadataOverride<T> = {
  add?: Partial<T>;
  remove?: Partial<T>;
  set?: Partial<T>;
};
```

Un objet de surcharge de métadonnées peut soit ajouter et retirer des éléments dans les propriétés de métadonnées, soit réinitialiser complètement ces propriétés.
Cet exemple réinitialise les métadonnées `providers` du composant.

Le paramètre de type, `T`, correspond au type de métadonnées que vous passeriez au décorateur `@Component` :

```ts
selector?: string;
template?: string;
templateUrl?: string;
providers?: any[];
…
```

### Fournir un _spy stub_ (`HeroDetailServiceSpy`) {#provide-a-spy-stub-herodetailservicespy}

Cet exemple remplace complètement le tableau `providers` du composant par un nouveau tableau contenant un `HeroDetailServiceSpy`.

`HeroDetailServiceSpy` est une version stub du vrai `HeroDetailService` qui simule toutes les fonctionnalités nécessaires de ce service.
Il n'injecte pas le `HeroService` de plus bas niveau et ne lui délègue rien : il n'est donc pas nécessaire de fournir une doublure de test pour celui-ci.

Les tests associés du composant `HeroDetail` vérifieront que des méthodes du `HeroDetailService` ont été appelées en espionnant les méthodes du service.
Le stub implémente donc ses méthodes sous forme de spies :

```ts
import {vi} from 'vitest';

class HeroDetailServiceSpy {
  testHero: Hero = {...testHero};

  /* emit cloned test hero */
  getHero = vi.fn(() => asyncData({...this.testHero}));

  /* emit clone of test hero, with changes merged in */
  saveHero = vi.fn((hero: Hero) => asyncData(Object.assign(this.testHero, hero)));
}
```

### Les tests de surcharge {#the-override-tests}

Désormais, les tests peuvent contrôler directement le héros du composant en manipulant le `testHero` du spy stub, et confirmer que les méthodes du service ont été appelées.

```ts
let hdsSpy: HeroDetailServiceSpy;

beforeEach(async () => {
  harness = await RouterTestingHarness.create();
  component = await harness.navigateByUrl(`/heroes/${testHero.id}`, HeroDetail);
  page = new Page();
  // get the component's injected HeroDetailServiceSpy
  hdsSpy = harness.routeDebugElement!.injector.get(HeroDetailService) as any;

  harness.detectChanges();
});

it('should have called `getHero`', () => {
  expect(hdsSpy.getHero, 'getHero called once').toHaveBeenCalledTimes(1);
});

it("should display stub hero's name", () => {
  expect(page.nameDisplay.textContent).toBe(hdsSpy.testHero.name);
});

it('should save stub hero change', async () => {
  const origName = hdsSpy.testHero.name;
  const newName = 'New Name';

  page.nameInput.value = newName;

  page.nameInput.dispatchEvent(new Event('input')); // tell Angular

  expect(component.hero.name, 'component hero has new name').toBe(newName);
  expect(hdsSpy.testHero.name, 'service hero unchanged before save').toBe(origName);

  click(page.saveBtn);
  expect(hdsSpy.saveHero, 'saveHero called once').toHaveBeenCalledTimes(1);

  await harness.fixture.whenStable();
  expect(hdsSpy.testHero.name, 'service hero has new name after save').toBe(newName);
  expect(TestBed.inject(Router).url).toEqual('/heroes');
});
```

### Autres surcharges {#more-overrides}

La méthode `TestBed.overrideComponent` peut être appelée plusieurs fois pour le même composant ou pour des composants différents.
Le `TestBed` propose des méthodes similaires, `overrideDirective`, `overrideModule` et `overridePipe`, pour explorer et remplacer des parties de ces autres classes.

Explorez par vous-même les options et les combinaisons.
