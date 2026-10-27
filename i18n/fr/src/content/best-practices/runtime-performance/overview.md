<!-- source: 2023383227550fdfc419341b5e75c393a6a11367 -->
# Optimisation des performances à l'exécution

Un rendu rapide est essentiel pour Angular, et nous avons conçu le framework avec de nombreuses optimisations pour vous aider à développer des applications performantes. Pour mieux comprendre les performances de votre application, nous proposons [Angular DevTools](tools/devtools) ainsi qu'un [guide vidéo](https://www.youtube.com/watch?v=FjyX_hkscII) sur l'utilisation de Chrome DevTools pour le profilage. Cette section présente les techniques d'optimisation des performances les plus courantes.

La **détection de changements** est le processus par lequel Angular vérifie si l'état de votre application a changé et si une partie du DOM doit être mise à jour. Dans les grandes lignes, Angular parcourt vos composants de haut en bas à la recherche de changements. Angular exécute périodiquement son mécanisme de détection de changements afin que les modifications du modèle de données soient reflétées dans la vue d'une application. La détection de changements peut être déclenchée soit manuellement, soit par un événement asynchrone (par exemple, une interaction de l'utilisateur ou la fin d'un XMLHttpRequest).

La détection de changements est très optimisée et performante, mais elle peut tout de même provoquer des ralentissements si l'application l'exécute trop fréquemment.

Dans ce guide, vous apprendrez à contrôler et à optimiser le mécanisme de détection de changements en ignorant certaines parties de votre application et en n'exécutant la détection de changements que lorsque c'est nécessaire.

Regardez cette vidéo si vous préférez en apprendre davantage sur l'optimisation des performances sous forme de média :

<docs-video src="https://www.youtube.com/embed/f8sA-i6gkGQ"/>
