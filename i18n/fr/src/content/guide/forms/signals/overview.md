<!-- source: d8b76649945833cd9b4e2934c34f5e8deac11065 -->
<docs-decorative-header title="Les formulaires avec les signals Angular" imgSrc="adev/src/assets/images/signals.svg"> <!-- markdownlint-disable-line -->
</docs-decorative-header>

Les Signal Forms sont une library qui vous permet de gérer l'état des formulaires dans les applications Angular en s'appuyant sur la base réactive des signals. Grâce au two-way binding automatique, à l'accès typé aux champs et à la validation par schéma, les Signal Forms vous aident à créer des formulaires robustes.

TIP: Pour une introduction rapide aux Signal Forms, consultez le [guide des Fondamentaux sur les Signal Forms](essentials/signal-forms).

## Pourquoi les Signal Forms ? {#why-signal-forms}

Créer des formulaires dans une application web implique de gérer plusieurs aspects interdépendants : suivre les valeurs des champs, valider les saisies de l'utilisateur, gérer les états d'erreur et garder l'interface synchronisée avec votre modèle de données. Gérer ces aspects séparément entraîne du code répétitif et de la complexité.

Les Signal Forms répondent à ces défis en :

- **Synchronisant l'état automatiquement** - Synchronise automatiquement le model de données du formulaire avec les champs de formulaire liés
- **Garantissant la sûreté du typage** - Prend en charge des schémas et des bindings entièrement typés entre vos contrôles d'interface et votre modèle de données
- **Centralisant la logique de validation** - Définissez toutes les règles de validation en un seul endroit grâce à un schéma de validation

Les Signal Forms sont particulièrement adaptés aux nouvelles applications construites avec les signals. Si vous travaillez sur une application existante qui utilise les reactive forms, les reactive forms restent un choix solide.

NOTE: Si vous venez des template-driven forms ou des reactive forms, le [guide de comparaison](guide/forms/signals/comparison) peut vous intéresser.

## Prérequis {#prerequisites}

Les Signal Forms nécessitent :

- Angular v21 ou supérieur

## Mise en place {#setup}

Les Signal Forms sont déjà inclus dans le package `@angular/forms`. Importez les fonctions et directives nécessaires depuis `@angular/forms/signals` :

```ts
import {form, FormField, required, email} from '@angular/forms/signals';
```

La directive `FormField` doit être importée dans tout composant qui lie des champs de formulaire à des éléments HTML de saisie :

```ts
@Component({
  // ...
  imports: [FormField],
})
```

## Prochaines étapes {#next-steps}

Pour en savoir plus sur le fonctionnement des Signal Forms, consultez les guides suivants :

<docs-pill-row>
  <docs-pill href="essentials/signal-forms" title="Fondamentaux des Signal Forms" />
  <docs-pill href="guide/forms/signals/models" title="Models de formulaire" />
  <docs-pill href="guide/forms/signals/model-design" title="Concevoir votre model de formulaire" />
  <docs-pill href="guide/forms/signals/field-state-management" title="Gestion de l'état des champs" />
  <docs-pill href="guide/forms/signals/validation" title="Validation" />
  <docs-pill href="guide/forms/signals/custom-controls" title="Contrôles personnalisés" />
  <docs-pill href="guide/forms/signals/comparison" title="Comparaison avec les autres systèmes de formulaires" />
</docs-pill-row>
