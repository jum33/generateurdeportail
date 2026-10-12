# Dépôt « générateur de portail » : consignes pour Claude

Ce dépôt contient, pour l'instant, le cahier des charges fonctionnel d'un générateur de portails IA. Il n'y a pas encore de code : on décrit le besoin (le « quoi ») avant le « comment ». Le contenu de chaque fichier est listé dans `README.md`.

## Règles de travail

- Écris en français.
- `docs/cahier-des-charges.md` est la référence. Après une modification, répercute-la dans `docs/synthese.html` et, si elle touche les écrans, dans `maquette/maquette.html`.
- Chaque fonctionnalité garde son identifiant `F-<DOMAINE>-<n°>`. N'en renumérote jamais une existante ; une nouvelle prend le numéro suivant libre.
- Priorités MoSCoW (`M`, `S`, `C`, `W`) et statuts (`À valider`, `Validé`, `Reporté`, `Abandonné`) : n'utilise que ces valeurs. Seul Jean-Marc passe une fonctionnalité en `Validé`.
- Pour détailler une fonctionnalité, pars de `docs/modele-fonctionnalite.md`.
- Ne choisis pas la stack technique à la place de Jean-Marc : `docs/stack-technique.md` reste une recommandation tant qu'il ne l'a pas validée.
- Le dépôt est public : n'y mets aucune donnée client ni information confidentielle.
