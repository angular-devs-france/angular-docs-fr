<!-- source: ad37428dab6ad78d745694cfbd2586e6e329a17e -->
# Mettre en place le model de formulaire

Chaque Signal Form commence par un model de données de formulaire : un signal qui définit la forme de vos données et stocke les données de votre formulaire.

Dans cette leçon, vous allez apprendre à :

- Définir une interface TypeScript pour les données de votre formulaire
- Créer un signal pour contenir les valeurs du formulaire
- Utiliser la fonction `form()` pour créer un Signal Form

Posons les fondations de notre formulaire de connexion !

<hr />

<docs-workflow>

<docs-step title="Définir l'interface LoginData {#define-the-logindata-interface}">
Créez une interface TypeScript qui définit la structure des données de votre formulaire de connexion. Le formulaire comportera :

- Un champ `email` (string)
- Un champ `password` (string)
- Un champ `rememberMe` (boolean)

```ts
interface LoginData {
  email: string;
  password: string;
  rememberMe: boolean;
}
```

Ajoutez cette interface au-dessus du décorateur `@Component`.
</docs-step>

<docs-step title="Importer signal et form {#import-signal-and-form}">
Importez la fonction `signal` depuis `@angular/core` et la fonction `form` depuis `@angular/forms/signals` :

```ts
import {Component, signal} from '@angular/core';
import {form} from '@angular/forms/signals';
```

</docs-step>

<docs-step title="Créer le signal du model de formulaire {#create-the-form-model-signal}">
Dans la classe de votre composant, créez un signal `loginModel` avec des valeurs initiales. Utilisez l'interface `LoginData` comme paramètre de type :

```ts
loginModel = signal<LoginData>({
  email: '',
  password: '',
  rememberMe: false,
});
```

Les valeurs initiales sont des chaînes vides pour les champs texte et `false` pour la case à cocher.
</docs-step>

<docs-step title="Créer le formulaire {#create-the-form}">
Créez maintenant le formulaire en passant le signal de votre model à la fonction `form()` :

```ts
loginForm = form(this.loginModel);
```

La fonction `form()` crée un formulaire à partir de votre model et vous donne accès à l'état des champs et à la validation.
</docs-step>

</docs-workflow>

Excellent ! Vous avez mis en place votre model de formulaire. Le signal `loginModel` contient les données de votre formulaire, et `loginForm` donne accès à chaque champ avec la sûreté du typage.

Ensuite, vous allez apprendre [à connecter votre formulaire au template](/tutorials/signal-forms/2-connect-form-template) !
