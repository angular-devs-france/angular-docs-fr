<!-- source: 77fdc825942186617ef6446459cb4fa2931b1bd5 -->
# Métadonnées des champs

Les métadonnées d'un champ sont des données réactives que vous pouvez attacher à un champ individuel. Les validateurs de contrainte intégrés d'Angular comme `required()` et `min()` utilisent ce système en interne. Autrement dit, chaque fois que vous appelez un validateur, vous contribuez à une clé de métadonnées pour ce champ précis.

Ce guide présente le système de métadonnées en détail : comment les reducers combinent les contributions de plusieurs règles de schéma, comment écrire des reducers personnalisés, comment la lecture se combine avec `hasMetadata()`, et comment les métadonnées gérées associent des objets dotés d'un cycle de vie à des champs individuels.

## Vous utilisez déjà les métadonnées {#you-have-already-been-using-metadata}

Quand vous appelez `required()` dans un schéma et lisez `.required()` sur le champ obtenu dans un template, vous utilisez le système de métadonnées. `state.required` n'est pas une propriété à part. C'est un getter pratique qui renvoie la valeur courante d'une clé de métadonnées intégrée, `REQUIRED`.

```angular-ts
import {Component, signal} from '@angular/core';
import {form, required, FormField} from '@angular/forms/signals';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `
    <form>
      <label>
        Username
        @if (registrationForm.username().required()) {
          <span class="required-marker" aria-hidden="true">*</span>
        }
        <input [formField]="registrationForm.username" />
      </label>
    </form>
  `,
})
export class Registration {
  registrationModel = signal({username: ''});

  registrationForm = form(this.registrationModel, (path) => {
    required(path.username);
  });
}
```

Appeler `required(path.username)` apporte une valeur à la clé de métadonnées `REQUIRED` de ce champ. Lire `registrationForm.username().required()` renvoie la valeur accumulée. La clé de métadonnées est le pont qui relie les deux.

Plusieurs validateurs de contrainte intégrés suivent ce pattern :

| Validateur    | Clé de métadonnées         | Type                  | Getter de `FieldState` |
| ------------- | -------------------------- | --------------------- | ------------------- |
| `required()`  | `REQUIRED`                 | `boolean`             | `required`          |
| `min()`       | `MIN` sélectionne `MIN_NUMBER` | `number \| undefined` | `min`               |
| `max()`       | `MAX` sélectionne `MAX_NUMBER` | `number \| undefined` | `max`               |
| `minDate()`   | `MIN` sélectionne `MIN_DATE`   | `Date \| undefined`   | `min`               |
| `maxDate()`   | `MAX` sélectionne `MAX_DATE`   | `Date \| undefined`   | `max`               |
| `minLength()` | `MIN_LENGTH`               | `number \| undefined` | `minLength`         |
| `maxLength()` | `MAX_LENGTH`               | `number \| undefined` | `maxLength`         |
| `pattern()`   | `PATTERN`                  | `RegExp[]`            | `pattern`           |

Les validateurs qui ne sont pas des contraintes, comme `email()` et `validate()`, ne contribuent pas aux métadonnées. Ils effectuent leur vérification et produisent une erreur de validation, mais ne publient pas de valeur réactive que les templates pourraient lire.

## Quand utiliser des métadonnées personnalisées {#when-to-use-custom-metadata}

Quand vous avez besoin de données réactives attachées à un champ précis et que les signals d'état intégrés comme `valid()`, `disabled()` et `touched()` ne les couvrent pas, utilisez des **métadonnées personnalisées**.

Voici quelques exemples :

- **Une configuration attachée à des schémas de champ réutilisables.** Un symbole monétaire sur un champ de prix, pour que tout template ou contrôle personnalisé qui affiche le champ puisse l'afficher. Ou `MIN_DATE` et `MAX_DATE` sur un champ de date, lus par un sélecteur de plage réutilisable.
- **Des valeurs analysées partagées entre les règles d'un même champ.** Un numéro de téléphone analysé une seule fois au format E.164, pour qu'un validateur de format et une vérification d'unicité lisent tous deux la même forme canonique sans nouvelle analyse.
- **Des indications d'affichage construites à partir de l'état du champ.** Un niveau de gravité (`'info' | 'warning' | 'error'`) que l'interface associe à des badges et des icônes, ou un message d'aide contextuel qui change selon ce que l'utilisateur a saisi et les autres champs remplis.

Si vous vous surprenez à maintenir une `Map<fieldKey, value>` parallèle à votre formulaire pour suivre une information par champ, c'est le signe que les métadonnées sont le bon outil. Les métadonnées restent à côté du schéma, restent réactives et suivent le cycle de vie du champ.

## Créer une clé de métadonnées {#creating-a-metadata-key}

Pour créer une clé personnalisée, appelez `createMetadataKey<TWrite>()`. Le paramètre de type décrit la valeur que vos règles de schéma apporteront.

```ts
import {createMetadataKey} from '@angular/forms/signals';

export const USERNAME_HELP = createMetadataKey<string>();
```

Chaque appel à `createMetadataKey()` crée une nouvelle clé unique. Deux appels avec les mêmes paramètres de type produisent malgré tout deux clés distinctes : définissez donc chaque clé une seule fois au niveau du module et importez-la partout où vous en avez besoin.

NOTE: Une clé créée sans reducer applique par défaut une sémantique de « remplacement » (override) : si plusieurs règles définissent la clé, la dernière contribution l'emporte.

## Définir des valeurs depuis un schéma {#setting-values-from-a-schema}

Pour enregistrer une valeur pour la clé sur un champ précis, utilisez `metadata(path, key, logic)` dans une fonction de schéma.

```angular-ts
import {Component, computed, signal} from '@angular/core';
import {form, metadata, FormField} from '@angular/forms/signals';
import {USERNAME_HELP} from './metadata-keys';

@Component({
  selector: 'app-registration',
  imports: [FormField],
  template: `
    <form>
      <label>
        Username
        <input [formField]="registrationForm.username" />
      </label>
      <p class="help">{{ usernameHelp() }}</p>
    </form>
  `,
})
export class Registration {
  registrationModel = signal({username: ''});

  registrationForm = form(this.registrationModel, (path) => {
    metadata(path.username, USERNAME_HELP, ({value}) => {
      const username = value();
      if (username.length === 0) {
        return 'Choose a unique username between 3 and 20 characters.';
      }
      if (username.length < 3) {
        return 'Keep typing, usernames are at least 3 characters.';
      }
      if (username.length > 20) {
        return 'Usernames are at most 20 characters.';
      }
      return 'Looks good.';
    });
  });

  usernameHelp = computed(() => this.registrationForm.username().metadata(USERNAME_HELP)?.() ?? '');
}
```

La fonction de logique reçoit le contexte du champ, qui expose `value`, un signal de la valeur courante du champ, `state`, le `FieldState` du champ, et des méthodes comme `valueOf(path)` et `stateOf(path)` pour lire d'autres champs du même formulaire. Tout signal lu par la fonction devient une dépendance réactive : quand `value()` change, les métadonnées sont recalculées et tout template qui lit la clé est mis à jour.

## Lire les métadonnées d'un champ {#reading-metadata-from-a-field}

`hasMetadata(key)` renvoie `true` si une règle de schéma a enregistré la clé sur ce champ. `state.metadata(key)` renvoie `undefined` quand aucune règle n'a enregistré la clé, et sinon un signal de la valeur réduite courante.

```ts
registrationForm.username().hasMetadata(USERNAME_HELP); // true if any metadata() rule registered this key
```

La forme de cette valeur interne (si elle peut elle-même valoir `undefined`, quel type elle contient) dépend du reducer de la clé. Les reducers sont présentés dans la section suivante.

Quand la clé peut ne pas être enregistrée, protégez la lecture avec `hasMetadata()` :

```angular-html
@if (registrationForm.username().hasMetadata(USERNAME_HELP)) {
  <p class="help">{{ registrationForm.username().metadata(USERNAME_HELP)!() }}</p>
}
```

Quand vous savez qu'une règle enregistre toujours la clé (parce que le schéma du même fichier le fait), vous pouvez vous passer de la vérification `hasMetadata()` et utiliser le chaînage optionnel, plus compact :

```ts
const message = registrationForm.username().metadata(USERNAME_HELP)?.();
// message: string | undefined
```

Ou, quand l'enregistrement par la règle est garanti, supprimez le chaînage optionnel et utilisez une assertion :

```ts
const message = registrationForm.username().metadata(USERNAME_HELP)!();
// message: string | undefined (still, because the inner value may be undefined)
```

L'exemple de composant ci-dessus utilise le chaînage optionnel dans un `computed()` pour que le template se lie à une simple `string`, avec une valeur de repli vide pour le premier rendu.

C'est toute l'API pour un contributeur unique. La section suivante explique ce qui se passe quand plusieurs règles de schéma contribuent à la même clé, et comment combiner ces contributions avec des reducers.

## Combiner les contributions avec des reducers {#combining-contributions-with-reducers}

La sémantique de remplacement fonctionne quand une seule règle contribue à une clé sur un champ donné. Dès que deux règles y contribuent, la première valeur est ignorée sans avertissement :

```ts
const HELP = createMetadataKey<string>();

form(model, (path) => {
  metadata(path.username, HELP, () => 'Choose something unique across the system.');
  metadata(path.username, HELP, () => 'Usernames are 3 to 20 characters.');
});
```

Une fois les deux règles exécutées, `state.metadata(HELP)!()` ne renvoie que le second message. Ce n'est presque jamais ce que vous voulez. Les contributions proviennent souvent de sources différentes : deux schémas composés avec `apply()` qui attachent chacun un texte d'aide, ou plusieurs règles de validation qui apportent chacune une indication.

Pour combiner les contributions, passez un reducer à `createMetadataKey()`. Un reducer décrit comment agréger les valeurs individuelles en un résultat accumulé :

```ts
import {createMetadataKey, MetadataReducer, form, metadata} from '@angular/forms/signals';
import {model} from '@angular/core';

const HELP = createMetadataKey<string, string[]>(MetadataReducer.list());

form(model, (path) => {
  metadata(path.username, HELP, () => 'Choose something unique across the system.');
  metadata(path.username, HELP, () => 'Usernames are 3 to 20 characters.');
});

// state.metadata(HELP)!() === [
//   'Choose something unique across the system.',
//   'Usernames are 3 to 20 characters.',
// ]
```

Remarquez les deux paramètres de type de `createMetadataKey<TWrite, TAcc>` : le premier est le type apporté par chaque règle, le second le type produit par le reducer. Avec `list()`, les règles apportent une `string` et le champ renvoie un `string[]`.

### Reducers intégrés {#built-in-reducers}

Angular fournit six reducers intégrés dans le namespace [`MetadataReducer`](api/forms/signals/MetadataReducer). `override()` existe sous deux formes à la sémantique légèrement différente, présentées séparément dans le tableau :

| Reducer        | Type de l'accumulateur | Ce qu'il fait                                                          | Valeur initiale |
| -------------- | --------------------- | ---------------------------------------------------------------------- | ------------- |
| `list<T>()`    | `T[]`                 | Accepte des contributions `T \| undefined` ; ajoute les valeurs autres que `undefined` | `[]`          |
| `or()`         | `boolean`             | `true` si au moins une contribution vaut `true`                        | `false`       |
| `and()`        | `boolean`             | `true` uniquement si toutes les contributions valent `true`            | `true`        |
| `min()`        | `number \| undefined` | Conserve le plus petit nombre apporté                                  | `undefined`   |
| `max()`        | `number \| undefined` | Conserve le plus grand nombre apporté                                  | `undefined`   |
| `override()`   | `T \| undefined`      | La dernière contribution remplace la précédente (comportement par défaut) | `undefined`   |
| `override(fn)` | `T`                   | Identique, mais avec une valeur initiale fournie                       | `fn()`        |

`list()` est le seul reducer intégré dont le type d'élément est plus large que le type des éléments de son accumulateur. Une règle peut apporter `undefined` et le reducer l'ignore sans avertissement. C'est ainsi que la clé intégrée `PATTERN` gère les règles `pattern()` dynamiques dont la fonction de logique renvoie `undefined` : la contribution `undefined` est ignorée au lieu d'être incluse dans la liste finale d'expressions régulières.

### Comment les clés des validateurs intégrés utilisent les reducers {#how-built-in-validator-keys-use-reducers}

Bien que `MetadataReducer.min()` et `MetadataReducer.max()` soient des reducers, vous serez peut-être surpris d'apprendre que ce ne sont pas des validateurs. `MetadataReducer.min()` choisit la plus petite contribution à une clé, tandis que le validateur `min()` impose une borne inférieure à la valeur d'un champ. Ils partagent un nom, mais répondent à des problèmes différents.

Les clés de contrainte intégrées choisissent leur reducer en fonction de ce que signifie « le plus strict » pour la contrainte, ce qui est souvent l'inverse de ce que suggère le nom de la clé :

| Clé          | Reducer          | Raisonnement                                                                                                                           |
| ------------ | ---------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `REQUIRED`   | `or()`           | Si une règle `required()` est évaluée à `true`, le champ est obligatoire.                                                              |
| `MIN_NUMBER` | `max()`          | Une contrainte de nombre minimum est la plus stricte quand elle est la plus grande. Si une règle exige `>= 5` et une autre `>= 10`, le minimum effectif est `10`. |
| `MIN_DATE`   | `max()`          | Même logique que `MIN_NUMBER` : la date requise la plus tardive l'emporte.                                                             |
| `MAX_NUMBER` | `min()`          | Une contrainte de nombre maximum est la plus stricte quand elle est la plus petite. Si une règle plafonne à `100` et une autre à `50`, le maximum effectif est `50`. |
| `MAX_DATE`   | `min()`          | Même logique que `MAX_NUMBER` : la date autorisée la plus précoce l'emporte.                                                           |
| `MIN_LENGTH` | `max()`          | Même logique que `MIN_NUMBER` : la longueur requise la plus grande l'emporte.                                                          |
| `MAX_LENGTH` | `min()`          | Même logique que `MAX_NUMBER` : la longueur autorisée la plus courte l'emporte.                                                        |
| `PATTERN`    | `list<RegExp>()` | Chaque appel à `pattern()` apporte une expression régulière ; la valeur doit toutes les respecter.                                     |

`MIN` et `MAX` sont des clés de sélection. Elles pointent vers la clé concrète qui correspond au type de valeur du champ, comme `MIN_NUMBER` pour `min()` et `MIN_DATE` pour `minDate()`. C'est pourquoi `field().min()` et `field().max()` fonctionnent pour les champs numériques comme pour les champs de date.

C'est grâce à cette règle du « plus strict l'emporte » qu'appeler `min(path.age, 18)` et `min(path.age, 21)` dans deux schémas composés fonctionne correctement. Chaque appel enregistre son propre validateur, qui impose sa borne spécifique (une valeur inférieure à l'une ou l'autre borne échoue donc à la validation). Par ailleurs, chaque appel contribue à la clé `MIN_NUMBER`, et `state.min!()` indique la valeur agrégée (`21`) pour que l'interface et les contrôles personnalisés puissent lire le minimum effectif.

### Écrire un reducer personnalisé {#writing-a-custom-reducer}

Pour écrire votre propre reducer, implémentez un objet conforme à l'interface `MetadataReducer<TAcc, TItem>` :

```ts
interface MetadataReducer<TAcc, TItem> {
  reduce: (acc: TAcc, item: TItem) => TAcc;
  getInitial: () => TAcc;
}
```

Vous pouvez définir un reducer personnalisé quand aucun des reducers intégrés ne correspond à la sémantique dont vous avez besoin. Par exemple, une clé `SEVERITY` qui conserve le niveau le plus grave apporté par une règle :

```ts
import {createMetadataKey, type MetadataReducer} from '@angular/forms/signals';

type Severity = 'info' | 'warning' | 'error';

const SEVERITY_RANK: Record<Severity, number> = {info: 0, warning: 1, error: 2};

const maxSeverity: MetadataReducer<Severity | undefined, Severity> = {
  reduce(acc, item) {
    if (acc === undefined) return item;
    return SEVERITY_RANK[item] > SEVERITY_RANK[acc] ? item : acc;
  },
  getInitial: () => undefined,
};

export const SEVERITY = createMetadataKey<Severity, Severity | undefined>(maxSeverity);
```

N'importe quel nombre de règles peut désormais apporter une gravité, et le champ indique la plus élevée :

```ts
form(model, (path) => {
  metadata(path.password, SEVERITY, () => 'info');
  metadata(path.password, SEVERITY, ({value}) => (value().length < 12 ? 'warning' : 'info'));
  metadata(path.password, SEVERITY, ({value}) =>
    /password|1234/i.test(value()) ? 'error' : 'info',
  );
});
```

Le reducer s'exécute dès que les signals d'une contribution changent : `state.metadata(SEVERITY)!()` reste donc synchronisé avec le pire cas courant parmi toutes les règles.

TIP: Gardez vos reducers purs : `reduce()` ne doit dépendre que de ses deux arguments, et `getInitial()` doit renvoyer la même valeur à chaque appel. Les reducers s'exécutent dans un calcul réactif qui se réexécute dès que les signals d'une contribution changent ; des reducers impurs produisent donc des métadonnées incohérentes.

## Attacher des objets dotés d'un cycle de vie avec les métadonnées gérées {#attaching-lifecycle-aware-objects-with-managed-metadata}

Les métadonnées gérées stockent sur un champ un objet doté d'un cycle de vie plutôt qu'une valeur réactive. Utilisez-les pour des objets propres à chaque champ, comme une `resource()` qui récupère des données externes, un `effect()` qui se synchronise avec un système extérieur, ou une référence de service limitée à un seul champ.

### Créer une clé gérée {#creating-a-managed-key}

Pour définir une clé gérée, appelez `createManagedMetadataKey<TRead, TWrite>(create)`. La fonction `create` que vous passez produit la valeur que contient la clé.

```ts
import {Signal} from '@angular/core';
import {httpResource} from '@angular/common/http';
import {createManagedMetadataKey} from '@angular/forms/signals';

export interface UrlPreview {
  title: string;
  description?: string;
  image?: string;
}

export const URL_PREVIEW = createManagedMetadataKey((_state, url: Signal<string | undefined>) => {
  return httpResource<UrlPreview>(() => {
    const currentUrl = url();
    return currentUrl ? {url: '/api/url-preview', params: {url: currentUrl}} : undefined;
  });
});
```

La fonction `create` reçoit le `FieldState` du champ et un `Signal<TAcc>` des données apportées par les règles `metadata()` pour cette clé, et renvoie l'objet qui doit vivre sur le champ. La valeur renvoyée est stockée telle quelle : contrairement aux clés non gérées, le framework ne l'enveloppe pas dans un `computed()`.

`create` s'exécute une seule fois, à la construction du champ, dans le contexte d'injection du champ. Cela vous permet d'appeler `inject()`, `resource()` et `effect()` dans `create`, et lie le nettoyage au cycle de vie du champ : quand le champ est détruit, Angular détruit le contexte d'injection, et toute `resource()`, tout `effect()` ou tout callback `DestroyRef` que vous y avez enregistré est nettoyé automatiquement.

Comme `create` n'est pas réactive en elle-même, tout comportement qui doit réagir aux changements de signals doit vivre dans un `effect()`, une `resource()` ou une `httpResource()` mis en place lors de cet appel initial. `URL_PREVIEW` illustre ce pattern : l'`httpResource()` lit le signal de l'URL dans sa fonction de requête, si bien que la requête est relancée dès que le signal change. La règle de schéma (`metadata(path.url, URL_PREVIEW, ({value}) => value())`) décide des données à fournir ; la clé gérée décide de ce qu'il faut en faire.

### Utiliser une clé gérée dans un formulaire {#using-a-managed-key-in-a-form}

Pour utiliser une clé gérée dans un formulaire, enregistrez une règle `metadata()` pour la clé, puis lisez l'objet renvoyé depuis l'état du champ.

```angular-ts
import {Component, computed, signal} from '@angular/core';
import {applyEach, form, metadata, FormField} from '@angular/forms/signals';
import {URL_PREVIEW} from './url-preview';

@Component({
  selector: 'app-link-editor',
  imports: [FormField],
  template: `
    <form>
      @for (link of linksForm.links; track link) {
        <fieldset>
          <label>
            URL
            <input [formField]="link.url" />
          </label>
          <!-- Read the URL_PREVIEW key for this link's url field; the result is the resource its create function produced -->
          @let preview = link.url().metadata(URL_PREVIEW);
          @if (preview?.isLoading()) {
            <p>Loading preview...</p>
          } @else if (preview?.hasValue() && preview.value(); as data) {
            <article class="preview">
              <h3>{{ data.title }}</h3>
              @if (data.description) {
                <p>{{ data.description }}</p>
              }
            </article>
          } @else if (preview?.error()) {
            <p class="error">Could not load preview.</p>
          }
        </fieldset>
      }
      <button type="button" (click)="addLink()">Add link</button>
    </form>
  `,
})
export class LinkEditor {
  URL_PREVIEW = URL_PREVIEW;

  linksModel = signal({links: [{url: ''}]});

  linksForm = form(this.linksModel, (path) => {
    // Register the URL_PREVIEW key on each link's url field.
    // applyEach runs the schema per item, so create() runs once per link
    // and each link gets its own resource.
    applyEach(path.links, (itemPath) => {
      metadata(itemPath.url, URL_PREVIEW, ({value}) => value());
    });
  });

  addLink() {
    this.linksForm.links().value.update((links) => [...links, {url: ''}]);
  }
}
```

Chaque élément du tableau obtient sa propre resource `URL_PREVIEW`, car `applyEach` enregistre les règles de schéma pour chaque élément indépendamment. Quand l'utilisateur ajoute un lien, `create` s'exécute pour le champ du nouvel élément. Quand un lien est supprimé (non illustré ici, mais c'est un pattern courant), le framework détruit l'injector de ce champ ainsi que la resource.

## Étapes suivantes {#next-steps}

Retenez que les métadonnées existent pour que des données réactives accompagnent le champ à travers la composition de schémas, s'accumulent d'une règle à l'autre et soient détruites avec le cycle de vie du champ. Elles s'appuient sur le même système que les validateurs intégrés d'Angular et peuvent être adaptées à vos propres cas d'usage.

Pour la documentation détaillée de l'API, consultez :

- [`createMetadataKey()`](api/forms/signals/createMetadataKey) - Définir une clé de métadonnées avec un reducer facultatif
- [`createManagedMetadataKey()`](api/forms/signals/createManagedMetadataKey) - Définir une clé de métadonnées dotée d'un cycle de vie
- [`metadata()`](api/forms/signals/metadata) - Apporter une valeur à une clé de métadonnées dans un schéma
- [`MetadataReducer`](api/forms/signals/MetadataReducer) - Reducers intégrés pour combiner les contributions

Pour d'autres guides sur les Signal Forms, consultez :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/form-logic" title="Ajouter de la logique de formulaire" />
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/async-operations" title="Opérations asynchrones" />
</docs-pill-row>
