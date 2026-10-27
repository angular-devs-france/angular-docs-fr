---
lang: fr
name: Français
direction: ltr
formality: formal
lineParity: strict
quotes: ["«", "»"]
spaceBeforePunctuation:
  chars: [":", ";", "?", "!"]
  space: " "
headingCase: sentence
numberFormat: "1 234,56"
dateFormat: "27 octobre 2026"
fonts: []
---

# Règles de traduction : français

Ces règles s'appliquent à toute page sous `i18n/fr/src/`. Le glossaire (`glossary.json`) fait foi pour la terminologie. En cas de doute, cherchez une page déjà traduite qui traite du même sujet et alignez-vous.

## Registre et ton

- Vouvoiement, toujours : « vous pouvez », jamais « tu peux ».
- Phrases directes, voix active quand l'anglais l'emploie. Ne rien ajouter : pas d'explication supplémentaire, pas d'avis, pas de promotion.
- Les instructions utilisent l'impératif à la deuxième personne du pluriel (« Créez un composant »), pas l'infinitif, sauf dans les listes de tâches courtes où l'anglais utilise des fragments.

## Typographie

- Espace fine insécable (U+202F) avant `:` `;` `?` `!`. Le vérificateur signale une espace ordinaire ou une absence d'espace.
- Guillemets français « » avec espaces insécables à l'intérieur. Les guillemets droits restent dans le code, les sorties de commande et les citations en anglais.
- Titres en casse de phrase : « Composants et templates », pas « Composants Et Templates ».
- Noms de produits et d'API tels quels : Angular, Angular CLI, TypeScript, `signal`, `computed`, `inject`.
- Nombres : espace insécable pour les milliers, virgule décimale (1 234,56). Dates en toutes lettres dans la prose (27 octobre 2026).

## Ce qui reste en anglais

- Les noms d'API, de fichiers, de commandes et d'options : `@Component`, `ng generate`, `--standalone`.
- Les termes marqués `keep` dans le glossaire : *signal*, *template*, *pipe*, *standalone*, *zoneless*, *computed*, *provider*, *schematic*, *package*, *locale*, *observable*, les noms de produits (Angular DevTools, Angular Language Service, Zone.js) et les termes de la plateforme web (*service worker*, *Shadow DOM*). Les acronymes restent (SSR, SSG, i18n, l10n, a11y, e2e) même quand le terme développé est traduit : *hydration* → hydratation, *server-side rendering* → rendu côté serveur (SSR).
- Le code, y compris ses commentaires (règle v1), les exemples de sortie, les URL, les ancres.

## Pièges

- *Component* → composant (masculin). *Directive* → directive (féminin). *Signal* reste signal, masculin : un signal, des signals (pas « signaux », qui désigne le concept général).
- *Template* reste template (masculin : le template). Jamais « gabarit » ni « modèle ».
- *Dependency injection* → injection de dépendances (pluriel). *Change detection* → détection de changements.
- Beaucoup de concepts Angular restent en anglais (`keep` dans le glossaire) : *binding*, *selector*, *host*, *injector*, *query*, *effect*, *resource*, *model*, *library*, *router*, *guard*… Le genre suit le mot français équivalent : le binding, le selector, le host, l'injector, le model, l'effect ; la query, la library, la resource, la promise, la guard, l'error boundary.
- Seul le concept Angular ou d'API web passe en anglais : « une requête HTTP », « en effet », un attribut HTML générique restent en français.
- *Form* → formulaire ; *reactive forms* et *template-driven forms* restent en anglais (les reactive forms) ; *Signal Forms* reste tel quel (nom de la fonctionnalité : les Signal Forms) ; l'expression générique *forms with signals* → formulaires à base de signals.
- *Lazy loading* reste en anglais (le lazy loading) ; « chargement différé » est réservé à `@defer` et aux *deferrable views* → vues différables.
- *Essentials* (la section) → Fondamentaux ; *Playground* reste tel quel.
- Les noms d'API restent en anglais, et le concept aussi quand le glossaire le marque `keep` : « un effect » se crée avec `effect()`, « une resource » avec `resource()`.
- *You* dans une instruction → « vous » ; *your application* → « votre application ».
- Ne traduisez pas les faux amis à l'identique : *actually* → en réalité ; *eventually* → finalement, à terme ; *consistent* → cohérent.

## Chaînes d'interface

- Libellés de navigation et de boutons courts : nom ou infinitif, sans point final (« Guide », « Commencer », « Référence »).
- Respectez la longueur : un libellé de menu dépassant deux mots de plus que l'anglais déborde sur mobile.

## Échantillon

> **EN** Signals are Angular's primitive for reactive state. A signal holds a value and notifies interested consumers when that value changes.
>
> **FR** Les signals sont la primitive d'Angular pour l'état réactif. Un signal contient une valeur et notifie les consommateurs intéressés quand cette valeur change.
