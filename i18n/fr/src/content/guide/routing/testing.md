<!-- source: ab0fabbc02fd84239e5906b7510a0bba2dc566a4 -->
# Tester le routing et la navigation

Tester le routing et la navigation est essentiel pour garantir que votre application se comporte correctement quand les utilisateurs naviguent entre différentes routes. Ce guide présente plusieurs stratégies pour tester les fonctionnalités de routing dans les applications Angular.

## Prérequis {#prerequisites}

Ce guide suppose que vous connaissez les outils et libraries suivants :

- **[Vitest](https://vitest.dev/)** - Framework de test JavaScript qui fournit la syntaxe de test (`describe`, `it`, `expect`)
- **[Utilitaires de test d'Angular](guide/testing)** - Les outils de test intégrés d'Angular ([`TestBed`](api/core/testing/TestBed), [`ComponentFixture`](api/core/testing/ComponentFixture))
- **[`RouterTestingHarness`](api/router/testing/RouterTestingHarness)** - Harnais de test pour tester les composants routés, avec des fonctionnalités intégrées de navigation et de test de composants

## Scénarios de test {#testing-scenarios}

### Paramètres de route {#route-parameters}

Les composants s'appuient souvent sur les paramètres de route de l'URL pour récupérer des données, comme un ID d'utilisateur pour une page de profil.

L'exemple suivant montre comment tester un composant `UserProfile` qui affiche un ID d'utilisateur issu de la route.

```ts { header: 'user-profile.spec.ts'}
import {TestBed} from '@angular/core/testing';
import {provideRouter, Router} from '@angular/router';
import {RouterTestingHarness} from '@angular/router/testing';
import {UserProfile} from './user-profile';

describe('UserProfile', () => {
  it('should display user ID from route parameters', async () => {
    TestBed.configureTestingModule({
      imports: [UserProfile],
      providers: [provideRouter([{path: 'user/:id', component: UserProfile}])],
    });

    const harness = await RouterTestingHarness.create();
    await harness.navigateByUrl('/user/123', UserProfile);

    expect(harness.routeNativeElement?.textContent).toContain('User Profile: 123');
  });
});
```

```angular-ts {header: 'user-profile.ts'}
import {Component, inject} from '@angular/core';
import {ActivatedRoute} from '@angular/router';

@Component({
  template: '<h1>User Profile: {{userId}}</h1>',
})
export class UserProfile {
  private route = inject(ActivatedRoute);
  userId: string | null = this.route.snapshot.paramMap.get('id');
}
```

### Guards de route {#route-guards}

Les guards de route contrôlent l'accès aux routes selon des conditions comme l'authentification ou les permissions. Pour tester des guards, concentrez-vous sur le mock des dépendances et la vérification du résultat de la navigation.

L'exemple suivant teste une guard `authGuard` qui autorise la navigation pour les utilisateurs authentifiés et redirige les utilisateurs non authentifiés vers une page de connexion.

```ts {header: 'auth.guard.spec.ts'}
import {describe, expect, it, vi, type Mocked} from 'vitest';
import {RouterTestingHarness} from '@angular/router/testing';
import {provideRouter, Router} from '@angular/router';
import {authGuard} from './auth.guard';
import {AuthStore} from './auth-store';
import {Component} from '@angular/core';
import {TestBed} from '@angular/core/testing';

@Component({template: '<h1>Protected Page</h1>'})
class Protected {}

@Component({template: '<h1>Login Page</h1>'})
class Login {}

describe('authGuard', () => {
  let authStore: Mocked<AuthStore>;
  let harness: RouterTestingHarness;

  async function setup(isAuthenticated: boolean) {
    authStore = {isAuthenticated: vi.fn().mockReturnValue(isAuthenticated)} as Mocked<AuthStore>;

    TestBed.configureTestingModule({
      providers: [
        {provide: AuthStore, useValue: authStore},
        provideRouter([
          {path: 'protected', component: Protected, canActivate: [authGuard]},
          {path: 'login', component: Login},
        ]),
      ],
    });

    harness = await RouterTestingHarness.create();
  }

  it('allows navigation when user is authenticated', async () => {
    await setup(true);
    await harness.navigateByUrl('/protected', Protected);
    // The protected component should render when authenticated
    expect(harness.routeNativeElement?.textContent).toContain('Protected Page');
  });

  it('redirects to login when user is not authenticated', async () => {
    await setup(false);
    await harness.navigateByUrl('/protected', Login);
    // The login component should render after redirect
    expect(harness.routeNativeElement?.textContent).toContain('Login Page');
  });
});
```

```ts {header: 'auth.guard.ts'}
import {inject} from '@angular/core';
import {CanActivateFn, Router} from '@angular/router';
import {AuthStore} from './auth-store';

export const authGuard: CanActivateFn = () => {
  const authStore = inject(AuthStore);
  const router = inject(Router);
  return authStore.isAuthenticated() ? true : router.parseUrl('/login');
};
```

### Router outlets {#router-outlets}

Les tests de router outlet relèvent davantage du test d'intégration, puisque vous testez essentiellement l'intégration entre le [`Router`](api/router/Router), l'outlet et les composants affichés.

Voici un exemple de mise en place d'un test qui vérifie que des composants différents sont affichés pour des routes différentes :

```ts {header: 'app.spec.ts'}
import {TestBed} from '@angular/core/testing';
import {RouterTestingHarness} from '@angular/router/testing';
import {provideRouter} from '@angular/router';
import {Component} from '@angular/core';
import {App} from './app';

@Component({
  template: '<h1>Home Page</h1>',
})
class MockHome {}

@Component({
  template: '<h1>About Page</h1>',
})
class MockAbout {}

describe('App Router Outlet', () => {
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [App],
      providers: [
        provideRouter([
          {path: '', component: MockHome},
          {path: 'about', component: MockAbout},
        ]),
      ],
    });

    harness = await RouterTestingHarness.create();
  });

  it('should display home component for default route', async () => {
    await harness.navigateByUrl('');

    expect(harness.routeNativeElement?.textContent).toContain('Home Page');
  });

  it('should display about component for about route', async () => {
    await harness.navigateByUrl('/about');

    expect(harness.routeNativeElement?.textContent).toContain('About Page');
  });
});
```

```angular-ts {header: 'app.ts'}
import {Component} from '@angular/core';
import {RouterOutlet, RouterLink} from '@angular/router';

@Component({
  imports: [RouterOutlet, RouterLink],
  template: `
    <nav>
      <a routerLink="/">Home</a>
      <a routerLink="/about">About</a>
    </nav>
    <router-outlet />
  `,
})
export class App {}
```

### Routes imbriquées {#nested-routes}

Tester les routes imbriquées garantit que les composants parent et enfant s'affichent correctement lors de la navigation vers des URL imbriquées. C'est important, car les routes imbriquées impliquent plusieurs niveaux.

Vous devez vérifier que :

1. Le composant parent s'affiche correctement.
2. Le composant enfant s'affiche à l'intérieur.
3. Les deux composants peuvent accéder à leurs données de route respectives.

Voici un exemple de test d'une structure de routes parente-enfant :

```ts {header: 'nested-routes.spec.ts'}
import {TestBed} from '@angular/core/testing';
import {RouterTestingHarness} from '@angular/router/testing';
import {provideRouter} from '@angular/router';
import {Parent, Child} from './nested-components';

describe('Nested Routes', () => {
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [Parent, Child],
      providers: [
        provideRouter([
          {
            path: 'parent',
            component: Parent,
            children: [{path: 'child', component: Child}],
          },
        ]),
      ],
    });

    harness = await RouterTestingHarness.create();
  });

  it('should render parent and child components for nested route', async () => {
    await harness.navigateByUrl('/parent/child');

    expect(harness.routeNativeElement?.textContent).toContain('Parent Component');
    expect(harness.routeNativeElement?.textContent).toContain('Child Component');
  });
});
```

```angular-ts {header: 'nested.ts'}
import {Component} from '@angular/core';
import {RouterOutlet} from '@angular/router';

@Component({
  imports: [RouterOutlet],
  template: `
    <h1>Parent Component</h1>
    <router-outlet />
  `,
})
export class Parent {}

@Component({
  template: '<h2>Child Component</h2>',
})
export class Child {}
```

### Query parameters et fragments {#query-parameters-and-fragments}

Les query parameters (comme `?search=angular&category=web`) et les fragments d'URL (comme `#section1`) fournissent des données supplémentaires via l'URL qui n'influent pas sur le composant chargé, mais qui influent sur son comportement. Les composants qui lisent des query parameters via [`ActivatedRoute.queryParams`](api/router/ActivatedRoute#queryParams) doivent être testés pour garantir qu'ils gèrent correctement les différents cas de paramètres.

Contrairement aux paramètres de route, qui font partie de la définition de la route, les query parameters sont facultatifs et peuvent changer sans déclencher de navigation de route. Vous devez donc tester à la fois le chargement initial et les mises à jour réactives quand les query parameters changent.

Voici un exemple de test des query parameters et des fragments :

```ts {header: 'search.spec.ts'}
import {TestBed} from '@angular/core/testing';
import {Router, provideRouter} from '@angular/router';
import {RouterTestingHarness} from '@angular/router/testing';
import {Search} from './search';

describe('Search', () => {
  let component: Search;
  let harness: RouterTestingHarness;

  beforeEach(async () => {
    TestBed.configureTestingModule({
      imports: [Search],
      providers: [provideRouter([{path: 'search', component: Search}])],
    });

    harness = await RouterTestingHarness.create();
  });

  it('should read search term from query parameters', async () => {
    component = await harness.navigateByUrl('/search?q=angular', Search);

    expect(component.searchTerm()).toBe('angular');
  });
});
```

```angular-ts {header: 'search.ts'}
import {Component, inject, computed} from '@angular/core';
import {ActivatedRoute} from '@angular/router';
import {toSignal} from '@angular/core/rxjs-interop';

@Component({
  template: '<div>Search term: {{searchTerm()}}</div>',
})
export class Search {
  private route = inject(ActivatedRoute);
  private queryParams = toSignal(this.route.queryParams, {initialValue: {}});

  searchTerm = computed(() => this.queryParams()['q'] || null);
}
```

## Bonnes pratiques pour tester le router {#best-practices-for-router-testing}

1. **Utilisez RouterTestingHarness** - Pour tester des composants routés, utilisez [`RouterTestingHarness`](api/router/testing/RouterTestingHarness), qui fournit une API plus claire et évite d'avoir recours à des composants host de test. Il offre un accès direct au composant, une navigation intégrée et une meilleure sûreté de typage. Il est toutefois moins adapté à certains scénarios, comme le test d'outlets nommés, pour lesquels vous devrez peut-être créer des composants host personnalisés.
2. **Gérez les dépendances externes avec discernement** - Privilégiez les implémentations réelles quand c'est possible pour des tests plus réalistes. Si les implémentations réelles ne sont pas envisageables (par ex. des API externes), utilisez des fakes qui reproduisent approximativement le comportement réel. N'utilisez des mocks ou des stubs qu'en dernier recours, car ils peuvent rendre les tests fragiles et moins fiables.
3. **Testez l'état de navigation** - Vérifiez à la fois l'action de navigation et l'état de l'application qui en résulte, y compris les changements d'URL et l'affichage des composants.
4. **Gérez les opérations asynchrones** - La navigation du router est asynchrone. Utilisez `async/await` pour gérer correctement le timing dans vos tests.
5. **Testez les scénarios d'erreur** - Incluez des tests pour les routes invalides, les navigations échouées et les rejets de guards afin de garantir que votre application gère les cas limites sans accroc.
6. **Ne mockez pas le router d'Angular** - Fournissez plutôt de vraies configurations de routes et utilisez le harnais pour naviguer. Vos tests seront ainsi plus robustes et moins susceptibles de casser lors des mises à jour internes d'Angular, tout en vous assurant de détecter les vrais problèmes lors des mises à jour du router, car les mocks peuvent masquer des breaking changes.
