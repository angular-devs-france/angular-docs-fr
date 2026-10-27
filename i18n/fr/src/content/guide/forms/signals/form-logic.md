<!-- source: f7f15cfa265ff68c3baea2ed32b21d85846dff90 -->
# Ajouter de la logique de formulaire

Les Signal Forms vous permettent d'ajouter de la logique à votre formulaire à l'aide de schémas. La logique de validation est traitée dans le [guide de validation](guide/forms/signals/validation) ; ce guide présente les autres règles disponibles dans les schémas. Vous pouvez désactiver des champs sous condition, les masquer en fonction d'autres valeurs, les mettre en lecture seule, appliquer un debounce à la saisie de l'utilisateur et associer des métadonnées pour les contrôles personnalisés.

Ce guide vous montre comment utiliser des règles comme `disabled()`, `hidden()`, `readonly()`, `debounce()` et `metadata()` pour contrôler le comportement des champs.

## Quand ajouter de la logique de formulaire {#when-to-add-form-logic}

Utilisez des règles lorsque le comportement d'un champ dépend des valeurs d'autres champs ou doit se mettre à jour de manière réactive. Par exemple :

- Un champ de code de réduction désactivé lorsque le total de la commande est trop faible
- Un champ d'adresse masqué sauf si une livraison est nécessaire
- Un champ de recherche avec un debounce pour réduire les appels d'API

## Fonctionnement des règles {#how-rules-work}

Les règles associent une logique réactive à des champs précis de votre formulaire. La plupart des règles conditionnelles acceptent un objet d'options avec une fonction `when`. La fonction `when` se recalcule automatiquement chaque fois que les signals qu'elle référence changent, exactement comme un `computed`.

```ts
const orderForm = form(this.orderModel, (schemaPath) => {
  disabled(schemaPath.couponCode, {when: ({valueOf}) => valueOf(schemaPath.total) < 50});
  //~~~~~~ ~~~~~~~~~~~~~~~~~~~~~  ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
  //rule     path                   reactive logic function
});
```

Les fonctions de logique réactive reçoivent un objet `FieldContext` qui donne accès aux valeurs et à l'état des champs via des fonctions utilitaires comme `valueOf()` et `stateOf()`. Il est souvent déstructuré pour accéder directement à ces fonctions.

NOTE: Le paramètre du callback de schéma (`schemaPath` dans ces exemples) est un objet `SchemaPathTree` qui fournit les chemins vers tous les champs de votre formulaire. Vous pouvez nommer ce paramètre comme vous le souhaitez.

Pour le détail complet des propriétés et méthodes de `FieldContext`, consultez le [guide de validation](guide/forms/signals/validation).

## Empêcher la modification des champs avec `disabled()` {#prevent-field-updates-with-disabled}

La règle `disabled()` configure l'état désactivé d'un champ.

Elle fonctionne avec la directive `[formField]` pour lier automatiquement l'attribut `disabled` en fonction de l'état du champ : vous n'avez donc pas besoin d'ajouter manuellement `[disabled]="yourForm.fieldName().disabled()"` à votre template.

NOTE: Les champs désactivés ne sont pas validés : ils ne participent pas aux vérifications de validation du formulaire. La valeur du champ est conservée mais n'est pas validée. Pour plus de détails sur le comportement de la validation, consultez le [guide de validation](guide/forms/signals/validation).

### Toujours désactivé {#always-disabled}

Pour désactiver un champ de façon permanente, appelez `disabled()` avec uniquement le chemin du champ :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, disabled} from '@angular/forms/signals';

@Component({
  selector: 'app-settings',
  imports: [FormField],
  template: `
    <label>
      System ID (cannot be changed)
      <input [formField]="settingsForm.systemId" />
    </label>
  `,
})
export class Settings {
  settingsModel = signal({
    systemId: 'SYS-12345',
    userName: '',
  });

  settingsForm = form(this.settingsModel, (schemaPath) => {
    disabled(schemaPath.systemId);
  });
}
```

### Désactivation conditionnelle {#conditional-disabling}

Pour désactiver un champ selon des conditions, fournissez une fonction `when` qui renvoie `true` (désactivé) ou `false` (activé) :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, disabled} from '@angular/forms/signals';

@Component({
  selector: 'app-order',
  imports: [FormField],
  template: `
    <label>
      Order Total
      <input type="number" [formField]="orderForm.total" />
    </label>

    <label>
      Coupon Code
      <input [formField]="orderForm.couponCode" />
    </label>
  `,
})
export class Order {
  orderModel = signal({
    total: 25,
    couponCode: '',
  });

  orderForm = form(this.orderModel, (schemaPath) => {
    disabled(schemaPath.couponCode, {when: ({valueOf}) => valueOf(schemaPath.total) < 50});
  });
}
```

Dans cet exemple, lorsque le total de la commande est inférieur à 50 $, le champ du code de réduction est désactivé.

### Raisons de désactivation {#disabled-reasons}

Lorsque vous désactivez un champ, fournissez des explications destinées à l'utilisateur en renvoyant une chaîne au lieu de `true` :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, disabled} from '@angular/forms/signals';

@Component({
  selector: 'app-order',
  imports: [FormField],
  template: `
    <label>
      Order Total
      <input type="number" [formField]="orderForm.total" />
    </label>

    <label>
      Coupon Code
      <input [formField]="orderForm.couponCode" />
    </label>

    @if (orderForm.couponCode().disabled()) {
      <div class="info">
        @for (reason of orderForm.couponCode().disabledReasons(); track reason) {
          <p>{{ reason.message }}</p>
        }
      </div>
    }
  `,
})
export class Order {
  orderModel = signal({
    total: 25,
    couponCode: '',
  });

  orderForm = form(this.orderModel, (schemaPath) => {
    disabled(schemaPath.couponCode, {
      when: ({valueOf}) =>
        valueOf(schemaPath.total) < 50 ? 'Order must be $50 or more to use a coupon' : false,
    });
  });
}
```

La fonction `when` renvoie :

- une **chaîne** pour désactiver le champ avec une raison
- `false` pour activer le champ (pas n'importe quelle valeur falsy : utilisez explicitement `false`)

Accédez aux raisons via le signal `disabledReasons()` de l'état du champ. Chaque raison possède une propriété `message` contenant la chaîne que vous avez renvoyée.

#### Plusieurs raisons de désactivation {#multiple-disabled-reasons}

Vous pouvez aussi appeler `disabled()` plusieurs fois sur le même champ, et toutes les raisons renvoyées s'accumulent :

```angular-ts
orderForm = form(this.orderModel, (schemaPath) => {
  disabled(schemaPath.promoCode, {
    when: ({valueOf}) =>
      !valueOf(schemaPath.hasAccount) ? 'You must have an account to use promo codes' : false,
  });
  disabled(schemaPath.promoCode, {
    when: ({valueOf}) => (valueOf(schemaPath.total) < 25 ? 'Order must be at least $25' : false),
  });
});
```

Si les deux conditions sont vraies, le champ affiche les deux raisons de désactivation. Cette approche est utile pour des règles de disponibilité complexes que vous voulez garder séparées.

## Configurer l'état `hidden()` des champs {#configuring-hidden-state-on-fields}

La règle `hidden()` configure l'état masqué d'un champ. Cependant, elle ne définit qu'un état programmatique. **C'est vous qui contrôlez si le champ apparaît dans l'interface**.

IMPORTANT: Contrairement à `disabled` et `readonly`, il n'existe pas de propriété DOM native pour l'état `hidden`. La directive `[formField]` n'applique pas d'attribut `hidden` aux éléments. Vous devez utiliser `@if` ou du CSS dans votre template pour afficher les champs de façon conditionnelle selon l'état `hidden()`.

NOTE: Comme les champs désactivés, les champs masqués ne sont pas validés. Consultez le [guide de validation](guide/forms/signals/validation) pour plus de détails.

### Masquage simple d'un champ {#basic-field-hiding}

Utilisez `hidden()` avec une fonction `when` qui renvoie `true` (masqué) ou `false` (visible) :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, hidden} from '@angular/forms/signals';

@Component({
  selector: 'app-profile',
  imports: [FormField],
  template: `
    <label>
      <input type="checkbox" [formField]="profileForm.isPublic" />
      Make profile public
    </label>

    @if (!profileForm.publicUrl().hidden()) {
      <label>
        Public URL
        <input [formField]="profileForm.publicUrl" />
      </label>
    }
  `,
})
export class Profile {
  profileModel = signal({
    isPublic: false,
    publicUrl: '',
  });

  profileForm = form(this.profileModel, (schemaPath) => {
    hidden(schemaPath.publicUrl, {when: ({valueOf}) => !valueOf(schemaPath.isPublic)});
  });
}
```

### Toujours masqué {#always-hidden}

Pour masquer un champ de façon permanente, appelez `hidden()` avec uniquement le chemin du champ :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, hidden} from '@angular/forms/signals';

@Component({
  selector: 'app-profile',
  imports: [FormField],
  template: `
    <label>
      <input type="checkbox" [formField]="profileForm.isPublic" />
      Make profile public
    </label>

    <!-- The publicUrl is permanently hidden, so this block never renders -->
    @if (!profileForm.publicUrl().hidden()) {
      <label>
        Public URL
        <input [formField]="profileForm.publicUrl" />
      </label>
    }
  `,
})
export class Profile {
  profileModel = signal({
    isPublic: false,
    publicUrl: '',
  });

  profileForm = form(this.profileModel, (schemaPath) => {
    // This field is now permanently hidden and excluded from active validation
    hidden(schemaPath.publicUrl);
  });
}
```

## Afficher des champs non modifiables avec `readonly()` {#display-uneditable-fields-with-readonly}

La règle `readonly()` empêche les utilisateurs de modifier un champ. La directive `[FormField]` lie automatiquement cet état à l'attribut HTML `readonly`, qui empêche la modification tout en permettant aux utilisateurs de donner le focus au champ et d'en sélectionner le texte.

NOTE: Les champs en lecture seule ne sont pas soumis à la [validation](guide/forms/signals/validation).

### Toujours en lecture seule {#always-readonly}

Pour mettre un champ en lecture seule de façon permanente, appelez `readonly()` avec uniquement le chemin du champ :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, readonly} from '@angular/forms/signals';

@Component({
  selector: 'app-account',
  imports: [FormField],
  template: `
    <label>
      Username (cannot be changed)
      <input [formField]="accountForm.username" />
    </label>

    <label>
      Email
      <input [formField]="accountForm.email" />
    </label>
  `,
})
export class Account {
  accountModel = signal({
    username: 'johndoe',
    email: 'john@example.com',
  });

  accountForm = form(this.accountModel, (schemaPath) => {
    readonly(schemaPath.username);
  });
}
```

La directive `[FormField]` lie automatiquement l'attribut `readonly` en fonction de l'état du champ.

### Lecture seule conditionnelle {#conditional-readonly}

Pour mettre un champ en lecture seule selon des conditions, fournissez une fonction `when` :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, readonly} from '@angular/forms/signals';

@Component({
  selector: 'app-document',
  imports: [FormField],
  template: `
    <label>
      <input type="checkbox" [formField]="documentForm.isLocked" />
      Lock document
    </label>

    <label>
      Document Title
      <input [formField]="documentForm.title" />
    </label>
  `,
})
export class Document {
  documentModel = signal({
    isLocked: false,
    title: 'Untitled',
  });

  documentForm = form(this.documentModel, (schemaPath) => {
    readonly(schemaPath.title, {when: ({valueOf}) => valueOf(schemaPath.isLocked)});
  });
}
```

Lorsque `isLocked` vaut true, le champ du titre passe en lecture seule.

## Choisir entre masqué, désactivé et en lecture seule {#choose-between-hidden-disabled-and-readonly}

Ces trois fonctions de configuration contrôlent la disponibilité des champs de différentes manières :

Choisissez `hidden()` lorsque le champ :

- ne doit pas du tout apparaître dans l'interface
- n'est pas pertinent pour l'état courant du formulaire
- Exemple : les champs d'adresse de livraison lorsque « identique à l'adresse de facturation » est coché

Choisissez `disabled()` lorsque le champ :

- doit être visible mais non modifiable
- doit indiquer pourquoi il est indisponible (à l'aide des raisons de désactivation)
- doit être exclu de la soumission du formulaire HTML
- Exemple : un bouton de soumission désactivé tant que le formulaire n'est pas valide, des champs d'approbation désactivés pour les utilisateurs non administrateurs

Choisissez `readonly()` lorsque le champ :

- doit être visible mais non modifiable
- contient des données que les utilisateurs doivent voir, sélectionner ou copier
- doit être inclus dans la soumission du formulaire HTML
- Exemple : un numéro de confirmation de commande, des codes de référence générés par le système

Les trois ignorent la validation et empêchent la modification par l'utilisateur lorsqu'ils sont actifs. Les principales différences :

| Caractéristique                              | `hidden()` | `disabled()` | `readonly()` |
| -------------------------------- | ---------- | ------------ | ------------ |
| Visible dans l'interface                     | Non        | Oui          | Oui          |
| Focus/sélection possible par l'utilisateur   | Non        | Non          | Oui          |
| Inclus dans la soumission du formulaire HTML | Non        | Non          | Oui          |

## Retarder les opérations de saisie avec `debounce()` {#delay-input-operations-with-debounce}

La règle `debounce()` retarde la mise à jour du model de formulaire. C'est utile pour optimiser les performances et réduire les opérations inutiles lors d'une saisie rapide.

### Ce que fait le debounce {#what-debouncing-does}

Sans debounce, chaque frappe met immédiatement à jour le model de formulaire. Cela peut déclencher :

- des signals computed coûteux qui se recalculent à chaque modification
- des vérifications de validation après chaque caractère
- des appels d'API ou d'autres effets de bord liés à la valeur du model

Le debounce retarde ces mises à jour et réduit le travail inutile.

### Debounce simple {#basic-debouncing}

Vous pouvez appliquer un debounce à un champ en indiquant un délai en millisecondes :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, debounce} from '@angular/forms/signals';

@Component({
  selector: 'app-search',
  imports: [FormField],
  template: `
    <label>
      Search
      <input [formField]="searchForm.query" />
    </label>

    <p>Searching for: {{ searchForm.query().value() }}</p>
  `,
})
export class Search {
  searchModel = signal({
    query: '',
  });

  searchForm = form(this.searchModel, (schemaPath) => {
    debounce(schemaPath.query, 300);
  });
}
```

Avec un debounce de 300 ms :

- L'utilisateur saisit du texte dans le champ
- Le model de formulaire n'est mis à jour qu'après 300 ms sans saisie
- Si l'utilisateur continue de taper, le minuteur est réinitialisé à chaque frappe
- Dès que l'utilisateur marque une pause de 300 ms, le model est mis à jour avec la valeur finale

### Garanties temporelles {#timing-guarantees}

La fonction `debounce()` garantit que les utilisateurs ne perdent pas de données grâce à ces mécanismes :

- **Lorsque le champ est marqué comme touché :** la valeur est synchronisée immédiatement, ce qui annule tout délai de debounce en attente. Cela se produit lorsque le champ perd le focus (blur) ou lorsqu'il est explicitement marqué comme touché.
- **Lors de la soumission du formulaire :** tous les champs sont marqués comme touchés avant la validation, ce qui garantit que toutes les valeurs soumises à un debounce sont synchronisées immédiatement.

Ainsi, les utilisateurs peuvent taper rapidement, quitter le champ avec Tab ou soumettre le formulaire sans attendre l'expiration des délais de debounce.

### Logique de debounce personnalisée {#custom-debounce-logic}

Pour un contrôle avancé, fournissez une fonction de debounce qui détermine quand synchroniser la valeur. Cette fonction est appelée à chaque mise à jour de la valeur du contrôle et peut renvoyer soit `undefined` pour synchroniser immédiatement, soit une promise qui empêche la synchronisation jusqu'à sa résolution :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, debounce} from '@angular/forms/signals';

@Component({
  selector: 'app-search',
  imports: [FormField],
  template: `
    <label>
      Search
      <input [formField]="searchForm.query" />
    </label>
  `,
})
export class Search {
  searchModel = signal({
    query: '',
  });

  searchForm = form(this.searchModel, (schemaPath) => {
    debounce(schemaPath.query, () => {
      // Return a promise that resolves after 500ms
      return new Promise<void>((resolve) => {
        setTimeout(() => resolve(), 500);
      });
    });
  });
}
```

La fonction de debounce peut renvoyer :

- `undefined` pour synchroniser la valeur immédiatement
- une `Promise<void>` qui empêche la synchronisation jusqu'à sa résolution

Cas d'usage d'une logique de debounce personnalisée :

- Implémenter une logique de timing personnalisée qui va au-delà de simples délais
- Coordonner la synchronisation avec des événements externes
- Appliquer un debounce conditionnel selon l'état de l'application

### Quand utiliser le debounce {#when-to-use-debouncing}

Le debounce est surtout utile lorsque :

- vous avez des signals computed coûteux qui dépendent de la valeur du champ
- le champ déclenche des appels d'API ou d'autres effets de bord
- vous voulez réduire le coût de la validation pendant une saisie rapide
- le profilage des performances montre que les mises à jour du model provoquent des ralentissements

N'utilisez pas le debounce si :

- le champ a besoin de mises à jour immédiates pour une bonne UX (comme les champs d'une calculatrice)
- le gain de performances est négligeable
- les utilisateurs attendent un retour en temps réel

## Associer des données à un champ avec `metadata()` {#associate-data-with-a-field-using-metadata}

Les métadonnées associent des données réactives à un champ. Les règles de validation utilisent ce système en interne, et vous pouvez publier vos propres clés pour des informations propres à votre application, comme un texte d'aide, une configuration ou des valeurs d'affichage calculées.

Les Signal Forms fournissent des clés de métadonnées prédéfinies que les validateurs intégrés renseignent automatiquement :

| Clé          | Renseignée par       | Lue via               |
| ------------ | -------------------- | --------------------- |
| `REQUIRED`   | `required()`         | `field().required()`  |
| `MIN`        | `min()`, `minDate()` | `field().min()`       |
| `MAX`        | `max()`, `maxDate()` | `field().max()`       |
| `MIN_LENGTH` | `minLength()`        | `field().minLength()` |
| `MAX_LENGTH` | `maxLength()`        | `field().maxLength()` |
| `PATTERN`    | `pattern()`          | `field().pattern()`   |

La directive `[formField]` lie automatiquement cinq d'entre elles (`REQUIRED`, `MIN`, `MAX`, `MIN_LENGTH` et `MAX_LENGTH`) à l'attribut HTML correspondant d'un contrôle de formulaire natif. `PATTERN` fait exception, car les Signal Forms prennent en charge plusieurs patterns par champ alors que l'attribut HTML `pattern` n'accepte qu'une seule expression régulière.

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, required, min, max} from '@angular/forms/signals';

@Component({
  selector: 'app-age',
  imports: [FormField],
  template: `
    <label>
      Age (between {{ ageForm.age().min?.() }} and {{ ageForm.age().max?.() }})
      <input type="number" [formField]="ageForm.age" />
    </label>

    @if (ageForm.age().required()) {
      <span class="required-indicator">*</span>
    }
  `,
})
export class Age {
  ageModel = signal({age: 0});

  ageForm = form(this.ageModel, (schemaPath) => {
    required(schemaPath.age);
    min(schemaPath.age, 18);
    max(schemaPath.age, 120);
  });
}
```

### Métadonnées réactives {#reactive-metadata}

Les règles de validation peuvent dériver leurs contraintes d'autres champs, ce qui rend les métadonnées publiées réactives :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, max} from '@angular/forms/signals';

@Component({
  selector: 'app-inventory',
  imports: [FormField],
  template: `
    <label>
      Item
      <select [formField]="inventoryForm.item">
        <option value="widget">Widget</option>
        <option value="gadget">Gadget</option>
      </select>
    </label>

    <label>
      Quantity (max: {{ inventoryForm.quantity().max?.() }})
      <input type="number" [formField]="inventoryForm.quantity" />
    </label>
  `,
})
export class Inventory {
  inventoryModel = signal({
    item: 'widget',
    quantity: 0,
  });

  inventoryForm = form(this.inventoryModel, (schemaPath) => {
    max(schemaPath.quantity, ({valueOf}) => {
      const item = valueOf(schemaPath.item);
      return item === 'widget' ? 100 : 50;
    });
  });
}
```

La règle de validation `max()` définit les métadonnées `MAX` de manière réactive en fonction de l'article sélectionné : tout template ou contrôle qui lit `field().max()` se met donc à jour chaque fois que l'article change.

Pour aller plus loin, notamment pour savoir comment définir des clés personnalisées, combiner des contributions avec des reducers et utiliser des métadonnées gérées pour des objets liés au cycle de vie, consultez le [guide des métadonnées de champ](guide/forms/signals/field-metadata).

## Combiner des règles {#combining-rules}

Vous pouvez appliquer plusieurs règles au même champ, et utiliser une logique conditionnelle pour appliquer des groupes entiers de règles selon l'état du formulaire.

### Plusieurs règles sur un même champ {#multiple-rules-on-one-field}

Appliquez plusieurs règles pour configurer tous les aspects du comportement d'un champ :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, disabled, hidden, debounce, metadata} from '@angular/forms/signals';
import {PLACEHOLDER} from './metadata-keys';

@Component({
  selector: 'app-promo',
  imports: [FormField],
  template: `
    @if (!promoForm.promoCode().hidden()) {
      <label>
        Promo Code
        <input [formField]="promoForm.promoCode" />
      </label>
    }
  `,
})
export class Promo {
  promoModel = signal({
    hasAccount: false,
    subscriptionType: 'free' as 'free' | 'premium',
    promoCode: '',
  });

  promoForm = form(this.promoModel, (schemaPath) => {
    disabled(schemaPath.promoCode, {
      when: ({valueOf}) => (!valueOf(schemaPath.hasAccount) ? 'You must have an account' : false),
    });
    hidden(schemaPath.promoCode, {
      when: ({valueOf}) => valueOf(schemaPath.subscriptionType) === 'free',
    });
    debounce(schemaPath.promoCode, 300);
    metadata(schemaPath.promoCode, PLACEHOLDER, () => 'Enter promo code');
  });
}
```

Ces règles fonctionnent ensemble :

- L'état masqué a la priorité : si le champ est masqué, l'état désactivé n'a pas d'importance
- L'état désactivé empêche la modification, quel que soit l'état de lecture seule
- Le debounce agit sur les mises à jour du model, quel que soit l'état par ailleurs
- Les métadonnées sont indépendantes et toujours disponibles

### Logique conditionnelle avec applyWhen {#conditional-logic-with-applywhen}

Utilisez `applyWhen()` pour appliquer de façon conditionnelle des groupes entiers de règles :

```angular-ts
import {Component, signal} from '@angular/core';
import {form, FormField, applyWhen, required, pattern} from '@angular/forms/signals';

@Component({
  selector: 'app-address',
  imports: [FormField],
  template: `
    <label>
      Country
      <select [formField]="addressForm.country">
        <option value="US">United States</option>
        <option value="CA">Canada</option>
      </select>
    </label>

    <label>
      Zip/Postal Code
      <input [formField]="addressForm.zipCode" />
    </label>
  `,
})
export class Address {
  addressModel = signal({
    country: 'US',
    zipCode: '',
  });

  addressForm = form(this.addressModel, (schemaPath) => {
    applyWhen(
      schemaPath,
      ({valueOf}) => valueOf(schemaPath.country) === 'US',
      (schemaPath) => {
        // Only applied when country is US
        required(schemaPath.zipCode);
        pattern(schemaPath.zipCode, /^\d{5}(-\d{4})?$/);
      },
    );
  });
}
```

La fonction `applyWhen()` reçoit :

1. Un chemin auquel appliquer la logique (souvent le chemin racine du formulaire)
2. Une fonction de logique réactive qui renvoie `true` (appliquer) ou `false` (ne pas appliquer)
3. Une fonction de schéma qui définit les règles conditionnelles

Les règles conditionnelles ne s'exécutent que lorsque la condition est vraie. C'est utile pour les formulaires complexes dont les règles de validation ou le comportement changent selon les choix de l'utilisateur.

### Fonctions de schéma réutilisables {#reusable-schema-functions}

Extrayez les configurations de règles courantes dans des fonctions réutilisables :

```ts
import {SchemaPath, debounce, metadata, maxLength, form} from '@angular/forms/signals';
import {PLACEHOLDER} from './metadata-keys';

function emailFieldConfig(path: SchemaPath<string>) {
  debounce(path, 300);
  metadata(path, PLACEHOLDER, () => 'user@example.com');
  maxLength(path, 255);
}

// Use in multiple forms
const contactForm = form(contactModel, (schemaPath) => {
  emailFieldConfig(schemaPath.email);
  emailFieldConfig(schemaPath.alternateEmail);
});

const registrationForm = form(registrationModel, (schemaPath) => {
  emailFieldConfig(schemaPath.email);
});
```

Cette approche est utile lorsque vous avez des configurations de champ standard que vous réutilisez dans plusieurs formulaires de votre application.

## Étapes suivantes {#next-steps}

Pour en savoir plus sur les Signal Forms, consultez ces guides associés :

- [Gestion de l'état des champs](guide/forms/signals/field-state-management) - Découvrez comment utiliser, dans vos templates et dans la logique de vos composants, les signals d'état créés par ces fonctions
- [Validation](guide/forms/signals/validation) - Découvrez les règles de validation et la gestion des erreurs
- [Contrôles personnalisés](guide/forms/signals/custom-controls) - Découvrez comment les contrôles personnalisés peuvent lire les métadonnées et l'état pour se configurer automatiquement
