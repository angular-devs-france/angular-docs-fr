<!-- source: c40d0d463e80fe467ded67a73b7b44fc0c9da7f6 -->
# Déboguer les tests

Si vos tests ne fonctionnent pas comme prévu, vous pouvez les déboguer aussi bien dans l'environnement Node.js par défaut que dans un vrai navigateur.

## Déboguer dans Node.js {#debugging-in-nodejs}

Le débogage dans l'environnement Node.js par défaut est souvent le moyen le plus rapide de diagnostiquer les problèmes qui ne sont pas liés aux API propres au navigateur ni au rendu.

1.  Exécutez la commande `ng test` avec le flag `--debug` :
    ```shell
    ng test --debug
    ```
2.  Le test runner démarre en mode débogage et attend qu'un débogueur s'y attache.
3.  Vous pouvez maintenant attacher le débogueur de votre choix. Par exemple, vous pouvez utiliser le débogueur Node.js intégré à VS Code ou les Chrome DevTools pour Node.js.

## Déboguer dans un navigateur {#debugging-in-a-browser}

De la même manière que vous démarrez une session de débogage dans Node, vous pouvez utiliser `ng test` avec le flag `--debug` avec Vitest et le [mode navigateur](/guide/testing/migrating-to-vitest#5-configure-browser-mode-optional).

Le test runner démarre en mode débogage et attend que vous ouvriez les devtools du navigateur pour déboguer les tests.
