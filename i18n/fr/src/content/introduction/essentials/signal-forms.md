<!-- source: f31051425bf275e567591b8258d1b6f02896f197 -->
<docs-decorative-header title="Les formulaires avec les signals" imgSrc="adev/src/assets/images/signals.svg"> </docs-decorative-header>

Les Signal Forms gèrent l'état des formulaires à l'aide des signals Angular, pour synchroniser automatiquement votre model de données et l'interface grâce aux Angular Signals.

Ce guide vous présente les concepts essentiels pour créer des formulaires avec les Signal Forms. Voici comment cela fonctionne :

## Créer votre premier formulaire {#creating-your-first-form}

### 1. Créer un model de formulaire avec `signal()` {#1-create-a-form-model-with-signal}

Chaque formulaire commence par la création d'un signal qui contient le model de données du formulaire :

```ts
interface LoginData {
  email: string;
  password: string;
}

const loginModel = signal<LoginData>({
  email: '',
  password: '',
});
```

### 2. Passer le model de formulaire à `form()` pour créer un `FieldTree` {#2-pass-the-form-model-to-form-to-create-a-fieldtree}

Ensuite, vous passez votre model de formulaire à la fonction `form()` pour créer un **arbre de champs** (field tree) : une structure d'objets qui reproduit la forme de votre model et vous permet d'accéder aux champs avec la notation pointée.

L'objet racine du formulaire comme ses propriétés imbriquées sont des nœuds `FieldTree` :

```ts
const loginForm = form(loginModel);

loginForm; // is a FieldTree
loginForm.email; // is also a FieldTree
```

### 3. Lier les champs HTML avec la directive `[formField]` {#3-bind-html-inputs-with-formfield-directive}

Ensuite, vous liez vos champs HTML au formulaire avec la directive `[formField]`, qui crée un two-way binding entre eux :

```html
<input type="email" [formField]="loginForm.email" />
<input type="password" [formField]="loginForm.password" />
```

Ainsi, les modifications de l'utilisateur (comme la saisie dans le champ) mettent automatiquement à jour le formulaire.

NOTE: La directive `[formField]` synchronise aussi l'état du champ pour des attributs comme `required`, `disabled` et `readonly` lorsque c'est pertinent.

### 4. Lire l'état avec les signals de `FieldTree` {#4-read-state-with-fieldtree-signals}

Vous pouvez accéder à l'état de n'importe quelle partie de l'arbre en appelant le nœud `FieldTree` comme une fonction. Cela renvoie un objet d'état contenant des signals réactifs pour la valeur, le statut de validation et l'état d'interaction :

```ts
loginForm(); // Returns state for the whole form
loginForm.email(); // Returns state for the email field
```

Pour lire la valeur courante, accédez au signal `value()` :

```html
<!-- Render values that update automatically as user types -->
<p>Form value: {{ loginForm().value() | json }}</p>
<p>Email: {{ loginForm.email().value() }}</p>
```

```ts
// Get the current value
const currentEmail = loginForm.email().value();
```

### 5. Mettre à jour les valeurs avec `set()` {#5-update-values-with-set}

Vous pouvez mettre à jour les valeurs par programmation avec la méthode `value.set()` sur n'importe quel nœud. Cela met à jour à la fois le `FieldTree` et le signal du model sous-jacent :

```ts
// Update the value programmatically
loginForm.email().value.set('alice@wonderland.com');
```

Ainsi, la valeur du champ et le signal du model sont tous deux mis à jour automatiquement :

```ts
// The model signal is also updated
console.log(loginModel().email); // 'alice@wonderland.com'
```

### Exemple complet {#complete-example}

<docs-code-multifile preview path="adev/src/content/examples/signal-forms/src/login-simple/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/signal-forms/src/login-simple/app/app.ts"/>
  <docs-code header="app.html" path="adev/src/content/examples/signal-forms/src/login-simple/app/app.html"/>
  <docs-code header="app.css" path="adev/src/content/examples/signal-forms/src/login-simple/app/app.css"/>
</docs-code-multifile>

## Utilisation de base {#basic-usage}

La directive `[formField]` fonctionne avec tous les types de champs HTML standard. Voici les cas les plus courants :

### Champs texte {#text-inputs}

Les champs texte fonctionnent avec différents attributs `type` et avec les zones de texte :

```html
<!-- Text and email -->
<input type="text" [formField]="form.name" />
<input type="email" [formField]="form.email" />
```

#### Nombres {#numbers}

Les champs numériques convertissent automatiquement entre chaînes et nombres :

```html
<!-- Number - automatically converts to number type -->
<input type="number" [formField]="form.age" />
```

#### Date et heure {#date-and-time}

Les champs de date stockent les valeurs sous forme de chaînes `YYYY-MM-DD`, et les champs d'heure utilisent le format `HH:mm` :

```html
<!-- Date and time - stores as ISO format strings -->
<input type="date" [formField]="form.eventDate" />
<input type="time" [formField]="form.eventTime" />
```

Si vous devez convertir les chaînes de date en objets Date, vous pouvez le faire en passant la valeur du champ à `Date()` :

```ts
const dateObject = new Date(form.eventDate().value());
```

#### Texte multiligne {#multiline-text}

Les zones de texte fonctionnent de la même façon que les champs texte :

```html
<!-- Textarea -->
<textarea [formField]="form.message" rows="4"></textarea>
```

### Cases à cocher {#checkboxes}

Les cases à cocher sont liées à des valeurs booléennes :

```html
<!-- Single checkbox -->
<label>
  <input type="checkbox" [formField]="form.agreeToTerms" />
  I agree to the terms
</label>
```

#### Plusieurs cases à cocher {#multiple-checkboxes}

Pour plusieurs options, créez un `formField` booléen distinct pour chacune :

```html
<label>
  <input type="checkbox" [formField]="form.emailNotifications" />
  Email notifications
</label>
<label>
  <input type="checkbox" [formField]="form.smsNotifications" />
  SMS notifications
</label>
```

### Boutons radio {#radio-buttons}

Les boutons radio fonctionnent comme les cases à cocher. Tant que les boutons radio utilisent la même valeur `[formField]`, les Signal Forms leur attribuent automatiquement le même attribut `name` :

```html
<label>
  <input type="radio" value="free" [formField]="form.plan" />
  Free
</label>
<label>
  <input type="radio" value="premium" [formField]="form.plan" />
  Premium
</label>
```

Quand un utilisateur sélectionne un bouton radio, le `formField` du formulaire stocke la valeur de l'attribut `value` de ce bouton. Par exemple, sélectionner « Premium » définit `form.plan().value()` à `"premium"`.

### Listes déroulantes {#select-dropdowns}

Les éléments select fonctionnent avec des options statiques comme dynamiques :

```angular-html
<!-- Static options -->
<select [formField]="form.country">
  <option value="">Select a country</option>
  <option value="us">United States</option>
  <option value="ca">Canada</option>
</select>

<!-- Dynamic options with @for -->
<select [formField]="form.productId">
  <option value="">Select a product</option>
  @for (product of products; track product.id) {
    <option [value]="product.id">{{ product.name }}</option>
  }
</select>
```

NOTE: La sélection multiple (`<select multiple>`) n'est pas prise en charge par la directive `[formField]` pour le moment.

## Validation et état {#validation-and-state}

Les Signal Forms fournissent des validateurs intégrés que vous pouvez appliquer aux champs de votre formulaire. Pour ajouter de la validation, passez une fonction de schéma comme second argument à `form()` :

```ts
const loginForm = form(loginModel, (schemaPath) => {
  debounce(schemaPath.email, 500);
  required(schemaPath.email);
  email(schemaPath.email);
});
```

La fonction de schéma reçoit un paramètre **chemin de schéma** (schema path) qui fournit les chemins vers vos champs pour configurer les règles de validation.

Les validateurs courants sont notamment :

- **`required()`** - Garantit que le champ a une valeur
- **`email()`** - Valide le format de l'adresse e-mail
- **`min()`** / **`max()`** - Valide des plages de nombres
- **`minLength()`** / **`maxLength()`** - Valide la longueur d'une chaîne ou d'une collection
- **`pattern()`** - Valide par rapport à une expression régulière

Vous pouvez aussi personnaliser les messages d'erreur en passant un objet d'options comme second argument du validateur :

```ts
required(schemaPath.email, {message: 'Email is required'});
email(schemaPath.email, {message: 'Please enter a valid email address'});
```

Chaque nœud du `FieldTree` expose son état de validation et d'interaction via des signals réactifs.

### Signals d'état de FieldTree {#fieldtree-state-signals}

Chaque nœud de l'arbre, y compris l'objet racine du formulaire, fournit les mêmes signals pour suivre son état. Comme chaque nœud est un `FieldTree`, l'API pour surveiller la validité et l'interaction est identique à tous les niveaux.

| État         | Description                                                                     |
| ------------ | ------------------------------------------------------------------------------- |
| `valid()`    | Renvoie `true` si le nœud respecte toutes les règles de validation              |
| `invalid()`  | Renvoie `true` s'il y a des erreurs de validation                               |
| `pending()`  | Renvoie `true` si une validation asynchrone est en cours                        |
| `touched()`  | Renvoie `true` si l'utilisateur a donné puis retiré le focus au champ ou à un champ enfant |
| `dirty()`    | Renvoie `true` si la valeur a été modifiée par l'utilisateur                    |
| `disabled()` | Renvoie `true` si le nœud est désactivé                                         |
| `readonly()` | Renvoie `true` si le nœud est en lecture seule                                  |
| `errors()`   | Renvoie un tableau d'erreurs de validation avec les propriétés `kind` et `message` |

### Exemple complet {#complete-example-validation}

<docs-code-multifile preview path="adev/src/content/examples/signal-forms/src/login-validation/app/app.ts">
  <docs-code header="app.ts" path="adev/src/content/examples/signal-forms/src/login-validation/app/app.ts"/>
  <docs-code header="app.html" path="adev/src/content/examples/signal-forms/src/login-validation/app/app.html"/>
  <docs-code header="app.css" path="adev/src/content/examples/signal-forms/src/login-validation/app/app.css"/>
</docs-code-multifile>

## Prochaines étapes {#next-steps}

Pour en savoir plus sur les Signal Forms et leur fonctionnement, consultez les guides approfondis :

- [Vue d'ensemble](guide/forms/signals/overview) - Introduction aux Signal Forms et aux cas où les utiliser
- [Models de formulaire](guide/forms/signals/models) - Créer et gérer les données de formulaire avec les signals
- [Gestion de l'état des champs](guide/forms/signals/field-state-management) - Travailler avec l'état de validation, le suivi des interactions et la visibilité des champs
- [Validation](guide/forms/signals/validation) - Validateurs intégrés, règles de validation personnalisées et validation asynchrone

<docs-pill-row>
  <docs-pill title="Une conception modulaire avec l'injection de dépendances" href="essentials/dependency-injection" />
</docs-pill-row>
