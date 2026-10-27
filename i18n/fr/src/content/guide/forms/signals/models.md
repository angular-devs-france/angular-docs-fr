<!-- source: 01c1cbd8c35a66810fa4b989097886fb258f4a50 -->
# Models de formulaire

Les models de formulaire sont le socle des Signal Forms : ils constituent la source de vérité unique pour les données de votre formulaire. Ce guide explique comment créer des models de formulaire, les mettre à jour et les concevoir pour qu'ils restent maintenables.

NOTE: Les models de formulaire sont distincts du signal `model()` d'Angular utilisé pour le two-way binding entre composants. Un model de formulaire est un signal modifiable qui stocke les données du formulaire, tandis que `model()` crée des inputs/outputs pour la communication entre composants parent et enfant.

## Ce que résolvent les models de formulaire {#what-form-models-solve}

Les formulaires nécessitent de gérer des données qui évoluent dans le temps. Sans structure claire, ces données peuvent se retrouver dispersées entre plusieurs propriétés du composant, ce qui rend difficile le suivi des changements, la validation des saisies ou l'envoi des données à un serveur.

Les models de formulaire résolvent ce problème en centralisant les données du formulaire dans un seul signal modifiable. Quand le model est mis à jour, le formulaire reflète automatiquement ces changements. Quand les utilisateurs interagissent avec le formulaire, le model est mis à jour en conséquence.

## Créer des models {#creating-models}

Un model de formulaire est un signal modifiable créé avec la fonction `signal()` d'Angular. Le signal contient un objet qui représente la structure des données de votre formulaire.

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField} from '@angular/forms/signals';

@Component({
  selector: 'app-login',
  imports: [FormField],
  template: `
    <input type="email" [formField]="loginForm.email" />
    <input type="password" [formField]="loginForm.password" />
  `,
})
export class LoginComponent {
  loginModel = signal({
    email: '',
    password: '',
  });

  loginForm = form(this.loginModel);
}
```

La fonction [`form()`](api/forms/signals/form) accepte le signal du model et crée un **arbre de champs** (field tree) : une structure d'objets particulière qui reproduit la forme de votre model. L'arbre de champs est à la fois navigable (accédez aux champs enfants avec la notation pointée, comme `loginForm.email`) et appelable (appelez un champ comme une fonction pour accéder à son état).

La directive `[formField]` lie chaque élément de saisie au champ correspondant dans l'arbre de champs, ce qui permet une synchronisation bidirectionnelle automatique entre l'interface et le model.

### Structures de model prises en charge {#supported-model-structures}

Les Signal Forms construisent l'arbre de champs en parcourant votre model. Les objets et tableaux parcourus (la **couche structurelle**) doivent être des objets et tableaux JavaScript simples. Les valeurs situées aux **feuilles** (positions sans champs imbriqués) sont généralement des primitives (chaînes, nombres, booléens) ou `null`. Les inputs natifs `date`, `month`, `time` et `week` acceptent aussi `Date`, et les contrôles personnalisés peuvent accepter n'importe quel type de valeur qu'ils savent gérer.

```ts {prefer, header: 'Plain structure'}
interface UserFormModel {
  name: string;
  birthday: Date | null;
  preferences: {
    theme: string;
    notifications: boolean;
  };
  tags: string[];
}

const userModel = signal<UserFormModel>({
  name: '',
  birthday: null,
  preferences: {
    theme: 'dark',
    notifications: true,
  },
  tags: [],
});
```

IMPORTANT: Les instances de classe, `Map` et `Set` ne sont **pas prises en charge dans la couche structurelle**, même si TypeScript les accepte. Les Signal Forms ne valident pas la forme du model à l'exécution : le framework accepte donc ces valeurs sans lever d'erreur, puis produit un comportement incorrect de différentes manières selon la forme :

- **Les instances de classe** perdent leur prototype à la première écriture, car les Signal Forms font une copie superficielle des objets parents lors d'une mise à jour. Les méthodes, les getters et les vérifications `instanceof` disparaissent ensuite.
- **Les objets non extensibles ou gelés dans des tableaux** lèvent une erreur quand les Signal Forms leur assignent un symbole de suivi pour préserver l'identité des éléments lors des réordonnancements.
- **`Map` et `Set`** produisent des arbres de champs vides, car les Signal Forms énumèrent les enfants avec `Object.keys`.

Si votre application utilise des classes pour modéliser le domaine, convertissez-les en objets simples à la frontière du formulaire. Consultez [Convertir entre model de formulaire et modèle de domaine](guide/forms/signals/model-design#translating-between-form-model-and-domain-model).

### Utiliser les types TypeScript {#using-typescript-types}

Même si TypeScript infère les types à partir des littéraux d'objet, définir des types explicites améliore la qualité du code et offre une meilleure prise en charge d'IntelliSense.

```ts
interface LoginData {
  email: string;
  password: string;
}

export class LoginComponent {
  loginModel = signal<LoginData>({
    email: '',
    password: '',
  });

  loginForm = form(this.loginModel);
}
```

Avec des types explicites, l'arbre de champs offre une sûreté de typage complète. L'accès à `loginForm.email` est typé `FieldTree<string>`, et toute tentative d'accès à une propriété inexistante provoque une erreur à la compilation.

```ts
// TypeScript knows this is FieldTree<string>
const emailField = loginForm.email;

// TypeScript error: Property 'username' does not exist
const usernameField = loginForm.username;
```

### Initialiser tous les champs {#initializing-all-fields}

Les models de formulaire doivent fournir des valeurs initiales pour tous les champs que vous voulez inclure dans l'arbre de champs.

```ts {prefer}
// Good: All fields initialized
const userModel = signal({
  name: '',
  email: '',
  age: 0,
});
```

```ts {avoid}
// Avoid: Missing initial value
const userModel = signal({
  name: '',
  email: '',
  // age field is not defined - cannot access userForm.age
});
```

Pour les champs optionnels, affectez-leur explicitement une valeur vide ou `null` :

```ts
interface UserData {
  name: string;
  email: string;
  phoneNumber: string | null;
}

const userModel = signal<UserData>({
  name: '',
  email: '',
  phoneNumber: null,
});
```

HELPFUL: Les contrôles texte natifs comme `<input type=text>` et `<textarea>` ne prennent pas en charge `null` : utilisez `''` pour représenter une valeur vide.

Les champs définis à `undefined` sont exclus de l'arbre de champs. Un model `{value: undefined}` se comporte exactement comme `{}` : accéder au champ renvoie `undefined` au lieu d'un `FieldTree`.

## Lire les valeurs du model {#reading-model-values}

Vous pouvez accéder aux valeurs du formulaire de deux manières : directement depuis le signal du model, ou via les champs individuels. Chaque approche répond à un besoin différent.

### Lire depuis le model {#reading-from-the-model}

Accédez au signal du model quand vous avez besoin de l'ensemble des données du formulaire, par exemple lors de la soumission :

```ts
async onSubmit() {
  const formData = this.loginModel();
  console.log(formData.email, formData.password);

  // Send to server
  await this.authService.login(formData);
}
```

Le signal du model renvoie l'objet de données complet, ce qui le rend idéal pour les opérations qui portent sur l'état complet du formulaire.

### Lire depuis l'état du champ {#reading-from-field-state}

Chaque champ de l'arbre de champs est une fonction. Appeler un champ renvoie un objet `FieldState` contenant des signals réactifs pour la valeur du champ, son statut de validation et son état d'interaction.

Accédez à l'état du champ quand vous travaillez avec des champs individuels dans les templates ou dans des calculs réactifs :

```angular-ts
@Component({
  template: `
    <p>Current email: {{ loginForm.email().value() }}</p>
    <p>Password length: {{ passwordLength() }}</p>
  `,
})
export class LoginComponent {
  loginModel = signal({email: '', password: ''});
  loginForm = form(this.loginModel);

  passwordLength = computed(() => {
    return this.loginForm.password().value().length;
  });
}
```

L'état du champ fournit des signals réactifs pour la valeur de chaque champ, ce qui le rend adapté à l'affichage d'informations propres à un champ ou à la création d'un état dérivé.

TIP: L'état du champ comprend bien d'autres signals que `value()`, comme l'état de validation (par exemple valide, invalide, erreurs), le suivi des interactions (par exemple touché, modifié (dirty)) et la visibilité (par exemple masqué, désactivé).

Consultez le [guide de gestion de l'état des champs](guide/forms/signals/field-state-management) pour une présentation complète.

## Mettre à jour les models de formulaire par programmation {#updating-form-models-programmatically}

### Remplacer les models de formulaire avec `set()` {#replacing-form-models-with-set}

Utilisez `set()` sur le model de formulaire pour remplacer la valeur entière :

```ts
loadUserData() {
  this.userModel.set({
    name: 'Alice',
    email: 'alice@example.com',
    age: 30,
  });
}

resetForm() {
  this.userModel.set({
    name: '',
    email: '',
    age: 0,
  });
}
```

Cette approche convient bien pour charger des données depuis une API ou réinitialiser tout le formulaire.

### Mettre à jour un seul champ directement avec `set()` ou `update()` {#update-a-single-field-directly-with-set-or-update}

Utilisez `set()` sur la valeur d'un champ pour mettre à jour directement l'état de ce champ :

```ts
clearEmail() {
  this.userForm.email().value.set('');
}

incrementAge() {
  this.userForm.age().value.update(currentAge => currentAge + 1);
}
```

On parle aussi de « mises à jour au niveau du champ ». Elles se propagent automatiquement au signal du model et maintiennent les deux synchronisés.

### Exemple : charger des données depuis une API {#example-loading-data-from-an-api}

Un pattern courant consiste à récupérer des données et à en alimenter le model :

```ts
export class UserProfileComponent {
  userModel = signal({
    name: '',
    email: '',
    bio: '',
  });

  userForm = form(this.userModel);
  private userService = inject(UserService);

  ngOnInit() {
    this.loadUserProfile();
  }

  async loadUserProfile() {
    const userData = await this.userService.getUserProfile();
    this.userModel.set(userData);
  }
}
```

Les champs du formulaire se mettent à jour automatiquement quand le model change, et affichent les données récupérées sans code supplémentaire.

### Exemple : synchroniser avec un service {#example-synchronizing-to-a-service}

Dans certains cas, un service peut stocker un model de formulaire :

```ts
@Service()
export class FormService {
  private readonly formModel = signal<UserFormModel>({ name: '', birthday: null });
  public readonly model = this.formModel.asReadonly();

  public syncFormModel(value: UserFormModel) {
    this.formModel.set(value);
  }

  // Model set synchronously or asynchronously elsewhere in service
}

@Component(...)
export class UserForm {
  private readonly formService = inject(FormService);

  private readonly formModel = linkedSignal(() => this.formService.model(), {
    set: (value) => this.formService.syncFormModel(value),
  });

  protected readonly form = form(this.formModel);
}
```

La fonction de calcul de `linkedSignal` lit le model de formulaire du service, et la fonction `set`
des options permet de synchroniser le model de formulaire du composant avec celui du service. Les
champs du formulaire se mettent automatiquement à jour quand le model du service change, et mettent
aussi automatiquement à jour le model du service quand les champs modifient le model du composant.

## Two-way binding des données {#two-way-data-binding}

La directive `[formField]` crée une synchronisation bidirectionnelle automatique entre le model, l'état du formulaire et l'interface.

### Circulation des données {#how-data-flows}

Les changements circulent dans les deux sens :

**Saisie utilisateur → Model :**

1. L'utilisateur saisit du texte dans un élément de saisie
2. La directive `[formField]` détecte le changement
3. L'état du champ est mis à jour
4. Le signal du model est mis à jour

**Mise à jour par programmation → Interface :**

1. Le code met à jour le model avec `set()` ou `update()`
2. Le signal du model notifie ses abonnés
3. L'état du champ est mis à jour
4. La directive `[formField]` met à jour l'élément de saisie

Cette synchronisation se fait automatiquement. Vous n'écrivez ni abonnements ni event handlers pour garder le model et l'interface synchronisés.

### Exemple : dans les deux sens {#example-both-directions}

```angular-ts
@Component({
  template: `
    <input type="text" [formField]="userForm.name" />
    <button (click)="setName('Bob')">Set Name to Bob</button>
    <p>Current name: {{ userModel().name }}</p>
  `,
})
export class UserComponent {
  userModel = signal({name: ''});
  userForm = form(this.userModel);

  setName(name: string) {
    this.userForm.name().value.set(name);
    // Input automatically displays 'Bob'
  }
}
```

Quand l'utilisateur saisit du texte dans le champ, `userModel().name` est mis à jour. Quand le bouton est cliqué, la valeur du champ devient « Bob ». Aucun code de synchronisation manuelle n'est nécessaire.

## Patterns de structure de model {#model-structure-patterns}

Les models de formulaire peuvent être des objets plats ou contenir des objets et des tableaux imbriqués. La structure choisie influence la manière d'accéder aux champs et d'organiser la validation.

### Models plats ou imbriqués {#flat-vs-nested-models}

Les models de formulaire plats placent tous les champs au premier niveau :

```ts
// Flat structure
const userModel = signal({
  name: '',
  email: '',
  street: '',
  city: '',
  state: '',
  zip: '',
});
```

Les models imbriqués regroupent les champs liés :

```ts
// Nested structure
const userModel = signal({
  name: '',
  email: '',
  address: {
    street: '',
    city: '',
    state: '',
    zip: '',
  },
});
```

**Utilisez des structures plates quand :**

- Les champs n'ont pas de regroupement conceptuel évident
- Vous voulez un accès plus simple aux champs (`userForm.city` plutôt que `userForm.address.city`)
- Les règles de validation couvrent plusieurs groupes potentiels

**Utilisez des structures imbriquées quand :**

- Les champs forment un groupe conceptuel évident (comme une adresse)
- Les données regroupées correspondent à la structure de votre API
- Vous voulez valider le groupe comme un tout

### Travailler avec des objets imbriqués {#working-with-nested-objects}

Vous pouvez accéder aux champs imbriqués en suivant le chemin de l'objet :

```ts
const userModel = signal({
  profile: {
    firstName: '',
    lastName: '',
  },
  settings: {
    theme: 'light',
    notifications: true,
  },
});

const userForm = form(userModel);

// Access nested fields
userForm.profile.firstName; // FieldTree<string>
userForm.settings.theme; // FieldTree<string>
```

Dans les templates, vous liez les champs imbriqués de la même manière que les champs de premier niveau :

```angular-ts
@Component({
  template: `
    <input [formField]="userForm.profile.firstName" />
    <input [formField]="userForm.profile.lastName" />

    <select [formField]="userForm.settings.theme">
      <option value="light">Light</option>
      <option value="dark">Dark</option>
    </select>
  `,
})
```

### Travailler avec des tableaux {#working-with-arrays}

Les models peuvent inclure des tableaux pour des collections d'éléments :

```ts
const orderModel = signal({
  customerName: '',
  items: [{product: '', quantity: 0, price: 0}],
});

const orderForm = form(orderModel);

// Access array items by index
orderForm.items[0].product; // FieldTree<string>
orderForm.items[0].quantity; // FieldTree<number>
```

Les éléments de tableau contenant des objets reçoivent automatiquement des identités de suivi, ce qui permet de conserver l'état des champs même quand les éléments changent de position dans le tableau. L'état de validation et les interactions de l'utilisateur sont ainsi correctement conservés quand les tableaux sont réordonnés.

<!-- TBD: For dynamic arrays and complex array operations, see the [Working with arrays guide](guide/forms/signals/arrays). -->

## Étapes suivantes {#next-steps}

Ce guide a présenté la création de models et la mise à jour des valeurs. D'autres guides explorent d'autres aspects des Signal Forms :

<docs-pill-row>
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/custom-controls" title="Contrôles personnalisés" />
  <!-- <docs-pill href="guide/forms/signals/arrays" title="Working with Arrays" /> -->
</docs-pill-row>
