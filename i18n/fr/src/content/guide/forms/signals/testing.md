<!-- source: 2263a009aef369946339dbcfb88f02016a41d507 -->
# Tester les Signal Forms

Les formulaires sont souvent critiques pour les applications, et les tests vous donnent l'assurance qu'ils se comportent correctement à mesure que la base de code évolue. Les Signal Forms placent l'essentiel de leur logique dans le schéma plutôt que dans le template : vous pouvez donc tester la majeure partie du comportement d'un formulaire sans rendre de composant.

Ce guide explique comment mettre en place ces tests, en commençant par des tests de logique isolés, puis en abordant les tests liés à un composant pour les cas où l'interaction avec le DOM compte.

## Tester la logique du formulaire de manière isolée {#testing-form-logic-in-isolation}

Lorsque vous avez seulement besoin de vérifier la validation, l'état désactivé, l'état obligatoire ou les erreurs produites, testez directement le formulaire au lieu de rendre un composant. Les tests isolés gardent une mise en place réduite et permettent au test de se concentrer sur le comportement du formulaire.

L'exigence principale est l'injector. Les Signal Forms ont besoin d'un contexte d'injection lors de la création du formulaire. Si un test appelle `form()` sans contexte d'injection, l'appel lève une exception avant que le test ne puisse vérifier quoi que ce soit sur le formulaire.

La façon la plus directe de satisfaire cette exigence est de passer explicitement un injector. Le test suivant crée un formulaire avec une règle `required` et vérifie que le champ devient valide après avoir reçu une valeur :

```ts {header: 'profile-form.spec.ts'}
import {Injector, signal} from '@angular/core';
import {TestBed} from '@angular/core/testing';
import {form, required} from '@angular/forms/signals';
import {describe, expect, it} from 'vitest';

describe('profile form', () => {
  it('marks required fields as invalid until they have a value', () => {
    const model = signal({name: ''});

    const profileForm = form(
      model,
      (path) => {
        required(path.name);
      },
      {injector: TestBed.inject(Injector)},
    );

    expect(profileForm.name().valid()).toBe(false);
    expect(profileForm.name().errors()).toEqual([expect.objectContaining({kind: 'required'})]);

    profileForm.name().value.set('Ada');

    expect(profileForm.name().valid()).toBe(true);
    expect(profileForm.name().errors()).toEqual([]);
  });
});
```

Ce pattern convient bien à la plupart des tests isolés, car l'exigence d'injector reste visible au point d'appel. Il reflète aussi la façon dont les tests unitaires des Signal Forms dans le code source d'Angular créent les formulaires.

Lorsque le code testé appelle `form()` en interne, vous ne pourrez peut-être pas passer l'injector directement. Dans ce cas, encapsulez l'appel dans un contexte d'injection ambiant :

```ts {header: 'profile-form.spec.ts'}
import {signal} from '@angular/core';
import {TestBed} from '@angular/core/testing';
import {form, required} from '@angular/forms/signals';
import {describe, expect, it} from 'vitest';

describe('profile form', () => {
  it('can create a form inside an injection context', () => {
    const model = signal({name: ''});

    TestBed.runInInjectionContext(() => {
      const profileForm = form(model, (path) => {
        required(path.name);
      });

      expect(profileForm.name().valid()).toBe(false);
    });
  });
});
```

Les deux patterns produisent le même type de formulaire. Passer `{injector}` est souvent le choix le plus clair lorsque le test crée directement le formulaire. `TestBed.runInInjectionContext()` est utile lorsque le code testé appelle `form()` en interne et que vous devez fournir le contexte d'injection environnant.

Une fois le formulaire créé, testez-le via les signals d'état des champs. Les assertions courantes incluent `valid()`, `invalid()`, `disabled()`, `required()` et `errors()`. Pour la plupart de la logique de formulaire, cela suffit à vérifier le comportement sans faire intervenir le DOM.

## Tester un formulaire avec plusieurs règles {#testing-a-form-with-multiple-rules}

Une fois la mise en place de l'injector effectuée, une bonne étape suivante est un test complet qui exerce plusieurs éléments de logique de formulaire ensemble. Ce type de test reste isolé, mais il ressemble beaucoup plus à un vrai formulaire d'application.

Par exemple, ce test vérifie à la fois une règle d'obligation simple et une règle d'obligation conditionnelle qui dépend d'un autre champ :

```ts {header: 'profile-form.spec.ts'}
import {Injector, signal} from '@angular/core';
import {TestBed} from '@angular/core/testing';
import {form, required} from '@angular/forms/signals';
import {describe, expect, it} from 'vitest';

describe('profile form', () => {
  it('updates validation state when related fields change', () => {
    const model = signal({
      name: '',
      age: 5,
    });

    const profileForm = form(
      model,
      (path) => {
        required(path.name);
        required(path.name, {
          error: (ctx) => ({kind: `required-${ctx.valueOf(path.age)}`}),
          when: ({valueOf}) => valueOf(path.age) > 10,
        });
      },
      {injector: TestBed.inject(Injector)},
    );

    expect(profileForm.name().invalid()).toBe(true);
    expect(profileForm.name().errors()).toEqual([expect.objectContaining({kind: 'required'})]);

    profileForm.age().value.set(15);

    expect(profileForm.name().errors()).toEqual([
      expect.objectContaining({kind: 'required'}),
      expect.objectContaining({kind: 'required-15'}),
    ]);

    profileForm.name().value.set('Ada');

    expect(profileForm.name().valid()).toBe(true);
    expect(profileForm.name().errors()).toEqual([]);
  });
});
```

Cet exemple illustre un pattern de test important : modifiez un champ, puis vérifiez l'état d'un autre champ. Comme les règles des Signal Forms sont réactives, la validation d'un champ peut dépendre de valeurs sœurs, de valeurs parentes ou d'autres conditions dérivées. Les tests doivent vérifier directement ces relations au lieu de contrôler uniquement le champ modifié.

Pour les tests centrés sur la validation, `errors()` est généralement l'assertion la plus utile. `valid()` et `invalid()` indiquent si le champ passe actuellement la validation, mais `errors()` montre quelle règle a produit l'échec. Cela devient particulièrement utile dès qu'un champ a plusieurs validateurs ou des règles conditionnelles.

La même structure fonctionne pour la plupart des tests de formulaire du quotidien :

1. Créez un signal de model avec la forme la plus réduite qui reproduit le comportement.
1. Construisez le formulaire avec un injector explicite.
1. Vérifiez l'état initial du champ.
1. Modifiez un champ avec `.value.set(...)`, y compris les champs frères lorsque vous testez des règles inter-champs.
1. Vérifiez les signals d'état mis à jour, généralement `errors()`, `valid()` ou `invalid()`.

Lorsqu'un test porte sur le comportement du schéma plutôt que sur le rendu, adoptez par défaut ce style isolé. Il est plus rapide qu'un test de composant et permet de voir plus facilement quelle règle est responsable quand le comportement change.

## Tester des formulaires liés à des composants {#testing-forms-bound-to-components}

Lorsque vous devez vérifier un comportement qui dépend des bindings du template, d'une interaction utilisateur via `dispatchEvent` ou de contrôles de formulaire personnalisés qui gèrent leur propre rendu, les tests isolés ne suffisent pas. Vous avez besoin de tests liés au composant qui rendent le template afin de pouvoir interagir avec de vrais éléments du DOM.

### Mettre en place un test de composant {#setting-up-a-component-test}

Les tests liés au composant rendent le composant afin que vous puissiez interagir avec de vrais éléments du DOM. Créez le composant avec `TestBed.createComponent()` et attendez la fin du rendu avant de faire vos assertions :

```angular-ts {header: 'profile-form.ts'}
import {Component, signal} from '@angular/core';
import {form, FormField, required} from '@angular/forms/signals';

@Component({
  selector: 'app-profile-form',
  imports: [FormField],
  template: `<input [formField]="profileForm.name" />`,
})
export class ProfileForm {
  readonly model = signal({name: 'Ada'});
  readonly profileForm = form(this.model, (path) => {
    required(path.name);
  });
}
```

```ts {header: 'profile-form.spec.ts'}
import {TestBed} from '@angular/core/testing';
import {describe, expect, it} from 'vitest';
import {ProfileForm} from './profile-form';

describe('ProfileForm', () => {
  it('reflects model values in the DOM and updates the model on user input', async () => {
    const fixture = TestBed.createComponent(ProfileForm);
    await fixture.whenStable();

    const input = fixture.nativeElement.querySelector('input') as HTMLInputElement;

    // Model → View: the input reflects the model's initial value
    expect(input.value).toBe('Ada');

    // View → Model: simulate the user clearing the field
    input.value = '';
    input.dispatchEvent(new Event('input'));
    await fixture.whenStable();

    expect(fixture.componentInstance.profileForm.name().value()).toBe('');
    expect(fixture.componentInstance.profileForm.name().valid()).toBe(false);
  });
});
```

Remarquez que le composant utilise `form()` sans injector explicite, car le contexte d'injection du composant le fournit automatiquement. Après chaque modification, `await fixture.whenStable()` attend la fin du rendu et des effects avant de faire les assertions.

Le même pattern fonctionne pour les opérations asynchrones comme les validateurs asynchrones ou les appels serveur. Appelez `await fixture.whenStable()` une fois le travail asynchrone résolu.

## Quand utiliser chaque approche {#when-to-use-each-approach}

| Ce que vous devez vérifier                           | Approche        |
| ---------------------------------------------------- | --------------- |
| Règles de validation, `errors()`, `valid()`, `invalid()` | Isolée      |
| État désactivé, obligatoire ou en lecture seule      | Isolée          |
| Dépendances réactives inter-champs                   | Isolée          |
| Schémas conditionnels (`applyWhen`, `applyWhenValue`) | Isolée         |
| Rendu des valeurs des champs dans le DOM             | Liée au composant |
| Saisie de l'utilisateur mettant à jour le model      | Liée au composant |
| Contrôles de formulaire personnalisés avec leurs propres templates | Liée au composant |
| Gestion du focus ou attributs d'accessibilité        | Liée au composant |

La plupart des formulaires n'ont besoin que de tests isolés. La logique du formulaire (comme la validation, l'état désactivé, les règles inter-champs) réside dans le schéma, et les schémas n'ont pas besoin de template pour s'exécuter. Les tests liés au composant apportent de la valeur lorsque le comportement qui vous intéresse franchit la frontière entre le formulaire et le DOM.

## Prochaines étapes {#next-steps}

Ce guide a présenté comment tester les Signal Forms de manière isolée et avec des templates de composant. Voici des guides connexes qui explorent d'autres aspects des Signal Forms :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/form-submission" title="Soumission de formulaire" />
</docs-pill-row>
