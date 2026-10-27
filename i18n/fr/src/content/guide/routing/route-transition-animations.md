<!-- source: 0c71423442bbf277b133fc26b042b87db7973673 -->
# Animations de transition de route

Les animations de transition de route améliorent l'expérience utilisateur en offrant des transitions visuelles fluides lors de la navigation entre les différentes vues de votre application Angular. Le [router Angular](/guide/routing) prend en charge nativement l'API View Transitions du navigateur, ce qui permet des animations fluides entre les changements de route dans les navigateurs compatibles.

HELPFUL: L'intégration native des View Transitions dans le router est actuellement en [developer preview](/reference/releases#developer-preview). Les View Transitions natives sont une fonctionnalité de navigateur relativement récente, dont la prise en charge reste limitée selon les navigateurs.

## Fonctionnement des view transitions {#how-view-transitions-work}

Les view transitions utilisent l'[API native `document.startViewTransition`](https://developer.mozilla.org/en-US/docs/Web/API/Document/startViewTransition) du navigateur pour créer des animations fluides entre les différents états de votre application. L'API fonctionne ainsi :

1. **Capture de l'état courant** - Le navigateur prend une capture d'écran de la page courante
2. **Exécution de la mise à jour du DOM** - Votre fonction de callback s'exécute pour mettre à jour le DOM
3. **Capture du nouvel état** - Le navigateur capture l'état mis à jour de la page
4. **Lecture de la transition** - Le navigateur anime le passage de l'ancien état au nouveau

Voici la structure de base de l'API `startViewTransition` :

```ts
document.startViewTransition(async () => {
  await updateTheDOMSomehow();
});
```

Pour plus de détails sur l'API du navigateur, consultez l'[explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions).

## Comment le router utilise les view transitions {#how-the-router-uses-view-transitions}

Le router Angular intègre les view transitions au cycle de vie de la navigation pour créer des changements de route fluides. Pendant la navigation, le router :

1. **Termine la préparation de la navigation** - La correspondance des routes, le [lazy loading](guide/routing/loading-strategies#lazily-loaded-components-and-routes), les [guards](/guide/routing/route-guards), les [resources](/guide/routing/data-fetching-with-resources) et les [resolvers](/guide/routing/data-resolvers) s'exécutent
2. **Lance la view transition** - Le router appelle `startViewTransition` quand les routes sont prêtes à être activées
3. **Met à jour le DOM** - Le router active les nouvelles routes et désactive les anciennes dans le callback de la transition
4. **Finalise la transition** - La promise de la transition se résout quand Angular a terminé le rendu

L'intégration des view transitions dans le router agit comme une [amélioration progressive](https://developer.mozilla.org/en-US/docs/Glossary/Progressive_Enhancement). Quand les navigateurs ne prennent pas en charge l'API View Transitions, le router effectue des mises à jour normales du DOM sans animation, ce qui garantit que votre application fonctionne dans tous les navigateurs.

## Activer les view transitions dans le router {#enabling-view-transitions-in-the-router}

Activez les view transitions en ajoutant la fonctionnalité `withViewTransitions` à la [configuration de votre router](/guide/routing/define-routes#adding-the-router-to-your-application). Angular prend en charge les deux approches de bootstrap, standalone et NgModule :

### Bootstrap standalone {#standalone-bootstrap}

```ts
import {bootstrapApplication} from '@angular/platform-browser';
import {provideRouter, withViewTransitions} from '@angular/router';
import {routes} from './app.routes';

bootstrapApplication(MyApp, {
  providers: [provideRouter(routes, withViewTransitions())],
});
```

### Bootstrap avec NgModule {#ngmodule-bootstrap}

```ts
import {NgModule} from '@angular/core';
import {RouterModule} from '@angular/router';

@NgModule({
  imports: [RouterModule.forRoot(routes, {enableViewTransitions: true})],
})
export class AppRouting {}
```

[Essayez l'exemple « count » sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-2dnvtm?file=src%2Fmain.ts)

Cet exemple montre comment la navigation du router peut remplacer les appels directs à `startViewTransition` pour les mises à jour du compteur.

## Personnaliser les transitions avec CSS {#customizing-transitions-with-css}

Vous pouvez personnaliser les view transitions avec CSS pour créer des effets d'animation uniques. Le navigateur crée des éléments de transition distincts que vous pouvez cibler avec des sélecteurs CSS.

Pour créer des transitions personnalisées :

1. **Ajoutez view-transition-name** - Attribuez des noms uniques aux éléments que vous voulez animer
2. **Définissez des animations globales** - Créez des animations CSS dans vos styles globaux
3. **Ciblez les pseudo-éléments de transition** - Utilisez les sélecteurs `::view-transition-old()` et `::view-transition-new()`

Voici un exemple qui ajoute un effet de rotation à un élément compteur :

```css
/* Define keyframe animations */
@keyframes rotate-out {
  to {
    transform: rotate(90deg);
  }
}

@keyframes rotate-in {
  from {
    transform: rotate(-90deg);
  }
}

/* Target view transition pseudo-elements */
::view-transition-old(count),
::view-transition-new(count) {
  animation-duration: 200ms;
  animation-name: -ua-view-transition-fade-in, rotate-in;
}

::view-transition-old(count) {
  animation-name: -ua-view-transition-fade-out, rotate-out;
}
```

IMPORTANT: Définissez les animations de view transition dans votre fichier de styles globaux, pas dans les styles des composants. L'[encapsulation des vues](/guide/components/styling#style-scoping) d'Angular limite la portée des styles de composant, ce qui les empêche de cibler correctement les pseudo-éléments de transition.

[Essayez l'exemple « count » mis à jour sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-fwn4i7?file=src%2Fmain.ts)

## Contrôle avancé des transitions avec onViewTransitionCreated {#advanced-transition-control-with-onviewtransitioncreated}

La fonctionnalité `withViewTransitions` accepte un objet d'options avec un callback `onViewTransitionCreated` pour un contrôle avancé des view transitions. Ce callback :

- S'exécute dans un [contexte d'injection](/guide/di/dependency-injection-context#run-within-an-injection-context)
- Reçoit un objet [`ViewTransitionInfo`](/api/router/ViewTransitionInfo) contenant :
  - L'instance de `ViewTransition` issue de `startViewTransition`
  - L'[`ActivatedRouteSnapshot`](/api/router/ActivatedRouteSnapshot) de la route que l'on quitte
  - L'[`ActivatedRouteSnapshot`](/api/router/ActivatedRouteSnapshot) de la route vers laquelle on navigue

Utilisez ce callback pour personnaliser le comportement de la transition selon le contexte de navigation. Par exemple, vous pouvez ignorer les transitions pour certains types de navigation :

```ts
import {inject} from '@angular/core';
import {Router, withViewTransitions, isActive, IsActiveMatchOptions} from '@angular/router';

withViewTransitions({
  onViewTransitionCreated: ({transition}) => {
    const router = inject(Router);
    const targetUrl = router.currentNavigation()!.finalUrl!;

    // Skip transition if only fragment or query params change
    const config: Partial<IsActiveMatchOptions> = {
      paths: 'exact',
      matrixParams: 'exact',
      fragment: 'ignored',
      queryParams: 'ignored',
    };

    const isTargetRouteCurrent = isActive(targetUrl, router, config);

    if (isTargetRouteCurrent()) {
      transition.skipTransition();
    }
  },
});
```

Cet exemple ignore la view transition quand la navigation ne modifie que le [fragment de l'URL ou les query parameters](/guide/routing/read-route-state#query-parameters) (comme les liens d'ancre au sein d'une même page). La méthode `skipTransition()` empêche l'animation tout en laissant la navigation se terminer.

## Respecter les préférences de mouvement de l'utilisateur {#respecting-the-users-motion-preferences}

Le router exécute les view transitions quel que soit le réglage [`prefers-reduced-motion`](https://developer.mozilla.org/en-US/docs/Web/CSS/@media/prefers-reduced-motion) de l'utilisateur. Une préférence pour un mouvement réduit ne signifie pas forcément que l'utilisateur ne veut aucun mouvement, le framework laisse donc le choix à l'application : vous pouvez [fournir des animations plus discrètes](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#react_to_the_reduced_motion_preference) à ces utilisateurs, ou ignorer entièrement la transition avec le callback `onViewTransitionCreated` :

```ts
withViewTransitions({
  onViewTransitionCreated: ({transition}) => {
    if (matchMedia('(prefers-reduced-motion: reduce)').matches) {
      transition.skipTransition();
    }
  },
});
```

## Exemples de l'explainer de Chrome adaptés à Angular {#examples-from-the-chrome-explainer-adapted-to-angular}

Les exemples suivants illustrent différentes techniques de view transition, adaptées de la documentation de l'équipe Chrome pour être utilisées avec le router Angular :

### Les éléments en transition n'ont pas besoin d'être le même élément du DOM {#transitioning-elements-dont-need-to-be-the-same-dom-element}

Les éléments peuvent passer en douceur d'un élément du DOM à un autre tant qu'ils partagent le même `view-transition-name`.

- [Explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#transitioning_elements_dont_need_to_be_the_same_dom_element)
- [Exemple Angular sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-dh8npr?file=src%2Fmain.ts)

### Animations d'entrée et de sortie personnalisées {#custom-entry-and-exit-animations}

Créez des animations uniques pour les éléments qui entrent dans le viewport ou en sortent pendant les transitions de route.

- [Explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#custom_entry_and_exit_transitions)
- [Exemple Angular sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-8kly3o)

### Mises à jour asynchrones du DOM et attente du contenu {#async-dom-updates-and-waiting-for-content}

Le router Angular privilégie les transitions immédiates plutôt que l'attente du chargement de contenu supplémentaire.

- [Explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#async_dom_updates_and_waiting_for_content)

NOTE: Le router Angular ne fournit aucun moyen de retarder les view transitions. Ce choix de conception évite que les pages deviennent non interactives pendant l'attente de contenu supplémentaire. Comme le souligne la documentation de Chrome : « Pendant ce temps, la page est figée, les délais doivent donc être réduits au minimum… dans certains cas, il vaut mieux éviter complètement le délai et utiliser le contenu dont vous disposez déjà. »

### Gérer plusieurs styles de view transition avec les types de view transition {#handle-multiple-view-transition-styles-with-view-transition-types}

Utilisez les types de view transition pour appliquer différents styles d'animation selon le contexte de navigation.

- [Explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#view-transition-types)
- [Exemple Angular sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-vxzcam)

### Gérer plusieurs styles de view transition avec un nom de classe sur la racine de la view transition (obsolète) {#handle-multiple-view-transition-styles-with-a-class-name-on-the-view-transition-root-deprecated}

Cette approche utilise des classes CSS sur l'élément racine de la transition pour contrôler les styles d'animation.

- [Explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#changing-on-navigation-type)
- [Exemple Angular sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-nmnzzg?file=src%2Fmain.ts)

### Effectuer une transition sans figer les autres animations {#transitioning-without-freezing-other-animations}

Conservez les autres animations de la page pendant les view transitions pour créer des expériences utilisateur plus dynamiques.

- [Explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#transitioning-without-freezing)
- [Exemple Angular sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-76kgww)

### Animer avec JavaScript {#animating-with-javascript}

Contrôlez les view transitions par programmation avec les API JavaScript pour des scénarios d'animation complexes.

- [Explainer de Chrome](https://developer.chrome.com/docs/web-platform/view-transitions/same-document#animating-with-javascript)
- [Exemple Angular sur StackBlitz](https://stackblitz.com/edit/stackblitz-starters-cklnkm)
