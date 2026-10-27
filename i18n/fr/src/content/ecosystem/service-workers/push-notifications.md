<!-- source: 054b95826a968a7ab630c2df61e87cef395470c2 -->
# Notifications push

Les notifications push sont un moyen efficace d'impliquer les utilisateurs.
Grâce à la puissance des service workers, les notifications peuvent être délivrées sur un appareil même quand votre application n'a pas le focus.

Le service worker Angular permet d'afficher des notifications push et de gérer les événements de clic sur les notifications.

HELPFUL: Avec le service worker Angular, les interactions avec les notifications push sont gérées par le service `SwPush`.
Pour en savoir plus sur les API de navigateur concernées, consultez [Push API](https://developer.mozilla.org/docs/Web/API/Push_API) et [Using the Notifications API](https://developer.mozilla.org/docs/Web/API/Notifications_API/Using_the_Notifications_API).

## Payload de notification {#notification-payload}

Déclenchez des notifications push en envoyant un message avec un payload valide.
Consultez `SwPush` pour plus de détails.

HELPFUL: Dans Chrome, vous pouvez tester les notifications push sans backend.
Ouvrez Devtools -> Application -> Service Workers et utilisez le champ `Push` pour envoyer un payload de notification au format JSON.

## Gestion des clics sur les notifications {#notification-click-handling}

Le comportement par défaut de l'événement `notificationclick` consiste à fermer la notification et à notifier `SwPush.notificationClicks`.

Vous pouvez spécifier une opération supplémentaire à exécuter lors de `notificationclick` en ajoutant une propriété `onActionClick` à l'objet `data` et en fournissant une entrée `default`.
C'est particulièrement utile quand aucun client n'est ouvert au moment où l'on clique sur une notification.

```json
{
  "notification": {
    "title": "New Notification!",
    "data": {
      "onActionClick": {
        "default": {"operation": "openWindow", "url": "foo"}
      }
    }
  }
}
```

### Opérations {#operations}

Le service worker Angular prend en charge les opérations suivantes :

| Opérations                  | Détails                                                                                                                                          |
| :-------------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------- |
| `openWindow`                | Ouvre un nouvel onglet à l'URL spécifiée.                                                                                                        |
| `focusLastFocusedOrOpen`    | Donne le focus au dernier client qui l'a eu. Si aucun client n'est ouvert, ouvre un nouvel onglet à l'URL spécifiée.                             |
| `navigateLastFocusedOrOpen` | Donne le focus au dernier client qui l'a eu et le fait naviguer vers l'URL spécifiée. Si aucun client n'est ouvert, ouvre un nouvel onglet à l'URL spécifiée. |
| `sendRequest`               | Envoie une simple requête GET à l'URL spécifiée.                                                                                                 |

IMPORTANT: Les URL sont résolues par rapport à la portée d'enregistrement du service worker.<br />Si un élément `onActionClick` ne définit pas d'`url`, c'est la portée d'enregistrement du service worker qui est utilisée.

### Actions {#actions}

Les actions permettent de personnaliser la façon dont l'utilisateur peut interagir avec une notification.

Avec la propriété `actions`, vous pouvez définir un ensemble d'actions disponibles.
Chaque action est représentée par un bouton d'action sur lequel l'utilisateur peut cliquer pour interagir avec la notification.

De plus, avec la propriété `onActionClick` de l'objet `data`, vous pouvez associer chaque action à une opération à effectuer quand l'utilisateur clique sur le bouton d'action correspondant :

```json
{
  "notification": {
    "title": "New Notification!",
    "actions": [
      {"action": "foo", "title": "Open new tab"},
      {"action": "bar", "title": "Focus last"},
      {"action": "baz", "title": "Navigate last"},
      {"action": "qux", "title": "Send request in the background"},
      {"action": "other", "title": "Just notify existing clients"}
    ],
    "data": {
      "onActionClick": {
        "default": {"operation": "openWindow"},
        "foo": {"operation": "openWindow", "url": "/absolute/path"},
        "bar": {"operation": "focusLastFocusedOrOpen", "url": "relative/path"},
        "baz": {"operation": "navigateLastFocusedOrOpen", "url": "https://other.domain.com/"},
        "qux": {"operation": "sendRequest", "url": "https://yet.another.domain.com/"}
      }
    }
  }
}
```

IMPORTANT: Si une action n'a pas d'entrée `onActionClick` correspondante, la notification est fermée et `SwPush.notificationClicks` est notifié sur les clients existants.

## En savoir plus sur les service workers Angular {#more-on-angular-service-workers}

Ces sujets pourraient aussi vous intéresser :

<docs-pill-row>
  <docs-pill href="ecosystem/service-workers/communications" title="Communiquer avec le service worker"/>
  <docs-pill href="ecosystem/service-workers/devops" title="DevOps du service worker"/>
</docs-pill-row>
