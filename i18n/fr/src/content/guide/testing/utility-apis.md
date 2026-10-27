<!-- source: e6f3c70e3b5cd053730088551f02fb54d89c9905 -->
# API utilitaires de test

NOTE: Pendant la mise à jour de ce guide pour Vitest, certaines descriptions et certains exemples d'API utilitaires sont présentés dans le contexte de Karma/Jasmine. Nous travaillons activement à fournir des équivalents Vitest et des recommandations à jour lorsque c'est pertinent.

Cette page décrit les fonctionnalités de test d'Angular les plus utiles.

Les utilitaires de test d'Angular comprennent le `TestBed`, la `ComponentFixture` et quelques fonctions qui contrôlent l'environnement de test.
Les classes [`TestBed`](#testbed-class-summary) et [`ComponentFixture`](#the-componentfixture) sont traitées séparément.

Voici un récapitulatif des fonctions autonomes, par ordre d'utilité probable :

| Fonction | Détails |
| :----------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [`inject`] | Injecte un ou plusieurs services depuis l'injector actuel du `TestBed` dans une fonction de test. Elle ne peut pas injecter un service fourni par le composant lui-même. Consultez la discussion sur [debugElement.injector](guide/testing/components-scenarios#get-injected-services). |
| `getTestBed` | Récupère l'instance actuelle du `TestBed`. Généralement inutile, car les méthodes de classe statiques de `TestBed` suffisent le plus souvent. L'instance de `TestBed` expose quelques membres rarement utilisés qui ne sont pas disponibles sous forme de méthodes statiques. |

Pour gérer des scénarios asynchrones complexes ou tester d'anciennes applications basées sur Zone.js, consultez le guide [Utilitaires de test de Zone.js](guide/testing/zone-js-testing-utilities).

## Récapitulatif de la classe `TestBed` {#testbed-class-summary}

La classe `TestBed` est l'un des principaux utilitaires de test d'Angular.
Son API est assez vaste et peut sembler écrasante tant que vous ne l'avez pas explorée, petit à petit.
Lisez d'abord le début de ce guide pour acquérir les bases avant d'essayer d'assimiler l'API complète.

La définition de module passée à `configureTestingModule` est un sous-ensemble des propriétés de métadonnées de `@NgModule`.

```ts
type TestModuleMetadata = {
  providers?: any[];
  declarations?: any[];
  imports?: any[];
  schemas?: Array<SchemaMetadata | any[]>;
};
```

Chaque méthode de surcharge prend un `MetadataOverride<T>` où `T` est le type de métadonnées adapté à la méthode, c'est-à-dire le paramètre d'un `@NgModule`, `@Component`, `@Directive` ou `@Pipe`.

```ts
type MetadataOverride<T> = {
  add?: Partial<T>;
  remove?: Partial<T>;
  set?: Partial<T>;
};
```

L'API de `TestBed` se compose de méthodes de classe statiques qui mettent à jour ou référencent une instance _globale_ du `TestBed`.

En interne, toutes les méthodes statiques recouvrent des méthodes de l'instance de `TestBed` en cours d'exécution, qui est aussi renvoyée par la fonction `getTestBed()`.

Appelez les méthodes de `TestBed` _dans_ un `beforeEach()` pour garantir un nouveau départ avant chaque test.

Voici les méthodes statiques les plus importantes, par ordre d'utilité probable.

| Méthodes | Détails |
| :----------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `configureTestingModule` | Les shims de test établissent l'[environnement de test initial](guide/testing) et un module de test par défaut. Le module de test par défaut est configuré avec des déclarations de base et quelques substituts de services Angular dont tout testeur a besoin. <br /> Appelez `configureTestingModule` pour affiner la configuration du module de test pour un ensemble de tests donné, en ajoutant et en retirant des imports, des déclarations \(de composants, de directives et de pipes\) et des providers. |
| `compileComponents` | Compile le module de test de manière asynchrone une fois que vous avez fini de le configurer. Vous **devez** appeler cette méthode si _l'un_ des composants du module de test possède des ressources chargées de manière asynchrone (comme des blocs @defer). |
| `createComponent<T>` | Crée une instance d'un composant de type `T` à partir de la configuration actuelle du `TestBed`. Après l'appel à `createComponent`, la configuration du `TestBed` est figée pour la durée de la spec en cours. |
| `overrideComponent` | Remplace les métadonnées de la classe de composant donnée, qui peut être imbriquée profondément dans un module interne. |
| `overrideDirective` | Remplace les métadonnées de la classe de directive donnée, qui peut être imbriquée profondément dans un module interne. |
| `overridePipe` | Remplace les métadonnées de la classe de pipe donnée, qui peut être imbriquée profondément dans un module interne. |
| `overrideModule` | Remplace les métadonnées du `NgModule` donné. Rappelez-vous que des modules peuvent importer d'autres modules. La méthode `overrideModule` peut atteindre en profondeur le module de test actuel pour modifier l'un de ces modules internes. |
| `inject` | Récupère un service depuis l'injector actuel du `TestBed`. La fonction `inject` suffit souvent pour cela. Mais `inject` lève une erreur si elle ne peut pas fournir le service. <br /> Et si le service est optionnel ? <br /> La méthode `TestBed.inject()` prend un second paramètre optionnel, l'objet à renvoyer si Angular ne trouve pas le provider \(`null` dans cet exemple\) : `expect(TestBed.inject(NotProvided, null)).toBeNull();` Après l'appel à `TestBed.inject`, la configuration du `TestBed` est figée pour la durée de la spec en cours. |
| `initTestEnvironment` | Initialise l'environnement de test pour toute l'exécution des tests. <br /> Les shims de test l'appellent pour vous, il y a donc rarement une raison de l'appeler vous-même. <br /> Appelez cette méthode _une seule fois_. Pour changer ce comportement par défaut en cours d'exécution des tests, appelez d'abord `resetTestEnvironment`. <br /> Indiquez un module de test Angular par défaut et une `PlatformRef`. Des alternatives pour les plateformes autres que le navigateur sont disponibles sous la forme générale `@angular/platform-<platform_name>/testing`. |
| `resetTestEnvironment` | Réinitialise l'environnement de test initial, y compris le module de test par défaut. |

Quelques méthodes d'instance de `TestBed` ne sont pas couvertes par les méthodes _de classe_ statiques de `TestBed`.
Elles sont rarement nécessaires.

## La `ComponentFixture` {#the-componentfixture}

`TestBed.createComponent<T>` crée une instance du composant `T` et renvoie une `ComponentFixture` fortement typée pour ce composant.

Les propriétés et méthodes de `ComponentFixture` donnent accès au composant, à sa représentation DOM et à certains aspects de son environnement Angular.

### Propriétés de `ComponentFixture` {#componentfixture-properties}

Voici les propriétés les plus importantes pour les testeurs, par ordre d'utilité probable.

| Propriétés | Détails |
| :------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `componentInstance` | L'instance de la classe du composant créée par `TestBed.createComponent`. |
| `debugElement` | Le `DebugElement` associé à l'élément racine du composant. <br /> Le `debugElement` donne un aperçu du composant et de son élément DOM pendant les tests et le débogage. C'est une propriété essentielle pour les testeurs. Ses membres les plus intéressants sont présentés [ci-dessous](#debugelement). |
| `nativeElement` | L'élément DOM natif à la racine du composant. |
| `changeDetectorRef` | Le `ChangeDetectorRef` de la host view du composant, et non du composant lui-même. Consultez [Utiliser Zoneless dans `TestBed`](guide/zoneless#using-zoneless-in-testbed). |

### Méthodes de `ComponentFixture` {#componentfixture-methods}

Les méthodes de la _fixture_ amènent Angular à effectuer certaines tâches sur l'arbre de composants.
Appelez ces méthodes pour déclencher le comportement d'Angular en réponse à une action utilisateur simulée.

Voici les méthodes les plus utiles pour les testeurs.

| Méthodes | Détails |
| :------------------ | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `detectChanges` | Déclenche un cycle de détection de changements pour le composant. <br /> Appelez-la pour initialiser le composant \(elle appelle `ngOnInit`\) et, après que votre code de test a modifié les valeurs des propriétés liées par binding du composant. Angular ne voit pas que vous avez modifié `personComponent.name` et ne met pas à jour le binding `name` tant que vous n'appelez pas `detectChanges`. <br /> Exécute ensuite `checkNoChanges` pour confirmer qu'il n'y a pas de mises à jour circulaires, sauf si elle est appelée sous la forme `detectChanges(false)` ; |
| `autoDetectChanges` | Définissez-la à `true` quand vous voulez que la fixture détecte automatiquement les changements. <br /> Quand l'autodétection vaut `true`, la fixture de test appelle `detectChanges` immédiatement après la création du composant. Elle écoute ensuite les événements de zone pertinents et appelle `detectChanges` en conséquence. Quand votre code de test modifie directement les valeurs des propriétés du composant, vous devrez probablement encore appeler `fixture.detectChanges` pour déclencher les mises à jour du data binding. <br /> La valeur par défaut est `true` avec la détection de changements zoneless (la valeur par défaut du `TestBed`) et `false` avec la détection de changements basée sur Zone. |
| `checkNoChanges` | Effectue une passe de détection de changements pour s'assurer qu'il n'y a aucun changement en attente. Lève une exception s'il y en a. |
| `isStable` | Si la fixture est actuellement _stable_, renvoie `true`. S'il reste des tâches asynchrones non terminées, renvoie `false`. |
| `whenStable` | Renvoie une promise qui se résout quand la fixture est stable. <br /> Pour reprendre le test après la fin d'une activité asynchrone ou d'une détection de changements asynchrone, accrochez-vous à cette promise. Consultez [whenStable](guide/testing/components-scenarios#whenstable). |
| `destroy` | Déclenche la destruction du composant. |

#### `DebugElement` {#debugelement}

Le `DebugElement` donne des informations essentielles sur la représentation DOM du composant.

À partir du `DebugElement` du composant racine de test, renvoyé par `fixture.debugElement`, vous pouvez parcourir \(et interroger\) l'ensemble des sous-arbres d'éléments et de composants de la fixture.

Voici les membres de `DebugElement` les plus utiles pour les testeurs, par ordre approximatif d'utilité :

| Membres | Détails |
| :-------------------- | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `nativeElement` | L'élément DOM correspondant dans le navigateur |
| `query` | Appeler `query(predicate: Predicate<DebugElement>)` renvoie le premier `DebugElement` qui correspond au prédicat, à n'importe quelle profondeur du sous-arbre. |
| `queryAll` | Appeler `queryAll(predicate: Predicate<DebugElement>)` renvoie tous les `DebugElements` qui correspondent au prédicat, à n'importe quelle profondeur du sous-arbre. |
| `injector` | L'injector de dépendances du host. Par exemple, l'injector de l'instance du composant de l'élément racine. |
| `componentInstance` | L'instance de composant propre à l'élément, s'il en a une. |
| `context` | Un objet qui fournit le contexte parent de cet élément. Souvent une instance de composant ancêtre qui gouverne cet élément. <br /> Quand un élément est répété dans un bloc `@for`, le contexte est un `RepeaterContext` dont la propriété `$implicit` est la valeur de l'instance de la ligne. Par exemple, le `hero` dans `@for(hero of heroes; ...)`. |
| `children` | Les enfants `DebugElement` directs. Parcourez l'arbre en descendant dans `children`. `DebugElement` possède aussi `childNodes`, une liste d'objets `DebugNode`. `DebugElement` dérive des objets `DebugNode`, et il y a souvent plus de nœuds que d'éléments. Les testeurs peuvent généralement ignorer les nœuds simples. |
| `parent` | Le `DebugElement` parent. Null s'il s'agit de l'élément racine. |
| `name` | Le nom de balise de l'élément, s'il s'agit d'un élément. |
| `triggerEventHandler` | Déclenche l'événement par son nom s'il existe un listener correspondant dans la collection `listeners` de l'élément. Le second paramètre est l'_objet événement_ attendu par le handler. <br /> Si l'événement n'a pas de listener ou s'il y a un autre problème, envisagez d'appeler `nativeElement.dispatchEvent(eventObject)`. |
| `listeners` | Les callbacks attachés aux propriétés `@Output` du composant et/ou aux propriétés d'événement de l'élément. |
| `providerTokens` | Les jetons de recherche de l'injector de ce composant. Comprend le composant lui-même ainsi que les jetons que le composant liste dans ses métadonnées `providers`. |
| `references` | Dictionnaire des objets associés aux variables locales du template \(par exemple, `#foo`\), indexés par le nom de la variable locale. |

Les méthodes `DebugElement.query(predicate)` et `DebugElement.queryAll(predicate)` prennent un prédicat qui filtre le sous-arbre de l'élément source pour trouver les `DebugElement` correspondants.

Le prédicat est n'importe quelle méthode qui prend un `DebugElement` et renvoie une valeur _truthy_.
L'exemple suivant trouve tous les `DebugElements` ayant une référence à une variable locale de template nommée « content » :

```ts
// Filter for DebugElements with a #content reference
const contentRefs = el.queryAll((de) => de.references['content']);
```

La classe `By` d'Angular possède trois méthodes statiques pour les prédicats courants :

| Méthode statique | Détails |
| :------------------------ | :------------------------------------------------------------------------- |
| `By.all` | Renvoie tous les éléments |
| `By.css(selector)` | Renvoie les éléments correspondant aux selectors CSS |
| `By.directive(directive)` | Renvoie les éléments qu'Angular a associés à une instance de la classe de directive |

```ts
// Can find DebugElement either by css selector or by directive
const h2 = fixture.debugElement.query(By.css('h2'));
const directive = fixture.debugElement.query(By.directive(Highlight));
```
