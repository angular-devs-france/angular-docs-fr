<!-- source: c38c778bdca497d1f726d8a5b9048044ffa64c65 -->
# Angular AI Tutor

L'Angular AI Tutor est conçu pour vous guider de façon interactive, étape par étape, dans la construction d'une application Angular complète et moderne, à partir de zéro. Vous apprendrez les derniers patterns et les bonnes pratiques en construisant un projet réel et concret : une **« Smart Recipe Box »** pour créer et gérer des recettes.

Notre objectif est de développer votre esprit critique et de vous aider à retenir ce que vous apprenez. Au lieu de simplement vous donner du code, le tuteur explique les concepts, vous montre des exemples, puis vous propose des exercices propres au projet à résoudre par vous-même.

## Premiers pas {#get-started}

Vous pouvez accéder à l'Angular AI Tutor via le [serveur MCP d'Angular](ai/mcp).

1. [Installez](ai/mcp#get-started) le serveur MCP d'Angular
2. Créez un nouveau projet Angular `ng new <project-name>`
3. Ouvrez votre nouveau projet (`cd <project-name>`) dans un éditeur ou un outil propulsé par l'IA, comme [Gemini CLI](https://geminicli.com/)
4. Saisissez un prompt comme `launch the Angular AI tutor`
   ![Une capture d'écran montrant comment lancer l'Angular AI Tutor dans Gemini CLI.](assets/images/launch-ai-tutor.png 'Lancer Angular AI Tutor')

## Utiliser l'AI Tutor {#using-the-ai-tutor}

Chaque module commence par une brève explication du concept.
![Une capture d'écran de l'Angular AI Tutor présentant une brève explication d'un concept.](assets/images/ai-tutor-preview-1.png 'Angular AI Tutor : explication')
Le cas échéant, le tuteur présente un exemple de code pour illustrer le concept.
![Une capture d'écran de l'Angular AI Tutor montrant un exemple de code.](assets/images/ai-tutor-preview-2.png 'Angular AI Tutor : exemple de code')
Le tuteur propose aussi un exercice ouvert pour tester votre compréhension.
![Une capture d'écran de l'Angular AI Tutor proposant un exercice.](assets/images/ai-tutor-preview-3.png 'Angular AI Tutor : exercice')
Enfin, le tuteur vérifie votre travail avant de passer au module suivant.
![Une capture d'écran de l'Angular AI Tutor vérifiant le travail de l'utilisateur.](assets/images/ai-tutor-preview-4.png 'Angular AI Tutor : vérification')

## Comment ça marche : le cycle d'apprentissage {#how-it-works-the-learning-cycle}

Pour chaque nouveau sujet, vous suivez une boucle d'apprentissage qui met l'accent sur l'esprit critique pour vous aider à mieux retenir ce que vous apprenez.

1. **Comprendre le concept :** le tuteur explique brièvement une fonctionnalité centrale d'Angular et vous montre un exemple de code générique pour l'illustrer.
2. **Appliquer vos connaissances :** vous obtenez immédiatement un exercice pratique. Le tuteur présente ces exercices à haut niveau, avec des objectifs et des résultats attendus, pour vous encourager à réfléchir vous-même à la solution.
3. **Obtenir des retours et de l'aide :** quand vous êtes prêt, prévenez le tuteur. Il **lit automatiquement les fichiers de votre projet** pour vérifier que votre solution est correcte. Si vous êtes bloqué, vous gardez le contrôle total. Vous pouvez demander un **« indice »** pour être davantage guidé, ou obtenir des instructions pas à pas en tapant **« guide détaillé »** ou **« instructions étape par étape »**.

Une fois l'exercice réussi, le tuteur passe directement au sujet suivant. Vous pouvez aussi lui demander plus d'informations sur un sujet ou lui poser à tout moment n'importe quelle question liée à Angular.

---

## **Fonctionnalités et commandes** {#features--commands}

Vous gardez le contrôle de votre expérience d'apprentissage. Utilisez ces fonctionnalités à tout moment :

### **Partir et revenir** {#leave-and-come-back}

N'hésitez pas à faire une pause. Votre progression est liée au code de votre projet. Quand vous revenez pour une nouvelle session, le tuteur analyse automatiquement vos fichiers pour déterminer précisément où vous vous êtes arrêté, ce qui vous permet de reprendre sans accroc là où vous en étiez.

**Astuce :** nous vous recommandons vivement d'utiliser Git pour sauvegarder votre progression. Après avoir terminé un module, c'est une excellente idée de commiter vos changements (par ex. `git commit -m "Complete Phase 1, Module 8"`). Cela constitue un point de sauvegarde personnel auquel vous pouvez toujours revenir.

### **Ajuster votre niveau d'expérience** {#adjust-your-experience-level}

Vous pouvez définir votre niveau d'expérience sur **Débutant (1-3)**, **Intermédiaire (4-7)** ou **Expérimenté (8-10)**. Vous pouvez modifier ce réglage à tout moment pendant votre session, et le tuteur adapte immédiatement son style d'enseignement en conséquence.

**Exemples de prompts :**

- « Règle mon niveau d'expérience sur débutant. »
- « Change mon niveau à 8. »

### **Voir le plan d'apprentissage complet** {#see-the-full-learning-plan}

Vous voulez avoir une vue d'ensemble ou voir le chemin parcouru ? Demandez simplement la table des matières.

**Exemples de prompts :**

- « Où en sommes-nous ? »
- « Affiche la table des matières. »
- « Affiche le plan. »

Le tuteur affiche le plan d'apprentissage complet et indique où vous en êtes.

### **Une remarque sur les styles** {#a-note-on-styling}

Le tuteur applique des styles de base à votre application pour qu'elle garde un aspect soigné. Nous vous encourageons vivement à appliquer vos propres styles pour personnaliser l'application.

### **Passer le module en cours** {#skip-the-current-module}

Si vous préférez passer au sujet suivant du parcours d'apprentissage, vous pouvez demander au tuteur de passer l'exercice en cours.

**Exemples de prompts :**

- « Passe cette section. »
- « Termine automatiquement cette étape pour moi. »

Le tuteur vous demande une confirmation, puis vous présente la solution complète du module en cours et tente d'appliquer automatiquement les mises à jour nécessaires pour que vous puissiez continuer sans difficulté avec le module suivant.

### **Aller directement à un sujet** {#jump-to-any-topic}

Si vous voulez étudier un sujet précis dans le désordre (par ex. passer des bases aux formulaires), c'est possible. Le tuteur fournit le code nécessaire pour mettre votre projet au bon point de départ pour le module choisi et tente d'appliquer automatiquement les mises à jour nécessaires.

**Exemples de prompts :**

- « Emmène-moi à la leçon sur les formulaires. »
- « Je veux maintenant apprendre les route guards. »
- « Va à la section sur les services. »

---

## **Dépannage** {#troubleshooting}

### Problèmes de configuration {#setup-issues}

**« launch the Angular AI tutor » ne fait rien ?**

Assurez-vous d'abord d'avoir un projet ouvert. Le tuteur a besoin d'un vrai projet Angular pour fonctionner :

```bash
ng new my-app
cd my-app
code .
```

Vérifiez ensuite que votre serveur MCP est en cours d'exécution. Dans VS Code, ouvrez le fichier `.vscode/mcp.json` et cliquez sur le bouton **« Start »** en haut du fichier.

Quand vous tapez « launch the Angular AI tutor », vous devriez voir une coche indiquant
« Reviewed .vscode/mcp.json and ran start task » et une invite demandant
« Allow task run? » : cliquez sur Allow.

**Ça ne fonctionne toujours pas ?**

Essayez de taper d'abord `#angular-cli` pour charger le contexte Angular, puis collez l'URL du tutoriel : `https://angular.dev/ai/ai-tutor`

**Comment vérifier que le serveur est en cours d'exécution**

Ouvrez la palette de commandes (`Ctrl+Shift+P`), tapez « MCP: List Running Servers » et cherchez « angular-cli » dans la liste.

---

### Problèmes généraux {#general-issues}

Si le tuteur ne répond pas correctement ou si vous soupçonnez un problème dans votre application, voici quelques pistes :

1. **Tapez « continue » :** cela suffit souvent à relancer le tuteur vers l'étape suivante s'il est bloqué.
2. **Corrigez le tuteur :** si le tuteur se trompe sur votre progression (par ex. il indique que vous êtes au module 3 alors que vous avez terminé le module 8), dites-le-lui simplement. Par exemple : _« En réalité, je suis au module 8. »_ Le tuteur réévalue votre code et s'ajuste.
3. **Vérifiez votre interface :** si vous voulez confirmer à quoi l'interface utilisateur de votre application doit ressembler, demandez simplement au tuteur. Par exemple : _« Que devrais-je voir dans mon interface ? »_
4. **Rechargez la fenêtre du navigateur :** un rafraîchissement peut résoudre de nombreux problèmes liés à votre application.
5. **Redémarrez complètement le navigateur :** les erreurs n'apparaissent parfois que dans la console de développement du navigateur. Un redémarrage complet peut aider à éliminer des problèmes sous-jacents liés à l'application.
6. **Démarrez une nouvelle conversation :** vous pouvez toujours démarrer une nouvelle conversation pour effacer l'historique existant et repartir de zéro. Le tuteur lira vos fichiers pour retrouver la dernière étape où vous en étiez.

## **Votre parcours d'apprentissage : un chemin en phases** {#your-learning-journey-the-phased-path}

Vous construisez votre application au fil d'un parcours en cinq phases. Vous pouvez suivre ce chemin du début à la fin pour créer une application Angular complète et entièrement fonctionnelle. Chaque module s'appuie logiquement sur le précédent et vous mène des bases jusqu'à des fonctionnalités avancées et concrètes.

**Une remarque sur la configuration automatisée :** certains modules nécessitent une étape de configuration, comme créer des interfaces ou des données de mock. Dans ce cas, le tuteur vous présente le code et les instructions sur les fichiers. C'est à vous de créer et de modifier ces fichiers comme indiqué avant le début de l'exercice.

### **Phase 1 : Fondamentaux d'Angular** {#phase-1-angular-fundamentals}

- **Module 1 :** Premiers pas
- **Module 2 :** Texte dynamique avec l'interpolation
- **Module 3 :** Event listeners (`(click)`)

### **Phase 2 : État et signals** {#phase-2-state-and-signals}

- **Module 4 :** Gestion d'état avec les signals modifiables (partie 1 : `set`)
- **Module 5 :** Gestion d'état avec les signals modifiables (partie 2 : `update`)
- **Module 6 :** Signals computed

### **Phase 3 : Architecture des composants** {#phase-3-component-architecture}

- **Module 7 :** Binding de template (propriétés et attributs)
- **Module 8 :** Créer et imbriquer des composants
- **Module 9 :** Inputs de composants avec les signals
- **Module 10 :** Styliser les composants
- **Module 11 :** Rendu de listes avec `@for`
- **Module 12 :** Rendu conditionnel avec `@if`

### **Phase 4 : Fonctionnalités avancées et architecture** {#phase-4-advanced-features--architecture}

- **Module 13 :** Two-way binding
- **Module 14 :** Services et injection de dépendances (DI)
- **Module 15 :** Routing de base
- **Module 16 :** Introduction aux formulaires
- **Module 17 :** Introduction à Angular Material

### **Phase 5 : Signal Forms** {#phase-5-signal-forms}

- **Module 18** : **Introduction aux Signal Forms**
- **Module 19** : **Soumission et réinitialisation**
- **Module 20** : **Validation dans les Signal Forms**
- **Module 21** : **État des champs et messages d'erreur**

---

## **Une remarque sur l'IA et vos retours** {#a-note-on-ai--feedback}

Ce tuteur est propulsé par un grand modèle de langage (LLM). Même si nous avons travaillé dur pour en faire un expert, les IA peuvent se tromper. Si vous rencontrez une explication ou un exemple de code qui semble incorrect, faites-le-nous savoir. Vous pouvez corriger le tuteur, et il utilisera vos retours pour ajuster sa réponse.

Pour tout bug technique ou toute demande de fonctionnalité, [ouvrez une issue](https://github.com/angular/angular-cli/issues).
