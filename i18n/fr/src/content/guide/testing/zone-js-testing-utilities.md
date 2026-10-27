<!-- source: ce459dc873e00fefe88f9142121f4ab2b99b8fa5 -->
# Utilitaires de test de Zone.js

Ce guide décrit des utilitaires de test principalement utilisés pour gérer et contrôler les tâches asynchrones dans les tests unitaires. Ces utilitaires sont essentiellement les utilitaires de mock d'horloge propres à Zone.js, particulièrement utiles pour contrôler le déroulement des opérations asynchrones dans les tests.

Pour les utilitaires de test généraux d'Angular, dont `TestBed` et `ComponentFixture`, consultez le guide [API utilitaires de test](guide/testing/utility-apis).

Voici un récapitulatif des fonctions propres à Zone.js :

| Fonction | Détails |
| :--------------------- | :----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `waitForAsync` | Suit les tâches asynchrones et ne termine les tests qu'une fois qu'il ne reste plus aucune microtâche ni macrotâche dans la zone de test. |
| `fakeAsync` | Exécute le corps d'un test \(`it`\) dans une _zone de test fakeAsync_ spéciale, ce qui permet d'écrire du code avec un flux de contrôle linéaire. |
| `tick` | Simule l'écoulement du temps et l'achèvement des activités asynchrones en attente en vidant les files de _timers_ et de _microtâches_ dans la _zone de test fakeAsync_. Le lecteur curieux et assidu pourra apprécier ce long article de blog, [« _Tasks, microtasks, queues and schedules_ »](https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules). Accepte un argument optionnel qui avance l'horloge virtuelle du nombre de millisecondes indiqué, en exécutant les activités asynchrones planifiées dans cet intervalle. |
| `discardPeriodicTasks` | Abandonne toutes les tâches périodiques (par exemple `setInterval`) créées dans la zone `fakeAsync`. |
| `flushMicrotasks` | Quand un test `fakeAsync()` se termine avec des _microtâches_ en attente, comme des promises non résolues, le test échoue avec un message d'erreur explicite. <br /> En général, un test doit attendre la fin des microtâches. Quand des microtâches en attente sont prévues, appelez `flushMicrotasks` pour vider la file des _microtâches_ et éviter l'erreur. |
