<!-- source: 0ee75ffad85a929bfe1bef954202331009f5bf31 -->
# Agent Skills

Les Agent Skills sont des instructions et des capacités spécialisées, propres à un domaine, conçues pour les agents IA comme Gemini CLI. Ces skills fournissent des conseils d'architecture, génèrent du code Angular idiomatique et aident à créer la structure de nouveaux projets selon les bonnes pratiques modernes.

En utilisant les Agent Skills, vous pouvez vous assurer que l'agent IA avec lequel vous travaillez dispose des informations les plus récentes sur les conventions d'Angular, ses modèles de réactivité (comme les signals) et la structure des projets.

## Skills disponibles {#available-skills}

L'équipe Angular maintient une collection de skills officielles, régulièrement mises à jour pour rester en phase avec les dernières améliorations du framework.

| Skill                   | Description                                                                                                                                                                                                                                                                                       |
| :---------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **`angular-developer`** | Génère du code Angular et fournit des conseils d'architecture. Utile pour créer des composants, des services, ou obtenir les bonnes pratiques sur la réactivité (signals, linkedSignal, resource), les formulaires, l'injection de dépendances, le routing, le SSR, l'accessibilité (ARIA), les animations, les styles, les tests ou l'outillage de la CLI. |
| **`angular-new-app`**   | Crée une nouvelle application Angular avec l'Angular CLI. Fournit des recommandations importantes pour configurer et structurer efficacement une application Angular moderne.                                                                                                                                           |

## Utiliser les agent skills {#using-agent-skills}

Les Agent Skills sont conçues pour être utilisées avec des outils de code agentiques comme [Gemini CLI](https://geminicli.com/docs/cli/skills/), [Antigravity](https://antigravity.google/docs/skills) et d'autres. Activer une skill charge les instructions et les ressources spécifiques nécessaires à cette tâche.

Pour utiliser ces skills dans votre propre environnement, vous pouvez suivre les instructions de votre outil ou utiliser un outil communautaire comme [skills.sh](https://skills.sh/).

```bash
npx skills add https://github.com/angular/skills
```
