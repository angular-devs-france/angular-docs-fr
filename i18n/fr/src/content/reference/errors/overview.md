<!-- source: 8aac30345a54b9b9614487874447423910586510 -->
# Encyclopédie des erreurs

## Erreurs d'exécution {#runtime-errors}

| Code      | Nom                                                                                  |
| :-------- | :----------------------------------------------------------------------------------- |
| `NG0100`  | [Expression modifiée après vérification](errors/NG0100) |
| `NG0200`  | [Dépendance circulaire dans l'injection de dépendances](errors/NG0200) |
| `NG0201`  | [Aucun provider trouvé](errors/NG0201) |
| `NG0203`  | [`inject()` doit être appelé depuis un contexte d'injection](errors/NG0203) |
| `NG0204`  | [Jeton d'injection invalide](errors/NG0204) |
| `NG0205`  | [L'injector a déjà été détruit](errors/NG0205) |
| `NG0207`  | [EnvironmentProviders dans le mauvais contexte](errors/NG0207) |
| `NG0209`  | [Multi provider invalide](errors/NG0209) |
| `NG0300`  | [Collision de selectors](errors/NG0300) |
| `NG0301`  | [Export introuvable](errors/NG0301) |
| `NG0302`  | [Pipe introuvable](errors/NG0302) |
| `NG0318`  | [Valeur invalide pour un property binding de style](errors/NG0318) |
| `NG0401`  | [Plateforme manquante](errors/NG0401) |
| `NG0403`  | [Le NgModule utilisé pour le bootstrap n'indique pas quel composant initialiser](errors/NG0403) |
| `NG0500`  | [Incohérence de nœud lors de l'hydratation](errors/NG0500) |
| `NG0501`  | [Nœuds frères manquants lors de l'hydratation](errors/NG0501) |
| `NG0502`  | [Nœud manquant lors de l'hydratation](errors/NG0502) |
| `NG0503`  | [Hydratation : projection de nœuds DOM non prise en charge](errors/NG0503) |
| `NG0504`  | [L'indicateur de désactivation de l'hydratation est appliqué à un nœud invalide](errors/NG0504) |
| `NG0505`  | [Aucune information d'hydratation dans la réponse du serveur](errors/NG0505) |
| `NG0506`  | [L'application reste instable](errors/NG0506) |
| `NG0507`  | [Le contenu HTML a été modifié après le rendu côté serveur](errors/NG0507) |
| `NG0600`  | [Écriture de signal dans un contexte interdit](errors/NG0600) |
| `NG0602`  | [Appel de fonction interdit dans un contexte réactif](errors/NG0602) |
| `NG0750`  | [Échec du chargement des dépendances de @defer](errors/NG0750) |
| `NG0751`  | [Comportement de @defer quand le HMR est activé](errors/NG0751) |
| `NG0904`  | [Valeur non sûre utilisée dans un contexte d'URL de ressource](errors/NG0904) |
| `NG0910`  | [Bindings non sécurisés sur un élément](errors/NG0910) |
| `NG0912`  | [Collision lors de la génération d'identifiants de composants](errors/NG0912) |
| `NG0913`  | [Avertissements de performances à l'exécution](errors/NG0913) |
| `NG0919`  | [Dépendance circulaire détectée](errors/NG0919) |
| `NG0950`  | [Un input obligatoire est lu avant qu'une valeur ne soit définie](errors/NG0950) |
| `NG0951`  | [Le résultat d'une child query est obligatoire mais aucune valeur n'est disponible](errors/NG0951) |
| `NG0955`  | [L'expression de suivi a produit des clés en double pour une collection donnée](errors/NG0955) |
| `NG0956`  | [L'expression de suivi a provoqué la recréation de la structure du DOM](errors/NG0956) |
| `NG0991`  | [Resource terminée avant de produire une valeur](errors/NG0991) |
| `NG01002` | [Valeur de contrôle manquante](errors/NG01002) |
| `NG01055` | [Contrôle manquant pour `formControlName`](errors/NG01055) |
| `NG01101` | [Type de retour incorrect pour un validateur asynchrone](errors/NG01101) |
| `NG01203` | [Value accessor manquant](errors/NG01203) |
| `NG01902` | [Champ orphelin dans les Signal Forms](errors/NG01902) |
| `NG02200` | [Iterable differ manquant](errors/NG02200) |
| `NG02800` | [Prise en charge de JSONP dans la configuration de HttpClient](errors/NG02800) |
| `NG02802` | [En-têtes non transférés par HttpTransferCache](errors/NG02802) |
| `NG02825` | [Le corps de la réponse fetch dépasse la limite configurée](errors/NG02825) |
| `NG05000` | [Hydratation avec une instance de Zone.js non prise en charge.](errors/NG05000) |
| `NG05104` | [Élément racine introuvable.](errors/NG05104) |
| `NG05106` | [Nœud de référence d'insertion introuvable](errors/NG05106) |
| `NG05201` | [URL de ressource non sûre passée à DomSanitizer](errors/NG05201) |
| `NG05703` | [Changement suspect d'origine d'URL](errors/NG05703) |

## Erreurs du compilateur {#compiler-errors}

| Code     | Nom                                                         |
| :------- | :---------------------------------------------------------- |
| `NG1001` | [Argument non littéral](errors/NG1001) |
| `NG2003` | [Jeton manquant](errors/NG2003) |
| `NG2009` | [Selector Shadow DOM invalide](errors/NG2009) |
| `NG3003` | [Cycle d'imports détecté](errors/NG3003) |
| `NG6100` | [Anti-pattern NgModule.id défini sur module.id](errors/NG6100) |
| `NG8001` | [Élément invalide](errors/NG8001) |
| `NG8002` | [Attribut invalide](errors/NG8002) |
| `NG8003` | [Cible de référence manquante](errors/NG8003) |
| `NG8011` | [Control flow empêchant la projection de contenu](errors/NG8011) |
| `NG8023` | [Plusieurs composants correspondent au même élément](errors/NG8023) |
| `NG8024` | [Binding de host directive en conflit](errors/NG8024) |
