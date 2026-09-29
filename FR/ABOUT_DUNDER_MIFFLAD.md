# 🎬 À propos de Dunder MifflAD

Ce document présente la couche narrative du lab : d'où vient le nom, pourquoi ce décor, et comment il habille l'annuaire de démonstration sans jamais toucher à l'infrastructure. 

La partie 2 sert de référence pour l'annuaire de démo : sites, organigramme, comptes, conventions de nommage, rôles de chacun et scénarios de panne.

## Sommaire

**Partie 1 · À propos de Dunder MifflAD**

1. [Pourquoi ce nom](#1-pourquoi-ce-nom)
2. [La suite de Dunder MiffLAN](#2-la-suite-de-dunder-mifflan)
3. [Une entrerprise comme on en a tous vu](#3-une-entreprise-comme-on-en-a-tous-vu)
4. [Le contraste : un bureau banal, une infrastructure d'identité sérieuse](#4-le-contraste--un-bureau-banal-une-infrastructure-didentité-sérieuse)
5. [La règle d'or : un thème riche, strictement cosmétique](#5-la-règle-dor--un-thème-riche-strictement-cosmétique)
6. [Pourquoi sabre.local](#6-pourquoi-sabrelocal)

**Partie 2 · L'annuaire de démo et le lore**

7. [Les sites : Scranton, Utica et le fantôme de Stamford](#7-les-sites--scranton-utica-et-le-fantôme-de-stamford)
8. [L'organigramme devient l'annuaire : OU et groupes](#8-lorganigramme-devient-lannuaire--ou-et-groupes)
9. [Les comptes de démo et les conventions de nommage](#9-les-comptes-de-démo-et-les-conventions-de-nommage)
10. [Le casting : qui joue quel rôle](#10-le-casting--qui-joue-quel-rôle)
11. [Ce que j'ai voulu prouver avec ce thème](#11-ce-que-jai-voulu-prouver-avec-ce-thème)

---

# Partie 1 · À propos de Dunder MifflAD

## 1. Pourquoi ce nom

**Dunder MifflAD** est une entreprise fictive inspirée de *The Office* (*Dunder Mifflin*), relue à travers le prisme de l'annuaire et de l'identité.

La retouche est minime et voulue : **Dunder Mifflin → Dunder Miffl*AD***.

L'entreprise reste le même environnement de bureau. Le suffixe *AD* (Active Directory) tourne simplement la caméra vers l'infrastructure d'identité qui décide qui est qui, et qui a le droit de faire quoi.

Je n'ai pas cherché à recréer le vrai réseau de la série. Je voulais une entreprise fictive assez familière pour ressembler à un vrai lieu de travail, et assez générique pour se projeter sur presque n'importe quelle organisation.

## 2. La suite de Dunder MiffLAN

Ce lab prolonge un premier projet, **[Dunder MiffLAN](https://github.com/Joupow/DunderMiffLAN-Network-Portfolio)**, consacré au réseau de la même entreprise fictive (VLAN, routage, pare-feu, VoIP, Wi-Fi sous Packet Tracer), qui m'a accompagné jusqu'à la certification CompTIA Network+. Ensemble, les deux dépôts dessinent le profil visé : administrateur systèmes et réseaux.

Le nom raconte cette filiation. Le suffixe change, l'entreprise reste : **MiffLAN (le réseau) → MifflAD (l'annuaire)**.

La progression suit celle d'une vraie infrastructure. On pose d'abord la plomberie qui fait circuler les paquets, puis on installe au-dessus l'annuaire qui décide qui se connecte, et à quoi.

Les deux dépôts restent indépendants (l'un sous Packet Tracer, l'autre sous Hyper-V). La continuité est narrative et pédagogique, sans dépendance technique entre les deux.

## 3. Une entreprise comme on en a tous vu

*The Office* fonctionne autant par ses personnages que par son **décor**. 

Une entreprise familière, sans domaine métier de niche à apprendre. Ventes, comptabilité, RH, réception, service client existent à peu près partout. Des employés, des départements, des ressources partagées, des postes, des serveurs, et un annuaire qui relie tout ce monde en répondant à une seule question : « qui es-tu, et qu'as-tu le droit de faire ? ».

Cette lisibilité donne à chaque brique technique un sens métier :

- 🧩 les départements deviennent des **OU** et des **groupes globaux métier** (`GG-Sales`, `GG-Accounting`...) ;
- 💻 les employés deviennent des **comptes utilisateurs** ;
- 🗄️ les ressources partagées (le dossier de paie, l'imprimante du couloir) deviennent des **groupes de ressource** (`DL-*`), reliés aux employés par le modèle **AGDLP** ;
- 🔐 le niveau de privilège de chacun devient un **tier** (T0 / T1 / T2) et un poste d'administration dédié (**PAW**) ;
- 🧑‍🔧 le prestataire de passage devient un **compte externe borné** (OU dédiée, expiration automatique, suffixe `.ext`) ;
- 🚨 un incident vécu par un utilisateur devient une **mission panne** qu'on diagnostique et qu'on répare.

## 4. Le contraste : un bureau banal, une infrastructure d'identité sérieuse

Là où *Dunder Mifflin* n'est qu'un bureau, *Dunder MifflAD* montre l'infrastructure d'identité qui vit dessous. Ce qui démarre comme un simple domaine Windows monte en puissance saison après saison :

- 🏗️ **Saison 1, Déploiement** : contrôleurs de domaine, DNS, DHCP, réplication, catalogue global, rôles FSMO, puis OU, comptes, AGDLP, GPO, LAPS, cycle de vie des comptes, deux sites routés, services de fichiers et d'impression ;
- 🛡️ **Saison 2, Sécurité et résilience** : modèle en tiers et PAW, audit, PKI à deux niveaux, sauvegarde et reprise (System State, Veeam, RODC), puis l'examen Break/Fix transversal ;
- 🚀 **Saison 3, Exploitation** : accès distant (NPS/RADIUS, VPN), identité hybride (Entra Connect) et administration Microsoft 365 du quotidien, bases Linux, supervision, inventaire et tickets (GLPI).

C'est tout l'intérêt du contraste : un bureau d'apparence banale peut reposer sur une infrastructure d'identité étonnamment rigoureuse. Dunder MifflAD, c'est l'entreprise vue depuis le siège de l'administrateur Active Directory.

## 5. La règle d'or : un thème riche, strictement cosmétique

La couche narrative est développée en profondeur (sites, casting, organigramme, storylines de panne). Elle obéit pourtant à une règle : **ne jamais franchir la couche technique**.

Le thème vit à trois endroits, et nulle part ailleurs :

1. les **comptes de démonstration** ;
2. leurs **données d'identité** (nom affiché, département, fonction) ;
3. la **documentation** (titres, README, scénarios de panne).

Il ne touche à rien de ce qui compte pour l'infrastructure :

| Couche | Statut | Valeur |
|---|---|---|
| Nom du projet | thème libre | *Dunder MifflAD* |
| Domaine AD | technique, figé | `sabre.local` |
| Noms de machines | techniques | `DC01`, `DC02`, `SRV-FILE`, `WIN11-A`, `RTR`, `PKI-CA`, `PAW-T0`... |
| Adressage (IP, DNS, sous-réseaux, vSwitch) | technique | défini par l'[IPAM](./IPAM.md), jamais par le thème |
| Noms d'OU | neutres et professionnels | `OU=Sales`, `OU=Accounting`, `OU=HR`... |
| Comptes de démo | thème autorisé | `jhalpert`, `mscott`, `amartin`... |
| Découpage en saisons | regroupement de lecture | n'altère jamais la numérotation des épisodes `N1` à `N11` |

Un lecteur qui ne connaît pas la série lit le dépôt sans perdre une seule information technique.

## 6. Pourquoi sabre.local

Le domaine porte le nom de **Sabre**, la société qui rachète Dunder Mifflin dans la série. Il joue ici le rôle de la couche d'identité corporate : le domaine est le socle qui possède l'annuaire, et les branches (Scranton, Utica) vivent à l'intérieur. Ce qui est au-dessus des branches porte le nom de la maison mère.

Anachronisme assumé : dans la série, Sabre n'arrive que tard, bien après l'ouverture de Scranton. Le nom de domaine est un choix de nommage, pas un événement daté. Aucune storyline ne met en scène le rachat.

---

# Partie 2 · L'annuaire de démo et le lore

> **Construit ou prévu.** Chaque élément ci-dessous est marqué ✅ (construit, fin de Saison 1) ou 🔜 (prévu, avec son épisode).

## 7. Les sites : Scranton, Utica et le fantôme de Stamford

| Site AD | Sous-réseau | Rôle technique | Pourquoi ce nom | État |
|---|---|---|---|---|
| **Scranton** | SITE1 | site principal : DC01 (tous les rôles FSMO), SRV-FILE, WIN11-A | la branche qui survit à tout, comme un DC01 jamais éteint | ✅ N3 |
| **Utica** | SITE2 | succursale : DC02 (futur RODC), WIN11-B | la branche distante et plus petite, à qui on ne confiera pas d'identifiants inscriptibles | ✅ N3 |
| **Stamford** | sous-réseau fictif, aucun réseau réel | site orphelin, sans DC | dans la série, Stamford ferme et fusionne dans Scranton | ✅ posé en N3 · 🔜 retiré en N7 |

- **Stamford, le plant et le payoff.** 

En N3, pendant la création des sites, un site `Stamford` est posé avec son objet sous-réseau et un lien de site dédié, sans aucun DC ni réseau réel. 

L'histoire : Stamford a fusionné dans Scranton, et personne n'a jamais nettoyé son objet Site. En N7, on le découvre et on le décommissionne proprement. Ce qui pourrait être le quotidien d'un annuaire hérité, plein de sites fantômes laissés par d'anciennes fusions.

Il s'agit d'**hygiène de topologie**, pas de *metadata cleanup* : ce dernier concerne le retrait des objets d'un **contrôleur** disparu, et il est traité à part en N7.

## 8. L'organigramme devient l'annuaire : OU et groupes

Chaque département de l'organigramme devient une OU (délégation et ciblage des GPO) et un groupe global métier (appartenance).

| Département      | OU                   | Groupe global       |
| ---------------- | -------------------- | ------------------- |
| Management       | `OU=Management`      | `GG-Management`     |
| Sales            | `OU=Sales`           | `GG-Sales`          |
| Accounting       | `OU=Accounting`      | `GG-Accounting`     |
| Reception        | `OU=Reception`       | `GG-Reception`      |
| Human Resources  | `OU=HR`              | `GG-HR`             |
| Customer Service | `OU=CustomerService` | `GG-CS`             |
| Warehouse        | `OU=Warehouse`       | `GG-Warehouse`      |
| Prestataires     | `OU=Externals`       | aucun groupe métier |

- **Les externes, hors organigramme.** 

Un prestataire n'est pas un employé : il ne figure pas dans l'organigramme, et son OU vit à part. C'est voulu, ça matérialise « ces gens ne sont pas des nôtres, accès borné ».

- **Les OU techniques, hors thème.** 

Deux OU rangent des ordinateurs, pas des personnes : `OU=Workstations` (postes clients, cible de LAPS, ✅ N2) et `OU=Servers` (serveurs membres, cible des GPO serveur, ✅ N4). Leurs noms restent neutres : le thème ne s'applique qu'aux objets d'identité.

**AGDLP, l'exemple de référence du lab.** Une comptable n'a jamais de droit en direct sur un dossier :

```
Angela Martin (compte)
  → GG-Accounting              (groupe global : appartenance métier)
    → DL-Share-Payroll-Modify  (groupe domaine local : ressource et niveau d'accès)
      → NTFS « Modifier » sur \\sabre.local\Partages\Compta
```

Construits en N4 : `DL-Share-Payroll-Modify` (la compta écrit), `DL-Share-Payroll-Read` (la direction lit) et `DL-Print-Compta` (l'imprimante, réservée à la compta et à la direction).

**Le Party Planning Committee, un groupe transverse.** 

Dans la série, le comité des fêtes pioche dans plusieurs départements. C'est l'illustration d'un groupe qui ne suit pas la structure d'OU, et la raison pour laquelle on sépare la structure de **délégation** (les OU) de la structure d'**appartenance** (les groupes). 🔜 construit le jour où une ressource transverse le justifie.

## 9. Les comptes de démo et les conventions de nommage

**Identifiant de connexion (`sAMAccountName`)** : première lettre du prénom, puis le nom, en minuscules et sans accent. Jim Halpert donne `jhalpert`. Réaliste pour un recruteur, reconnaissable pour un fan.

**Prestataires externes** : même règle, suffixée `.ext`. Bob Vance donne `bvance.ext`. L'OU seule ne se voit pas partout ; le suffixe, lui, apparaît dans chaque journal et chaque ACL.

**Suffixe UPN.** Le domaine AD reste `sabre.local`. Les comptes internes se connectent en `@sabre.local` jusqu'en N10, où le suffixe routable `@DunderMifflAD.com` est ajouté, vérifié, puis appliqué aux comptes internes avant la synchronisation vers le cloud. Domaine et suffixe UPN sont deux couches distinctes.

**Les externes restent en `@sabre.local`.** Leur exclusion du cloud ne repose pas sur le suffixe (un suffixe non vérifié est simplement réécrit par Entra Connect), mais sur le **filtrage d'OU** de la synchronisation : `OU=Externals` reste hors périmètre. Le suffixe routable est un privilège d'accès, pas un réglage par défaut.

| Nom affiché                                                              | `sAMAccountName`                                        | OU                               | État   |
| ------------------------------------------------------------------------ | ------------------------------------------------------- | -------------------------------- | ------ |
| Michael Scott, Ryan Howard                                               | `mscott`, `rhoward`                                     | Management                       | ✅ N2   |
| Karen Filippelli                                                         | `kfilippelli`                                           | Management                       | ✅ N4   |
| Jim Halpert, Dwight Schrute, Andy Bernard, Phyllis Vance, Stanley Hudson | `jhalpert`, `dschrute`, `abernard`, `pvance`, `shudson` | Sales                            | ✅ N2   |
| Pam Beesly                                                               | `pbeesly`                                               | Sales (arrivée depuis Reception) | ✅ N2   |
| Todd Packer                                                              | `tpacker`                                               | Sales, désactivé                 | ✅ N2   |
| Angela Martin, Kevin Malone, Oscar Martinez                              | `amartin`, `kmalone`, `omartinez`                       | Accounting                       | ✅ N2   |
| Erin Hannon                                                              | `ehannon`                                               | Reception                        | ✅ N2   |
| Toby Flenderson                                                          | `tflenderson`                                           | HR                               | ✅ N2   |
| Kelly Kapoor                                                             | `kkapoor`                                               | CustomerService                  | ✅ N2   |
| Darryl Philbin                                                           | `dphilbin`                                              | Warehouse                        | ✅ N2   |
| Bob Vance                                                                | `bvance.ext`                                            | Externals, déprovisionné         | ✅ N2   |
| David Wallace                                                            | `dwallace`                                              | compte d'urgence Tier 0          | 🔜 N5  |
| Holly Flax                                                               | `hflax`                                                 | HR (succession de Toby)          | 🔜 N10 |

## 10. Le casting : qui joue quel rôle

Le modèle en tiers répond à une question simple : « depuis quelle machine ce privilège est-il utilisé ? ». La série y répond toute seule. Les rôles ci-dessous sont un cadrage pédagogique : l'architecture de sécurité réelle se construit en N5.

| Personnage | Rôle dans l'annuaire | Tier | Pourquoi lui |
|---|---|---|---|
| **David Wallace** | administrateur d'urgence (*break-glass*) | T0 | la vraie autorité, calme et quasi absente : un compte qu'on n'utilise presque jamais et qu'on protège à l'extrême |
| **Dwight Schrute** | administrateur Tier 0 au quotidien, depuis `PAW-T0` | T0 | obsédé de sécurité et de plans de secours ; c'est lui qui saisira les rôles FSMO quand DC01 tombera (N7) |
| **Jim Halpert** | administrateur serveurs, depuis `PAW-T1` | T1 | compétent, peu de drame, fait le vrai travail |
| **Michael Scott** | directeur régional, compte ordinaire | T2 | se fait piéger, clique sur tout : l'argument vivant pour qu'un manager ne détienne jamais de droits d'administration |
| **Karen Filippelli** | directrice d'Utica, compte ordinaire | T2 | même règle que Michael, sur l'autre site : l'autorité hiérarchique ne donne aucun privilège AD |
| **Pam Beesly → Erin Hannon** | réception, délégation ciblée sur les comptes | T2 | la réception gère les arrivées ; la succession de Pam par Erin sert l'exercice de passation de délégation (N5) |
| **Angela Martin** | comptable senior, données de paie | T2 | le compte le plus sensible du domaine : cible de Protected Users et d'une stratégie de mot de passe renforcée (N5) |
| **Kevin Malone** | utilisateur négligent | T2 | l'argument pour LAPS et le moindre privilège ; grand pourvoyeur de tickets (N11) |
| **Todd Packer** | compte à désactiver | T2 | le départ à traiter proprement (Leaver) |
| **Bob Vance** | prestataire externe, accès temporaire | T2 maximum | autre société, mission bornée : OU dédiée, expiration automatique, jamais T0 ou T1 |
| **Toby Flenderson** | incarnation des GPO | | les RH, ce sont les règles que tout le monde subit ; une GPO qui ne s'applique pas, c'est « Michael ignore le règlement » |
| **Charles Miner** | incarnation d'une baseline trop stricte | | un durcissement trop serré qui casse un usage légitime, et l'équilibre à trouver entre sécurité et disponibilité (N5) |
| **Creed Bratton** | la menace jamais auditée | | personne ne sait ce qu'il fait depuis des années : l'argument pour l'audit (N5) et la supervision (N11) |

**Deux métaphores récurrentes.**

- **La PKI, ce sont les badges d'employé** (N6) : la CA racine hors ligne est la charte fondatrice qui reste au coffre, la CA émettrice délivre les badges du quotidien, la liste de révocation est la liste des badges désactivés. Et un certificat qui ne chaîne pas, c'est un *Dundie* : un trophée signé par une autorité qui n'est pas reconnue.
- **Les caméras de Dwight, c'est la supervision** (N11) : on ne protège que ce qu'on observe, et la sonde qui manque, c'est l'angle mort.

## 11. Ce que j'ai voulu prouver avec ce thème

Je voulais que la fiction serve de vraie couche de traçabilité, du besoin métier vers la décision technique :

- les OU portent le nom des départements réels de l'organigramme, pas de libellés génériques ;
- les règles d'accès (AGDLP, ACL NTFS) traduisent un besoin métier explicite, par exemple qui accède à la paie et qui n'y accède pas ;
- les incidents partent d'un impact métier (une session qui ne s'ouvre plus, une succursale qui ne reçoit plus le règlement) et remontent jusqu'à la cause technique.

Ce que j'ai retenu en montant ce lab : le thème ne remplace jamais l'exercice technique, il l'emballe. Et il ouvre des pistes, d'autres scénarios à explorer et de nouvelles briques à construire.

---

📎 Retour au [README](./README.md) · 📘 [Vue d'ensemble technique](./TECHNICAL_OVERVIEW.md)
