<!-- source: 8eb008d6e16d42a30f1d5e456bca1fc97d902a18 -->
# Réagir aux changements de signals avec effect

Maintenant que vous avez appris [à accéder aux éléments enfants avec des queries signal](/tutorials/signals/9-query-child-elements-with-signal-queries), voyons comment réagir aux changements de signals avec des effects. Les effects sont des fonctions qui s'exécutent automatiquement quand leurs dépendances changent, ce qui les rend parfaits pour des effets de bord comme la journalisation, la manipulation du DOM ou les appels d'API.

**Important : les effects doivent être la dernière API à laquelle vous recourez.** Préférez toujours `computed()` pour les valeurs dérivées et `linkedSignal()` pour les valeurs qui peuvent être à la fois dérivées et définies manuellement. Si vous vous retrouvez à copier des données d'un signal à un autre avec un effect, c'est le signe que vous devriez remonter votre source de vérité plus haut et utiliser `computed()` ou `linkedSignal()` à la place. Les effects sont surtout adaptés à la synchronisation de l'état des signals avec des API impératives qui ne reposent pas sur les signals.

Dans cette activité, vous allez apprendre à utiliser la fonction `effect()` à bon escient, pour des effets de bord légitimes qui réagissent aux changements de signals.

<hr />

Vous disposez d'une application de gestion de thème dont les signals sont déjà en place. Vous allez maintenant ajouter des effects pour réagir automatiquement aux changements de signals.

<docs-workflow>

<docs-step title="Importer la fonction effect {#import-effect-function}">
Ajoutez `effect` à vos imports existants.

```ts
// Add effect to existing imports
import {Component, signal, computed, effect, ChangeDetectionStrategy} from '@angular/core';
```

La fonction `effect` crée un effet de bord réactif qui s'exécute automatiquement quand l'un des signals qu'il lit change.
</docs-step>

<docs-step title="Créer un effect pour le stockage local {#create-an-effect-for-local-storage}">
Ajoutez un effect qui enregistre automatiquement le thème dans le stockage local quand il change.

```ts
constructor() {
  // Save theme to localStorage whenever it changes
  effect(() => {
    localStorage.setItem('theme', this.theme());
    console.log('Theme saved to localStorage:', this.theme());
  });
}
```

Cet effect s'exécute chaque fois que le signal du thème change, ce qui conserve automatiquement la préférence de l'utilisateur.
</docs-step>

<docs-step title="Créer un effect pour journaliser l'activité de l'utilisateur {#create-an-effect-for-logging-user-activity}">
Ajoutez un effect qui journalise la connexion ou la déconnexion de l'utilisateur.

```ts
constructor() {
  // ... previous effect

  // Log user activity changes
  effect(() => {
    const status = this.isLoggedIn() ? 'logged in' : 'logged out';
    const user = this.username();
    console.log(`User ${user} is ${status}`);
  });
}
```

Cet effect montre comment les effects peuvent lire plusieurs signals et réagir aux changements de n'importe lequel d'entre eux.
</docs-step>

<docs-step title="Créer un effect avec nettoyage {#create-an-effect-with-cleanup}">
Ajoutez un effect qui met en place un minuteur et le nettoie quand le composant est détruit.

```ts
constructor() {
  // ... previous effects

  // Timer effect with cleanup
  effect((onCleanup) => {
    const interval = setInterval(() => {
      console.log('Timer tick - Current theme:', this.theme());
    }, 5000);

    // Clean up the interval when the effect is destroyed
    onCleanup(() => {
      clearInterval(interval);
      console.log('Timer cleaned up');
    });
  });
}
```

Cet effect ne lit aucun signal pendant son exécution, car `this.theme()` n'est appelé que plus tard, dans le callback de `setInterval`. Il s'exécute une seule fois et ne se réexécute jamais : le nettoyage n'a donc lieu que lorsque le composant est détruit, par exemple quand l'aperçu se recharge.
</docs-step>

<docs-step title="Tester les effects {#test-the-effects}">
Ouvrez la console du navigateur et interagissez avec l'application :

- **Toggle Theme** : observez les enregistrements dans localStorage
- **Login/Logout** : observez la journalisation de l'activité de l'utilisateur
- **Watch Timer** : observez la journalisation périodique du thème toutes les 5 secondes

Les effects s'exécutent une fois à la création du composant : certains de ces logs apparaissent donc avant même que vous cliquiez sur quoi que ce soit. Ensuite, ils se réexécutent chaque fois que les signals qu'ils lisent changent.
</docs-step>

</docs-workflow>

Excellent ! Vous savez maintenant utiliser les effects avec les signals. Concepts clés à retenir :

- **Les effects sont réactifs** : ils s'exécutent automatiquement quand l'un des signals qu'ils lisent change
- **Uniquement pour les effets de bord** : parfaits pour la journalisation, la manipulation du DOM, les appels d'API et la synchronisation avec des API impératives
- **Nettoyage** : utilisez le callback `onCleanup` pour libérer des ressources comme des minuteurs ou des abonnements
- **Suivi automatique** : les effects suivent automatiquement les signals qu'ils lisent et se réexécutent quand ces signals changent

**Rappelez-vous : utilisez les effects avec parcimonie !** Les exemples de cette leçon (synchronisation avec localStorage, journalisation, minuteurs) sont des usages appropriés. Évitez les effects pour :

- Dériver des valeurs d'autres signals : utilisez plutôt `computed()`
- Créer un état dérivé modifiable : utilisez plutôt `linkedSignal()`
- Copier des données entre signals : restructurez votre code pour utiliser une source de vérité partagée

Les effects sont puissants, mais ils doivent rester votre dernier recours lorsque `computed()` et `linkedSignal()` ne peuvent pas répondre à votre cas d'usage.
