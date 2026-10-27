<!-- source: 296764274318b88799568f09d297aca7d6ce1033 -->
# Utiliser les signals avec les directives

Maintenant que vous avez appris [à utiliser les signals avec les services](/tutorials/signals/7-using-signals-with-services), voyons comment les directives utilisent les signals. **Bonne nouvelle : les signals fonctionnent exactement de la même manière dans les directives que dans les composants !** La principale différence est que, les directives n'ayant pas de template, vous utiliserez surtout les signals dans des host bindings pour mettre à jour le host element de manière réactive.

Dans cette activité, vous allez construire une directive de surlignage qui montre comment les signals créent un comportement réactif dans les directives.

<hr />

<docs-workflow>

<docs-step title="Mettre en place les signals comme dans un composant {#set-up-signals-just-like-in-a-component}">
Importez les fonctions de signals et créez votre état réactif. Cela fonctionne exactement comme dans les composants :

```ts
import {Directive, input, signal, computed} from '@angular/core';

@Directive({
  selector: '[highlight]',
})
export class HighlightDirective {
  // Signal inputs - same as components!
  color = input<string>('yellow');
  intensity = input<number>(0.3);

  // Internal state - same as components!
  private isHovered = signal(false);

  // Computed signals - same as components!
  backgroundStyle = computed(() => {
    const baseColor = this.color();
    const alpha = this.isHovered() ? this.intensity() : this.intensity() * 0.5;

    const colorMap: Record<string, string> = {
      'yellow': `rgba(255, 255, 0, ${alpha})`,
      'blue': `rgba(0, 100, 255, ${alpha})`,
      'green': `rgba(0, 200, 0, ${alpha})`,
      'red': `rgba(255, 0, 0, ${alpha})`,
    };

    return colorMap[baseColor] || colorMap['yellow'];
  });
}
```

Remarquez que c'est identique aux patterns des composants : la seule différence est que nous sommes dans une `@Directive` au lieu d'un `@Component`.
</docs-step>

<docs-step title="Utiliser les signals dans les host bindings {#use-signals-in-host-bindings}">
Comme les directives n'ont pas de template, vous utiliserez les signals dans des **host bindings** pour mettre à jour le host element de manière réactive. Ajoutez la configuration `host` et les event handlers :

```ts
@Directive({
  selector: '[highlight]',
  host: {
    '[style.backgroundColor]': 'backgroundStyle()',
    '(mouseenter)': 'onMouseEnter()',
    '(mouseleave)': 'onMouseLeave()',
  },
})
export class HighlightDirective {
  // ... signals from previous step ...

  onMouseEnter() {
    this.isHovered.set(true);
  }

  onMouseLeave() {
    this.isHovered.set(false);
  }
}
```

Les host bindings sont automatiquement réévalués quand les signals changent, exactement comme les bindings de template dans les composants ! Quand `isHovered` change, le signal computed `backgroundStyle` se recalcule, et le host binding met à jour le style de l'élément.
</docs-step>

<docs-step title="Utiliser la directive dans votre template {#use-the-directive-in-your-template}">
Mettez à jour le template de l'application pour montrer la directive réactive :

```angular-html
template: `
<div>
  <h1>Directive with Signals</h1>

  <div highlight color="yellow" [intensity]="0.2">Hover me - Yellow highlight</div>

  <div highlight color="blue" [intensity]="0.4">Hover me - Blue highlight</div>

  <div highlight color="green" [intensity]="0.6">Hover me - Green highlight</div>
</div>
`,
```

La directive applique automatiquement un surlignage réactif en fonction des inputs signal !
</docs-step>

</docs-workflow>

Parfait ! Vous avez vu comment les signals fonctionnent avec les directives. Voici quelques points clés de cette leçon :

- **Les signals sont universels** : toutes les API de signals (`input()`, `signal()`, `computed()`, `effect()`) fonctionnent de la même manière dans les directives et dans les composants
- **Les host bindings sont le cas d'usage principal** : comme les directives n'ont pas de template, vous utilisez les signals dans des host bindings pour modifier le host element de manière réactive
- **Les mêmes patterns réactifs** : les mises à jour de signals déclenchent la réévaluation automatique des signals computed et des host bindings, exactement comme dans les templates de composants

Dans la prochaine leçon, vous [apprendrez à accéder aux éléments enfants avec des queries signal](/tutorials/signals/9-query-child-elements-with-signal-queries) !
