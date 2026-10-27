<!-- source: 9047bee9cfe679edd15dfbdc274b5a98f1d45955 -->
# Rendre des templates d'un composant parent avec `ng-content`

`<ng-content>` est un élément spécial qui accepte du balisage ou un fragment de template et contrôle la manière dont les composants rendent le contenu. Il ne rend pas de véritable élément du DOM.

Voici un exemple de composant `BaseButton` qui accepte n'importe quel balisage de son parent.

```angular-ts {header:'base-button/base-button.ts'}
import {Component} from '@angular/core';

@Component({
  selector: 'button[baseButton]',
  template: `<ng-content />`,
})
export class BaseButton {}
```

```angular-ts {header:'app.ts'}
import {Component} from '@angular/core';
import {BaseButton} from './base-button';

@Component({
  selector: 'app-root',
  imports: [BaseButton],
  template: `<button baseButton>Next <span class="icon arrow-right"></span></button>`,
})
export class App {}
```

Pour plus de détails, consultez le [guide approfondi sur `<ng-content>`](/guide/components/content-projection) pour découvrir d'autres façons de tirer parti de ce pattern.
