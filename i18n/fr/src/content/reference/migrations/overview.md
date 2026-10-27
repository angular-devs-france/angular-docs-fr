<!-- source: f481bd167635705249fa48360c93c982f85f5f89 -->
# Migrations

Découvrez comment migrer progressivement votre projet Angular existant vers les dernières fonctionnalités.

<docs-card-container>
  <docs-card title="Standalone" link="Migrer maintenant" href="reference/migrations/standalone">
    Les composants standalone offrent une façon simplifiée de construire des applications Angular. Les composants standalone déclarent leurs dépendances directement au lieu de les obtenir via des NgModules.
  </docs-card>
  <docs-card title="Syntaxe du control flow" link="Migrer maintenant" href="reference/migrations/control-flow">
    La syntaxe intégrée du control flow vous permet d'utiliser une syntaxe plus ergonomique, proche de JavaScript, avec une meilleure vérification de types. Elle évite d'avoir à importer `CommonModule` pour utiliser des fonctionnalités comme `*ngFor`, `*ngIf` et `*ngSwitch`.
  </docs-card>
  <docs-card title="Fonction inject()" link="Migrer maintenant" href="reference/migrations/inject-function">
    La fonction [`inject`](/api/core/inject) d'Angular offre des types plus précis et une meilleure compatibilité avec les décorateurs standards que l'injection par constructeur.
  </docs-card>
  <docs-card title="Routes en lazy loading" link="Migrer maintenant" href="reference/migrations/route-lazy-loading">
    Convertissez les routes de composants chargées immédiatement en routes chargées en lazy loading. Le processus de build peut ainsi découper les bundles de production en chunks plus petits, pour charger moins de JavaScript au chargement initial de la page.
  </docs-card>
  <docs-card title="Nouvelle API `input()`" link="Migrer maintenant" href="reference/migrations/signal-inputs">
    Convertissez les champs `@Input` existants vers la nouvelle API des inputs signal, désormais prête pour la production.
  </docs-card>
  <docs-card title="Nouvelle fonction `output()`" link="Migrer maintenant" href="reference/migrations/outputs">
    Convertissez les événements personnalisés `@Output` existants vers la nouvelle fonction output, désormais prête pour la production.
  </docs-card>
  <docs-card title="Queries signal" link="Migrer maintenant" href="reference/migrations/signal-queries">
    Convertissez les champs de query existants basés sur des décorateurs vers l'API améliorée des queries signal. L'API est désormais prête pour la production.
  </docs-card>
  <docs-card title="Nettoyer les imports inutilisés" link="Essayer maintenant" href="reference/migrations/cleanup-unused-imports">
    Nettoyez les imports inutilisés de votre projet.
  </docs-card>
  <docs-card title="Balises auto-fermantes" link="Migrer maintenant" href="reference/migrations/self-closing-tags">
    Convertissez les templates des composants pour utiliser des balises auto-fermantes quand c'est possible.
  </docs-card>
  <docs-card title="De NgClass aux class bindings" link="Migrer maintenant" href="reference/migrations/ngclass-to-class">
      Convertissez les templates des composants pour privilégier les class bindings plutôt que la directive `NgClass` quand c'est possible.
  </docs-card>
  <docs-card title="De NgStyle aux style bindings" link="Migrer maintenant" href="reference/migrations/ngstyle-to-style">
      Convertissez les templates des composants pour privilégier les style bindings plutôt que la directive `NgStyle` quand c'est possible.
  </docs-card>
  <docs-card title="Migration de RouterTestingModule" link="Migrer maintenant" href="reference/migrations/router-testing-module-migration">
    Convertissez les utilisations de `RouterTestingModule` en `RouterModule` dans les configurations du TestBed et ajoutez `provideLocationMocks()` si nécessaire.
  </docs-card>
  <docs-card title="De CommonModule aux imports standalone" link="Migrer maintenant" href="reference/migrations/common-to-standalone">
    Remplacez les imports de `CommonModule` par des imports des directives et pipes individuels utilisés dans les templates quand c'est possible.
  </docs-card>
  <docs-card title="De Injectable à Service" link="Migrer maintenant" href="reference/migrations/injectable-to-service">
    Convertissez les classes `@Injectable` éligibles vers le décorateur `@Service`.
  </docs-card>
</docs-card-container>
