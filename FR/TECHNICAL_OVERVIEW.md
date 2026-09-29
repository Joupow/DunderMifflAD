# Dunder MifflAD · Vue d'ensemble technique

> Vue transversale du lab, mise à jour à la fin de chaque saison.
> **État actuel : fin de Saison 1 (N1 à N4).**

## <a id="sommaire"></a>Sommaire

1. [Objectif et état actuel](#objectif)
2. [Le parc virtuel](#parc)
   - 2.1 [Quinze VM, un rôle chacune](#vm-roles)
   - 2.2 [Capacité et placement : VM chaudes, VM froides](#placement)
3. [Les pannes, deux axes de lecture](#pannes)
   - 3.1 [Les origines](#origines)
   - 3.2 [Le cran de réversibilité](#reversibilite)
4. [Ce qui a été cassé, épisode par épisode](#casse)
5. [Ce qu'un relecteur technique challengera](#relecteur)
6. [Écarts entre le lab et la production](#ecarts)

---

## <a id="objectif"></a>1. Objectif et état actuel

Ce document s'adresse au lecteur technique qui veut juger le lab sans ouvrir chaque épisode. 

Il décrit comment l'infrastructure tient sur un seul hôte ([§2](#parc)), comment les pannes sont choisies et encadrées ([§3](#pannes)) et ce qui a réellement été cassé jusqu'ici ([§4](#casse)). Les deux dernières sections disent ce qu'un relecteur contestera et où le lab s'écarte de la production.

La méthode elle-même (les quatre temps d'une panne, la différence entre impact et symptôme) est présentée dans le [README](./README.md#methode). Ce document montre l'infrastructure qui la rend possible, et le bilan.

**Où en est le lab à la fin de la Saison 1 :**

| Domaine | État |
|---|---|
| Socle | forêt et domaine uniques, deux contrôleurs de domaine inscriptibles, DNS intégré à AD, DHCP |
| Réseau | deux sites AD sur deux sous-réseaux routés, relais DHCP entre les deux |
| Services | un serveur de fichiers membre : partages SMB/NTFS, DFS Namespace, DFS-R, FSRM, impression |
| Systèmes | Windows Server 2025 et Windows 11 Entreprise, en versions d'évaluation |
| VM construites | 6 sur 15 |
| Missions panne | 5 (plus une annexe), toutes au cran 1 de réversibilité |
| Incidents imprévus documentés | 5 |

---

## <a id="parc"></a>2. Le parc virtuel

### <a id="vm-roles"></a>2.1 Quinze VM, un rôle chacune

Quinze VM sont prévues, jamais plus de quatre ou cinq allumées en même temps. Les adresses sont définies dans l'[IPAM](./IPAM.md).

| VM | Rôle | Épisode | État | RAM |
|---|---|---|---|---|
| **DC01** | contrôleur de domaine principal (site Scranton) : DNS, DHCP, catalogue global, les 5 rôles FSMO, racine de temps | N1 | ✅ | 3 Go statique |
| **DC02** | second contrôleur de domaine (site Utica) : DNS, catalogue global ; deviendra RODC en N7 | N1 | ✅ | 2 Go statique |
| **WIN11-A** | poste client du site Scranton, poste utilisateur des missions panne | N1 | ✅ | 4 Go dynamique |
| **RTR** | routeur Windows (RRAS) entre les deux sous-réseaux, relais DHCP | N3 | ✅ | 1 Go dynamique |
| **WIN11-B** | poste client de la succursale d'Utica | N3 | ✅ | 4 Go dynamique |
| **SRV-FILE** | premier serveur membre (Tier 1) : fichiers, DFS, FSRM, impression | N4 | ✅ | 2 Go dynamique |
| **PAW-T0 / PAW-T1** | postes d'administration privilégiée, Tier 0 (annuaire, PKI) et Tier 1 (serveurs) | N5 | 🔜 | 4 Go dynamique |
| **PKI-ROOT** | autorité de certification racine, hors ligne | N6 | 🔜 | 2 Go dynamique |
| **PKI-CA** | autorité de certification émettrice, en ligne | N6 | 🔜 | 2 Go dynamique |
| **SRV-BACKUP** | serveur de sauvegarde (Veeam) | N7 | 🔜 | 2 Go dynamique |
| **SRV-NPS** | NPS / RADIUS pour l'accès distant | N9 | 🔜 | 2 Go dynamique |
| **SRV-AADC** | Entra Connect (synchronisation vers le cloud) | N10 | 🔜 | 2 Go dynamique |
| **SRV-MON** | supervision (Linux) | N11 | 🔜 | 2 Go dynamique |
| **SRV-GLPI** | inventaire et tickets (Linux, GLPI) | N11 | 🔜 | 2 Go dynamique |

Les contrôleurs de domaine gardent une mémoire **statique** : la mémoire dynamique peut reprendre de la RAM à AD DS sous charge. Clients et serveurs membres passent en dynamique pour rendre la RAM quand ils sont inactifs.

### <a id="placement"></a>2.2 Capacité et placement : VM chaudes, VM froides

L'hôte a 32 Go de RAM, un SSD (système et VM rapides) et un HDD (grand, lent). Placer un disque de VM est une décision de capacité, pas un détail d'installation.

**Le critère est la sensibilité au disque**, pas « allumée dans l'épisode ». Une VM interactive ou très sollicitée en lecture et écriture va sur le SSD. Un service au repos disque s'accommode du HDD.

| Couche | VM | Règle |
|---|---|---|
| Noyau chaud, fixe | DC01, DC02, WIN11-A, SRV-FILE | sur SSD, jamais déplacées |
| Services non interactifs | RTR, PKI, NPS, Entra Connect, sauvegarde, supervision, GLPI | sur HDD, allumées à la demande |
| VM pivots | WIN11-B, PAW-T0, PAW-T1 | déplacées vers le SSD le temps des épisodes qui les utilisent (WIN11-B y est depuis N3) |

Garde-fou : quatre à cinq disques virtuels au maximum sur le SSD, et toujours plus de 15 % d'espace libre.

Deux décisions ont été prises sur mesure, avec un test à l'appui :

- **WIN11-B remontée sur le SSD** (N3) : un client Windows 11 sur disque mécanique est inutilisable au démarrage.
- **RTR descendu sur le HDD** (N4), pour libérer la place de SRV-FILE. Le déplacement n'a été validé qu'après un test : latence inter-site inchangée (0 à 2 ms) et réplication saine depuis le HDD. Le routage ne dépend pas du disque.

---

## <a id="pannes"></a>3. Les pannes, deux axes de lecture

Une panne se lit sur deux axes indépendants.

Le premier dit d'où elle vient. Le second dit ce qu'il faudra pour en revenir, et c'est lui qui décide si on a le droit de l'injecter.

### <a id="origines"></a>3.1 Les origines

- **🎯 Origine 1 : La mission panne délibérée, par épisode.**

Juste après avoir construit un composant, on le casse volontairement. L'exercice suit quatre temps obligatoires : injection, impact sur un utilisateur réel, diagnostic jusqu'à la cause racine, réparation validée en rejouant l'opération qui échouait. L'impact se prouve par une opération d'utilisateur (une session, un bail, une stratégie qui n'arrive pas), jamais par la même sonde que le diagnostic. Quand une panne n'a aucun impact mesurable dans un lab isolé, c'est écrit tel quel, sans fabriquer de faux impact.

- **⚡Origine 2 : Les incidents imprévus, documentés comme les missions.**

Une grande partie de l'apprentissage vient des pannes qu'on n'a pas choisies : une commande qui n'existe plus sur la version de Windows, un service qui ne redémarre pas, un outil de diagnostic qui accuse la mauvaise cause. Chacun est diagnostiqué et documenté dans la section « Dépannage » du BREAKFIX de l'épisode (bilan en [section 4](#casse)).

- **🎲 Origine 3 : Le Break/Fix transversal, sans étiquette** (N8, 🔜).

L'examen final : des pannes combinées, injectées sans savoir lesquelles, sur n'importe quelle couche construite pendant les deux premières saisons. Une seule entrée, du type « un utilisateur ne peut plus se connecter », et c'est à moi de trouver la couche (conception en cours).

### <a id="reversibilite"></a>3.2 Le cran de réversibilité

Avant toute injection, une question passe devant les autres : **qu'est-ce qui répare ?** La réponse fixe le cran de réversibilité, donc qui peut lancer l'injection.

| Cran | Ce qui répare | Exemples | Qui peut l'injecter |
|---|---|---|---|
| **1 · Retour arrière direct** | le geste inverse, depuis un état initial connu | service arrêté, DNS ou DHCP modifié, port bloqué, compte verrouillé, ACL sauvegardée | moi ou un injecteur automatisé |
| **2 · Restauration externe** | un filet préparé avant la panne : System State, Veeam, corbeille AD | objets AD supprimés et répliqués, SYSVOL corrompu, retour arrière d'un DC | moi seul, après vérification du filet |
| **3 · Pas de retour** | rien : il faut reconstruire ou accepter le changement | perte de clé privée de CA, extension du schéma, certains changements de niveau fonctionnel, activation de la corbeille AD | personne dans la forêt du lab |

- **🟢 Cran 1 : réversible seulement si le retour est maîtrisé.**

Trois conditions doivent être réunies : l'état d'avant est connu, le canal d'annulation survit à la panne, et l'effet reste limité dans le temps et à la cible prévue.

Par exemple, remettre une carte en DHCP n'est pas un retour valide si elle était auparavant en IP statique. De même, casser le réseau impose un canal hors réseau, comme PowerShell Direct sous Hyper-V.

La cible compte aussi. Décaler l'heure d'un poste reste facilement réversible, toucher au PDC peut laisser des effets persistants dans le domaine. Une panne laissée trop longtemps peut également sortir du cran 1.

- **🟠 Cran 2 : le filet doit exister avant l'injection.**

Certaines pannes ne s'annulent pas par une commande inverse, il faut restaurer. C'est le cas d'une suppression AD déjà répliquée, d'un SYSVOL corrompu propagé par DFS-R ou de certaines pertes liées au retour arrière d'un DC.

Le VM-Generation ID protège un DC virtualisé contre l'USN rollback lors d'un retour à un point de contrôle, mais il ne recrée pas les modifications locales perdues avant réplication. Pour cela, il faut une vraie sauvegarde.

Une sauvegarde jamais restaurée reste une hypothèse : **le cran 2 n'est utilisé qu'à partir de N7**, quand les mécanismes de restauration ont été construits et testés.

- **🔴 Cran 3 : on ne répare pas, on décide.**

Certains changements n'ont pas de véritable retour arrière : perte de la clé privée d'une CA racine, extension du schéma, activation de la corbeille AD ou certains changements de niveau fonctionnel.

Ils ne sont donc jamais injectés comme des pannes. Ils sont seulement étudiés ou réalisés comme **décisions d'architecture documentées**, après validation dans le lab.

**Où en est le lab.**

Les missions de la Saison 1 relèvent toutes du cran 1. N7 sera le premier épisode à franchir volontairement le cran 2, parce que c'est lui qui pose les filets.

Si le Break/Fix de N8 est automatisé, son injecteur se limitera au cran 1 (conception de l'épisode en cours).

---

## <a id="casse"></a>4. Ce qui a été cassé, épisode par épisode

C'est le cœur du lab. Chaque épisode a son document **BREAKFIX** : les pannes injectées volontairement, et les incidents imprévus rencontrés en construisant, diagnostiqués jusqu'à la cause racine.

### N1 · Fondations ([BREAKFIX](./episodes/N1/BREAKFIX.md))

|              | Ce qui casse                                                                                                                                           | Ce qu'on observe et ce qu'on en retient                                                                                                              |
| ------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 🎯 Mission 1 | Trois défaillances DNS cumulées : DNS client du poste pointé vers une adresse morte, enregistrement de localisation des DC supprimé, redirecteur cassé | Le poste ne trouve plus aucun contrôleur, un changement de mot de passe échoue.                                                                      |
| 🎯 Mission 2 | Un contrôleur de domaine éteint                                                                                                                        | L'authentification bascule sur le second. Le DHCP, porté par un seul serveur, disparaît : un poste qui renouvelle son bail se retrouve sans adresse. |
| ⚡ Imprévu    | Une injection exécutée sur la mauvaise machine vide le DNS client du premier contrôleur                                                                | Deux pannes empilées, et un outil de diagnostic qui accuse une fausse cause. Démêlées par des sondes indépendantes et les journaux d'événements.     |

### N2 · Administration ([BREAKFIX](./episodes/N2/BREAKFIX.md))

| | Ce qui casse | Ce qu'on observe et ce qu'on en retient |
|---|---|---|
| 🎯 Mission | Le compte d'un prestataire reste actif après la fin de sa mission | Il ouvre une session et parcourt l'annuaire. Le radar d'expiration classique ne le voit pas, il faut chercher l'absence de date. Après réparation, la même connexion est refusée. |

### N3 · Réseau entreprise ([BREAKFIX](./episodes/N3/BREAKFIX.md))

| | Ce qui casse | Ce qu'on observe et ce qu'on en retient |
|---|---|---|
| 🎯 Mission | La succursale ne reçoit plus d'adresse | Un poste témoin sur l'autre site disculpe le serveur DHCP, l'état de l'étendue désigne la cause, sans démonter le routage à l'aveugle. |
| ⚡ Imprévu | Réplication inter-site en échec après la renumérotation d'un contrôleur | Plusieurs causes superposées (route de retour absente, routeur qui ne redémarre pas seul, enregistrements DNS périmés) et un pare-feu qui brouillait les tests. Chaque couche isolée, la cause au démarrage prouvée par reproduction. |

### N4 · Services de fichiers ([BREAKFIX](./episodes/N4/BREAKFIX.md))

| | Ce qui casse | Ce qu'on observe et ce qu'on en retient |
|---|---|---|
| 🎯 Mission | La réplication des stratégies de groupe (SYSVOL) s'arrête sur un contrôleur, celle de l'annuaire reste saine | La succursale garde l'ancien règlement et l'outil de réplication habituel n'affiche aucune erreur. Deux moteurs distincts, deux outils de diagnostic. |
| 📎 Annexe | Un conflit de réplication de fichiers provoqué | On voit où DFS-R range la version perdante, et pourquoi une réplication n'est pas une sauvegarde. |
| ⚡ Imprévu | Un objet recréé avant que la réplication AD ait convergé | Un objet de conflit apparaît dans l'annuaire. Un nom n'est pas un identifiant : on désambiguïse avant toute action destructive. |
| ⚡ Imprévu | Une imprimante qui refuse de se déployer | Cinq obstacles empilés, levés un par un. |
| ⚡ Imprévu | Une stratégie Point and Print trop permissive | Repérée à la relecture, corrigée, puis vérifiée sur le poste. |

---

## <a id="relecteur"></a>5. Ce qu'un relecteur technique challengera

Les questions ci-dessous portent sur le lab entier. Les plus ciblées (DHCP sur un seul serveur, mot de passe commun des comptes de démo, absence de redondance matérielle) trouvent leur réponse dans les [écarts avec la production](#ecarts).

- **« Pourquoi seulement de l'on-premise, et pas de l'hybride à fond ? »**

Parce que l'hybride repose sur un annuaire local sain. Entra Connect synchronise ce qui existe on-premise, erreurs comprises.

Le marché demande pourtant les deux : Microsoft 365 revient presque aussi souvent qu'Active Directory (31% contre 33%), et Entra ID apparaît dans 34% des CDI.

L'hybride arrive donc en N10, borné volontairement (le concept, une synchronisation de test, l'administration de M365 au quotidien, Intune observé sur un poste), une fois le socle on-premise construit et cassé. Un parcours cloud complet serait un autre projet.

- **« Pourquoi Hyper-V et pas VMware ? »**

Le marché penche vers VMware, surtout en CDI : 50% des offres lyonnaises le citent contre 23% pour Hyper-V qui citent également VMware. En alternance, l'écart disparaît (9% chacun).

J'ai choisi Hyper-V parce qu'il est livré avec Windows et tourne sur le poste disponible sans licence supplémentaire. D'autant plus que ce qu'il enseigne se transfère : commutateurs virtuels, mémoire dynamique, placement des disques, point de contrôle contre sauvegarde.

La console et l'écosystème VMware, les clusters et la haute disponibilité restent hors de portée d'un hôte unique. 

- **« Et Linux ? »**

Linux apparaît dans une offre CDI sur deux (54%) et dans une alternance sur trois (35%). Dans ces relevés, plus le poste suppose d'expérience, plus il revient. Pour une première embauche, c'est un plus qui devient vite une attente.

Avec des bases acquises sur TryHackMe, Linux arrive dans ce lab en N11 avec les serveurs de supervision et d'inventaire sous Debian : accès SSH par clé, services, journaux, paquets, pare-feu local, disques. C'est une administration de base, pas une maîtrise complète de Linux, et le lab ne prétend pas le contraire.

- **« Tu as utilisé une IA. Qu'est-ce qui vient de toi ? »**

Le tableau répond ligne par ligne, étape par étape : 

| Étape                     | Ce que j'ai fait                                                                                                                         | Ce que j'ai demandé à l'IA                                                                                                                             |
| ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Cadrage de l'IA           | écrire ses consignes (rôle de mentor, méthode pédagogique, règles de progression)                                                        | appliquer ce cadre à chaque étape                                                                                                                      |
| Conception du programme   | relever les offres, fixer les grandes lignes, valider ou refuser chaque retouche du plan                                                 | dépouiller les offres relevées, proposer un découpage et des retouches dans ce cadre, relier chaque étape aux attentes d'un poste                      |
| Sources de vérité         | tenir à jour le plan d'adressage et le plan maître, arbitrer quand deux sources se contredisent ou quand le matériel ne suit pas         | ne jamais inventer une adresse, un nom ou un chemin, s'arrêter et me demander en cas de contradiction                                                  |
| Apprentissage             | répondre à deux questions de vérification à la fin de chaque étape, et reprendre tant que la réponse restait incomplète                  | expliquer chaque concept (métaphores, démonstrations, sources), réexpliquer autrement en cas d'erreur, ne pas me laisser avancer avant d'avoir compris |
| Construction (PowerShell) | taper chaque commande à la main pour la mémoriser plutôt que la coller, lire les sorties, trancher entre les options                     | proposer chaque commande avec son rôle, ce qui casse si elle manque, son usage en production et son équivalent graphique                               |
| Pannes                    | injecter, observer l'impact, mener le diagnostic, affronter les imprévus, prononcer moi-même le verdict « réparé »                       | proposer des scénarios adaptés à chaque épisode, refuser de passer à la suite tant que le composant n'a pas été cassé et réparé                        |
| Documentation             | fixer la structure des documents, produire les preuves, relire, réécrire, refuser ce qui ne tenait pas, creuser quand le détail manquait | remplir la structure à partir de mes notes et de mes captures, sans rien inventer                                                                      |

---

## <a id="ecarts"></a>6. Écarts entre le lab et la production

N'ayant pas encore d'expérience en production, la colonne « Pratique de référence » ne décrit donc pas ce que j'ai vu en entreprise, mais ce que recommandent l'éditeur et les référentiels publics : 

- Documentation Microsoft Learn
- Guides de l'ANSSI (*Recommandations relatives à l'administration sécurisée des systèmes d'information*, ANSSI-PA-022, et sa déclinaison pour Active Directory, ANSSI-PA-099) 
- Et quelques bulletins de sécurité. 

Quand une ligne repose seulement sur un usage courant, sans document précis derrière, c'est écrit.

Seuls les écarts qui traversent le lab figurent ici, rangés par couche. Les limites propres à un épisode restent dans le registre de son README. 

### 🖥️ Hôte et virtualisation

| Écart                                | Dans le lab                                        | Pratique de référence                                                                                          | Source                                                                  |
| ------------------------------------ | -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| Haute disponibilité de l'hyperviseur | hôte unique, pas de cluster ni de stockage partagé | cluster d'hyperviseurs et stockage partagé                                                                     | Microsoft Learn, Hyper-V en cluster de basculement                      |
| Sécurité physique                    | inexistante, l'hôte est hors du périmètre simulé   | l'hyperviseur qui porte un DC est traité comme Tier 0 : accès physique restreint, TPM, chiffrement des disques | ANSSI-PA-099 ; Microsoft Learn, sécurisation des contrôleurs de domaine |

### 🌐 Réseau

| Écart | Dans le lab | Pratique de référence | Source |
|---|---|---|---|
| Routeur | RRAS Windows en routage seul, sans filtrage | pare-feu dédié, flux AD entre sites limités aux ports nécessaires | Microsoft Learn, ports requis pour Active Directory à travers un pare-feu |
| Relais DHCP | administrable seulement en ligne de commande héritée | relais porté par l'équipement réseau ou le pare-feu | usage courant (relais IP Helper côté Cisco, pratiqué dans Dunder MiffLAN) |
| Administration à distance | WinRM bloqué entre les deux sites | WinRM ouvert et restreint aux postes d'administration (N5) | ANSSI-PA-022, postes et flux d'administration dédiés |

### 🪪 Identité et services d'infrastructure

| Écart | Dans le lab | Pratique de référence | Source |
|---|---|---|---|
| Source de temps | le PDC suit sa propre horloge (réseau isolé) | PDC calé sur une source NTP externe fiable | Microsoft Learn, service de temps Windows |
| DHCP | un seul serveur, sur un DC | basculement DHCP entre deux serveurs, idéalement hors DC | Microsoft Learn, basculement DHCP ; le « hors DC » relève de l'usage courant |
| Mots de passe des comptes de démo | un mot de passe commun, sans changement forcé | mot de passe propre à chaque utilisateur, stratégies renforcées sur les comptes sensibles (N5) | NIST SP 800-63B ; Microsoft Learn, stratégies de mot de passe affinées et Protected Users |

### 🗂️ Fichiers et impression

| Écart | Dans le lab | Pratique de référence | Source |
|---|---|---|---|
| Serveur de fichiers | un seul serveur, dossier sur le disque système | volume de données séparé, second serveur si la disponibilité l'exige | usage courant |
| Point and Print | installation sans invite, depuis SRV-FILE seulement | même principe, avec un serveur d'impression durci et surveillé (N5) | Microsoft, KB5005652 (installation des pilotes Point and Print après PrintNightmare) |

---

⬆️ [Sommaire](#sommaire) · 📎 [README](./README.md) · 🎭 [À propos de Dunder MifflAD](./ABOUT_DUNDER_MIFFLAD.md)
