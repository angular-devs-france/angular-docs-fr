<!-- source: cdd92b17350681d5eafd89c93d4f2c60557f791b -->
# Logique inter-champs

La **logique inter-champs** est nécessaire dès qu'une règle, une validation ou un comportement d'un champ dépend de la valeur ou de l'état d'un autre champ.

Les Signal Forms fournissent un **contexte de champ** (field context) à chaque fonction de règle. Le contexte de champ donne accès à la valeur et à l'état du champ courant, et vous permet de lire les autres champs du formulaire avec `valueOf()`, `stateOf()` et `fieldTreeOf()`.

Ce guide présente en détail l'API du contexte de champ et montre des patterns inter-champs courants. Pour la validation d'un seul champ, consultez le [guide de la validation](/guide/forms/signals/validation).

## Comprendre le contexte de champ {#understanding-the-field-context}

Chaque fonction de règle des Signal Forms reçoit un paramètre **contexte de champ**, un objet qui décrit le champ courant et donne accès au reste du formulaire.

Trois propriétés vous donnent accès au champ courant :

| Propriété   | Type                 | Description                                                          |
| ----------- | -------------------- | -------------------------------------------------------------------- |
| `value`     | `Signal<TValue>`     | La valeur du champ courant, sous forme de signal                     |
| `state`     | `FieldState<TValue>` | L'état du champ courant (validité, erreurs, touché, modifié…)        |
| `fieldTree` | `FieldTree<TValue>`  | L'arbre du champ courant, pour accéder aux champs enfants par programmation |

Pour la logique inter-champs, les trois propriétés suivantes vous permettent d'accéder à d'autres parties du formulaire :

| Propriété       | Type                           | Description                                                                                                                        |
| --------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| `valueOf()`     | `(path) => PValue`             | La plus courante. Utilisez-la quand vous avez besoin de la valeur brute d'un autre champ pour des comparaisons ou des calculs.      |
| `stateOf()`     | `(path) => FieldState<PValue>` | Utilisez-la quand votre logique dépend de l'état d'un autre champ, par exemple s'il est valide, touché ou modifié.                  |
| `fieldTreeOf()` | `(path) => FieldTree<PModel>`  | Utilisez-la quand vous avez besoin d'accéder par programmation à l'arbre d'un autre champ, par exemple pour envoyer des erreurs à un champ enfant précis avec validateTree. |

Voici un exemple qui utilise `value` et `valueOf()` pour vérifier que le champ courant (la date de fin) est postérieur à la date de début du formulaire :

```ts
import {Component, signal} from '@angular/core';
import {form, validate} from '@angular/forms/signals';

@Component({/* ... */})
export class EventForm {
  eventModel = signal({
    startDate: new Date('2026-06-01'),
    endDate: new Date('2026-06-05'),
  });

  eventForm = form(this.eventModel, (schemaPath) => {
    validate(schemaPath.endDate, ({value, valueOf}) => {
      if (value() <= valueOf(schemaPath.startDate)) {
        return {
          kind: 'invalidDateRange',
          message: 'End date must be after start date',
        };
      }

      return null;
    });
  });
}
```

NOTE: Le paramètre `fieldContext` est généralement déstructuré pour n'en extraire que ce dont la règle a besoin. Les exemples suivants de ce guide utilisent ce pattern.

## Patterns de validation inter-champs {#cross-field-validation-patterns}

L'exemple de plage de dates de la section précédente valide la date de fin par rapport à la date de début. Comme la règle lit `valueOf(schemaPath.startDate)`, elle est réévaluée automatiquement dès que l'une des deux dates change. Autrement dit, un seul validateur suffit à maintenir un état d'erreur correct.

Cependant, ce validateur unique place l'erreur uniquement sur le champ de la date de fin. Si vous voulez que les deux champs affichent une erreur quand la plage est invalide, ajoutez une règle de validation correspondante sur chaque champ :

```ts
import {Component, signal} from '@angular/core';
import {form, validate} from '@angular/forms/signals';

@Component({/* ... */})
export class EventForm {
  eventModel = signal({
    startDate: new Date('2026-06-01'),
    endDate: new Date('2026-06-05'),
  });

  eventForm = form(this.eventModel, (schemaPath) => {
    validate(schemaPath.startDate, ({value, valueOf}) => {
      if (value() >= valueOf(schemaPath.endDate)) {
        return {
          kind: 'invalidDateRange',
          message: 'Start date must be before end date',
        };
      }
      return null;
    });

    validate(schemaPath.endDate, ({value, valueOf}) => {
      if (value() <= valueOf(schemaPath.startDate)) {
        return {
          kind: 'invalidDateRange',
          message: 'End date must be after start date',
        };
      }
      return null;
    });
  });
}
```

Les deux règles utilisent `valueOf()` pour lire l'autre champ. Comme chaque règle est réactive, modifier l'une des deux dates réévalue automatiquement les deux validations.

NOTE: Quand une règle implique plusieurs champs, vous devez décider où placer l'erreur : sur un champ précis, sur plusieurs champs ou sur le parent. En règle générale, placez l'erreur là où l'utilisateur irait le plus probablement corriger le problème.

### Champs obligatoires sous condition {#conditional-requirements}

Dans certains formulaires, certains champs ne sont obligatoires que sous certaines conditions. Par exemple, un formulaire d'inscription peut exiger un nom d'entreprise uniquement quand l'utilisateur choisit un type de compte professionnel :

```ts
import {Component, signal} from '@angular/core';
import {form, required} from '@angular/forms/signals';

@Component({/* ... */})
export class RegistrationForm {
  registrationModel = signal({
    accountType: 'personal' as 'personal' | 'business',
    companyName: '',
  });

  registrationForm = form(this.registrationModel, (schemaPath) => {
    required(schemaPath.companyName, {
      when: ({valueOf}) => valueOf(schemaPath.accountType) === 'business',
      message: 'Company name is required for business accounts',
    });
  });
}
```

L'option `when` reçoit le même contexte de champ que n'importe quelle autre fonction de règle : `valueOf` fonctionne donc de la même manière. Quand l'utilisateur revient à `'personal'`, la condition est réévaluée et l'obligation — ainsi que son erreur — disparaît automatiquement.

Utiliser `required()` avec `when` plutôt qu'une vérification manuelle avec `validate()` ajoute aussi au champ les métadonnées « obligatoire » appropriées, ce qui active des fonctionnalités d'accessibilité comme l'indication du caractère obligatoire du champ aux lecteurs d'écran.

### Valider en fonction de l'état d'un autre champ {#validating-based-on-another-fields-state}

Les exemples précédents utilisent `valueOf()` pour lire la valeur d'un autre champ. Parfois, votre logique dépend plutôt de l'_état_ d'un autre champ : s'il est valide, touché ou modifié. Utilisez `stateOf()` dans ce cas.

Par exemple, un champ de confirmation du mot de passe ne devrait vérifier la correspondance qu'une fois que l'utilisateur a interagi avec le champ du mot de passe. Si l'utilisateur n'a pas encore touché au mot de passe, signaler une différence sur la confirmation est prématuré :

```ts
import {Component, signal} from '@angular/core';
import {form, validate} from '@angular/forms/signals';

@Component({/* ... */})
export class PasswordForm {
  passwordModel = signal({
    password: '',
    confirmPassword: '',
  });

  passwordForm = form(this.passwordModel, (schemaPath) => {
    validate(schemaPath.confirmPassword, ({value, valueOf, stateOf}) => {
      if (!stateOf(schemaPath.password).touched()) {
        return null;
      }
      if (value() !== valueOf(schemaPath.password)) {
        return {
          kind: 'passwordMismatch',
          message: 'Passwords do not match',
        };
      }
      return null;
    });
  });
}
```

L'appel à `stateOf()` renvoie l'[état du champ](api/forms/signals/FieldState) de l'autre champ, ce qui vous donne accès à des signals comme `invalid()`, `touched()` et `dirty()`. Comme ce sont des signals, la règle est réévaluée dès que la validité du champ du mot de passe change.

CRITICAL: Veillez à ne pas lire un état qui dépend de la validation de votre propre champ, car cela crée une boucle circulaire. Par exemple, un validateur qui vérifie si le champ parent est valide crée une boucle infinie, car la validité du parent dépend de celle de ses enfants (qui inclut votre validateur).

## Utiliser validateTree {#using-validatetree}

Les exemples précédents utilisent `validate()` pour vérifier des champs individuels. Parfois, vous devez valider un groupe de champs dont la logique concerne intrinsèquement plusieurs champs, et diriger les erreurs vers des enfants précis de ce groupe. `validateTree` est idéal pour ce genre de scénario.

Par exemple, dans une grille de Sudoku, chaque ligne doit contenir des nombres uniques. C'est une règle au niveau du groupe : vous vérifiez la ligne entière, puis vous signalez les cellules précises qui l'enfreignent. Ce type de validation ne s'exprime pas proprement avec `validate` sur des champs individuels, car chaque cellule devrait connaître toutes les autres.

```ts
import {Component, signal} from '@angular/core';
import {form, validateTree} from '@angular/forms/signals';

@Component({/* ... */})
export class SudokuRow {
  rowModel = signal({
    cell1: 1,
    cell2: 3,
    cell3: 1,
    cell4: 4,
  });

  rowForm = form(this.rowModel, (schemaPath) => {
    validateTree(schemaPath, ({value, fieldTreeOf}) => {
      const row = value();
      const entries = [
        {val: row.cell1, fieldTree: fieldTreeOf(schemaPath.cell1)},
        {val: row.cell2, fieldTree: fieldTreeOf(schemaPath.cell2)},
        {val: row.cell3, fieldTree: fieldTreeOf(schemaPath.cell3)},
        {val: row.cell4, fieldTree: fieldTreeOf(schemaPath.cell4)},
      ];

      const counts = new Map<number, number>();
      for (const {val} of entries) {
        if (val !== 0) {
          counts.set(val, (counts.get(val) ?? 0) + 1);
        }
      }

      const errors = entries
        .filter(({val}) => val !== 0 && (counts.get(val) ?? 0) > 1)
        .map(({val, fieldTree}) => ({
          kind: 'duplicateInRow',
          message: `${val} already appears in this row`,
          fieldTree,
        }));

      return errors.length > 0 ? errors : null;
    });
  });
}
```

Le validateur s'exécute sur le champ parent (la ligne), lit les valeurs de toutes les cellules, compte les doublons et renvoie une erreur pour chaque cellule qui contient un nombre répété. La propriété `fieldTree` de chaque erreur indique à Angular quelle cellule doit afficher l'erreur. Sans `fieldTree`, les erreurs s'appliqueraient à la ligne elle-même, et non là où l'utilisateur a besoin de les voir.

Comme `validateTree` peut renvoyer un tableau d'erreurs, un seul validateur peut signaler plusieurs cellules à la fois. Chaque erreur inclut un `fieldTree` qui pointe vers sa cible, ce qui permet à Angular d'acheminer les erreurs vers les bons champs.

### Quand utiliser validateTree ou validate {#when-to-use-validatetree-vs-validate}

Préférez `validate()` avec `valueOf()` quand l'erreur appartient au champ validé, même si la règle lit d'autres champs. Tournez-vous vers `validateTree` quand :

- La logique de validation concerne intrinsèquement un groupe de champs, et non un seul champ
- Le validateur doit renvoyer des erreurs ciblant différents champs enfants

TIP: Pour une introduction à `validateTree` et à son type de retour, consultez le [guide de la validation](/guide/forms/signals/validation).

## Étapes suivantes {#next-steps}

Ce guide a présenté l'API du contexte de champ et des patterns inter-champs courants. Pour en savoir plus sur les guides liés des Signal Forms, consultez :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/custom-controls" title="Contrôles personnalisés" />
</docs-pill-row>
