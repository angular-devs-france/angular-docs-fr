<!-- source: 0f3f71e8fec8caced739b7efc26c5ad5a6ffd4bd -->
# Formulaires dynamiques avec JSON

Certains formulaires ne peuvent pas définir leur structure à la compilation. Les formulaires pilotés par le serveur, les panneaux d'administration, les applications multi-tenant et les contenus gérés par un CMS doivent tous afficher des champs à partir d'une configuration fournie à l'exécution, généralement sous forme de JSON provenant d'un backend, d'un outil d'administration ou de paramètres propres à chaque tenant.

Ce guide montre comment créer des formulaires dont le model, le schéma, la validation et le rendu sont tous dérivés d'une seule configuration fournie à l'exécution.

## Quand utiliser des formulaires pilotés par JSON {#when-to-use-json-driven-forms}

Ce pattern est un bon choix lorsque :

- Un backend définit quels champs apparaissent selon le rôle de l'utilisateur, des feature flags ou des règles métier
- Des non-développeurs configurent la structure du formulaire via un panneau d'administration ou un CMS
- Chaque tenant d'une application multi-tenant a sa propre structure de formulaire stockée en configuration
- Les formulaires doivent évoluer sans redéployer le frontend

Utilisez un formulaire statique (avec des champs définis directement dans votre composant) lorsque la structure est connue au moment du build. Les formulaires statiques bénéficient d'une vérification TypeScript complète sur chaque champ, ainsi que de tests simples et du support de l'outillage.

## Définir une configuration de champ typée {#defining-a-typed-field-config}

Lorsque vous voulez afficher des champs à partir d'une configuration fournie à l'exécution, commencez par un type TypeScript qui décrit la forme de chaque champ. Une union discriminée par `kind` permet à chaque variante de déclarer ses propres options de validation :

```ts
type FieldConfig =
  | {kind: 'text'; name: string; label: string; required?: boolean}
  | {kind: 'number'; name: string; label: string; required?: boolean; min?: number; max?: number};
```

Chaque variante a un nom, un libellé et un indicateur `required` facultatif. Les champs numériques acceptent en plus des bornes `min` et `max`. Ajoutez de nouvelles variantes en ajoutant de nouvelles branches `kind`.

Une configuration concrète peut ressembler à ceci :

```ts
const profileConfig: FieldConfig[] = [
  {kind: 'text', name: 'fullName', label: 'Full Name', required: true},
  {kind: 'number', name: 'age', label: 'Age', required: true, min: 18, max: 120},
];
```

En pratique, ce `FieldConfig[]` provient généralement de votre backend, d'un panneau d'administration ou d'un CMS. Par souci de concision, les exemples ci-dessous utilisent un littéral défini dans le composant.

## Construire le model à partir de la configuration {#building-the-model-from-config}

Le model du formulaire a besoin d'une entrée par champ, avec une valeur par défaut adaptée au type du champ. Une petite fonction utilitaire s'en charge :

```ts
function buildModel(configs: FieldConfig[]): Record<string, string | number | null> {
  const initial: Record<string, string | number | null> = {};
  for (const config of configs) {
    initial[config.name] = config.kind === 'number' ? null : '';
  }
  return initial;
}
```

Le model utilise `Record<string, string | number | null>` car les clés ne sont pas connues à l'avance.

De plus, les champs numériques sont initialisés à `null` plutôt qu'à `0` afin qu'un champ vide soit lu comme vide. Avec `0`, [`required()`](api/forms/signals/required) considérerait le champ comme déjà rempli, et toute contrainte [`min()`](api/forms/signals/min) supérieure à zéro marquerait le champ comme invalide avant même que l'utilisateur ne saisisse quoi que ce soit.

## Construire le schéma à partir de la configuration {#building-the-schema-from-config}

Le schéma est lui aussi dérivé de la configuration. Vous pouvez parcourir chaque entrée et appliquer les validateurs correspondant à son type :

```ts
import {required, min, max, SchemaFn, SchemaPath} from '@angular/forms/signals';

function buildSchema(configs: FieldConfig[]): SchemaFn<Record<string, string | number | null>> {
  return (path) => {
    for (const config of configs) {
      const fieldPath = path[config.name];

      if (config.required) {
        required(fieldPath);
      }

      if (config.kind === 'number') {
        const numberPath = fieldPath as unknown as SchemaPath<number | null>;
        if (config.min !== undefined) min(numberPath, config.min);
        if (config.max !== undefined) max(numberPath, config.max);
      }
    }
  };
}
```

L'union discriminée affine le type de `config` dans chaque branche, si bien que `config.min` et `config.max` sont correctement typés lorsque `config.kind === 'number'`.

## Exprimer des règles conditionnelles dans la configuration {#expressing-conditional-rules-in-config}

Certaines règles de validation n'ont de sens que sous certaines conditions. Par exemple, les codes d'État américains ne doivent être validés que lorsque le pays est les États-Unis. Exprimez ces dépendances dans la configuration en ajoutant un discriminant `when` qui nomme un autre champ et la valeur à laquelle il doit être égal :

```ts
type WhenCondition = {field: string; equals: string | number};

type FieldConfig =
  | {kind: 'text'; name: string; label: string; required?: boolean; when?: WhenCondition}
  | {
      kind: 'number';
      name: string;
      label: string;
      required?: boolean;
      min?: number;
      max?: number;
      when?: WhenCondition;
    };
```

Mettez à jour `buildSchema()` pour traduire `when` en un appel à [`applyWhen()`](api/forms/signals/applyWhen). La logique commune d'application des règles est déplacée dans une petite closure afin que les branches conditionnelle et inconditionnelle appellent toutes deux la même fonction :

```ts
import {applyWhen, required, min, max, SchemaFn, SchemaPath} from '@angular/forms/signals';

function buildSchema(configs: FieldConfig[]): SchemaFn<Record<string, string | number | null>> {
  return (rootPath) => {
    for (const config of configs) {
      const applyRules = (path: typeof rootPath) => {
        const fieldPath = path[config.name];
        if (config.required) required(fieldPath);
        if (config.kind === 'number') {
          const numberPath = fieldPath as unknown as SchemaPath<number | null>;
          if (config.min !== undefined) min(numberPath, config.min);
          if (config.max !== undefined) max(numberPath, config.max);
        }
      };

      if (config.when) {
        const {field, equals} = config.when;
        applyWhen(rootPath, ({valueOf}) => valueOf(rootPath[field]) === equals, applyRules);
      } else {
        applyRules(rootPath);
      }
    }
  };
}
```

Lorsque la condition d'`applyWhen()` est vraie, les règles qu'elle contient s'activent. Lorsque la condition devient fausse, les règles se désactivent et l'état de validation du champ est réinitialisé. Comme la fonction de condition lit la valeur via `valueOf(rootPath[field])`, le formulaire réévalue la condition chaque fois que le champ référencé change.

Une configuration qui utilise `when` ressemble à ceci :

```ts
const addressConfig: FieldConfig[] = [
  {kind: 'text', name: 'country', label: 'Country', required: true},
  {
    kind: 'text',
    name: 'stateCode',
    label: 'State',
    required: true,
    when: {field: 'country', equals: 'US'},
  },
];
```

Le champ `stateCode` n'exige une valeur que lorsque `country` vaut `'US'`. Les utilisateurs qui saisissent un autre pays peuvent laisser `stateCode` vide sans bloquer la soumission.

Pour des conditions plus complexes (plusieurs champs, plages de valeurs ou tests autres que l'égalité), étendez `WhenCondition` avec des discriminants supplémentaires (comme `in: string[]` ou `notEquals: string | number`) et traduisez chaque variante dans `buildSchema()`. Le principe reste le même : la configuration porte les données, `buildSchema()` les traduit en appels à `applyWhen()`.

Pour conditionner la visibilité plutôt que la validation, suivez le même pattern en utilisant [`hidden()`](api/forms/signals/hidden) sur le schema path du champ. Consultez [Configurer l'état `hidden()` des champs](guide/forms/signals/form-logic#configuring-hidden-state-on-fields) pour plus de détails.

## Exprimer des champs répétés dans la configuration {#expressing-repeating-fields-in-config}

Certaines configurations ont besoin de champs qui s'agrandissent et se réduisent à l'exécution, comme une liste de numéros de téléphone, de tags ou de lignes de facture. Ajoutez un type `array` à la configuration et traduisez-le en [`applyEach()`](api/forms/signals/applyEach) afin que les règles par élément s'appliquent uniformément à mesure que des éléments sont ajoutés ou retirés.

Étendez `FieldConfig` avec une variante `array`. Cet exemple utilise des tableaux de chaînes ; la même approche s'étend aux tableaux d'objets en remplaçant la forme de l'élément par un record :

```ts
type FieldConfig =
  | {kind: 'text'; name: string; label: string; required?: boolean; when?: WhenCondition}
  | {
      kind: 'number';
      name: string;
      label: string;
      required?: boolean;
      min?: number;
      max?: number;
      when?: WhenCondition;
    }
  | {kind: 'array'; name: string; label: string; itemRequired?: boolean; when?: WhenCondition};
```

Mettez à jour `buildModel()` pour initialiser les champs de type tableau avec un tableau vide. Le type du model s'élargit pour inclure `string[]` :

```ts
function buildModel(configs: FieldConfig[]): Record<string, string | number | null | string[]> {
  const initial: Record<string, string | number | null | string[]> = {};
  for (const config of configs) {
    if (config.kind === 'number') initial[config.name] = null;
    else if (config.kind === 'array') initial[config.name] = [];
    else initial[config.name] = '';
  }
  return initial;
}
```

Mettez à jour `buildSchema()` pour appliquer des règles par élément avec `applyEach()`. Le schema path issu d'un model `Record<string, string | number | null | string[]>` est trop large pour que `applyEach()` (et `min()` / `max()`) puisse être vérifié directement par le typage ; convertissez donc `fieldPath` vers la forme appropriée dans chaque branche `kind` :

```ts
import {
  applyEach,
  applyWhen,
  required,
  min,
  max,
  SchemaFn,
  SchemaPath,
} from '@angular/forms/signals';

function buildSchema(
  configs: FieldConfig[],
): SchemaFn<Record<string, string | number | null | string[]>> {
  return (rootPath) => {
    for (const config of configs) {
      const applyRules = (path: typeof rootPath) => {
        const fieldPath = path[config.name];

        if (config.kind === 'array') {
          const arrayPath = fieldPath as unknown as SchemaPath<string[]>;
          if (config.itemRequired) {
            applyEach(arrayPath, (item) => required(item));
          }
          return;
        }

        if (config.required) required(fieldPath);

        if (config.kind === 'number') {
          const numberPath = fieldPath as unknown as SchemaPath<number | null>;
          if (config.min !== undefined) min(numberPath, config.min);
          if (config.max !== undefined) max(numberPath, config.max);
        }
      };

      if (config.when) {
        const {field, equals} = config.when;
        applyWhen(rootPath, ({valueOf}) => valueOf(rootPath[field]) === equals, applyRules);
      } else {
        applyRules(rootPath);
      }
    }
  };
}
```

Les casts dans chaque branche sont des échappatoires délibérées : vous échangez la garantie structurelle du compilateur contre un invariant à l'exécution que la vérification `kind` environnante fait respecter. Chaque cast est limité à un bloc `kind`, de sorte que l'hypothèse reste locale et simple à auditer.

Une configuration qui utilise le type `array` ressemble à ceci :

```ts
const contactConfig: FieldConfig[] = [
  {kind: 'text', name: 'fullName', label: 'Full name', required: true},
  {kind: 'array', name: 'phoneNumbers', label: 'Phone numbers', itemRequired: true},
];
```

Pour afficher un champ de type tableau, parcourez-le avec `@for` et laissez les utilisateurs ajouter ou retirer des éléments en mettant à jour le signal du model. Ajoutez un accesseur typé qui renvoie un [`FieldTree`](api/forms/signals/FieldTree) afin que l'itération voie la structure du tableau, et définissez des méthodes pour agrandir et réduire le model :

```ts
import {FieldTree} from '@angular/forms/signals';

// inside the component class
asArrayField(name: string): FieldTree<string[]> {
  return this.dynamicForm[name] as unknown as FieldTree<string[]>;
}

addItem(name: string) {
  this.model.update(current => ({
    ...current,
    [name]: [...(current[name] as string[]), ''],
  }));
}

removeItem(name: string, index: number) {
  this.model.update(current => ({
    ...current,
    [name]: (current[name] as string[]).filter((_, i) => i !== index),
  }));
}
```

`FieldTree<string[]>` est itérable, donc `@for` peut le parcourir ; chaque élément est un `FieldTree<string>` qui convient directement à `[formField]`. Les champs feuilles peuvent utiliser des accesseurs `Field<T>` à la place, puisque `Field<T>` est la signature appelable sans itération.

Dans le template, affichez un cas de tableau comme ceci :

```angular-html
@case ('array') {
  <fieldset>
    <legend>{{ config.label }}</legend>
    @for (item of asArrayField(config.name); track item) {
      <input type="text" [formField]="item" />
      <button type="button" (click)="removeItem(config.name, $index)">Remove</button>
    }
    <button type="button" (click)="addItem(config.name)">Add</button>
  </fieldset>
}
```

La méthode `addItem()` étend le model ; le formulaire redérive automatiquement les champs du tableau. Les nouveaux éléments démarrent avec un état de validation vierge. La méthode `removeItem()` filtre le model. Comme les tableaux de primitives sont [suivis par position](#tracking-item-identity), l'état du champ est lié à l'index plutôt qu'à la valeur : lorsqu'un élément est retiré, les valeurs suivantes se décalent vers les positions précédentes et héritent de l'état de champ existant, tandis que le dernier champ est supprimé.

### Suivre l'identité des éléments {#tracking-item-identity}

Les Signal Forms suivent chaque élément d'un tableau d'objets par son identité. Lorsque vous conservez une référence vers un champ à une position donnée, cette référence suit les données sous-jacentes, pas la position. Lire l'état via la référence conservée renvoie les données même si elles ont été déplacées :

```ts
const contactModel = signal([
  {name: 'Alice', phone: '555-0001'},
  {name: 'Bob', phone: '555-0002'},
]);

const contactForm = form(contactModel);

// Hold a reference to the field that's currently at index 0 (Alice).
const aliceField = contactForm[0];

// Swap the array items so Bob is at index 0, Alice at index 1.
contactModel.update(([alice, bob]) => [bob, alice]);

// The held reference still points to Alice's field, even after the swap.
console.log(aliceField().value().phone); // '555-0001' (Alice's number)
console.log(contactForm[0]().value().phone); // '555-0002' (Bob, now at index 0)
```

Ce suivi d'identité évite des bugs lors des tris, réordonnancements ou filtrages, tant que l'élément référencé reste dans le tableau. Les références de champ conservées restent valides même lorsque l'ordre du tableau change ; retirer l'élément référencé lui-même rend la référence conservée orpheline.

Pour les tableaux de primitives (l'exemple `phoneNumbers` ci-dessus), les Signal Forms suivent plutôt les éléments par position : l'index 0 désigne toujours la valeur qui se trouve actuellement en position 0.

L'identité repose ici sur la référence d'objet JavaScript, et non sur un identifiant logique comme une clé de base de données. Si vous remplacez le tableau par des objets fraîchement désérialisés (par exemple, après un rechargement depuis le serveur), l'état du champ ne suit pas l'élément logique, même lorsque l'`id` de chaque élément est inchangé. La garantie couvre les opérations en mémoire comme le tri, le réordonnancement et le filtrage, pas le rafraîchissement des données.

## Valider la configuration {#validating-the-config}

Les configurations provenant de sources externes doivent être validées avant la construction du formulaire. Plusieurs cas d'échec peuvent se cacher dans un JSON non fiable :

- Des valeurs `name` en double écrasent les entrées précédentes du model et cassent l'expression `track config.name` dans le template.
- Une clause `when` qui nomme un champ inexistant échoue à l'exécution lors de la première évaluation de la condition.
- Une clause `when` qui compare à un champ `array` n'a pas de sémantique d'égalité définie.
- Une valeur `when.equals` dont le type ne correspond pas au type du champ référencé ne correspond silencieusement jamais, ce qui masque le comportement conditionnel comme si la règle n'était jamais active.

Interceptez ces quatre cas à la frontière :

```ts
function validateConfigs(configs: FieldConfig[]): FieldConfig[] {
  const knownNames = new Set<string>();

  for (const config of configs) {
    if (knownNames.has(config.name)) {
      throw new Error(`Duplicate field name in config: "${config.name}"`);
    }
    knownNames.add(config.name);
  }

  for (const config of configs) {
    if (!config.when) continue;
    if (!knownNames.has(config.when.field)) {
      throw new Error(
        `Field "${config.name}" references unknown field "${config.when.field}" in its 'when' condition.`,
      );
    }
    const referenced = configs.find((c) => c.name === config.when!.field)!;
    if (referenced.kind === 'array') {
      throw new Error(
        `Field "${config.name}" cannot use 'when' to compare against array field "${config.when.field}".`,
      );
    }
    const expected = referenced.kind === 'text' ? 'string' : 'number';
    if (typeof config.when.equals !== expected) {
      throw new Error(
        `Field "${config.name}" compares ${referenced.kind} field "${config.when.field}" against a ${typeof config.when.equals} value; expected a ${expected}.`,
      );
    }
  }

  return configs;
}
```

La première passe garantit l'unicité ; la seconde parcourt chaque clause `when` pour confirmer que le champ référencé existe, n'est pas un tableau et est comparé à une valeur du bon type. La fonction renvoie les configurations inchangées en cas de succès, ce qui lui permet de se combiner proprement avec l'initialiseur du champ de classe qui contient les configurations dans le composant. Les erreurs apparaissent ainsi à la frontière entre votre application et la source en amont, plutôt que sous la forme d'un dysfonctionnement opaque du formulaire plus tard.

## Afficher le formulaire dynamiquement {#rendering-the-form-dynamically}

Dans le composant, utilisez `@for` pour parcourir les configurations et `@switch` sur `kind` pour choisir le bon contrôle de saisie :

```angular-ts
import {Component, signal} from '@angular/core';
import {Field, FieldTree, form, FormField, FormRoot} from '@angular/forms/signals';

@Component({
  selector: 'app-dynamic-form',
  imports: [FormField, FormRoot],
  template: `
    <form [formRoot]="dynamicForm">
      @for (config of configs; track config.name) {
        @switch (config.kind) {
          @case ('text') {
            <label>
              {{ config.label }}
              <input type="text" [formField]="asTextField(config.name)" />
            </label>
          }
          @case ('number') {
            <label>
              {{ config.label }}
              <input type="number" [formField]="asNumberField(config.name)" />
            </label>
          }
          @case ('array') {
            <fieldset>
              <legend>{{ config.label }}</legend>
              @for (item of asArrayField(config.name); track item; let i = $index) {
                <input type="text" [formField]="item" />
                <button type="button" (click)="removeItem(config.name, i)">Remove</button>
              }
              <button type="button" (click)="addItem(config.name)">Add</button>
            </fieldset>
          }
        }
      }
    </form>
  `,
})
export class DynamicForm {
  configs: FieldConfig[] = validateConfigs([
    {kind: 'text', name: 'fullName', label: 'Full Name', required: true},
    {kind: 'number', name: 'age', label: 'Age', required: true, min: 18, max: 120},
    {kind: 'array', name: 'phoneNumbers', label: 'Phone numbers', itemRequired: true},
  ]);

  model = signal(buildModel(this.configs));

  dynamicForm = form(this.model, buildSchema(this.configs));

  asTextField(name: string): Field<string> {
    // <input type="text"> requires Field<string>.
    return this.dynamicForm[name] as unknown as Field<string>;
  }

  asNumberField(name: string): Field<number | null> {
    // <input type="number"> requires Field<number | null>.
    return this.dynamicForm[name] as unknown as Field<number | null>;
  }

  asArrayField(name: string): FieldTree<string[]> {
    // FieldTree (not Field) so @for can iterate the array.
    return this.dynamicForm[name] as unknown as FieldTree<string[]>;
  }

  addItem(name: string) {
    this.model.update((current) => ({
      ...current,
      [name]: [...(current[name] as string[]), ''],
    }));
  }

  removeItem(name: string, index: number) {
    this.model.update((current) => ({
      ...current,
      [name]: (current[name] as string[]).filter((_, i) => i !== index),
    }));
  }
}
```

La vérification de types des templates traite `dynamicForm[name]` comme une expression indépendante, si bien que l'affinage de type du `@switch` sur `config.kind` n'atteint pas l'accès indexé. Les accesseurs reformulent cet affinage sous forme de cast au niveau du binding, et la branche `kind` correspondante garantit que le type affiné est correct à l'exécution.

Comme le model et le schéma sont tous deux dérivés du même `FieldConfig[]` lors de la construction du composant, ils ne peuvent pas diverger pour une configuration donnée. L'exemple ci-dessus suppose que la configuration est disponible de manière synchrone lors de la création du composant.

## Prochaines étapes {#next-steps}

Les formulaires pilotés par JSON gardent leur model et leur schéma alignés en dérivant les deux du même `FieldConfig[]`. Chaque extension de ce guide (règles conditionnelles, champs répétés) élargit le type et ajoute une étape de traduction dans `buildSchema()` tout en préservant cet alignement. Le model et le schéma restent liés, quelle que soit la provenance de la configuration ou la façon dont elle évolue.

Pour découvrir d'autres guides consacrés à d'autres aspects des Signal Forms, consultez :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/schemas" title="Schémas et composabilité des schémas" />
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/form-logic" title="Ajouter de la logique de formulaire" />
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
</docs-pill-row>

Pour la documentation détaillée de l'API, consultez :

- [`form()`](api/forms/signals/form) - Crée un formulaire à partir d'un signal de model
- [`applyWhen()`](api/forms/signals/applyWhen) - Applique un schéma de manière conditionnelle selon un état réactif
- [`applyEach()`](api/forms/signals/applyEach) - Applique un schéma à chaque élément d'un champ de type tableau
- [`FieldTree`](api/forms/signals/FieldTree) - Arbre de champs navigable exposé par `form()`
- [`SchemaFn`](api/forms/signals/SchemaFn) - Signature de type des fonctions de schéma
