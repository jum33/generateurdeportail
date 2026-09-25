# Cahier des charges fonctionnel — Générateur de portail

> **Version** : 0.43 (brouillon) · **Date** : 24/09/2026 · **Statut** : en cours de rédaction
>
> Toutes les fonctionnalités ont été revues et validées (sauf celles au statut *Reporté*). Les points encore ouverts sont listés en section 7.

---

## 1. Contexte et objectifs

### 1.1 Contexte
Les portails IA existants montrent l'intérêt d'un espace unique où les collaborateurs d'une organisation accèdent à l'IA (chat, agents, documents, automatisations). Ils sont en revanche jugés **peu souples** et **peu lisibles** pour l'utilisateur.

Le besoin est de pouvoir **proposer à chaque client son propre portail IA**, rapidement, à son nom et sur son nom de domaine.

### 1.2 Objectif principal
Disposer d'un **générateur** qui crée **à la demande un portail IA par client** :
- le client peut être une **entreprise** ou une **collectivité locale** ;
- le portail **s'installe sur le nom de domaine du client** (ex. `ia.nom-du-client.fr`) ;
- côté utilisateur, le portail **s'ouvre directement sur un chatbot** ;
- le chatbot est **multi-LLM** : il peut s'appuyer sur plusieurs modèles d'IA ;
- chaque portail est une **instance dédiée** au client, et non un compte dans un SaaS partagé, pour garantir la conformité **RGPD** et un alignement sur les normes **ISO**.

Le projet comprend **quatre outils** :
1. **Le générateur de portail et sa gestion** (opérateur) : il crée les portails et gère leurs **usages** — **quels outils** sur quel portail, **combien** chaque portail est utilisé (**coût** et **nombre de tokens**) — ainsi que **l'étendue des possibilités données à l'admin de chaque client** et les **forfaits** proposés aux clients (usage inclus ou non, sans licence par utilisateur) ;
2. **Le portail client, partie admin** (administrateur du client) ;
3. **Le portail client, partie utilisateur** (utilisateurs finaux), ouverte sur le chatbot ;
4. **L'espace partenaire**, à la marque de chaque partenaire : les partenaires y demandent la génération de portails pour leurs clients, y cochent / décochent les options et y suivent leurs KPI et commissions. **Seul l'opérateur utilise le générateur.**

Les portails sont vendus soit directement par l'opérateur, soit par un **réseau de partenaires sur trois niveaux au maximum**, situés entre l'opérateur et le client final. Chaque partenaire a son propre coefficient multiplicateur et règle, depuis l'espace partenaire, les options des portails de ses clients et de ses sous-partenaires. L'opérateur facture tous les partenaires et leur reverse des commissions selon leurs accords ; **le générateur calcule les montants mais n'émet pas les factures**.

### 1.3 Objectifs secondaires
- Être **plus souple** que les portails IA existants : chaque portail s'adapte au client (fonctions activées, sources de données, modèles d'IA, apparence).
- Être **plus lisible** que les portails IA existants : une interface simple, centrée sur le chatbot, où l'utilisateur trouve immédiatement ce qu'il cherche.
- **Configurer en parlant plutôt qu'en cliquant** (« vibe coding ») : le client décrit ce dont il a besoin et le portail le construit (cf. P-10).
- Réduire au minimum le temps et l'effort pour mettre en service un nouveau client.

### 1.4 Indicateurs de réussite
- **500 portails** (instances clients) gérés à terme
- *Ex. : temps de création d'un portail < 30 min (à confirmer)*

---

## 2. Grands principes

Principes structurants : chaque fonctionnalité du générateur doit les respecter.

| ID | Principe | Signification | Conséquences attendues |
|---|---|---|---|
| P-01 | **Agnostique au LLM** | Le générateur ne dépend d'aucun fournisseur ni modèle d'IA en particulier. | Choix du fournisseur et du modèle par configuration ; possibilité de changer de LLM sans refaire le portail ; possibilité d'utiliser des modèles différents selon l'usage ; possibilité d'utiliser un modèle hébergé en local. |
| P-02 | **Agnostique à l'hébergement** | Un portail généré peut être déployé chez n'importe quel hébergeur ou sur un serveur du client. **Par défaut, l'opérateur héberge en France** ; chaque client peut choisir un autre type d'hébergement. | Aucune dépendance obligatoire à un hébergeur ; export complet du portail ; déploiement possible chez l'opérateur (France, par défaut), sur un cloud souverain, un cloud public ou un serveur interne du client. |
| P-03 | **Agnostique à la marque** (marque blanche) | Le générateur n'impose aucune marque : chaque portail porte l'identité de son propriétaire. | Aucune mention imposée de l'éditeur du générateur ; nom, logo, couleurs, typographies, domaine, e-mails et textes entièrement personnalisables. Seule exception : l'application mobile commune est publiée dans les stores à la marque de l'opérateur (une application à la marque du client est proposée en option, cf. §4.26). |
| P-04 | **Agnostique à l'interface utilisateur** | Les fonctions et contenus du portail ne dépendent pas d'une interface précise : la même logique peut être présentée sous plusieurs formes. | Séparation entre le fond (contenus, services, IA) et la présentation ; interface remplaçable ou personnalisable ; diffusion possible sur plusieurs canaux (site web, mobile, widget intégré dans un site existant, messagerie collaborative…). |
| P-05 | **Agnostique aux sources de données** | Le portail peut s'appuyer sur n'importe quelle source de données, sans dépendance à un outil ou éditeur particulier. | Connecteurs vers des sources variées (fichiers, bases de données, API, espaces documentaires, CRM…) ; ajout ou remplacement d'une source sans refaire le portail ; formats d'échange ouverts. |
| P-06 | **Conformité RGPD et ISO** | Chaque portail respecte le RGPD et est conçu pour permettre la **certification** ISO de sécurité et de gouvernance de l'IA. | Protection des données dès la conception ; traçabilité ; documentation de conformité fournie pour chaque instance (cf. §5.1). |
| P-07 | **Une instance par client, pas de SaaS** | Chaque client dispose de sa propre instance du portail (application, données, paramètres), isolée de celles des autres clients. | Aucune donnée partagée entre clients ; instance déployable dans l'hébergement choisi par le client ou pour lui (cf. P-02) ; mises à jour pilotées instance par instance. |
| P-08 | **Pilotage centralisé des usages** | Toutes les instances sont gérées depuis le générateur de l'opérateur, sans que celui-ci accède aux contenus des clients. | Outils activés et consommation visibles pour chaque portail ; seules des données de pilotage (compteurs, coûts, état) remontent au générateur. |
| P-09 | **Distribution multi-niveaux** | Entre l'opérateur et le client final peuvent s'intercaler **jusqu'à trois niveaux** de partenaires. | Seul l'opérateur utilise le générateur ; les partenaires passent par l'espace partenaire. Chaque niveau ne peut que **restreindre** ce que le niveau au-dessus lui a ouvert ; chaque niveau a son propre coefficient multiplicateur ; un partenaire ne voit que **les KPI consolidés** de ses sous-partenaires, jamais la configuration des portails de leurs clients. |
| P-10 | **Configurer en parlant plutôt qu'en cliquant** (« vibe coding ») | Le client décrit ce qu'il veut, à l'écrit ou à la voix ; le portail construit la fonction. Les écrans à nombreuses options sont évités. | Création d'chats spécialisés, d'automatisations, de formulaires, de ensembles de sources par la conversation ; aperçu et test immédiats ; ajustements par dialogue ; options avancées cachées par défaut. |
| P-11 | *À compléter* | | |

---

## 3. Acteurs et rôles

| Acteur | Description | Droits principaux |
|---|---|---|
| Opérateur (super-administrateur) | Vous : utilisez le **générateur** pour créer et gérer les portails de vos clients | Tout, depuis le générateur : créer et déployer les instances, choisir les outils de chaque portail, **fixer les possibilités de l'admin client**, suivre la consommation et les coûts ; **aucun accès aux conversations ni aux documents des clients** |
| Partenaire | Revendeur de niveau 1, sous contrat avec l'opérateur | **Pas d'accès au générateur.** Depuis l'**espace partenaire** : demander la génération d'un portail pour un client, cocher / décocher les options des portails de ses clients et les options ouvertes à ses sous-partenaires, voir les KPI de ses clients, les **KPI consolidés** de ses sous-partenaires et ses commissions |
| Sous-partenaire (niveaux 2 et 3) | Revendeur rattaché à un partenaire de niveau 1 ou 2 | Mêmes droits que le partenaire dans l'espace partenaire, **dans les limites fixées par son partenaire parent** |
| Administrateur client | Référent chez le client (entreprise ou collectivité), disposant de la **partie admin** de son portail | Selon ce que le générateur lui délègue : gérer les droits (utilisateurs, profils, équipes), les fiches organisations et contacts, les intégrations (Microsoft, Google, suite bureautique française, MCP) et, selon le cas, les clés API de **son** portail |
| Contributeur client | Personne chez le client qui alimente le portail | Ajouter des documents, des chats spécialisés ou des contenus *(à valider)* |
| Utilisateur final | Collaborateur ou agent du client | Se connecter et utiliser le chatbot et les services du portail |

*À valider : l'utilité du rôle contributeur. Le périmètre de l'administrateur client est réglé portail par portail (cf. §4.4).*

Chaîne de distribution : **Opérateur → Partenaire (niveau 1) → Sous-partenaire (niveau 2) → Sous-partenaire (niveau 3) → Client final**, soit **trois niveaux de partenaires au maximum**. Un client peut être rattaché directement à l'opérateur ou à n'importe quel niveau de partenaire.

---

## 4. Fonctionnalités attendues

Légende priorité : **M** indispensable · **S** important · **C** souhaitable · **W** pas pour cette version.

Le périmètre comprend **quatre outils** :
- **Partie A — le générateur de portail**, avec sa gestion : utilisé **uniquement par l'opérateur**, en amont de tout ;
- **Partie B — le portail client, partie admin** : utilisée par l'administrateur du client, avec plus ou moins de possibilités selon ce que le générateur lui ouvre ;
- **Partie C — le portail client, partie utilisateur** : utilisée par les utilisateurs finaux, ouverte sur le chatbot, sur le web et dans les **applications Android et iPhone** ;
- **Partie D — l'espace partenaire** : utilisé par les partenaires pour demander des portails, régler les options et suivre KPI et commissions.

Les parties B et C forment ensemble **une instance par client** (cf. P-07).

### Partie A — Générateur de portail (opérateur)

Outil **réservé à l'opérateur** (les partenaires n'y ont pas accès ; ils passent par l'espace partenaire, Partie D) : il crée les portails, y compris ceux demandés par les partenaires, les déploie, choisit leurs outils, **fixe ce que chaque admin client peut faire** et suit la consommation.

#### 4.1 Création et déploiement des instances — `GEN`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-GEN-02 | Créer un portail à partir d'un modèle (template) | M | Validé |
| F-GEN-03 | Dupliquer un portail existant | S | Validé |
| F-GEN-05 | Générer une première version du portail à partir d'une description en langage naturel (IA, quel que soit le LLM, cf. P-01 et P-10) | M | Validé |
| F-GEN-06 | Archiver / supprimer une instance, avec restitution des données au client et effacement vérifiable | M | Validé |
| F-GEN-07 | Générer à la demande une **instance dédiée** pour un nouveau client, entreprise ou collectivité locale (cf. P-07) | M | Validé |
| F-GEN-08 | Bibliothèque de modèles de portail, notamment par type de client (ex. entreprise, collectivité), pré-configurant fonctions, textes et réglages | S | Validé |
| F-GEN-09 | Isolation totale : chaque client a sa propre instance, aucune donnée, utilisateur ou réglage partagé avec un autre (cf. P-07) | M | Validé |
| F-GEN-11 | Mettre à jour une instance vers une nouvelle version du portail, instance par instance ou par lot, avec retour arrière | M | Validé |
| F-GEN-12 | Voir l'état de chaque instance (en ligne, version, hébergement, dernière sauvegarde) | M | Validé |
| F-GEN-13 | Choisir le type d'hébergement de chaque instance : **hébergement opérateur en France (par défaut)**, cloud souverain, cloud public, serveur du client | M | Validé |
| F-GEN-14 | **Supervision technique** de toutes les instances : disponibilité, performances, erreurs, espace disque, sauvegardes ; alertes à l'opérateur | M | Validé |

#### 4.2 Réseau de partenaires — `PART`

Gestion, **par l'opérateur**, des partenaires qui revendent les portails (cf. P-09). Les partenaires eux-mêmes utilisent l'espace partenaire (Partie D).

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-PART-01 | Créer et gérer les partenaires sur **trois niveaux au maximum** (partenaire, sous-partenaire, sous-sous-partenaire) | M | Validé |
| F-PART-02 | Rattacher chaque client à l'opérateur ou à un partenaire de n'importe quel niveau ; déplacer un client ou un sous-partenaire d'un parent à un autre | M | Validé |
| F-PART-03 | Enregistrer l'**accord commercial** de chaque partenaire de niveau 1 (et des clients directs) : coefficient multiplicateur de l'opérateur, qui fixe la **part de l'opérateur** ; les coefficients des sous-partenaires sont fixés par leur partenaire parent (cf. F-ESP-15) | M | Validé |
| F-PART-04 | Recevoir les **demandes de génération de portail** envoyées par les partenaires depuis l'espace partenaire (cf. §4.28), les traiter et suivre leur état (reçue, en cours, livrée) | M | Validé |
| F-PART-05 | Les options cochées / décochées par les partenaires s'appliquent **immédiatement** aux portails concernés, sans validation de l'opérateur ; l'opérateur est informé et peut consulter l'historique | M | Validé |
| F-PART-10 | L'opérateur est **alerté à chaque fois** qu'un partenaire ferme une option sur un portail | M | Validé |
| F-PART-06 | Règle de cascade : un niveau ne peut jamais ouvrir ce que le niveau au-dessus a fermé ; une fermeture en haut se répercute automatiquement vers le bas | M | Validé |
| F-PART-07 | Autoriser ou non un partenaire à avoir des sous-partenaires (dans la limite de trois niveaux) | S | Validé |
| F-PART-08 | Les partenaires, comme l'opérateur, n'ont **aucun accès aux conversations ni aux documents** des clients (cf. P-08) | M | Validé |
| F-PART-09 | Journal des actions de chaque partenaire (demandes, options modifiées) | S | Validé |

#### 4.3 Outils et services de chaque portail — `OUT`

Premier volet des usages : **quels outils pour quel portail**.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-OUT-01 | Catalogue des outils disponibles (chatbot, modèles d'IA, chats spécialisés, bases documentaires, connecteurs, automatisations, comptes rendus…) | M | Validé |
| F-OUT-02 | Activer ou désactiver chaque outil, portail par portail | M | Validé |
| F-OUT-03 | Choisir les modèles d'IA ouverts sur chaque portail (cf. P-01, F-IA-01) | M | Validé |
| F-OUT-04 | Vue d'ensemble « portails × outils » : qui a quoi, en un coup d'œil | M | Validé |
| F-OUT-05 | Packs d'outils prédéfinis appliqués à un portail en un clic, repris dans les forfaits (cf. §4.6) | S | Validé |
| F-OUT-07 | Historique des activations / désactivations (qui, quand) | S | Validé |

#### 4.4 Niveau de délégation au portail admin — `DELEG`

L'opérateur, dans le générateur, décide (le cas échéant à partir des options cochées par le partenaire dans l'espace partenaire), portail par portail, **jusqu'où l'administrateur client peut aller** dans la partie admin. Les réglages se font **en cascade** : l'opérateur, puis chaque niveau de partenaire, puis l'admin client ne peuvent que restreindre ce que le niveau au-dessus leur a ouvert (cf. §4.2).

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-DELEG-01 | Régler, pour chaque portail, les possibilités données à la partie admin du client ; réglage par l'opérateur, ou par le partenaire qui a apporté le client depuis l'espace partenaire | M | Validé |
| F-DELEG-02 | Niveaux de délégation **calés sur les forfaits** : chaque forfait porte son niveau de délégation, appliqué automatiquement au portail (cf. F-FORF-02) ; ajustable ensuite module par module (cf. F-DELEG-03) | M | Validé |
| F-DELEG-03 | Réglage fin, fonction par fonction : ouvrir, fermer ou ouvrir en lecture seule chaque module de l'admin client (utilisateurs, profils, équipes, fiches, intégrations, MCP, clés API, sources, apparence, contenus, statistiques, consommation) | M | Validé |
| F-DELEG-04 | Fixer des limites techniques (ex. nombre maximal de serveurs MCP, de sources de données, budget) — sans limite ni tarification par utilisateur (cf. F-FORF-04) | S | Validé |
| F-DELEG-05 | Déléguer ou non au client la gestion des **modèles d'IA, abonnements fournisseurs et clés API** : pas du tout, partiellement (ex. choix parmi les modèles de l'opérateur) ou totalement (cf. §4.8) | M | Validé |
| F-DELEG-06 | Autoriser ou non le client à ajouter ses propres intégrations et serveurs MCP (cf. F-INT-05) | M | Validé |
| F-DELEG-07 | Autoriser ou non le client à créer ses propres profils (rôles) personnalisés | S | Validé |
| F-DELEG-08 | Les modules non délégués sont **masqués** dans l'admin client (et non simplement grisés), pour garder une interface lisible | S | Validé |
| F-DELEG-09 | Les réglages non délégués restent gérés par l'opérateur depuis le générateur, sans accès aux contenus du client (cf. P-08) | M | Validé |
| F-DELEG-10 | Historique des changements de niveau de délégation (qui, quand, quoi) | S | Validé |

#### 4.5 Consommation, coûts et KPI d'usage — `CONSO`

Second volet des usages : **combien chaque portail est utilisé**, en tokens et en coût. **Dans tous les cas**, y compris quand le client gère lui-même ses modèles, abonnements et clés, les KPI d'usage et de coût remontent dans la gestion du générateur.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CONSO-01 | Compter les tokens consommés (entrée et sortie) pour chaque portail | M | Validé |
| F-CONSO-02 | Calculer le coût correspondant, selon le tarif de chaque modèle | M | Validé |
| F-CONSO-03 | Détailler la consommation par portail, modèle, outil, période et équipe ; détail **par utilisateur nominatif** selon le réglage du portail (cf. F-CONSO-34) | M | Validé |
| F-CONSO-30 | Le détail nominatif est visible dans la partie admin du client ; dans le générateur et l'espace partenaire, il est **toujours pseudonymisé** (identifiant à la place du nom) (cf. P-08, §5.1) | M | Validé |
| F-CONSO-34 | **Réglage par portail** du niveau de détail de la consommation : **par équipe** (par défaut pour les collectivités) ou **par utilisateur nominatif** (activable, ex. entreprises) | M | Validé |
| F-CONSO-04 | Tableau de bord global : consommation et coût de tous les portails, évolution dans le temps | M | Validé |
| F-CONSO-05 | Gérer la grille des tarifs par modèle : prix d'achat et prix de revente au client | M | Validé |
| F-CONSO-06 | Fixer un budget ou un plafond de tokens par portail, avec alertes à l'approche du seuil | S | Validé |
| F-CONSO-08 | Exporter la consommation (tableur) pour la facturation, faite hors de l'outil, ou le reporting client | S | Validé |
| F-CONSO-09 | Donner à l'administrateur client une vue de la consommation de son propre portail, coefficient appliqué (cf. F-CONSO-17) | S | Validé |
| F-CONSO-10 | Remonter au générateur (et aux partenaires) uniquement des compteurs, jamais le contenu des échanges (cf. P-08) | M | Validé |
| F-CONSO-11 | KPI toujours disponibles dans le générateur, quel que soit le mode de gestion des modèles, abonnements et clés (opérateur ou client, cf. §4.8) | M | Validé |
| F-CONSO-12 | **KPI d'usage** par portail : utilisateurs actifs (jour, mois), taux d'adoption (actifs / inscrits), consommation par rapport à l'enveloppe du forfait (cf. §4.6), nombre de conversations et de requêtes, outils et chats spécialisés les plus utilisés | M | Validé |
| F-CONSO-13 | **KPI de coût** par portail : coût total, coût par utilisateur actif, par modèle, par outil, évolution et projection en fin de mois | M | Validé |
| F-CONSO-14 | Distinguer les coûts supportés par l'opérateur (ses clés) de ceux supportés directement par le client (ses propres clés) | M | Validé |
| F-CONSO-15 | Comparer les portails entre eux et alerter sur une anomalie (pic de consommation, portail inactif) | S | Validé |
| F-CONSO-33 | **KPI en tête du générateur** : coût réel, marge, consommation par rapport à l'enveloppe du forfait | M | Validé |
| F-CONSO-16 | **Coefficient multiplicateur à chaque niveau** : l'opérateur fixe celui de ses partenaires de niveau 1 et de ses clients directs ; **chaque partenaire fixe celui de ses sous-partenaires** et de ses clients, depuis l'espace partenaire | M | Validé |
| F-CONSO-17 | Les coefficients **se cumulent le long de la chaîne** : valeur vue par le client = valeur réelle × coefficient de chaque niveau entre l'opérateur et lui (ex. 1,3 × 1,2 = 1,56). Les KPI d'usage de la partie admin client intègrent ce coefficient cumulé | M | Validé |
| F-CONSO-31 | Le coefficient s'applique **uniquement aux tokens et aux coûts** (pas aux nombres d'utilisateurs, de conversations ou de requêtes) | M | Validé |
| F-CONSO-32 | Coefficient **différent par modèle d'IA** possible pour un même client ou partenaire (un coefficient par défaut, ajustable modèle par modèle) | M | Validé |
| F-CONSO-18 | Chaque partenaire voit ses valeurs « d'achat » (coefficients des niveaux au-dessus déjà appliqués) et ses valeurs « de revente » ; il ne voit **jamais** les coefficients ni les valeurs des niveaux au-dessus de lui. Le client ne voit jamais aucun coefficient | M | Validé |
| F-CONSO-19 | L'opérateur voit toute la chaîne : valeurs réelles, coefficient de chaque niveau, valeurs à chaque étape et écarts | M | Validé |
| F-CONSO-20 | Historique du coefficient avec date d'effet : un changement ne modifie pas les périodes passées | S | Validé |
| F-CONSO-21 | Le coefficient cumulé s'applique aussi au décompte de l'enveloppe du forfait et à l'usage refacturé (cf. F-FORF-03, F-FORF-08) | M | Validé |
| F-CONSO-22 | Tableau de bord de chaque partenaire dans l'espace partenaire : KPI d'usage et de coût de **ses propres clients** (cf. §4.28) | M | Validé |
| F-CONSO-23 | Pour les sous-partenaires, le partenaire ne voit que des **KPI consolidés** (par sous-partenaire), sans détail par portail ni accès à leur configuration | M | Validé |
| F-CONSO-24 | **Règle de marge** : la part de l'opérateur ne dépend que de son propre coefficient ; le coefficient qu'un partenaire accorde à ses sous-partenaires est **pris sur la marge de ce partenaire** et n'affecte jamais la part de l'opérateur (exemple ci-dessous) | M | Validé |

*Exemple (mode coefficient)* — coût réel de l'usage : 100 €. L'opérateur applique 1,3 au partenaire P (niveau 1) : **part de l'opérateur 130 €**. P fixe 1,2 pour son sous-partenaire S : S « achète » 156 €. S fixe 1,25 pour son client : le client voit 195 €. L'opérateur calcule 156 € dus par S, conserve 130 € et **reverse 26 € de commission à P**. S facture (ou fait facturer) 195 € au client et garde 39 € de marge.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CONSO-25 | **Deux modes de tarification**, au choix pour chaque relation (opérateur → partenaire, partenaire → sous-partenaire, partenaire → client) : **coefficient multiplicateur** appliqué à la valeur d'achat, ou **prix direct** saisi par le niveau qui vend | M | Validé |
| F-CONSO-26 | En mode prix direct, deux unités utilisables **séparément ou ensemble** : **prix par million de tokens pour chaque modèle d'IA** (distinguant si besoin tokens d'entrée et de sortie) et **montant fixe par mois** | M | Validé |
| F-CONSO-27 | Les deux modes peuvent coexister dans une même chaîne (ex. coefficient entre l'opérateur et P, prix direct entre P et S) ; les KPI, montants dus et commissions sont calculés de la même façon (cf. F-CONSO-24) | M | Validé |
| F-CONSO-28 | Alerte si un prix direct est inférieur au prix d'achat du vendeur (marge négative) | S | Validé |
| F-CONSO-29 | Les KPI vus par le client intègrent le mode retenu (coefficient ou prix direct), sans jamais afficher ni coefficient ni prix d'achat | M | Validé |

*Exemple (mode prix direct)* — même chaîne : la part de l'opérateur reste 130 €. P fixe pour S un prix direct équivalant à 150 € pour cet usage : l'opérateur calcule 150 € dus par S et reverse 20 € de commission à P.

#### 4.6 Forfaits clients — `FORF`

Objectif : **proposer des forfaits aux clients, qui incluent ou non l'usage**. La tarification **ne repose pas sur des licences par utilisateur**.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-FORF-01 | Catalogue de forfaits : nom, description, prix, périodicité (mensuelle, annuelle), durée d'engagement | M | Validé |
| F-FORF-02 | Chaque forfait définit les outils inclus (cf. §4.3) et le niveau de délégation au portail admin (cf. §4.4) | M | Validé |
| F-FORF-03 | Trois façons de traiter l'usage, au choix pour chaque forfait : **usage inclus** (enveloppe de tokens ou montant en euros par période) · **usage non inclus**, refacturé au réel selon la grille de revente (cf. F-CONSO-05) · **usage à la charge du client** via ses propres clés d'IA (cf. §4.8) | M | Validé |
| F-FORF-04 | Prix indépendant du nombre d'utilisateurs : pas de licence par utilisateur, utilisateurs illimités sur le portail | M | Validé |
| F-FORF-05 | Règle de dépassement de l'enveloppe incluse (ou du budget), par forfait : facturation du dépassement au tarif défini, bascule vers un modèle moins cher, ou blocage ; alertes au client et à l'opérateur à l'approche du seuil | M | Validé |
| F-FORF-06 | Affecter un forfait à un portail ; changer de forfait (montée ou descente) avec date d'effet | M | Validé |
| F-FORF-07 | Options à la carte ajoutées à un forfait (ex. outil supplémentaire, enveloppe d'usage additionnelle, application mobile à la marque du client) | S | Validé |
| F-FORF-08 | Suivi par portail : consommation par rapport à l'enveloppe incluse, reste disponible, projection en fin de période | M | Validé |
| F-FORF-09 | Calcul de la marge à chaque niveau (opérateur, partenaire, sous-partenaire) : prix de revente moins prix d'achat, coefficients multiplicateurs inclus | S | Validé |
| F-FORF-11 | Visibilité par l'admin client de son forfait, de son enveloppe et de sa consommation (si délégué, coefficient appliqué, cf. F-CONSO-09 et F-CONSO-17) | S | Validé |

#### 4.7 Calcul des montants et des commissions — `FACT`

Le générateur **calcule les montants** ; il **n'émet pas les factures** et ne gère ni les encaissements ni les paiements, qui se font dans les outils habituels de l'opérateur et des partenaires.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-FACT-01 | Indiquer, **pour chaque client**, qui le facture : l'opérateur ou le partenaire qui l'a apporté (le calcul en dépend) | M | Validé |
| F-FACT-02 | Calculer, par période, le **montant dû par chaque partenaire** à l'opérateur, quel que soit son niveau (forfaits, options, usage, coefficients appliqués) | M | Validé |
| F-FACT-03 | Calculer le **montant dû par chaque client** facturé directement par l'opérateur | M | Validé |
| F-FACT-04 | Calculer les **commissions** à reverser à chaque partenaire, selon sa place dans la hiérarchie : différence entre ce que paient ses sous-partenaires et la part de l'opérateur (cf. F-CONSO-24) ; un accord peut prévoir des règles complémentaires | M | Validé |
| F-FACT-05 | Relevés par période (montants dus, commissions), consultables dans le générateur et, pour ce qui le concerne, dans l'espace partenaire (cf. F-ESP-09) | M | Validé |
| F-FACT-06 | Export des montants calculés (tableur ou format comptable) pour l'outil de facturation / comptabilité | M | Validé |

#### 4.8 Modèles d'IA, abonnements et clés API — `IA`

Selon le client, les modèles d'IA, les abonnements aux fournisseurs et les clés API sont gérés **par l'opérateur dans le générateur (A)** ou **par l'administrateur du client dans la partie admin (B)**. Le choix se fait portail par portail (cf. F-DELEG-05). Les fonctions ci-dessous sont les mêmes dans les deux cas ; côté client, elles sont limitées à son portail (cf. §4.17).

| ID | Fonctionnalité | Géré en | Priorité | Statut |
|---|---|---|---|---|
| F-IA-01 | Choisir, pour chaque portail, qui gère modèles, abonnements et clés : l'opérateur (A), le client (B), ou un partage (ex. modèles par l'opérateur, clés par le client) | A | M | Validé |
| F-IA-02 | Choisir les fournisseurs de LLM et les modèles proposés sur le portail, sans développement (cf. P-01) | A ou B | M | Validé |
| F-IA-03 | Changer de fournisseur ou de modèle sans modifier le portail (cf. P-01) | A ou B | M | Validé |
| F-IA-04 | Utiliser un modèle différent selon l'usage (génération, rédaction, chat…) | A ou B | S | Validé |
| F-IA-05 | Utiliser un modèle hébergé en local ou sur une infrastructure privée | A ou B | S | Validé |
| F-IA-06 | Gérer les clés d'accès aux fournisseurs d'IA : saisie, stockage chiffré, affectation à un ou plusieurs portails | A ou B | M | Validé |
| F-IA-07 | Gérer les abonnements aux fournisseurs d'IA et outils tiers utilisés par le portail (fournisseur, conditions, dates de validité) — **aucune licence par utilisateur** | A ou B | S | Validé |
| F-IA-08 | Alertes : abonnement bientôt expiré, clé bientôt expirée ou en erreur | A et B | S | Validé |
| F-IA-09 | Quand le client gère lui-même, le générateur garde la vue de ce qui est configuré (modèles, abonnements, clés masquées) et des KPI (cf. F-CONSO-11) | A | M | Validé |
| F-IA-10 | **Confidentialité par usage** : pour chaque usage (chat, documents, comptes rendus, automatisations…), choisir les modèles autorisés selon la sensibilité des données (ex. données sensibles → modèles hébergés en France ou en local uniquement) | A ou B | M | Validé |
| F-IA-11 | Masquage automatique des données personnelles (noms, adresses, numéros…) avant envoi à un modèle externe, activable par usage | A ou B | S | Validé |
| F-IA-12 | Garantie que les fournisseurs d'IA retenus ne réutilisent pas les données pour entraîner leurs modèles ; affichage de cette information pour chaque modèle | A | M | Validé |

#### 4.9 Publication et déploiement — `PUB`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-PUB-01 | Mettre en service, suspendre ou réactiver un portail | M | Validé |
| F-PUB-02 | Le portail se cale sur le nom de domaine du client (ex. `ia.nom-du-client.fr`), certificat de sécurité inclus | M | Validé |
| F-PUB-04 | Historique des versions et retour arrière | S | Validé |
| F-PUB-05 | Export complet du portail (contenus, configuration, médias) dans un format ouvert (cf. P-02) | M | Validé |

#### 4.10 Administration du générateur — `ADM`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-ADM-01 | Gestion des comptes ayant accès au générateur et de leurs rôles | M | Validé |
| F-ADM-03 | Liste des clients et de leurs portails : contact, contrat, hébergement | S | Validé |
| F-ADM-04 | Journal des actions sur le générateur (qui a modifié quoi, quand) | M | Validé |

### Partie B — Portail client : partie admin

Utilisée par l'administrateur du client. **Chaque module n'est visible que si le générateur l'a délégué** (cf. §4.4).

#### 4.11 Création par la conversation (« vibe coding ») — `VIBE`

Cœur de la partie admin (cf. P-10) : **le client parle, le portail construit**. Les fonctions du portail (chats spécialisés, automatisations, formulaires, ensembles de sources, comptes rendus…) se créent ainsi, sans écrans à options multiples.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-VIBE-01 | Décrire en langage naturel, **à l'écrit ou à la voix**, ce que l'on veut créer ou modifier ; le portail le construit | M | Validé |
| F-VIBE-02 | Créer ainsi un **chat spécialisé** (rôle, ton, sources, actions autorisées), doté de sa propre politique de chat (cf. §4.12) | M | Validé |
| F-VIBE-03 | Créer ainsi une **automatisation** (ex. « chaque lundi, résume les nouveaux courriers et envoie-le au service »), détaillée au §4.13 | M | Validé |
| F-VIBE-04 | Créer ainsi un **formulaire** qui déclenche un traitement par l'IA | M | Validé |
| F-VIBE-05 | Créer ainsi un **ensemble de sources** (documents, dossiers, sources connectées) utilisable dans les politiques de chat | M | Validé |
| F-VIBE-06 | Le portail pose des **questions de clarification** avant de construire, plutôt que d'afficher des options ; il en pose **suffisamment** pour lever les ambiguïtés (préférence exprimée : plus de questions plutôt que moins) | M | Validé |
| F-VIBE-07 | **Aperçu et test immédiats** de ce qui a été créé, puis ajustements par le dialogue (« plus court », « ajoute tel document ») | M | Validé |
| F-VIBE-08 | Résumé en langage simple de ce qui a été créé (ce que fait l'objet, avec quelles données, pour qui) | M | Validé |
| F-VIBE-09 | Réglages avancés accessibles seulement sur demande, jamais affichés par défaut | S | Validé |
| F-VIBE-10 | Versions et retour arrière (« reviens à la version d'hier ») | S | Validé |
| F-VIBE-11 | Ce qui est créé respecte la délégation (cf. §4.4), les droits (cf. §4.14) et les outils ouverts sur le portail ; publication auprès des utilisateurs, équipes ou profils choisis | M | Validé |
| F-VIBE-17 | À la publication d'un objet créé en parlant, **l'opérateur et le partenaire** du client sont prévenus | M | Validé |
| F-VIBE-18 | Des **écrans classiques de consultation** (listes, tableaux) restent disponibles à côté de la création en parlant | M | Validé |
| F-VIBE-12 | Possibilité pour les utilisateurs finaux de créer leurs propres chats spécialisés personnels de la même façon, si l'admin l'autorise | C | Validé |
| F-VIBE-13 | Créer ainsi des **prompts partagés** (demandes types) proposés aux utilisateurs dans le chat | M | Validé |
| F-VIBE-14 | Créer ainsi des **consignes réutilisables** (savoir-faire partagés entre plusieurs chats spécialisés, ex. « rédiger une délibération ») | M | Validé |
| F-VIBE-15 | Ajouter par la conversation une **validation humaine** à une automatisation ou un chat spécialisé (« demande l'accord du directeur avant d'envoyer ») | M | Validé |
| F-VIBE-16 | Décrire par la conversation le **contexte de l'organisation** (identité, vocabulaire, règles de rédaction), pris en compte dans toutes les réponses de l'IA | M | Validé |

#### 4.12 Politiques de chat — `POL`

Une **politique de chat** fixe, pour un **utilisateur**, un **profil** ou une **équipe**, ce que le chat peut consulter, quels modèles il utilise, et ce qu'il peut faire. Le chat principal et chaque chat spécialisé en ont une.

**Règle de combinaison** : quand plusieurs politiques s'appliquent (profil, équipes, utilisateur, chat spécialisé), **la plus restrictive l'emporte** sur chaque réglage.

**Sources**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-POL-01 | Choisir les sources que le chat peut consulter : dossiers, documents, ensembles de sources, intégrations (SharePoint, Pennylane…) | M | Validé |
| F-POL-02 | **Mode strict** : le chat répond uniquement à partir des sources autorisées ; sinon il indique qu'il ne sait pas | M | Validé |
| F-POL-03 | Autoriser ou interdire la **recherche sur internet** | M | Validé |
| F-POL-04 | **Sources prioritaires** : ordre de priorité entre sources (ex. d'abord le règlement intérieur, puis le reste) | M | Validé |

**Modèles**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-POL-05 | **Liste des modèles autorisés** pour ce public, parmi ceux ouverts sur le portail | M | Validé |
| F-POL-06 | **Modèle par défaut** utilisé si l'utilisateur ne choisit rien | M | Validé |
| F-POL-07 | **Routage par sensibilité** : si la question ou un document contient des données sensibles, bascule automatique vers un modèle hébergé en France ou en local (cf. F-IA-10) | M | Validé |
| F-POL-25 | Définition des **données sensibles** : liste prédéfinie (santé, social, identité, données financières…) que l'admin peut compléter **en parlant** | M | Validé |
| F-POL-26 | Si l'utilisateur n'a pas le droit de choisir son modèle, un **routeur** choisit automatiquement le modèle adapté à chaque demande dans la liste autorisée | M | Validé |
| F-POL-08 | **Plafond de consommation** (tokens ou euros) par personne, par jour ou par mois | M | Validé |

**Processus**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-POL-09 | **Consignes permanentes** : ton, règles de rédaction, sujets interdits, façon de répondre (ex. « toujours citer l'article du règlement ») | M | Validé |
| F-POL-10 | **Actions autorisées** : ce que le chat peut faire (envoyer un e-mail, créer une fiche dans Zoho, déposer un document…) | M | Validé |
| F-POL-11 | **Validation humaine** : actions soumises à l'accord d'un responsable avant exécution | M | Validé |
| F-POL-12 | **Traitements enchaînés** déclenchables depuis le chat (ex. « instruire une demande de subvention »), créés par la conversation (cf. F-VIBE-03) | M | Validé |

**Libertés de l'utilisateur** (toujours dans les limites de sa politique)

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-POL-13 | Choisir son modèle parmi ceux autorisés | M | Validé |
| F-POL-14 | Ajouter ses propres documents à une conversation | M | Validé |
| F-POL-15 | Ajouter ses **consignes personnelles** (ton, format), qui s'ajoutent sans contredire la politique | M | Validé |
| F-POL-16 | **Voir sa politique** : sources, modèles et actions ouverts ; en cas de refus, **le motif et la personne à contacter** | M | Validé |
| F-POL-27 | **Demander une exception** depuis le chat (ex. accès à une source) : la demande part à l'admin, qui accepte ou refuse | M | Validé |

**Gestion et contrôle**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-POL-17 | Politiques gérées par : les **administrateurs**, un **groupe admin dédié** (« gestionnaires IA », sans les autres droits d'administration) et les **responsables d'équipe** pour leur équipe, qui peuvent seulement **durcir** la politique de leur équipe, jamais l'assouplir | M | Validé |
| F-POL-18 | Un chat spécialisé a sa propre politique ; à l'usage, celle de l'utilisateur **et** celle du chat spécialisé s'appliquent, la plus restrictive l'emporte | M | Validé |
| F-POL-19 | Création **en parlant** (« l'équipe RH n'utilise que des modèles français et les documents RH »), puis **fiche en quatre blocs** (sources, modèles, processus, libertés) pour relire et ajuster | M | Validé |
| F-POL-20 | **Voir comme…** : tester le chat exactement comme le voit un utilisateur, un profil ou une équipe | M | Validé |
| F-POL-21 | **Explication d'une réponse** : sources, modèle et règles qui ont joué, sans accès au reste de la conversation | M | Validé |
| F-POL-22 | **Journal des refus** : demandes bloquées par une politique (source, modèle ou action non autorisés) | M | Validé |
| F-POL-28 | **Politiques temporaires** avec date de fin (ex. stagiaire jusqu'au 31/12) | M | Validé |
| F-POL-29 | Quand une politique change, les utilisateurs concernés reçoivent un **message simple** | M | Validé |
| F-POL-30 | Panneau **« Politique effective »** : résultat de la combinaison pour une personne et politique à l'origine de chaque restriction | S | Validé |
| F-POL-31 | Les **consignes personnelles** des utilisateurs sont visibles par l'admin ; les utilisateurs en sont informés | M | Validé |
| F-POL-23 | L'admin **ne lit pas** les conversations des utilisateurs | M | Validé |
| F-POL-24 | **Politique par défaut** du portail, appliquée automatiquement à tout nouvel utilisateur tant qu'il n'a ni profil ni équipe | M | Validé |

#### 4.13 Automatisations — `AUTO`

Une **automatisation** enchaîne des étapes réalisées par l'IA (lire, résumer, rédiger, extraire, écrire dans un logiciel…) sans que personne n'ait à les lancer une à une. Elle se crée **en parlant** (cf. F-VIBE-03).

**Qui crée, avec quels droits**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-AUTO-01 | Création par les **administrateurs**, les **gestionnaires IA**, les **responsables d'équipe** (pour leur équipe) et les **utilisateurs pour eux-mêmes** (automatisations personnelles, si leur politique l'autorise) | M | Validé |
| F-AUTO-02 | Chaque automatisation s'exécute avec **sa propre politique** (sources, modèles, actions), indépendante des personnes, dans les limites de la politique de son créateur | M | Validé |

**Ce qui la lance**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-AUTO-03 | Un **horaire** (ex. chaque lundi à 8 h, chaque fin de mois) | M | Validé |
| F-AUTO-04 | Un **événement reçu** : e-mail reçu, document déposé dans un dossier, formulaire rempli | M | Validé |
| F-AUTO-05 | Un **audio de réunion** déposé ou enregistré | M | Validé |
| F-AUTO-06 | Un **événement d'un logiciel externe** (Zoho, Pennylane, Notion… : nouvelle facture, nouveau contact) | M | Validé |
| F-AUTO-07 | Une **demande dans le chat** (« lance la revue du courrier ») | M | Validé |

**Ce qu'elle produit**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-AUTO-08 | Un **document** (compte rendu, synthèse, courrier, rapport) déposé dans l'espace documentaire ou un dossier connecté | M | Validé |
| F-AUTO-09 | Un **e-mail ou un message** (e-mail, Teams, chat de l'utilisateur) | M | Validé |
| F-AUTO-10 | Une **mise à jour de logiciel** (fiche Zoho, écriture Pennylane, page Notion…) | M | Validé |
| F-AUTO-11 | Des **données structurées** (ex. tableau des décisions avec responsables et échéances), exportables ou réutilisables | M | Validé |
| F-AUTO-12 | L'**appel d'une autre automatisation**, sans limite de profondeur mais **sans boucle** (une automatisation ne peut pas se rappeler elle-même, directement ou indirectement) | M | Validé |
| F-AUTO-13 | L'**appel d'une validation humaine** comme étape | M | Validé |

**Validation humaine**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-AUTO-14 | Validation **selon l'action** : seules les actions marquées « avec validation » dans la politique attendent un accord | M | Validé |
| F-AUTO-15 | **Délai et relance** : sans réponse dans le délai fixé, relance, puis abandon ou escalade vers un autre responsable | M | Validé |

**Création, test et suivi**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-AUTO-16 | Relecture sous forme de **schéma simple** (étapes et flèches), en lecture seule ; les modifications se font **en parlant** | M | Validé |
| F-AUTO-17 | **Essai à blanc** avant activation, sur **le dernier cas réel** ou sur **un cas choisi** : exécution sans rien envoyer ni modifier ; on voit ce qu'elle aurait fait | M | Validé |
| F-AUTO-18 | **Historique détaillé** de chaque exécution : date, durée, coût, étapes, validations, résultat | M | Validé |
| F-AUTO-19 | **Relancer** manuellement une exécution échouée ou refusée | M | Validé |
| F-AUTO-20 | **Versions** à chaque modification, avec retour arrière | M | Validé |
| F-AUTO-21 | **Résultat sous forme de message dans le chat principal** de la personne concernée (« la revue du courrier est prête ») | M | Validé |

**Échecs et limites**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-AUTO-22 | En cas d'échec : **le créateur est prévenu** avec le motif, et l'erreur apparaît dans la **supervision de l'admin** (cf. F-SUP-02) ; pas de nouvel essai automatique | M | Validé |
| F-AUTO-23 | **Modèle de secours** : si le modèle prévu est indisponible, bascule sur un autre modèle autorisé par la politique de l'automatisation | M | Validé |
| F-AUTO-24 | **Budget par automatisation** (tokens ou euros, par exécution et par mois) | M | Validé |
| F-AUTO-25 | **Fréquence maximale** d'exécution | M | Validé |
| F-AUTO-26 | **Nombre maximal d'automatisations personnelles** par utilisateur | M | Validé |
| F-AUTO-27 | **Durée de vie** : date de fin, ou désactivation automatique si inutilisée depuis un délai réglable | M | Validé |
| F-AUTO-30 | **Arrêt d'urgence** : l'admin peut mettre en pause d'un coup toutes les automatisations du portail | M | Validé |
| F-AUTO-31 | **Coût imputé à la personne qui lance** l'automatisation ; pour un lancement automatique (horaire, événement, audio, logiciel externe), au **créateur** de l'automatisation | M | Validé |

**Réutilisation**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-AUTO-28 | **Catalogue du portail** : les admins publient des automatisations que chacun peut activer | M | Validé |
| F-AUTO-32 | Quand un utilisateur active une automatisation du catalogue, **lui seul** voit ses résultats | M | Validé |
| F-AUTO-29 | **Modèles fournis par l'opérateur ou le partenaire**, réutilisables sur tous les portails (ex. « revue du courrier », « compte rendu de conseil municipal ») | M | Validé |

#### 4.14 Utilisateurs, profils et équipes — `ACC`

La gestion des droits repose sur **trois niveaux** : droits par **utilisateur**, par **profil** (rôle) et par **équipe**.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-ACC-01 | Partie admin propre à chaque portail, réservée aux administrateurs du client | M | Validé |
| F-ACC-02 | Gestion des utilisateurs : invitation (unitaire ou import de fichier), activation, désactivation, suppression | M | Validé |
| F-ACC-03 | Gestion des profils (rôles) : profils prédéfinis (ex. Administrateur, Gestionnaire, Utilisateur, Lecteur) et profils personnalisés | M | Validé |
| F-ACC-04 | Gestion des équipes : créer des équipes, y rattacher des utilisateurs, désigner un responsable | M | Validé |
| F-ACC-05 | Droits attribuables à chacun des trois niveaux (utilisateur, profil, équipe) sur chaque outil, modèle d'IA, source de données, chat spécialisé, intégration | M | Validé |
| F-ACC-06 | Règles de combinaison des droits claires (le plus restrictif l'emporte, comme pour les politiques de chat, cf. §4.12) et simulateur « que peut faire cet utilisateur ? » | S | Validé |
| F-ACC-07 | Outils ouverts par équipe, dans la limite de ce que l'opérateur ou le partenaire a activé pour le portail (cf. F-OUT-02) | M | Validé |
| F-ACC-08 | Budget ou quota de consommation par équipe, par utilisateur ou par **étiquette** (projet, centre de coût) (cf. §4.5) | S | Validé |
| F-ACC-09 | Connexion : code par e-mail, mot de passe avec double authentification, ou connexion unique (SSO Microsoft, Google, ProConnect…) | M | Validé |
| F-ACC-12 | Portail **privé par défaut** : accès uniquement après connexion (seul le chatbot public, s'il est activé, est accessible sans connexion, cf. F-UI-06) | M | Validé |
| F-ACC-13 | **Demandes RGPD des utilisateurs** : file des demandes (accès, export, rectification, suppression), échéance légale d'un mois affichée, exécution et traçabilité | M | Validé |
| F-ACC-10 | Synchronisation des utilisateurs et équipes avec l'annuaire du client (Microsoft Entra ID, Google Workspace…) | S | Validé |
| F-ACC-11 | Journal des actions d'administration (qui a donné quel droit, quand) | M | Validé |

#### 4.15 Fiches organisations et contacts — `FICHE`

Fiches **organisations** et **contacts** servant à la **relation usagers / clients**, à la **prospection**, à un **mini-CRM**, à un **annuaire interne** et de **contexte pour l'IA**. Deux modes, au choix pour chaque portail.

**Modes**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-FICHE-01 | **Mode intégré** : fiches organisations et contacts gérées dans le portail | M | Validé |
| F-FICHE-02 | **Mode externe** : fiches gérées dans un logiciel externe, consultées depuis le portail via **MCP**, **sans aucune copie** dans le portail (interrogation à chaque besoin) ; l'**origine de l'information** est toujours affichée (ex. « lu dans Zoho à l'instant ») | M | Validé |
| F-FICHE-03 | Choix du mode (intégré, externe, ou les deux) par l'administrateur client | M | Validé |
| F-FICHE-31 | Fiche présentée **sur une seule page** : identité, contacts, suivi, chronologie | M | Validé |
| F-FICHE-10 | Logiciels externes visés en priorité : **Pennylane, Zoho, Notion** *(liste à compléter)* | M | Validé |
| F-FICHE-11 | En mode externe, **création et modification** de fiches dans le logiciel **après validation** d'une personne | M | Validé |

**Contenu et saisie**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-FICHE-04 | Lien entre fiches : un contact rattaché à une ou plusieurs organisations | M | Validé |
| F-FICHE-05 | Champs personnalisables par le client (ex. SIRET pour une entreprise, code INSEE pour une commune) | S | Validé |
| F-FICHE-12 | Créer ou modifier une fiche **en parlant** (« ajoute Julie Martin, présidente de l'association Les Amis du Moulin ») | M | Validé |
| F-FICHE-13 | Créer une fiche **depuis un document** : carte de visite photographiée, signature d'e-mail, courrier | M | Validé |
| F-FICHE-14 | **Formulaire classique** de saisie et de modification | M | Validé |
| F-FICHE-08 | **Import / export** (tableur, export d'un autre logiciel) | M | Validé |

**Alimentation automatique**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-FICHE-15 | **E-mails** échangés avec le contact rattachés à sa fiche | M | Validé |
| F-FICHE-16 | **Comptes rendus** où l'organisation ou le contact est cité ou présent, rattachés à sa fiche | M | Validé |
| F-FICHE-17 | **Documents** liés (courriers, factures, conventions) rattachés à la fiche | M | Validé |
| F-FICHE-18 | **Complétion par les données publiques** : SIRET, adresse, dirigeants (annuaire des entreprises), code INSEE | M | Validé |

**Mini-CRM et prospection**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-FICHE-19 | **Étapes** (prospect, contacté, rendez-vous, proposition, gagné / perdu), affichées en **liste** (pas de vue en colonnes) | M | Validé |
| F-FICHE-20 | **Opportunités** : montant, probabilité, date de décision | M | Validé |
| F-FICHE-21 | **Historique des échanges** en chronologie (e-mails, réunions, appels, documents) | M | Validé |
| F-FICHE-22 | **Rappels** de relance, dans les tâches du portail | M | Validé |
| F-FICHE-23 | **Trouver des cibles** selon des critères (secteur, taille, zone) dans les données publiques | M | Validé |
| F-FICHE-24 | **Enrichir les fiches** automatiquement (site, dirigeants, effectif, actualités) | M | Validé |
| F-FICHE-25 | **Rédiger des messages de prospection** personnalisés, envoyés après validation | M | Validé |
| F-FICHE-26 | **Séquences de relance** automatiques tant qu'il n'y a pas de réponse ; **nombre de relances réglable par séquence** (automatisation, cf. §4.13) | M | Validé |
| F-FICHE-06 | Utilisation des fiches par le chat principal et les chats spécialisés (« prépare un courrier pour tel contact »), dans le respect des droits | M | Validé |

**Qualité, visibilité et RGPD**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-FICHE-27 | **Détection des doublons** ; la fusion est faite par **l'utilisateur** | M | Validé |
| F-FICHE-28 | **Fiches privées**, visibles de leur seul créateur | M | Validé |
| F-FICHE-29 | **Fiches par équipe** (ex. contacts RH visibles de la seule équipe RH) | M | Validé |
| F-FICHE-07 | Droits d'accès aux fiches par utilisateur, profil et équipe (cf. §4.14) | M | Validé |
| F-FICHE-30 | **Opposition à la prospection** : un contact marqué ne reçoit plus aucun message de prospection, séquences arrêtées | M | Validé |
| F-FICHE-09 | Conformité RGPD : durée de conservation, droits des personnes (accès, rectification, effacement), traçabilité des consultations ; information des personnes prospectées et respect des règles de prospection (B2B / particuliers) (cf. §5.1) | M | Validé |

#### 4.16 Intégrations bureautiques et MCP — `INT`

Gérées depuis la **partie admin du client**, dans la limite des intégrations ouvertes par l'opérateur.

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-INT-01 | **Microsoft 365** : SharePoint, OneDrive, Teams, Outlook (e-mails, agenda), connexion Entra ID | M | Validé |
| F-INT-02 | **Google Workspace** : Drive, Gmail, Agenda, connexion Google | M | Validé |
| F-INT-03 | **Suite bureautique française** (ex. La Suite numérique de l'État) — *à étudier plus tard* | C | Reporté |
| F-INT-04 | **Serveurs MCP** : catalogue de serveurs MCP proposés par l'opérateur, activables par le client | M | Validé |
| F-INT-05 | Ajout par le client de ses propres serveurs MCP (adresse, authentification) | M | Validé |
| F-INT-06 | Droits sur chaque intégration et chaque serveur MCP par utilisateur, profil et équipe (cf. §4.14) | M | Validé |
| F-INT-07 | Choix, par intégration, des actions autorisées (lecture seule, ou lecture et écriture) et validation humaine avant une action sensible | S | Validé |
| F-INT-08 | État de chaque intégration (connectée, en erreur, dernière synchronisation) et test de connexion | S | Validé |
| F-INT-10 | Le portail lui-même exposé en serveur MCP, pour être utilisé depuis d'autres outils d'IA du client (cf. P-04) | C | Validé |

#### 4.17 Modèles d'IA, abonnements et clés API — `CLE`

Selon le client, ces éléments sont gérés par l'opérateur dans le générateur ou **délégués à l'admin client** (cf. §4.8 et F-DELEG-05). Quand ils sont délégués, le client dispose des mêmes fonctions que le générateur, limitées à son portail. Les clés des intégrations et les clés API du portail sont toujours gérées côté client.

| ID | Fonctionnalité | Niveau | Priorité | Statut |
|---|---|---|---|---|
| F-CLE-01 | Si délégué : choisir ses modèles d'IA, gérer ses propres clés et abonnements de fournisseurs d'IA (fonctions F-IA-02 à F-IA-08) | Client | M | Validé |
| F-CLE-02 | Clés d'accès aux intégrations et serveurs MCP (cf. §4.16) | Client | M | Validé |
| F-CLE-03 | Clés API du portail, pour que les applications du client l'interrogent : création, droits limités (portée), date d'expiration, restriction d'adresses IP, quota | Client | M | Validé |
| F-CLE-04 | Renouvellement (rotation) et révocation immédiate d'une clé | Générateur et client | M | Validé |
| F-CLE-05 | Stockage chiffré des clés ; une clé n'est affichée qu'une fois à sa création | Générateur et client | M | Validé |
| F-CLE-06 | Journal d'utilisation de chaque clé ; la consommation associée remonte au générateur (cf. F-CONSO-11) | Générateur et client | M | Validé |
| F-CLE-07 | Alerte avant expiration d'une clé | Générateur et client | C | Validé |

#### 4.18 Sources de données — `SRC`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-SRC-01 | Connecter une ou plusieurs sources de données à un portail, sans développement (cf. P-05) | M | Validé |
| F-SRC-02 | Types de sources pris en charge en V1 : **fichiers importés** (PDF, Word, Excel, PowerPoint, texte), **bases de données** en lecture seule (ex. PostgreSQL, MySQL), **API de logiciels métier** (quand il n'existe pas de serveur MCP) ; en plus des intégrations Microsoft 365, Google Workspace et MCP (cf. §4.16). Aspiration de sites web : hors V1 | M | Validé |
| F-SRC-03 | Ajouter, remplacer ou retirer une source sans modifier le portail (cf. P-05) | M | Validé |
| F-SRC-04 | Synchroniser les données (à la demande ou automatiquement, fréquence réglable) | S | Validé |
| F-SRC-05 | Choisir quelles données de chaque source sont visibles, et par qui | S | Validé |
| F-SRC-07 | Stocker de façon sécurisée les accès (identifiants, clés) aux sources | M | Validé |

#### 4.19 Personnalisation visuelle — `DSN`

L'apparence se règle aussi **en parlant** (ex. « mets nos couleurs, voici notre logo », cf. P-10).

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-DSN-01 | Logo, couleurs, typographies (charte graphique) | M | Validé |
| F-DSN-02 | Thèmes prédéfinis | S | Validé |
| F-DSN-03 | Aperçu en direct (ordinateur / tablette / mobile) | M | Validé |
| F-DSN-04 | Mode clair / sombre | C | Validé |
| F-DSN-05 | Marque blanche totale : aucune mention de l'éditeur du générateur sur le portail (cf. P-03) | M | Validé |
| F-DSN-06 | Personnaliser les e-mails envoyés (expéditeur, logo, textes) aux couleurs de la marque du portail (cf. P-03) | S | Validé |
| F-DSN-07 | Pages légales pré-remplies (mentions légales, confidentialité, CGU), adaptées au client | S | Validé |
| F-DSN-08 | Portail multilingue | C | Validé |

#### 4.20 Tableau de bord de l'admin client (KPI) — `STA`

KPI disponibles pour l'admin du client. Règles communes : **coûts affichés coefficient compris** (cf. F-CONSO-17) ; détail **nominatif ou par équipe selon le réglage du portail** (cf. F-CONSO-34) ; filtres par période, équipe, modèle, outil ; export tableur.

**Vue d'ensemble**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-STA-01 | **Utilisateurs actifs** (jour, semaine, mois), nouveaux utilisateurs, utilisateurs inactifs, **taux d'adoption** | M | Validé |
| F-STA-04 | **Conversations et messages**, tendance sur la période | M | Validé |
| F-STA-05 | **Tokens** (entrée / sortie) et **coût** de la période, évolution et projection de fin de mois | M | Validé |
| F-STA-06 | **Consommation par rapport à l'enveloppe du forfait** et aux budgets | M | Validé |

**Répartitions**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-STA-07 | **Par modèle** : requêtes, tokens, coût, temps de réponse moyen, taux d'erreur | M | Validé |
| F-STA-03 | **Par équipe** : tokens, coût, part du budget consommée, alertes de plafond | M | Validé |
| F-STA-08 | **Par utilisateur** (si le suivi nominatif est activé) : tokens, coût, part du plafond personnel | M | Validé |
| F-STA-09 | **Par étiquette** (projet, centre de coût…), avec **budget par étiquette** | M | Validé |
| F-STA-02 | **Par outil** : chat principal, chats spécialisés les plus utilisés, automatisations, comptes rendus, fiches | M | Validé |
| F-STA-10 | **Automatisations** : exécutions, taux de succès / échec, bascules sur modèle de secours, coût par automatisation | M | Validé |
| F-STA-11 | **Comptes rendus** : réunions traitées, heures d'audio transcrites | M | Validé |

**Ressources et environnement**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-STA-12 | **Ressources** : documents indexés, volume de stockage, sources connectées et état de leur synchronisation | M | Validé |
| F-STA-13 | **Empreinte carbone** estimée de l'usage de l'IA, par modèle et par période | M | Validé |

**Qualité et fiabilité** (cf. §4.21)

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-STA-14 | **Satisfaction** : avis utile / pas utile, par chat spécialisé et par modèle | M | Validé |
| F-STA-15 | **Erreurs des modèles** et indisponibilités, par modèle et par période | M | Validé |
| F-STA-16 | **Refus de politique** et **demandes d'exception** (nombre, motifs, délais de traitement) | M | Validé |
| F-STA-17 | **Demandes RGPD** en cours et respect du délai d'un mois (cf. F-ACC-13) | M | Validé |

**Diffusion**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-STA-18 | **Rapport périodique** envoyé par e-mail à l'admin (hebdomadaire ou mensuel) | M | Validé |
| F-STA-19 | Tableau de bord **simplifié pour les responsables d'équipe**, limité à leur équipe | M | Validé |

#### 4.21 Supervision et qualité — `SUP`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-SUP-01 | Tableau des **retours des utilisateurs** sur les réponses de l'IA (utile / pas utile, commentaires, cf. F-USR-05), par chat spécialisé et par modèle | M | Validé |
| F-SUP-02 | Suivi des **erreurs** (modèle indisponible, source de données en échec, intégration déconnectée) avec alertes à l'admin client | M | Validé |
| F-SUP-03 | Journal de conformité : actions d'administration, accès, validations humaines (cf. §5.1, §5.2) | M | Validé |
| F-SUP-04 | Suggestions d'amélioration issues des retours (ex. « ce chat spécialisé répond mal sur tel sujet, ajouter tel document ? ») | C | Validé |

### Partie C — Portail client : partie utilisateur

Utilisée par les collaborateurs ou agents du client. Elle s'ouvre sur le chatbot.

#### 4.22 Chatbot — `CHAT`

Le chatbot est la **porte d'entrée** de la partie utilisateur : un **chat principal**, plus des **chats spécialisés** attribués selon les besoins. Tous obéissent à des **politiques de chat** (cf. §4.12).

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CHAT-01 | Le portail s'ouvre directement sur le **chat principal** après connexion ; son comportement est fixé par la politique de chat de l'utilisateur (cf. §4.12) | M | Validé |
| F-CHAT-02 | Chatbot multi-LLM : plusieurs modèles d'IA disponibles sur un même portail (cf. P-01) | M | Validé |
| F-CHAT-03 | Par défaut, le modèle est **choisi automatiquement** par le routeur (cf. F-POL-26) ; l'admin peut, par la politique de chat, permettre à l'utilisateur de voir et choisir le modèle | M | Validé |
| F-CHAT-04 | Réponses appuyées sur les documents et sources du client, avec citation des sources (cf. P-05) | M | Validé |
| F-CHAT-05 | Envoi de fichiers dans la conversation (PDF, Word, images…) pour les analyser | S | Validé |
| F-CHAT-06 | Historique des conversations, recherche, renommage ; la suppression passe par une **demande à l'admin** (cf. F-USR-03) | M | Validé |
| F-CHAT-07 | **Chats spécialisés** (ex. « Urbanisme », « RH ») attribués à des utilisateurs, profils ou équipes, accessibles depuis le chat principal ; créés en parlant par l'admin client (cf. F-VIBE-02) ou fournis par l'opérateur ou le partenaire | M | Validé |
| F-CHAT-13 | Le chat principal **propose de lui-même** le chat spécialisé adapté à la question, et peut le **consulter en coulisse** si l'utilisateur y a droit | M | Validé |
| F-CHAT-14 | **« Pourquoi cette réponse ? »** sous chaque réponse (sources, modèle, règles appliquées), visible par l'utilisateur et par l'admin | M | Validé |
| F-CHAT-08 | Écran d'accueil du chat : **suggestions de questions**, **chats spécialisés disponibles**, **actualités du client** (ex. actualités de la mairie) et liens vers ses outils externes | S | Validé |
| F-CHAT-09 | Dictée vocale des questions | M | Validé |
| F-CHAT-10 | Export d'une réponse ou d'une conversation, et **partage d'une conversation avec un collègue** | M | Validé |
| F-CHAT-11 | Accès secondaire, depuis le chat, aux autres services du portail (documents, chats spécialisés, automatisations) sans surcharger l'écran | S | Validé |
| F-CHAT-12 | **Mémoire IA** : l'IA retient, d'une conversation à l'autre, des informations utiles sur l'utilisateur (fonction, préférences, dossiers en cours), si l'admin l'a activée | M | Validé |

#### 4.23 Documents et réunions — `DOC`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-DOC-01 | **Espace documentaire** : déposer, ranger et retrouver ses documents et ceux de ses équipes, selon les droits | M | Validé |
| F-DOC-02 | Interroger les documents de l'espace depuis le chat, avec citation des sources | M | Validé |
| F-DOC-03 | **Comptes rendus de réunion** : détaillés au §4.24 | M | Validé |

#### 4.24 Comptes rendus de réunion — `CR`

**Sources de la réunion**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CR-01 | Enregistrement depuis l'**application mobile** (cf. §4.26) | M | Validé |
| F-CR-02 | Enregistrement depuis le **navigateur**, dans le portail web | M | Validé |
| F-CR-03 | **Import** d'un fichier audio ou vidéo déjà enregistré | M | Validé |
| F-CR-04 | Récupération automatique de l'enregistrement d'une **visioconférence** (Teams, Google Meet, Visio…) | M | Validé |

**Types de comptes rendus**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CR-05 | Types fournis d'office : **compte rendu standard**, **conseil municipal / assemblée délibérante** (ordre du jour, délibérations, votes, rapporteurs), **relevé de décisions**, **verbatim nettoyé** (texte intégral par intervenant) | M | Validé |
| F-CR-24 | Pour les communes, production de **documents séparés** : procès-verbal, liste des délibérations et compte rendu *(conformité aux règles de publicité des actes issues de la réforme de 2022 à faire vérifier par un juriste)* | M | Validé |
| F-CR-06 | **Créer ses propres types** de comptes rendus, en parlant (« les présents, l'ordre du jour, puis un tableau des actions ») | M | Validé |
| F-CR-07 | **Importer un modèle** (document Word ou PDF, ancien compte rendu) : le portail en déduit la structure et **reprend la mise en page** (en-tête, logo, styles) | M | Validé |
| F-CR-08 | Chaque type fixe sa **règle d'accès** (ex. conseil municipal : public ; réunion RH : confidentiel), sa destination et ses destinataires | M | Validé |

**Contexte de la réunion**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CR-09 | **Liste des présents** : présents, excusés, absents, pouvoirs | M | Validé |
| F-CR-10 | **Ordre du jour** fourni avant la réunion ; le compte rendu suit sa structure | M | Validé |
| F-CR-11 | **Documents de séance** joints (rapport, budget) auxquels l'IA se réfère | M | Validé |
| F-CR-12 | **Quorum et votes** : vérification du quorum, décompte pour / contre / abstentions par délibération | M | Validé |

**Relecture**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CR-13 | **Nommer les locuteurs** : voix séparées automatiquement ; l'IA **propose** un nom d'après ce qui est dit (ex. « Madame la Maire ») ; l'utilisateur **confirme toujours** ; aucune empreinte vocale (cf. F-MOB-18) | M | Validé |
| F-CR-14 | **Corriger en parlant** (« ajoute que le vote a été reporté », « raccourcis la partie budget ») | M | Validé |
| F-CR-15 | **Édition directe** du texte, comme dans un traitement de texte | M | Validé |
| F-CR-16 | **Validation par un tiers** avant diffusion (ex. secrétaire de séance), selon le type | M | Validé |
| F-CR-25 | Après validation, **seul le validateur** peut modifier le compte rendu ; chaque modification crée une nouvelle version tracée | M | Validé |
| F-CR-17 | Les **marqueurs** posés pendant la réunion (décision, action, vote, question, important) sont repris et mis en évidence | M | Validé |

**Diffusion et suites**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CR-18 | **Envoi aux participants** (e-mail ou chat) | M | Validé |
| F-CR-19 | **Dépôt** dans l'espace documentaire ou un dossier connecté (SharePoint, Drive…), selon le type | M | Validé |
| F-CR-20 | **Actions transformées en tâches**, par défaut dans les **tâches du portail** (responsable, échéance, lien vers le compte rendu, rappels) ; envoi possible vers les outils connectés (Planner, Notion, Zoho…) | M | Validé |
| F-CR-21 | Compte rendu **interrogeable dans le chat** (« qu'a-t-on décidé sur la cantine ? »), dans le respect de sa règle d'accès | M | Validé |
| F-CR-22 | Poursuivre la discussion avec l'IA sur un compte rendu (« quelles actions pour moi ? ») | S | Validé |

**Conservation**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-CR-23 | Conservation de l'**audio** et de la **transcription**, chacun pour une **durée réglable**, puis suppression automatique | M | Validé |

#### 4.25 Interfaces et canaux — `UI`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-UI-01 | Séparer contenus/services et présentation : changer d'interface sans refaire le portail (cf. P-04) | M | Validé |
| F-UI-02 | Plusieurs interfaces au choix pour un même portail (ex. portail web classique, interface conversationnelle) | S | Validé |
| F-UI-03 | Intégrer tout ou partie du portail dans un site ou outil existant (widget, iframe) | S | Validé |
| F-UI-04 | Diffuser les services du portail sur d'autres canaux, dont les **applications mobiles Android et iPhone** (cf. §4.26) et la messagerie collaborative | M | Validé |
| F-UI-05 | Exposer les fonctions du portail via une API ouverte, pour qu'un tiers construise sa propre interface | C | Validé |
| F-UI-06 | **Chatbot public** : chatbot accessible sans connexion (ex. sur le site internet du client), créé par la conversation, limité aux sources publiques choisies, sa consommation étant comptée dans les KPI du portail | M | Validé |
| F-UI-07 | **Bot Microsoft Teams** : utiliser le chatbot et les chats spécialisés du portail depuis Teams, avec les mêmes droits | M | Validé |
| F-UI-08 | Protection du chatbot public contre les abus (limite de questions, filtrage des sujets hors périmètre) | M | Validé |

#### 4.26 Applications mobiles Android et iPhone — `MOB`

Applications pour les **utilisateurs** (pas d'administration, d'espace partenaire ni de générateur sur mobile), centrées sur **le chat et l'audio**.

**Publication**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-MOB-01 | **Application commune** Android et iPhone, publiée dans les stores **à la marque de l'opérateur** ; à la connexion (**QR code** ou adresse du portail), elle prend le nom, le logo et les couleurs du client | M | Validé |
| F-MOB-02 | **Application dédiée à la marque du client**, publiée à son nom dans les stores, en **option du forfait** (cf. F-FORF-07) | S | Validé |

**Chat**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-MOB-03 | Chat principal et chats spécialisés, avec **les mêmes politiques de chat que sur le web** (sources, modèles, actions, limites) | M | Validé |
| F-MOB-04 | **Parler au chat** : dicter ses questions et écouter les réponses lues à voix haute ; lecture automatique ou à la demande, **réglable par l'utilisateur** | M | Validé |
| F-MOB-05 | **Photographier un document** (courrier, facture) et l'envoyer au chat pour analyse | M | Validé |

**Audio de réunion**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-MOB-06 | **Enregistrer une réunion** sur le téléphone et l'envoyer au portail (compte rendu, automatisations, cf. F-AUTO-05) | M | Validé |
| F-MOB-07 | **Enregistrement hors connexion** ; envoi automatique au retour du réseau (enregistrement chiffré en attente) | M | Validé |
| F-MOB-08 | **Longues réunions**, jusqu'à **4 heures** par enregistrement, écran éteint, sans coupure | M | Validé |
| F-MOB-09 | **Rappel du consentement** avant d'enregistrer : informer les participants (RGPD), avec message type | M | Validé |
| F-MOB-10 | **Marqueurs** pendant la réunion : **Décision, Action, Vote, Question, Important**, repris dans le compte rendu | M | Validé |
| F-MOB-11 | À la fin, **choisir le traitement** (compte rendu standard, format conseil municipal, relevé de décisions…) parmi les automatisations autorisées | M | Validé |
| F-MOB-18 | **Séparation automatique des voix** (« Intervenant 1 », « Intervenant 2 »…), puis l'utilisateur est **invité à identifier chaque locuteur** (nom choisi parmi les participants ou saisi) avant la finalisation du compte rendu ; **aucune reconnaissance par empreinte vocale** | M | Validé |
| F-MOB-19 | **Continuité web / mobile** : une conversation commencée sur le téléphone se poursuit sur le web, et inversement | M | Validé |

**Notifications et validations**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-MOB-12 | **Notifications** : résultat d'une automatisation, compte rendu prêt, changement de politique | M | Validé |
| F-MOB-13 | **Valider depuis le téléphone** : approuver, refuser ou modifier une action soumise à validation humaine (cf. F-AUTO-14) | M | Validé |

**Sécurité**

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-MOB-14 | **Déverrouillage biométrique** (empreinte, visage) | M | Validé |
| F-MOB-15 | **Aucune donnée conservée** sur le téléphone après envoi, sauf les enregistrements en attente, chiffrés | M | Validé |
| F-MOB-16 | Compatible avec la **gestion de flotte** des téléphones professionnels (MDM, ex. Microsoft Intune) | M | Validé |
| F-MOB-17 | **Effacement à distance** : l'admin déconnecte un téléphone perdu et efface les données de l'app | M | Validé |

#### 4.27 Espace personnel de l'utilisateur — `USR`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-USR-01 | Profil personnel : nom, photo, langue, préférences d'affichage | S | Validé |
| F-USR-02 | Voir les outils et chats spécialisés auxquels on a accès (selon ses droits, son profil et ses équipes) | M | Validé |
| F-USR-03 | **Droits RGPD par demande à l'admin** : l'utilisateur n'exporte ni ne supprime lui-même ses données ; il envoie une demande (accès, export, rectification, suppression de conversations ou du compte), traitée par l'admin dans le délai légal (cf. F-ACC-13) | M | Validé |
| F-USR-04 | Aide intégrée et prise en main (visite guidée, questions fréquentes) | S | Validé |
| F-USR-05 | Donner un avis sur une réponse de l'IA (utile / pas utile, commentaire) | S | Validé |
| F-USR-06 | Consulter, corriger, supprimer ce que l'IA a mémorisé sur soi, ou désactiver sa mémoire (cf. F-CHAT-12) | M | Validé |
| F-USR-07 | **Mes tâches** : actions issues des comptes rendus, rappels des fiches, validations à faire | M | Validé |
| F-USR-08 | **Mes automatisations** : automatisations personnelles et celles du catalogue activées | M | Validé |
| F-USR-09 | **Mes documents** : documents envoyés dans le chat, comptes rendus où l'on est participant | M | Validé |
| F-USR-10 | **Ma consommation** du mois par rapport à son plafond | M | Validé |
| F-USR-11 | **Consignes personnelles** : ton, format, préférences de réponse (visibles par l'admin, cf. F-POL-31) | M | Validé |
| F-USR-12 | **Voix** : lecture à voix haute automatique ou à la demande, choix de la voix, vitesse | M | Validé |
| F-USR-13 | **Notifications** : événements, canal (e-mail, application, Teams), heures de silence | M | Validé |
| F-USR-14 | **Accessibilité** : taille du texte, contraste renforcé, mode clair / sombre | M | Validé |
| F-USR-15 | **Ma politique** : sources, modèles, actions ouverts et leurs limites (cf. F-POL-16) | M | Validé |
| F-USR-16 | **Historique des refus** et suivi de ses **demandes d'exception** en cours | M | Validé |

### Partie D — Espace partenaire

Outil des partenaires et sous-partenaires. Ils **n'ont pas accès au générateur** : ils demandent à l'opérateur de générer les portails et règlent les options depuis cet espace.

#### 4.28 Espace partenaire — `ESP`

| ID | Fonctionnalité | Priorité | Statut |
|---|---|---|---|
| F-ESP-01 | **Demander la génération d'un portail** pour un nouveau client : informations client, nom de domaine, forfait, options souhaitées ; suivi de l'état de la demande. **Par formulaire** (pas de demande en parlant) | M | Validé |
| F-ESP-02 | **Cocher / décocher toutes les options** des portails de ses clients : outils, modèles d'IA, intégrations, modules et niveau de délégation de l'admin client | M | Validé |
| F-ESP-03 | Cocher / décocher les options **ouvertes à ses sous-partenaires**, qui ne pourront pas aller au-delà pour leurs propres clients (cascade, cf. F-PART-06) | M | Validé |
| F-ESP-04 | Les options cochées / décochées s'appliquent **immédiatement** au portail concerné (cf. F-PART-05) | M | Validé |
| F-ESP-05 | Seules les options ouvertes au partenaire par le niveau au-dessus sont proposées | M | Validé |
| F-ESP-06 | KPI d'usage et de coût de **ses propres clients**, avec son coefficient appliqué (cf. F-CONSO-22) | M | Validé |
| F-ESP-16 | Le partenaire voit **sa marge en euros** pour chaque client | M | Validé |
| F-ESP-17 | Le partenaire peut **suspendre et réactiver** le portail de l'un de ses clients (ex. impayé) ; l'opérateur est alerté | M | Validé |
| F-ESP-18 | Pas de contrat ni de conditions à accepter dans l'espace partenaire (le contrat est géré hors de l'outil) | M | Validé |
| F-ESP-07 | **KPI consolidés** de ses sous-partenaires uniquement ; **aucun accès** à la configuration des portails des clients de ses sous-partenaires | M | Validé |
| F-ESP-08 | Gérer ses sous-partenaires (dans la limite de trois niveaux) : demander leur création à l'opérateur, voir leurs KPI consolidés | S | Validé |
| F-ESP-09 | Consulter les montants calculés qu'il doit à l'opérateur et ses relevés de commissions (cf. §4.7) | M | Validé |
| F-ESP-10 | Pour les clients qu'il facture lui-même, disposer des montants calculés (forfait, options, usage coefficienté) pour les facturer avec son propre outil | S | Validé |
| F-ESP-11 | Comptes utilisateurs et rôles au sein de la structure partenaire (ex. administrateur, commercial, lecture seule) | S | Validé |
| F-ESP-12 | Aucun accès aux conversations ni aux documents des clients (cf. P-08) | M | Validé |
| F-ESP-13 | **Fixer la tarification** de chacun de ses clients : coefficient multiplicateur **ou** prix direct (cf. F-CONSO-25) | M | Validé |
| F-ESP-14 | Espace partenaire **à la marque de chaque partenaire** : logo, couleurs, nom, adresse sur son propre nom de domaine, e-mails à son nom | M | Validé |
| F-ESP-15 | **Fixer la tarification de ses sous-partenaires** : coefficient multiplicateur **ou** prix direct ; l'espace montre l'effet sur sa propre marge (cf. F-CONSO-24) | M | Validé |

---

## 5. Exigences transverses (non fonctionnelles)

Indépendantes de la future stack technique.

| Thème | Exigence attendue |
|---|---|
| Ergonomie | Utilisable sans compétence technique ; interface en français |
| Lisibilité | Vocabulaire simple et propre au produit (chat principal, chat spécialisé, politique de chat, sources), sans le jargon des outils existants (« agent », « base de connaissances », « workflow »…). Interface utilisateur épurée, centrée sur le chatbot ; nettement plus lisible que les portails IA existants (peu de menus, vocabulaire simple, actions principales visibles immédiatement) |
| Responsive | Portails générés lisibles sur mobile, tablette et ordinateur |
| Accessibilité | Portails conformes au RGAA (obligatoire pour les collectivités) / WCAG niveau AA |
| Réglementation IA | Transparence envers l'utilisateur (il sait qu'il échange avec une IA) ; conformité AI Act *(à préciser)* |
| Sécurité | Authentification sécurisée (double authentification possible), chiffrement des données, isolation par instance |
| Performance | Chargement d'une page de portail < 2 s *(à confirmer)* |
| Capacité | Le générateur, la supervision et les mises à jour doivent gérer **500 instances** sans dégradation ; mises à jour par lot (cf. F-GEN-11) |
| Disponibilité | *À définir* |
| Sauvegarde | Sauvegarde régulière de chaque instance, tests de restauration |

### 5.1 Conformité RGPD

| Exigence | Détail |
|---|---|
| Localisation des données | **Par défaut, données et sauvegardes hébergées en France** par l'opérateur ; si le client choisit un autre hébergement, il reste dans l'Union européenne *(sauf choix explicite et documenté du client)* |
| LLM | Possibilité de n'utiliser, pour un portail, que des modèles hébergés en Europe ; aucune réutilisation des données par les fournisseurs d'IA pour entraîner leurs modèles |
| Minimisation | Collecter uniquement les données nécessaires ; le générateur ne reçoit que des compteurs (cf. P-08) |
| Suivi nominatif de la consommation | Utilisateurs informés du suivi de leur consommation (finalité : maîtrise des coûts) ; accès réservé aux administrateurs habilités ; durée de conservation définie ; pseudonymisation hors du portail du client |
| Mémoire IA | Informations mémorisées visibles, modifiables et supprimables par l'utilisateur ; désactivable ; durée de conservation définie |
| Enregistrements de réunion | Information des participants avant enregistrement ; voix séparées puis locuteurs nommés par l'utilisateur, **sans empreinte vocale** (pas de donnée biométrique) ; durées de conservation de l'audio et de la transcription réglables, suppression automatique à échéance |
| Durées de conservation | Durée de conservation des conversations et documents réglable par portail, avec suppression automatique |
| Droits des personnes | Accès, rectification, effacement et portabilité (export) pour chaque utilisateur |
| Rôles | Le client est responsable de traitement, l'opérateur sous-traitant (et, le cas échéant, les partenaires dans la chaîne) : modèles de contrats de sous-traitance (article 28) fournis ; les partenaires n'accèdent pas aux contenus |
| Documentation | Registre des traitements, liste des sous-traitants (hébergeur, fournisseurs d'IA), aide à l'analyse d'impact (AIPD) fournis par instance |
| Traçabilité | Journal des accès et des actions d'administration, conservé et consultable |
| Cookies | Uniquement les cookies nécessaires, ou recueil du consentement |
| Fiches nominatives | Fiches organisations et contacts (mode intégré) couvertes par le registre des traitements ; en mode externe (MCP), le logiciel tiers est déclaré comme sous-traitant ou traitement distinct (cf. §4.15) |

### 5.2 Certification ISO

Objectif : **certifications visées**. **Périmètre certifié : la société de l'opérateur et l'hébergement qu'elle opère en France** (exploitation du générateur et des portails hébergés par l'opérateur). Les portails hébergés ailleurs, au choix du client, sont **conçus pour permettre** la certification, sans être couverts par celle de l'opérateur.

| Norme | Objet | Attendu | Objectif |
|---|---|---|---|
| ISO/IEC 27001 | Sécurité de l'information | Gestion des accès, chiffrement, journalisation, sauvegarde, gestion des incidents, gestion des mises à jour | Certification visée |
| ISO/IEC 27701 | Protection de la vie privée (extension RGPD de 27001) | Mesures RGPD documentées (cf. §5.1) | Non visée : mesures RGPD appliquées sans certification |
| ISO/IEC 42001 | Système de management de l'IA | Inventaire des modèles utilisés, supervision humaine, gestion des risques liés à l'IA | Certification visée |

---

## 6. Hors périmètre (pour la première version)
- **Émission des factures**, encaissements et paiement des commissions : l'outil calcule les montants, la facturation se fait dans les outils existants (cf. §4.7).
- **Regroupement par projets** (dossiers réunissant documents et conversations d'un même sujet) : pas en V1.
- *À compléter.*

---

## 7. Questions ouvertes
1. ~~Qu'est-ce qu'un « portail » ?~~ → Un portail IA par client, ouvert sur un chatbot multi-LLM (cf. §1.2).
2. ~~Qui crée les portails ?~~ → L'opérateur, à la demande, pour chaque client (cf. §3). Le client modifie ensuite son portail selon le niveau de délégation de son forfait (cf. §4.4).
3. ~~Combien de portails ?~~ → 500 à terme (cf. §1.4).
4. ~~Portails privés par défaut ?~~ → Oui (cf. F-ACC-12).
5. ~~Suivi de la consommation d'IA par client ?~~ → Oui, en tokens et en coût (cf. §4.5). Les clients souscrivent des **forfaits**, usage inclus ou non (cf. §4.6). La facturation se fait hors de l'outil, qui calcule seulement les montants (cf. §4.7).
6. ~~Quelles fonctions en V1 ?~~ → Simplifiées et créées par la conversation : chats spécialisés, ensembles de sources, prompts, consignes réutilisables, automatisations, formulaires, validation humaine, comptes rendus, espace documentaire, contexte de l'organisation. Projets écartés en V1 
7. ~~Qui fournit les accès aux LLM ?~~ → Selon le client : l'opérateur ou le client lui-même (cf. §4.8). Les coûts sont distingués dans les KPI (cf. F-CONSO-14).
8. ~~Qui héberge ?~~ → Par défaut l'opérateur, en France ; chaque client peut choisir son type d'hébergement (cf. P-02, F-GEN-13).
9. ~~Alignement ou certification ISO ?~~ → Certification **ISO 27001 et ISO 42001** visée, périmètre : société de l'opérateur + hébergement en France (cf. §5.2).
10. ~~Consommation par utilisateur nominatif ?~~ → Oui (cf. F-CONSO-03), avec les garanties RGPD du §5.1.
11. ~~Suite bureautique française ?~~ → À voir plus tard (F-INT-03 reportée).
12. ~~Qui gère les clés API ?~~ → Selon le client : l'opérateur (A) ou le client (B) ; les KPI remontent toujours en A (cf. §4.8, §4.5).
13. Logiciels externes pour les fiches en mode MCP : Pennylane, Zoho, Notion (cf. F-FICHE-10). *Liste à compléter.*
14. ~~Que recouvre une « licence » ?~~ → Pas de licence par utilisateur : le client paie un forfait, usage inclus ou non (cf. §4.6).
15. ~~KPI prioritaires du générateur ?~~ → Coût réel, marge, consommation / enveloppe (cf. F-CONSO-33).
16. ~~Portée du coefficient ?~~ → Tokens et coûts uniquement, coefficient ajustable par modèle d'IA (cf. F-CONSO-31, F-CONSO-32).
17. ~~Combien de niveaux de partenaires ?~~ → Trois au maximum (cf. P-09).
18. ~~Qui facture ?~~ → Le client final est facturé, au choix, par l'opérateur ou par le partenaire ; l'opérateur facture tous les partenaires et leur reverse des commissions selon leurs accords (cf. §4.7).
19. ~~Marque blanche du générateur pour les partenaires ?~~ → Sans objet : seul l'opérateur utilise le générateur ; les partenaires passent par l'espace partenaire (cf. §4.28).
20. ~~Un partenaire voit-il les portails des clients de ses sous-partenaires ?~~ → Non : uniquement les KPI consolidés (cf. F-ESP-07).
21. ~~Application des options cochées par un partenaire ?~~ → Immédiate (cf. F-ESP-04).
22. ~~Qui fixe le coefficient d'un client final ?~~ → Le partenaire qui l'a apporté (cf. F-ESP-13).
23. ~~Marque de l'espace partenaire ?~~ → À la marque de chaque partenaire (cf. F-ESP-14).
24. ~~Qui fixe le coefficient d'un sous-partenaire ?~~ → Son partenaire parent, sur sa propre marge (cf. F-ESP-15, F-CONSO-24).
25. ~~Consommation nominative ou par équipe ?~~ → Réglage par portail : par équipe par défaut pour les collectivités, nominatif activable (cf. F-CONSO-34).

---

## 8. Glossaire

| Terme | Définition |
|---|---|
| Portail | Espace IA dédié à un client, sur son nom de domaine, ouvert côté utilisateur sur un chatbot multi-LLM |
| Générateur de portail | Outil de l'opérateur, en amont : crée les instances, gère leurs usages et fixe le niveau de délégation de chaque admin client |
| Partie admin | Outil d'administration d'un portail, utilisé par l'administrateur du client |
| Partie utilisateur | Outil utilisé au quotidien par les utilisateurs finaux du client, ouvert sur le chatbot |
| Politique de chat | Ensemble de règles (sources, modèles, processus, libertés) appliqué au chat d'un utilisateur, d'un profil ou d'une équipe |
| Chat principal | Chat ouvert par défaut à chaque utilisateur, réglé par sa politique de chat |
| Chat spécialisé | Chat dédié à un sujet (ex. Urbanisme), attribué à des utilisateurs, profils ou équipes, avec sa propre politique |
| Ensemble de sources | Groupe de documents et de sources connectées que les politiques de chat peuvent autoriser |
| Automatisation | Enchaînement d'étapes réalisées par l'IA, lancé par un horaire, un événement, un audio, un logiciel externe ou une demande dans le chat |
| Essai à blanc | Exécution d'essai d'une automatisation qui n'envoie rien et ne modifie rien |
| Délégation | Ensemble des possibilités que le générateur ouvre à la partie admin d'un portail |
| Forfait | Offre souscrite par un client pour son portail : outils inclus, niveau de délégation, usage inclus ou non ; **jamais facturé par utilisateur** |
| Prix direct | Mode de tarification où le vendeur saisit directement son prix de revente (ex. prix par million de tokens), au lieu d'un coefficient |
| Coefficient multiplicateur | Facteur fixé par un niveau (opérateur ou partenaire) pour chacun de ses partenaires ou clients ; les coefficients se cumulent le long de la chaîne et s'appliquent aux KPI d'usage (tokens, coûts) ; invisible pour les niveaux en dessous |
| Partenaire | Revendeur des portails (niveau 1, 2 ou 3), sous contrat avec l'opérateur ; n'a pas accès au générateur |
| Sous-partenaire | Revendeur rattaché à un partenaire ou à un autre sous-partenaire |
| Périmètre | Pour un partenaire : ses clients (détail) et ses sous-partenaires (KPI consolidés uniquement) |
| Espace partenaire | Outil des partenaires : demandes de portails, options, KPI, factures et commissions |
| Commission | Somme reversée par l'opérateur à un partenaire selon son accord et sa place dans la hiérarchie |
| Enveloppe d'usage | Quantité de tokens (ou montant en euros) incluse dans un forfait pour une période |
| Abonnement fournisseur | Contrat avec un fournisseur d'IA ou d'outil tiers utilisé par un portail |
| KPI | Indicateur clé : d'usage (utilisateurs actifs, adoption…) ou de coût (coût total, par utilisateur…) |
| Instance | Copie autonome et isolée du portail, dédiée à un seul client |
| Usage | Pour un portail : les outils qui y sont ouverts, et sa consommation (tokens, coût) |
| Token | Unité de mesure du texte traité par un LLM, base de sa facturation |
| Profil | Rôle attribué à un utilisateur (ex. Administrateur, Utilisateur), portant un ensemble de droits |
| Équipe | Groupe d'utilisateurs partageant des droits, des outils et éventuellement un budget |
| MCP | *Model Context Protocol* : standard ouvert qui permet à l'IA du portail d'utiliser des logiciels externes (lire ou écrire des données) |
| Fiche nominative | Fiche décrivant une organisation ou un contact (personne), donc contenant des données personnelles |
| Client | Entreprise ou collectivité locale pour laquelle un portail est généré |
| Opérateur | Personne qui exploite le générateur et crée les portails des clients |
| Multi-LLM | Capacité à utiliser plusieurs modèles d'IA, de fournisseurs différents, dans un même portail |
| Modèle (template) | Portail type servant de point de départ |
| Bloc | Élément de contenu réutilisable dans une page (texte, image, tuile…) |
| Tuile | Carte cliquable menant vers une page ou un service |
