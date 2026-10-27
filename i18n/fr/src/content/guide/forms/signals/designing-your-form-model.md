<!-- source: e4389576f2b2a79562dd3b64dbb3f16553cf9cdc -->
# Concevoir votre model de formulaire

Les Signal Forms suivent une approche pilotée par le model : l'état et la structure du formulaire sont dérivés directement du model que vous fournissez. Comme il sert de fondation à l'ensemble du formulaire, il est important de partir d'un model de formulaire bien conçu. Ce guide présente les bonnes pratiques de conception des models de formulaire.

## Model de formulaire ou modèle de domaine {#form-model-vs-domain-model}

Les formulaires servent à recueillir les saisies de l'utilisateur. Votre application dispose probablement d'un modèle de domaine qui représente ces saisies d'une manière optimisée pour la logique métier ou le stockage. Cependant, cette représentation est souvent _différente_ de la façon dont nous voulons modéliser les données dans notre formulaire.

Le model de formulaire représente les saisies brutes de l'utilisateur telles qu'elles apparaissent dans l'interface. Par exemple, dans un formulaire, vous pouvez demander à l'utilisateur de choisir une date et un créneau horaire pour un rendez-vous dans deux champs séparés, même si votre modèle de domaine les représente sous la forme d'un unique objet JavaScript `Date`.

```ts
interface AppointmentFormModel {
  name: string; // Appointment owner's name
  date: Date; // Appointment date (carries only date information, time component is unused)
  time: string; // Selected time as a string
}

interface AppointmentDomainModel {
  name: string; // Appointment owner's name
  time: Date; // Appointment time (carries both date and time information)
}
```

Les formulaires doivent utiliser un model de formulaire adapté à l'expérience de saisie, plutôt que de simplement réutiliser le modèle de domaine.

## Bonnes pratiques pour le model de formulaire {#form-model-best-practices}

### Utiliser des types précis {#use-specific-types}

Définissez toujours des interfaces ou des types pour vos models, comme le montre [Utiliser des types TypeScript](/guide/forms/signals/models#using-typescript-types). Les types explicites améliorent l'IntelliSense, détectent les erreurs à la compilation et documentent les données que contient le formulaire.

### Initialiser tous les champs {#initialize-all-fields}

Fournissez une valeur initiale pour chaque champ de votre model :

```ts {prefer, header: 'All fields initialized'}
const taskModel = signal({
  title: '',
  description: '',
  priority: 'medium',
  completed: false,
});
```

```ts {avoid, header: 'Partial initialization'}
const taskModel = signal({
  title: '',
  // Missing description, priority, completed
});
```

Sans valeur initiale, ces champs n'existent pas dans l'arbre de champs, ce qui les rend inaccessibles aux interactions du formulaire.

### Garder des models ciblés {#keep-models-focused}

Chaque model doit représenter un seul formulaire ou un ensemble cohérent de données liées :

```ts {prefer, header: 'Focused on a single purpose'}
const loginModel = signal({
  email: '',
  password: '',
});
```

```ts {avoid, header: 'Mixing unrelated concerns'}
const appModel = signal({
  // Login data
  email: '',
  password: '',
  // User preferences
  theme: 'light',
  language: 'en',
  // Shopping cart
  cartItems: [],
});
```

Des models distincts pour des préoccupations différentes rendent les formulaires plus faciles à comprendre et à réutiliser. Créez plusieurs formulaires si vous gérez des ensembles de données distincts.

### Tenir compte des besoins de validation {#consider-validation-requirements}

Concevez vos models en pensant à la validation. Regroupez les champs qui se valident ensemble :

```ts {prefer, header: 'Related fields grouped for comparison'}
// Password fields grouped for comparison
interface PasswordChangeData {
  currentPassword: string;
  newPassword: string;
  confirmPassword: string;
}
```

Cette structure rend la validation inter-champs (comme vérifier que `newPassword` correspond à `confirmPassword`) plus naturelle.

### Faire correspondre les types de données aux contrôles de l'interface {#match-data-types-to-ui-controls}

Les propriétés de votre model de formulaire doivent correspondre aux types de données attendus par vos contrôles d'interface.

Prenons par exemple un formulaire de commande de boissons avec un champ `size` (pack de 6, 12 ou 24) et un champ `quantity`. L'interface utilise une liste déroulante (`<select>`) pour la taille et un champ numérique (`<input type="number">`) pour la quantité.

Même si les options de taille ont l'air numériques, les éléments `<select>` fonctionnent avec des valeurs de type chaîne : `size` doit donc être modélisé comme une chaîne. Un `<input type="number">`, en revanche, fonctionne bien avec des nombres : `quantity` peut donc être modélisé comme un nombre.

```ts {prefer, header: 'Appropriate data types for the bound UI controls'}
interface BeverageOrderFormModel {
  size: string; // Bound to: <select> (option values: "6", "12", "24")
  quantity: number; // Bound to: <input type="number">
}
```

### Éviter `undefined` {#avoid-undefined}

Un model de formulaire ne doit pas contenir de valeurs ou de propriétés `undefined`. Dans les Signal Forms, la structure du formulaire est dérivée de la structure du model, et `undefined` signifie l'_absence de champ_, et non un champ dont la valeur est vide. Vous devez donc aussi éviter les champs optionnels (par exemple `{property?: string}`), car ils autorisent implicitement `undefined`.

Pour représenter une propriété à la valeur vide dans votre model de formulaire, utilisez une valeur que le contrôle d'interface comprend comme « vide » (par exemple `""` pour un `<input type="text">`). Si vous concevez un contrôle d'interface personnalisé, `null` est souvent une bonne valeur pour signifier « vide ».

```ts {prefer, header: 'Appropriate empty values'}
interface UserFormModel {
  name: string; // Bound to <input type="text">
  birthday: Date | null; // Bound to <input type="date">
}

// Initialize our form with empty values.
form(signal({name: '', birthday: null}));
```

### Éviter les models à structure dynamique {#avoid-models-with-dynamic-structure}

Un model de formulaire a une structure dynamique s'il change de forme (si les propriétés de l'objet changent) en fonction de sa valeur. C'est le cas quand le type du model autorise des valeurs de formes différentes, comme une union de types objet aux propriétés différentes, ou une union d'un objet et d'un type primitif. Les sections suivantes examinent quelques scénarios courants où les models à structure dynamique peuvent sembler séduisants, mais s'avèrent finalement problématiques.

#### Valeur vide pour un objet complexe {#empty-value-for-a-complex-object}

On utilise souvent des formulaires pour demander à l'utilisateur de saisir des données entièrement nouvelles, plutôt que de modifier des données existantes du système. Un formulaire de création de compte en est un bon exemple. Nous pourrions le modéliser avec le model de formulaire suivant.

```ts
interface CreateAccountFormModel {
  name: {
    first: string;
    last: string;
  };
  username: string;
}
```

À la création du formulaire, un dilemme se pose : quelle doit être la valeur initiale du model ? Il peut être tentant de créer un `form<CreateAccountFormModel | null>()`, puisque l'utilisateur n'a encore rien saisi.

```ts {avoid, header: 'Using null as empty value for complex object'}
createAccountForm = form<CreateAccountFormModel | null>(signal(/* what goes here, null? */));
```

Cependant, il est important de garder à l'esprit que les Signal Forms sont _pilotés par le model_. Si notre model vaut `null` et que `null` n'a pas de propriété `name` ni `username`, notre formulaire n'aura pas non plus ces sous-champs. Ce que nous voulons réellement, c'est une instance de `CreateAccountFormModel` dont tous les champs feuilles ont une valeur vide.

```ts {prefer, header: 'Same shape value with empty values for properties'}
createAccountForm = form<CreateAccountFormModel>(
  signal({
    name: {
      first: '',
      last: '',
    },
    username: '',
  }),
);
```

Avec cette représentation, tous les sous-champs dont nous avons besoin existent désormais, et nous pouvons les lier avec la directive `[formField]` dans notre template.

```html
First: <input [formField]="createAccountForm.name.first" /> Last:
<input [formField]="createAccountForm.name.last" /> Username:
<input [formField]="createAccountForm.username" />
```

#### Champs masqués ou indisponibles sous condition {#fields-that-are-conditionally-hidden-or-unavailable}

Les formulaires ne sont pas toujours linéaires. Vous devez souvent créer des parcours conditionnels en fonction des saisies précédentes de l'utilisateur. Un formulaire qui propose différents moyens de paiement en est un exemple. Commençons par imaginer à quoi pourrait ressembler l'interface d'un tel formulaire.

```html
Name: <input type="text" />

<section>
  <h2>Payment Info</h2>
  <input type="radio" /> Credit Card @if (/* credit card selected */) {
  <section>
    Card Number <input type="text" /> Security Code <input type="text" /> Expiration
    <input type="text" />
  </section>
  }
  <input type="radio" /> Bank Account @if (/* bank account selected */) {
  <section>Account Number <input type="text" /> Routing Number <input type="text" /></section>
  }
</section>
```

La meilleure façon de gérer ce cas est d'utiliser un model de formulaire à structure statique, qui inclut les champs de _tous_ les moyens de paiement possibles. Dans notre schéma, nous pouvons masquer ou désactiver les champs qui ne sont pas disponibles pour le moment.

```ts {prefer, header: 'Static structure model'}
interface BillPayFormModel {
  name: string;
  method: {
    type: string;
    card: {
      cardNumber: string;
      securityCode: string;
      expiration: string;
    };
    bank: {
      accountNumber: string;
      routingNumber: string;
    };
  };
}

const billPaySchema = schema<BillPayFormModel>((billPay) => {
  // Hide credit card details when user has selected a method other than credit card.
  hidden(billPay.method.card, {when: ({valueOf}) => valueOf(billPay.method.type) !== 'card'});
  // Hide bank account details when user has selected a method other than bank account.
  hidden(billPay.method.bank, {when: ({valueOf}) => valueOf(billPay.method.type) !== 'bank'});
});
```

Avec ce model, les objets `card` et `bank` sont toujours présents dans l'état du formulaire. Quand l'utilisateur change de moyen de paiement, nous mettons uniquement à jour la propriété `type`. Les données qu'il a saisies dans les champs de la carte restent stockées en toute sécurité dans l'objet `card`, prêtes à être réaffichées s'il revient en arrière.

À l'inverse, un model de formulaire dynamique peut sembler, de prime abord, bien adapté à ce cas d'usage. Après tout, nous n'avons pas besoin des champs de numéro de compte et de routing number si l'utilisateur a choisi « Carte bancaire ». Il peut être tentant de le modéliser comme une union discriminée :

```ts {avoid, header: 'Dynamic structure model'}
interface BillPayFormModel {
  name: string;
  method:
    | {
        type: 'card';
        cardNumber: string;
        securityCode: string;
        expiration: string;
      }
    | {
        type: 'bank';
        accountNumber: string;
        routingNumber: string;
      };
}
```

Cependant, voyez ce qui se passerait dans le scénario suivant :

1. L'utilisateur renseigne son nom et les informations de sa carte bancaire
2. Il s'apprête à soumettre, mais remarque au dernier moment les frais de service.
3. Il bascule plutôt sur l'option compte bancaire, en se disant qu'autant éviter ces frais.
4. Au moment de saisir ses informations bancaires, il hésite : il ne voudrait pas qu'elles se retrouvent dans une fuite de données.
5. Il revient à l'option carte bancaire, mais constate que toutes les informations qu'il vient de saisir ont disparu !

Cela illustre un autre problème des models de formulaire à structure dynamique : ils peuvent entraîner une perte de données. Un tel model suppose qu'une fois un champ masqué, les informations qu'il contient ne seront plus jamais nécessaires. Il remplace les informations de la carte bancaire par les informations bancaires, et n'a aucun moyen de récupérer les informations de la carte.

#### Exceptions {#exceptions}

Bien qu'une structure statique soit généralement préférable, il existe des scénarios précis où une structure dynamique est nécessaire et prise en charge.

##### Tableaux {#arrays}

Les tableaux sont l'exception la plus courante. Les formulaires doivent souvent recueillir un nombre variable d'éléments, comme une liste de numéros de téléphone, de participants ou de lignes d'une commande.

```ts
interface SendEmailFormModel {
  subject: string;
  recipientEmails: string[];
}
```

Dans ce cas, le tableau `recipientEmails` s'agrandit et se réduit au fil des interactions de l'utilisateur avec le formulaire. Si la longueur du tableau est dynamique, la structure de chaque élément doit rester cohérente (chaque élément doit avoir la même forme).

##### Champs traités de manière atomique par le contrôle d'interface {#fields-that-are-treated-atomically-by-the-ui-control}

Une structure dynamique est également acceptable quand un objet complexe est traité comme une valeur unique et atomique par le contrôle d'interface, c'est-à-dire quand le contrôle ne cherche pas à se lier à ses sous-champs ni à y accéder individuellement. Dans ce scénario, le contrôle met à jour la valeur en remplaçant l'objet entier d'un coup, plutôt qu'en modifiant ses propriétés internes. Comme la structure du formulaire n'a pas d'importance dans ce cas, il est acceptable qu'elle soit dynamique.

Prenons par exemple un formulaire de profil utilisateur qui inclut un champ `location`. La localisation est choisie à l'aide d'un widget complexe de « sélection de lieu » (une carte, ou une liste déroulante avec recherche) qui renvoie un objet de coordonnées. Lorsque la localisation n'a pas encore été choisie, ou que l'utilisateur préfère ne pas la partager, le sélecteur indique la localisation comme `null`.

```ts {prefer, header: 'Dynamic structure is ok when field is treated as atomic'}
interface Location {
  lat: number;
  lng: number;
}

interface UserProfileFormModel {
  username: string;
  // This property has dynamic structure,
  // but that's ok because the location picker treats this field as atomic.
  location: Location | null;
}
```

Dans le template, nous lions le champ `location` directement à notre contrôle personnalisé :

```html
Username: <input [formField]="userForm.username" /> Location:
<location-picker [formField]="userForm.location"></location-picker>
```

Ici, `<location-picker>` consomme et produit l'objet `Location` entier (ou `null`), et n'accède ni à `userForm.location.lat` ni à `userForm.location.lng`. `location` peut donc avoir une forme dynamique en toute sécurité, sans enfreindre les principes des formulaires pilotés par le model.

## Convertir entre model de formulaire et modèle de domaine {#translating-between-form-model-and-domain-model}

Puisque le model de formulaire et le modèle de domaine représentent le même concept différemment, nous avons besoin d'un moyen de passer de l'une de ces représentations à l'autre. Quand nous voulons présenter à l'utilisateur, dans un formulaire, des données existantes du système, nous devons les transformer de la représentation du modèle de domaine vers celle du model de formulaire. À l'inverse, quand nous voulons enregistrer les modifications de l'utilisateur, nous devons transformer les données de la représentation du model de formulaire vers celle du modèle de domaine.

Imaginons que nous disposions d'un modèle de domaine et d'un model de formulaire, et que nous ayons écrit des fonctions de conversion entre les deux.

```ts
interface MyDomainModel { ... }

interface MyFormModel { ... }

// Instance of `MyFormModel` populated with empty input (e.g. `''` for string inputs, etc.)
const EMPTY_MY_FORM_MODEL: MyFormModel = { ... };

function domainModelToFormModel(domainModel: MyDomainModel): MyFormModel { ... }

function formModelToDomainModel(formModel: MyFormModel): MyDomainModel { ... }
```

### Du modèle de domaine au model de formulaire {#domain-model-to-form-model}

Quand nous créons un formulaire pour modifier un modèle de domaine existant du système, nous recevons généralement ce modèle de domaine soit comme `input()` de notre composant de formulaire, soit depuis un backend (par exemple via une resource). Dans les deux cas, `linkedSignal` offre un excellent moyen d'appliquer notre transformation.

Lorsque nous recevons le modèle de domaine comme `input()`, nous pouvons utiliser `linkedSignal` pour créer un model de formulaire modifiable à partir du signal de l'input.

```ts {prefer, header: 'Use linkedSignal to convert domain model to form model'}
@Component(...)
class MyForm {
  // The domain model to initialize the form with, if not given we start with an empty form.
  readonly domainModel = input<MyDomainModel>();

  private readonly formModel = linkedSignal({
    // Linked signal based on the domain model
    source: this.domainModel,
    // If domain model is defined convert it to a form model, otherwise use an empty form model.
    computation: (domainModel) => domainModel
      ? domainModelToFormModel(domainModel)
      : EMPTY_MY_FORM_MODEL
  });

  protected readonly myForm = form(this.formModel);
}
```

De même, lorsque nous recevons le modèle de domaine depuis le backend via une resource, nous pouvons créer un `linkedSignal` basé sur sa valeur pour obtenir notre `formModel`. Dans ce scénario, la récupération du modèle de domaine peut prendre un certain temps, et nous devons désactiver le formulaire jusqu'au chargement des données.

```ts {prefer, header: 'Disable or hide the form when data is unavailable'}
@Component(...)
class MyForm {
  // Fetch the domain model from the backend.
  readonly domainModelResource: ResourceRef<MyDomainModel | undefined> = httpResource(...);

  private readonly formModel = linkedSignal({
    // Linked signal based on the domain model resource
    source: this.domainModelResource.value,
    // Convert the domain model once it loads, use an empty form model while loading.
    computation: (domainModel) => domainModel
      ? domainModelToFormModel(domainModel)
      : EMPTY_MY_FORM_MODEL
  });

  protected readonly myForm = form(this.formModel, (root) => {
    // Disable the entire form when the resource is loading.
    disabled(root, {when: () => this.domainModelResource.isLoading()});
  });
}
```

Les exemples ci-dessus montrent une dérivation pure du model de formulaire, directement à partir du modèle de domaine. Dans certains cas, cependant, vous pouvez vouloir effectuer une comparaison plus avancée entre la nouvelle valeur du modèle de domaine et les valeurs précédentes du modèle de domaine et du model de formulaire. Vous pouvez l'implémenter à partir de l'[état précédent](/guide/signals/linked-signal#accounting-for-previous-state) de `linkedSignal`.

### Du model de formulaire au modèle de domaine {#form-model-to-domain-model}

Quand nous sommes prêts à enregistrer les saisies de l'utilisateur dans le système, nous devons les convertir dans la représentation du modèle de domaine. Cela se produit généralement lorsque l'utilisateur soumet le formulaire, ou en continu pendant la saisie dans le cas d'un formulaire à enregistrement automatique.

Pour enregistrer lors de la soumission, nous pouvons gérer la conversion dans la fonction `submit`.

```ts {prefer, header: 'Convert form model to domain model on submit'}
@Component(...)
class MyForm {
  private readonly myDataService = inject(MyDataService);

  protected readonly myForm = form<MyFormModel>(...);

  handleSubmit() {
    submit(this.myForm, async () => {
      await this.myDataService.update(formModelToDomainModel(this.myForm().value()));
    });
  };
}
```

Vous pouvez aussi envoyer directement le model de formulaire au serveur et effectuer la conversion du
model de formulaire vers le modèle de domaine côté serveur.

Pour un enregistrement continu, mettez à jour le modèle de domaine dans un `effect`.

```ts {prefer, header: 'Convert form model to domain model in an effect for auto-saving'}
@Component(...)
class MyForm {
  readonly domainModel = model.required<MyDomainModel>()

  protected readonly myForm = form(...);

  constructor() {
    effect(() => {
      // When the form model changes to a valid value, update the domain model.
      if (this.myForm().valid()) {
        this.domainModel.set(formModelToDomainModel(this.myForm().value()));
      }
    });
  };
}
```

Les exemples ci-dessus montrent une conversion pure du model de formulaire vers le modèle de domaine. Il est toutefois parfaitement acceptable de prendre en compte l'état complet du formulaire, et pas seulement la valeur du model de formulaire. Par exemple, pour économiser des octets, nous pourrions vouloir n'envoyer au serveur que des mises à jour partielles, selon ce que l'utilisateur a modifié. Dans ce cas, notre fonction de conversion pourrait être conçue pour prendre l'état complet du formulaire et renvoyer un modèle de domaine partiel, basé sur les valeurs du formulaire et sur les champs modifiés (dirty).

```ts
type Sparse<T> = T extends object ? {
    [P in keyof T]?: Sparse<T[P]>;
} : T;

function formStateToPartialDomainModel(
  formState: FieldState<MyFormModel>
): Sparse<MyDomainModel> { ... }
```
