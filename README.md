# Générateur de portail

Ce dépôt contient, pour l'instant, le **cahier des charges fonctionnel** d'un générateur de portail à construire plus tard.

La stack technique n'est pas encore choisie : on décrit d'abord le besoin fonctionnel (le « quoi »), le « comment » viendra ensuite.

## Contenu

| Fichier | Rôle |
|---|---|
| [`docs/cahier-des-charges.md`](docs/cahier-des-charges.md) | Cahier des charges : contexte, principes, acteurs, fonctionnalités des quatre outils (générateur, portail admin, portail utilisateur, espace partenaire), exigences transverses |
| [`docs/synthese.html`](docs/synthese.html) | Page de synthèse du cahier des charges, à partager |
| [`docs/stack-technique.md`](docs/stack-technique.md) | Recommandation de stack technique et d'architecture |
| [`maquette/maquette.html`](maquette/maquette.html) | Maquette cliquable des quatre outils, avec questions pour affiner |
| [`docs/modele-fonctionnalite.md`](docs/modele-fonctionnalite.md) | Modèle de fiche pour détailler une fonctionnalité |

## Conventions

- Chaque fonctionnalité a un identifiant unique `F-<DOMAINE>-<n°>` (ex. `F-GEN-01`).
- Priorité selon la méthode **MoSCoW** : `M` (indispensable), `S` (important), `C` (souhaitable), `W` (pas pour cette version).
- Statut : `À valider`, `Validé`, `Reporté`, `Abandonné`.
