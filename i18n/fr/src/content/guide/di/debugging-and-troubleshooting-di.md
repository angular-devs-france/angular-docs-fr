<!-- source: 8407acc0053a2b0547c0b0dccb4d22a88bfdc0af -->
# Déboguer et résoudre les problèmes d'injection de dépendances

Les problèmes d'injection de dépendances (DI) proviennent généralement d'erreurs de configuration, de problèmes de portée ou de modes d'utilisation incorrects. Ce guide vous aide à identifier et à résoudre les problèmes de DI courants que rencontrent les développeurs.

## Pièges courants et solutions {#common-pitfalls-and-solutions}

### Services indisponibles là où on les attend {#services-not-available-where-expected}

L'un des problèmes de DI les plus courants survient lorsque vous essayez d'injecter un service mais qu'Angular ne le trouve ni dans l'injector courant ni dans aucun injector parent. Cela se produit généralement lorsque le service est fourni dans la mauvaise portée ou n'est pas fourni du tout.

#### Portée du provider inadaptée {#provider-scope-mismatch}

Lorsque vous fournissez un service dans le tableau `providers` d'un composant, Angular crée une instance dans l'injector de ce composant. Cette instance n'est disponible que pour ce composant et ses enfants. Les composants parents et les composants frères ne peuvent pas y accéder, car ils utilisent des injectors différents.

```angular-ts {header: 'child-view.ts'}
import {Component} from '@angular/core';
import {DataStore} from './data-store';

@Component({
  selector: 'app-child',
  template: '<p>Child</p>',
  providers: [DataStore], // Only available in this component and its children
})
export class ChildView {}
```

```angular-ts {header: 'parent-view.ts'}
import {Component, inject} from '@angular/core';
import {DataStore} from './data-store';

@Component({
  selector: 'app-parent',
  template: '<app-child />',
})
export class ParentView {
  private dataService = inject(DataStore); // ERROR: Not available to parent
}
```

Angular ne recherche qu'en remontant la hiérarchie, jamais en descendant. Les composants parents ne peuvent pas accéder aux services fournis dans les composants enfants.

**Solution :** fournissez le service à un niveau plus élevé (application ou composant parent).

```ts {prefer}
import {Service} from '@angular/core';

@Service()
export class DataStore {
  // Available everywhere
}
```

TIP: `@Service` rend les services disponibles partout et permet le tree-shaking. Si vous ne voulez pas lui donner la portée de toute l'application, indiquez `autoProvided: false`.

#### Services et routes en lazy loading {#services-and-lazy-loaded-routes}

Lorsque vous fournissez un service dans le tableau `providers` d'une route en lazy loading, Angular crée un injector enfant pour cette route. Cet injector et ses services ne deviennent disponibles qu'après le chargement de la route. Les composants des parties de votre application chargées immédiatement ne peuvent pas accéder à ces services, car ils utilisent des injectors différents, qui existent avant la création de l'injector chargé en lazy loading.

```ts {header: 'feature.routes.ts'}
import {Routes} from '@angular/router';
import {FeatureClient} from './feature-client';

export const featureRoutes: Routes = [
  {
    path: 'feature',
    providers: [FeatureClient],
    loadComponent: () => import('./feature-view'),
  },
];
```

```angular-ts {header: 'eager-view.ts'}
import {Component, inject} from '@angular/core';
import {FeatureClient} from './feature-client';

@Component({
  selector: 'app-eager',
  template: '<p>Eager Component</p>',
})
export class EagerView {
  private featureService = inject(FeatureClient); // ERROR: Not available yet
}
```

Les routes en lazy loading créent des injectors enfants qui ne sont disponibles qu'après le chargement de la route.

NOTE: Par défaut, les injectors de route et leurs services persistent même après avoir quitté la route. Ils ne sont pas détruits avant la fermeture de l'application. Pour le nettoyage automatique des injectors de route inutilisés, consultez [personnaliser le comportement des routes](guide/routing/customizing-route-behavior#automatic-cleanup-of-unused-route-injectors).

**Solution :** utilisez `@Service` pour les services qui doivent être partagés au-delà des frontières du lazy loading.

```ts {prefer, header: 'Provide at root for shared services'}
import {Service} from '@angular/core';

@Service()
export class FeatureClient {
  // Available everywhere, including before lazy load
}
```

Si le service doit être chargé en lazy loading tout en restant disponible pour les composants chargés immédiatement, injectez-le uniquement là où il est nécessaire et utilisez l'injection optionnelle pour gérer sa disponibilité.

### Plusieurs instances au lieu de singletons {#multiple-instances-instead-of-singletons}

Vous attendez une instance partagée unique (singleton), mais vous obtenez des instances distinctes dans différents composants.

#### Fournir dans le composant plutôt qu'à la racine {#providing-in-component-instead-of-root}

Lorsque vous ajoutez un service au tableau `providers` d'un composant, Angular crée une nouvelle instance de ce service pour chaque instance du composant. Chaque composant obtient sa propre instance du service, ce qui signifie que les modifications faites dans un composant n'affectent pas l'instance du service des autres composants. C'est souvent inattendu lorsque vous voulez un état partagé dans toute votre application.

```angular-ts {avoid, header: 'Component-level provider creates multiple instances'}
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-profile',
  template: '<p>Profile</p>',
  providers: [UserClient], // Creates new instance per component!
})
export class UserProfile {
  private userService = inject(UserClient);
}

@Component({
  selector: 'app-settings',
  template: '<p>Settings</p>',
  providers: [UserClient], // Different instance!
})
export class UserSettings {
  private userService = inject(UserClient);
}
```

Chaque composant obtient sa propre instance de `UserClient`. Les modifications faites dans l'un n'affectent pas l'autre.

**Solution :** utilisez `@Service` pour les singletons.

```ts {prefer, header: 'Root-level singleton'}
import {Service} from '@angular/core';

@Service()
export class UserClient {
  // Single instance shared across all components
}
```

#### Quand plusieurs instances sont voulues {#when-multiple-instances-are-intentional}

Il arrive que vous vouliez des instances distinctes par composant, pour un état propre à chaque composant.

```angular-ts {header: 'Intentional: Component-scoped state'}
import {Injectable, signal, Component, inject} from '@angular/core';

@Injectable() // No providedIn - must be provided explicitly
export class FormStateStore {
  private formData = signal({});

  setData(data: any) {
    this.formData.set(data);
  }

  getData() {
    return this.formData();
  }
}

@Component({
  selector: 'app-user-form',
  template: '<form>...</form>',
  providers: [FormStateStore], // Each form gets its own state
})
export class UserForm {
  private formState = inject(FormStateStore);
}
```

Ce pattern est utile pour :

- La gestion de l'état d'un formulaire (chaque formulaire a un état isolé)
- La mise en cache propre à un composant
- Les données temporaires qui ne doivent pas être partagées

### Utilisation incorrecte de inject() {#incorrect-inject-usage}

La fonction `inject()` ne fonctionne que dans des contextes précis, pendant la construction de la classe et l'exécution d'une fabrique.

#### Utiliser inject() dans les lifecycle hooks {#using-inject-in-lifecycle-hooks}

Lorsque vous appelez la fonction `inject()` dans des lifecycle hooks comme `ngOnInit()`, `ngAfterViewInit()` ou `ngOnDestroy()`, Angular lève une erreur, car ces méthodes s'exécutent en dehors du contexte d'injection. Le contexte d'injection n'est disponible que pendant l'exécution synchrone de la construction de la classe, qui a lieu avant l'appel des lifecycle hooks.

```angular-ts {avoid, header: 'inject() in ngOnInit'}
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-profile',
  template: '<p>User: {{userName}}</p>',
})
export class UserProfile {
  userName = '';

  ngOnInit() {
    const userService = inject(UserClient); // ERROR: Not an injection context
    this.userName = userService.getUser().name;
  }
}
```

**Solution :** capturez les dépendances et dérivez les valeurs dans les initialiseurs de champs.

```angular-ts {prefer, header: 'Derive values in field initializers'}
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-profile',
  template: '<p>User: {{userName}}</p>',
})
export class UserProfile {
  private userService = inject(UserClient);
  userName = this.userService.getUser().name;
}
```

#### Utiliser l'Injector pour une injection différée {#using-the-injector-for-deferred-injection}

Lorsque vous devez récupérer des services en dehors d'un contexte d'injection, utilisez directement l'`Injector` capturé avec `injector.get()` :

```angular-ts
import {Component, inject, Injector} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-profile',
  template: '<button (click)="delayedLoad()">Load Later</button>',
})
export class UserProfile {
  private injector = inject(Injector);

  delayedLoad() {
    setTimeout(() => {
      const userService = this.injector.get(UserClient);
      console.log(userService.getUser());
    }, 1000);
  }
}
```

#### Utiliser runInInjectionContext pour les callbacks {#using-runininjectioncontext-for-callbacks}

Utilisez `runInInjectionContext()` lorsque vous devez permettre à **un autre code** d'appeler `inject()`. C'est utile lorsque vous acceptez des callbacks susceptibles d'utiliser l'injection de dépendances :

```angular-ts
import {Component, inject, Injector, input, runInInjectionContext} from '@angular/core';

@Component({
  selector: 'app-data-loader',
  template: '<button (click)="load()">Load</button>',
})
export class DataLoader {
  private injector = inject(Injector);
  onLoad = input<() => void>();

  load() {
    const callback = this.onLoad();
    if (callback) {
      // Enable the callback to use inject()
      runInInjectionContext(this.injector, callback);
    }
  }
}
```

La fonction `runInInjectionContext()` crée un contexte d'injection temporaire, ce qui permet au code du callback d'appeler `inject()`.

IMPORTANT: Capturez toujours les dépendances au niveau de la classe lorsque c'est possible. Utilisez `injector.get()` pour une simple récupération différée, et `runInInjectionContext()` uniquement lorsqu'un code externe doit appeler `inject()`.

TIP: Utilisez `assertInInjectionContext()` pour vérifier que votre code s'exécute dans un contexte d'injection valide. C'est utile lorsque vous créez des fonctions réutilisables qui appellent `inject()`. Consultez [Vérifier le contexte](guide/di/dependency-injection-context#asserts-the-context) pour plus de détails.

### Confusion entre providers et viewProviders {#providers-vs-viewproviders-confusion}

La différence entre `providers` et `viewProviders` a un impact dans les scénarios de projection de contenu.

#### Comprendre la différence {#understanding-the-difference}

**providers :** disponibles pour le template du composant ET pour tout contenu projeté dans le composant (ng-content).

**viewProviders :** disponibles uniquement pour le template du composant, PAS pour le contenu projeté.

```angular-ts {header: 'parent-view.ts'}
import {Component, inject} from '@angular/core';
import {ThemeStore} from './theme-store';

@Component({
  selector: 'app-parent',
  template: `
    <div>
      <p>Theme: {{ themeService.theme() }}</p>
      <ng-content />
    </div>
  `,
  providers: [ThemeStore], // Available to content children
})
export class ParentView {
  protected themeService = inject(ThemeStore);
}

@Component({
  selector: 'app-parent-view',
  template: `
    <div>
      <p>Theme: {{ themeService.theme() }}</p>
      <ng-content />
    </div>
  `,
  viewProviders: [ThemeStore], // NOT available to content children
})
export class ParentViewOnly {
  protected themeService = inject(ThemeStore);
}
```

```angular-ts {header: 'child-view.ts'}
import {Component, inject} from '@angular/core';
import {ThemeStore} from './theme-store';

@Component({
  selector: 'app-child',
  template: '<p>Child theme: {{theme()}}</p>',
})
export class ChildView {
  private themeService = inject(ThemeStore, {optional: true});
  theme = () => this.themeService?.theme() ?? 'none';
}
```

```angular-ts {header: 'app.ts'}
@Component({
  selector: 'app-root',
  template: `
    <app-parent>
      <app-child />
      <!-- Can access ThemeStore -->
    </app-parent>

    <app-parent-view>
      <app-child />
      <!-- Cannot access ThemeStore -->
    </app-parent-view>
  `,
})
export class App {}
```

**Lorsqu'il est projeté dans `app-parent` :** le composant enfant peut injecter `ThemeStore`, car `providers` le rend disponible pour le contenu projeté.

**Lorsqu'il est projeté dans `app-parent-view` :** le composant enfant ne peut pas injecter `ThemeStore`, car `viewProviders` le restreint au seul template du parent.

#### Choisir entre providers et viewProviders {#choosing-between-providers-and-viewproviders}

Utilisez `providers` lorsque :

- Le service doit être disponible pour le contenu projeté
- Vous voulez que les enfants de contenu accèdent au service
- Vous fournissez des services à usage général

Utilisez `viewProviders` lorsque :

- Le service ne doit être disponible que pour le template de votre composant
- Vous voulez masquer des détails d'implémentation au contenu projeté
- Vous fournissez des services internes qui ne doivent pas fuiter à l'extérieur

**Recommandation par défaut :** utilisez `providers`, sauf si vous avez une raison précise de restreindre l'accès avec `viewProviders`.

### Problèmes avec InjectionToken {#injectiontoken-issues}

Lorsqu'ils utilisent `InjectionToken` pour des dépendances qui ne sont pas des classes, les développeurs rencontrent souvent des problèmes liés à l'identité du jeton, à la sûreté du typage et à la configuration des providers. Ces problèmes viennent généralement de la façon dont JavaScript gère l'identité des objets et dont TypeScript infère les types.

#### Confusion sur l'identité des jetons {#token-identity-confusion}

Lorsque vous créez une nouvelle instance d'`InjectionToken`, JavaScript crée un objet unique en mémoire. Même si vous créez un autre `InjectionToken` avec exactement la même chaîne de description, il s'agit d'un objet complètement différent. Angular utilise l'identité de l'objet jeton (et non sa description) pour faire correspondre les providers aux points d'injection : des jetons ayant la même description mais des identités d'objet différentes ne peuvent donc pas accéder aux valeurs l'un de l'autre.

```ts {header: 'config.token.ts'}
import {InjectionToken} from '@angular/core';

export interface AppConfig {
  apiUrl: string;
}

export const APP_CONFIG = new InjectionToken<AppConfig>('app config');
```

```ts {header: 'app.config.ts'}
import {bootstrapApplication} from '@angular/platform-browser';
import {APP_CONFIG} from './config.token';

export const appConfig: AppConfig = {
  apiUrl: 'https://api.example.com',
};

bootstrapApplication(App, {
  providers: [{provide: APP_CONFIG, useValue: appConfig}],
});
```

```angular-ts {avoid, header: 'feature-view.ts'}
// Creating new token with same description
import {InjectionToken, inject, Component} from '@angular/core';
import {AppConfig} from './config.token';

const APP_CONFIG = new InjectionToken<AppConfig>('app config');

@Component({
  selector: 'app-feature',
  template: '<p>Feature</p>',
})
export class FeatureView {
  private config = inject(APP_CONFIG); // ERROR: Different token instance!
}
```

Même si les deux jetons ont la description `'app config'`, ce sont des objets différents. Angular compare les jetons par référence, et non par description.

**Solution :** importez la même instance de jeton.

```angular-ts {prefer, header: 'feature-view.ts'}
import {inject, Component} from '@angular/core';
import {APP_CONFIG, AppConfig} from './config.token';

@Component({
  selector: 'app-feature',
  template: '<p>API: {{config.apiUrl}}</p>',
})
export class FeatureView {
  protected config = inject(APP_CONFIG); // Works: Same token instance
}
```

TIP: Exportez toujours les jetons depuis un fichier partagé et importez-les partout où ils sont nécessaires. Ne créez jamais plusieurs instances d'`InjectionToken` avec la même description.

#### Essayer d'injecter des interfaces {#trying-to-inject-interfaces}

Lorsque vous définissez une interface TypeScript, elle n'existe que pendant la compilation, pour la vérification des types. TypeScript efface toutes les définitions d'interface lors de la compilation en JavaScript : à l'exécution, il n'existe donc aucun objet qu'Angular puisse utiliser comme jeton d'injection. Si vous essayez d'injecter un type interface, Angular n'a rien à faire correspondre avec la configuration des providers.

```angular-ts {avoid, header: "Can't inject interface"}
interface UserConfig {
  name: string;
  email: string;
}

@Component({
  selector: 'app-profile',
  template: '<p>Profile</p>',
})
export class UserProfile {
  // ERROR: Interfaces don't exist at runtime
  constructor(private config: UserConfig) {}
}
```

**Solution :** utilisez `InjectionToken` pour les types interface.

```angular-ts {prefer, header: 'Use InjectionToken for interfaces'}
import {InjectionToken, inject, Component} from '@angular/core';
import {bootstrapApplication} from '@angular/platform-browser';

interface UserConfig {
  name: string;
  email: string;
}

export const USER_CONFIG = new InjectionToken<UserConfig>('user configuration');

// Provide the configuration
bootstrapApplication(App, {
  providers: [
    {
      provide: USER_CONFIG,
      useValue: {name: 'Alice', email: 'alice@example.com'},
    },
  ],
});

// Inject using the token
@Component({
  selector: 'app-profile',
  template: '<p>User: {{config.name}}</p>',
})
export class UserProfile {
  protected config = inject(USER_CONFIG);
}
```

L'`InjectionToken` existe à l'exécution et peut servir à l'injection, tandis que l'interface `UserConfig` apporte la sûreté du typage pendant le développement.

### Dépendances circulaires {#circular-dependencies}

Les dépendances circulaires surviennent lorsque des services s'injectent mutuellement, ce qui crée un cycle qu'Angular ne peut pas résoudre. Pour des explications détaillées et des exemples de code, consultez [NG0200 : Circular dependency](errors/NG0200).

**Stratégies de résolution** (par ordre de préférence) :

1. **Restructurer** - Extrayez la logique partagée dans un troisième service pour briser le cycle
2. **Utiliser des événements** - Remplacez les dépendances directes par une communication à base d'événements (comme `Subject`)
3. **Injection différée** - Utilisez `Injector.get()` pour différer une dépendance (en dernier recours)

NOTE: N'utilisez pas `forwardRef()` pour les dépendances circulaires entre services : il ne résout que les imports circulaires dans les configurations de composants standalone.

## Déboguer la résolution des dépendances {#debugging-dependency-resolution}

### Comprendre le processus de résolution {#understanding-the-resolution-process}

Angular résout les dépendances en remontant la hiérarchie des injectors. Lorsqu'une `NullInjectorError` se produit, comprendre cet ordre de recherche vous aide à identifier où ajouter le provider manquant.

Angular recherche dans cet ordre :

1. **Element injector** - Le composant ou la directive courant(e)
2. **Element injectors parents** - En remontant l'arbre du DOM à travers les composants parents
3. **Injector d'environnement** - L'injector de la route ou de l'application
4. **NullInjector** - Lève une `NullInjectorError` si rien n'est trouvé

Lorsque vous voyez une `NullInjectorError`, c'est que le service n'est fourni à aucun niveau accessible au composant. Vérifiez que :

- Le service a `@Service()`, ou
- Le service a `@Injectable({providedIn: 'root'})`, ou
- Le service figure dans un tableau `providers` que le composant peut atteindre

Vous pouvez modifier ce comportement de recherche avec des modificateurs de résolution comme `self`, `skipSelf`, `host` et `optional`. Pour une présentation complète des règles de résolution et des modificateurs, consultez le [guide des injectors hiérarchiques](guide/di/hierarchical-dependency-injection).

### Se servir d'Angular DevTools {#using-angular-devtools}

Angular DevTools inclut un inspecteur de l'arbre des injectors qui visualise l'ensemble de la hiérarchie des injectors et montre quels providers sont disponibles à chaque niveau. Pour l'installation et l'utilisation générale, consultez la [documentation des injectors d'Angular DevTools](tools/devtools/injectors).

Lorsque vous déboguez des problèmes de DI, utilisez les DevTools pour répondre à ces questions :

- **Le service est-il fourni ?** Sélectionnez le composant dont l'injection échoue et vérifiez si le service apparaît dans la section Injector.
- **À quel niveau ?** Remontez l'arbre des composants pour trouver où le service est réellement fourni (au niveau du composant, de la route ou de l'application).
- **Plusieurs instances ?** Si un service singleton apparaît dans plusieurs injectors de composants, il est probablement fourni dans les tableaux `providers` de composants au lieu d'utiliser `@Service` ou `providedIn: 'root'`.

Si un service n'apparaît dans aucun injector, vérifiez qu'il a le décorateur `@Service` ou qu'il figure dans un tableau `providers`.

### Journaliser et tracer l'injection {#logging-and-tracing-injection}

Lorsque les DevTools ne suffisent pas, utilisez la journalisation pour tracer le comportement de l'injection.

#### Journaliser la création des services {#logging-service-creation}

Ajoutez des logs console dans les constructeurs des services pour voir quand les services sont créés.

```ts
import {Service} from '@angular/core';

@Service()
export class UserClient {
  constructor() {
    console.log('UserClient created');
    console.trace(); // Shows call stack
  }

  getUser() {
    return {name: 'Alice'};
  }
}
```

Lorsque le service est créé, vous voyez le message de log et une trace de pile montrant où l'injection a eu lieu.

**Ce qu'il faut vérifier :**

- Combien de fois le constructeur est-il appelé ? (une seule fois pour un singleton)
- À quel endroit du code est-il injecté ? (consultez la trace de pile)
- Est-il créé au moment attendu ? (démarrage de l'application ou lazy loading)

#### Vérifier la disponibilité d'un service {#checking-service-availability}

Utilisez l'injection optionnelle avec de la journalisation pour déterminer si un service est disponible.

```angular-ts
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-debug',
  template: '<p>Debug Component</p>',
})
export class DebugView {
  private userService = inject(UserClient, {optional: true});

  constructor() {
    if (this.userService) {
      console.log('UserClient available:', this.userService);
    } else {
      console.warn('UserClient NOT available');
      console.trace(); // Shows where we tried to inject
    }
  }
}
```

Ce pattern vous aide à vérifier si un service est disponible sans faire planter l'application.

#### Journaliser les modificateurs de résolution {#logging-resolution-modifiers}

Testez différentes stratégies de résolution avec de la journalisation.

```angular-ts
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-debug',
  template: '<p>Debug Component</p>',
  providers: [UserClient],
})
export class DebugView {
  // Try to get local instance
  private localService = inject(UserClient, {self: true, optional: true});

  // Try to get parent instance
  private parentService = inject(UserClient, {
    skipSelf: true,
    optional: true,
  });

  constructor() {
    console.log('Local instance:', this.localService);
    console.log('Parent instance:', this.parentService);
    console.log('Same instance?', this.localService === this.parentService);
  }
}
```

Cela vous montre quelles instances sont disponibles aux différents niveaux d'injector.

### Méthode de débogage {#debugging-workflow}

Lorsque la DI échoue, suivez cette approche systématique :

**Étape 1 : lisez le message d'erreur**

- Identifiez le code d'erreur (NG0200, NG0203, etc.)
- Lisez le chemin de dépendances
- Notez quel jeton a échoué

**Étape 2 : vérifiez les bases**

- Le service a-t-il `@Service` ou `@Injectable()` ?
- Si vous utilisez `@Injectable`, `providedIn` est-il correctement défini ?
- Les imports sont-ils corrects ?
- Le fichier est-il inclus dans la compilation ?

**Étape 3 : vérifiez le contexte d'injection**

- `inject()` est-il appelé dans un contexte valide ?
- Recherchez les problèmes d'asynchronisme (await, setTimeout, promises)
- Vérifiez le moment de l'appel (pas après la destruction)

**Étape 4 : utilisez les outils de débogage**

- Ouvrez Angular DevTools
- Examinez la hiérarchie des injectors
- Ajoutez des logs console dans les constructeurs
- Utilisez l'injection optionnelle pour tester la disponibilité

**Étape 5 : simplifiez et isolez**

- Supprimez les dépendances une par une
- Testez dans un composant minimal
- Vérifiez chaque niveau d'injector séparément
- Créez un cas de reproduction

## Référence des erreurs de DI {#di-error-reference}

Cette section fournit des informations détaillées sur les codes d'erreur de DI d'Angular que vous pouvez rencontrer. Servez-vous-en comme référence lorsque ces erreurs apparaissent dans votre console.

### NullInjectorError : No provider for [Service] {#nullinjectorerror-no-provider-for-service}

**Code d'erreur :** [NG0201](errors/NG0201)

Cette erreur se produit lorsqu'Angular ne trouve aucun provider pour un jeton dans la hiérarchie des injectors. Le message d'erreur inclut un chemin de dépendances qui montre où l'injection a été tentée.

```
NG0201: No provider found for `UserClient`. Path: AuthClient -> UserClient.
```

Le chemin de dépendances montre que `AuthClient` a tenté d'injecter `UserClient`, mais qu'aucun provider n'a été trouvé.

#### Décorateur `@Service ` ou `@Injectable` manquant {#missing-the-service--or-injectable-decorator}

La cause la plus fréquente est l'oubli du décorateur `@Service` ou `@Injectable()` sur une classe de service.

```ts {avoid, header: 'Missing decorator'}
export class UserClient {
  getUser() {
    return {name: 'Alice'};
  }
}
```

Angular a besoin du décorateur `@Service()` pour générer les métadonnées nécessaires à l'injection de dépendances.

```ts {prefer, header: 'Include @Service'}
import {Service} from '@angular/core';

@Service()
export class UserClient {
  getUser() {
    return {name: 'Alice'};
  }
}
```

NOTE: Les classes dont le constructeur ne prend aucun argument peuvent fonctionner sans `@Service()`, mais ce n'est pas recommandé. Incluez toujours le décorateur, par cohérence et pour éviter des problèmes lorsque vous ajouterez des dépendances plus tard.

#### Configuration providedIn manquante {#missing-providedin-configuration}

Un service peut avoir `@Injectable()` sans préciser où il doit être fourni.

```ts {avoid, header: 'No providedIn specified'}
import {Injectable} from '@angular/core';

@Injectable()
export class UserClient {
  getUser() {
    return {name: 'Alice'};
  }
}
```

Utilisez le décorateur `@Service` pour rendre le service disponible dans toute votre application.

```ts {prefer, header: 'Specify providedIn'}
import {Service} from '@angular/core';

@Service()
export class UserClient {
  getUser() {
    return {name: 'Alice'};
  }
}
```

Le décorateur `@Service` rend le service disponible dans toute l'application et permet le tree-shaking (le service est retiré du bundle s'il n'est jamais injecté).

#### Imports manquants dans un composant standalone {#standalone-component-missing-imports}

Dans Angular v20+ avec des composants standalone, vous devez importer ou fournir explicitement les dépendances dans chaque composant.

```angular-ts {avoid, header: 'Missing service import'}
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-profile',
  template: '<p>User: {{user.name}}</p>',
})
export class UserProfile {
  private userService = inject(UserClient); // ERROR: No provider
  user = this.userService.getUser();
}
```

Assurez-vous que le service utilise `@Service` ou ajoutez-le au tableau `providers` du composant.

```angular-ts {prefer, header: 'Service uses providedIn: root'}
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-profile',
  template: '<p>User: {{user.name}}</p>',
})
export class UserProfile {
  private userService = inject(UserClient); // Works: providedIn: 'root'
  user = this.userService.getUser();
}
```

#### Déboguer avec le chemin de dépendances {#debugging-with-the-dependency-path}

Le chemin de dépendances dans le message d'erreur montre la chaîne d'injections qui a mené à l'échec.

```
NG0201: No provider found for `LoggerStore`. Path: DataStore -> ApiClient -> LoggerStore.
```

Ce chemin vous indique que :

1. `DataStore` a injecté `ApiClient`
2. `ApiClient` a tenté d'injecter `LoggerStore`
3. Aucun provider n'a été trouvé pour `LoggerStore`

Commencez votre investigation par la fin de la chaîne (`LoggerStore`) et vérifiez qu'il est correctement configuré.

#### Vérifier la disponibilité d'un provider avec l'injection optionnelle {#checking-provider-availability-with-optional-injection}

Utilisez l'injection optionnelle pour vérifier si un provider existe sans lever d'erreur.

```angular-ts
import {Component, inject} from '@angular/core';
import {UserClient} from './user-client';

@Component({
  selector: 'app-debug',
  template: '<p>Service available: {{serviceAvailable}}</p>',
})
export class DebugView {
  private userService = inject(UserClient, {optional: true});
  serviceAvailable = this.userService !== null;
}
```

L'injection optionnelle renvoie `null` si aucun provider n'est trouvé, ce qui vous permet de gérer son absence proprement.

### NG0203 : inject() must be called from an injection context {#ng0203-inject-must-be-called-from-an-injection-context}

**Code d'erreur :** NG0203

Cette erreur se produit lorsque vous appelez `inject()` en dehors d'un contexte d'injection valide. Angular exige que `inject()` soit appelé de façon synchrone pendant la construction de la classe ou l'exécution d'une fabrique.

```
NG0203: The `UserClient` token injection failed. `inject()` function must
be called from an injection context such as a constructor, a factory
function, a field initializer, or a function used with
`runInInjectionContext`.
```

#### Contextes d'injection valides {#valid-injection-contexts}

Angular autorise `inject()` aux emplacements suivants :

1. **Initialiseurs de champs de classe**

   ```angular-ts
   import {Component, inject} from '@angular/core';
   import {UserClient} from './user-client';

   @Component({
     selector: 'app-profile',
     template: '<p>User: {{user.name}}</p>',
   })
   export class UserProfile {
     private userService = inject(UserClient); // Valid
     user = this.userService.getUser();
   }
   ```

2. **Constructeur de classe**

   ```angular-ts
   import {Component, inject} from '@angular/core';
   import {UserClient} from './user-client';

   @Component({
     selector: 'app-profile',
     template: '<p>User: {{user.name}}</p>',
   })
   export class UserProfile {
     private userService: UserClient;
     user: ReturnType<UserClient['getUser']>;

     constructor() {
       this.userService = inject(UserClient); // Valid
       this.user = this.userService.getUser();
     }
   }
   ```

3. **Fonctions de fabrique de providers**

   ```ts
   import {inject, InjectionToken} from '@angular/core';
   import {UserClient} from './user-client';

   export const GREETING = new InjectionToken<string>('greeting', {
     factory() {
       const userService = inject(UserClient); // Valid
       const user = userService.getUser();
       return `Hello, ${user.name}`;
     },
   });
   ```

4. **Dans runInInjectionContext()**

   ```angular-ts
   import {Component, inject, Injector, runInInjectionContext} from '@angular/core';
   import {UserClient} from './user-client';

   @Component({
     selector: 'app-profile',
     template: '<button (click)="loadUser()">Load User</button>',
   })
   export class UserProfile {
     private injector = inject(Injector);

     loadUser() {
       runInInjectionContext(this.injector, () => {
         const userService = inject(UserClient); // Valid
         console.log(userService.getUser());
       });
     }
   }
   ```

Voici d'autres contextes d'injection dans lesquels `inject()` fonctionne également :

- [provideAppInitializer](api/core/provideAppInitializer)
- [provideEnvironmentInitializer](api/core/provideEnvironmentInitializer)
- Les [route guards](guide/routing/route-guards) fonctionnelles
- Les [resolvers de données](guide/routing/data-resolvers) fonctionnels
- Les [resources](guide/routing/data-fetching-with-resources) de route

#### Quand cette erreur se produit {#when-this-error-occurs}

Cette erreur se produit lorsque vous :

- Appelez `inject()` dans des lifecycle hooks (`ngOnInit`, `ngAfterViewInit`, etc.)
- Appelez `inject()` après un `await` dans des fonctions async
- Appelez `inject()` dans des callbacks (`setTimeout`, `Promise.then()`, etc.)
- Appelez `inject()` en dehors de la phase de construction de la classe

Consultez la section « Utilisation incorrecte de inject() » pour des exemples détaillés et des solutions.

#### Solutions et contournements {#solutions-and-workarounds}

**Solution 1 :** capturez les dépendances dans les initialiseurs de champs (le cas le plus courant)

```ts
private userService = inject(UserClient) // Capture at class level
```

**Solution 2 :** utilisez `runInInjectionContext()` pour les callbacks

```ts
private injector = inject(Injector)

someCallback() {
  runInInjectionContext(this.injector, () => {
    const service = inject(MyClient)
  })
}
```

**Solution 3 :** passez les dépendances en paramètres au lieu de les injecter

```ts
// Instead of injecting inside a callback
setTimeout(() => {
  const service = inject(MyClient) // ERROR
}, 1000)

// Capture first, then use
private service = inject(MyClient)

setTimeout(() => {
  this.service.doSomething() // Use captured reference
}, 1000)
```

### NG0200 : Circular dependency detected {#ng0200-circular-dependency-detected}

**Code d'erreur :** NG0200

Cette erreur se produit lorsque deux services ou plus dépendent les uns des autres, ce qui crée une dépendance circulaire qu'Angular ne peut pas résoudre.

```
NG0200: Circular dependency detected for `AuthClient`. Path: AuthClient -> UserClient -> AuthClient.
```

Le chemin de dépendances montre le cycle : `AuthClient` dépend de `UserClient`, qui dépend à son tour de `AuthClient`.

#### Comprendre l'erreur {#understanding-the-error}

Angular crée les instances de services en appelant leurs constructeurs et en injectant leurs dépendances. Lorsque des services dépendent les uns des autres de façon circulaire, Angular ne peut pas déterminer lequel créer en premier.

#### Causes courantes {#common-causes}

- Dépendance circulaire directe (Service A → Service B → Service A)
- Dépendance circulaire indirecte (Service A → Service B → Service C → Service A)
- Cycles d'imports dans des fichiers de modules qui ont aussi des dépendances de services

#### Stratégies de résolution {#resolution-strategies}

Consultez la section « Dépendances circulaires » pour des exemples détaillés et des solutions :

1. **Restructurer** - Extrayez la logique partagée dans un troisième service (recommandé)
2. **Utiliser des événements** - Remplacez les dépendances directes par une communication à base d'événements
3. **Injection différée** - Utilisez `Injector.get()` pour différer une dépendance (en dernier recours)

N'utilisez PAS `forwardRef()` pour les dépendances circulaires entre services. Il ne résout que les imports circulaires dans les configurations de composants.

### Autres codes d'erreur de DI {#other-di-error-codes}

Pour des explications détaillées et des solutions concernant ces erreurs, consultez la [référence des erreurs Angular](errors) :

| Code d'erreur           | Description                                                                                |
| ----------------------- | ------------------------------------------------------------------------------------------ |
| [NG0204](errors/NG0204) | Impossible de résoudre tous les paramètres - décorateur `@Injectable()` manquant          |
| [NG0205](errors/NG0205) | Injector déjà détruit - accès à des services après la destruction du composant             |
| [NG0207](errors/NG0207) | EnvironmentProviders dans un mauvais contexte - utilisation de `provideHttpClient()` dans les providers d'un composant |

## Prochaines étapes {#next-steps}

Lorsque vous rencontrez des erreurs de DI, pensez à :

1. Lire attentivement le message d'erreur et le chemin de dépendances
2. Vérifier la configuration de base (décorateurs, `providedIn`, imports)
3. Vérifier le contexte d'injection et le moment de l'appel
4. Utiliser les DevTools et la journalisation pour investiguer
5. Simplifier et isoler le problème

Pour approfondir certains sujets de l'injection de dépendances, consultez :

- [Comprendre l'injection de dépendances](guide/di) - Concepts et patterns fondamentaux de la DI
- [Injection de dépendances hiérarchique](guide/di/hierarchical-dependency-injection) - Fonctionnement de la hiérarchie des injectors
- [Tester avec l'injection de dépendances](guide/testing) - Utiliser TestBed et simuler des dépendances
