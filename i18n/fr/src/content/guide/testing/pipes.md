<!-- source: c2deb6df4dbade20f699e2d18d974ea3343bddeb -->
# Tester les pipes

Vous pouvez tester les [pipes](guide/templates/pipes) sans les utilitaires de test d'Angular.

## Tester le `TitleCasePipe` {#testing-the-titlecasepipe}

Une classe de pipe possède une seule méthode, `transform`, qui transforme la valeur d'entrée en une valeur de sortie.
L'implémentation de `transform` interagit rarement avec le DOM.
La plupart des pipes ne dépendent d'Angular que par les métadonnées `@Pipe` et une interface.

Prenons un `TitleCasePipe` qui met en majuscule la première lettre de chaque mot.
Voici une implémentation à base d'expression régulière.

```ts
import {Pipe, PipeTransform} from '@angular/core';

@Pipe({name: 'titlecase', pure: true})
/** Transform to Title Case: uppercase the first letter of the words in a string. */
export class TitleCasePipe implements PipeTransform {
  transform(input: string): string {
    return input.length === 0
      ? ''
      : input.replace(/\w\S*/g, (txt) => txt[0].toUpperCase() + txt.slice(1).toLowerCase());
  }
}
```

Tout ce qui utilise une expression régulière mérite d'être testé en profondeur. Vous pouvez utiliser les techniques classiques de tests unitaires pour explorer les cas attendus et les cas limites.

```ts
describe('TitleCasePipe', () => {
  // This pipe is a pure, stateless function so no need for BeforeEach
  const pipe = new TitleCasePipe();

  it('transforms "abc" to "Abc"', () => {
    expect(pipe.transform('abc')).toBe('Abc');
  });

  it('transforms "abc def" to "Abc Def"', () => {
    expect(pipe.transform('abc def')).toBe('Abc Def');
  });

  // ... more tests ...
});
```

## Écrire des tests du DOM pour compléter un test de pipe {#writing-dom-tests-to-support-a-pipe-test}

Il s'agit de tests du pipe _de manière isolée_.
Ils ne peuvent pas indiquer si le `TitleCasePipe` fonctionne correctement tel qu'il est appliqué dans les composants de l'application.

Envisagez d'ajouter des tests de composants comme celui-ci :

```ts
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
