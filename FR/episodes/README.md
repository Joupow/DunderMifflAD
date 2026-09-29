# Guide des épisodes

Onze épisodes pour monter, sécuriser puis exploiter le domaine `sabre.local`. <br>
Voici le guide complet de ce qui été produit à ce jour.<br>
Etat actuel : toute la saison 1, de l'épisode N1 à N4.<br>
A venir : Saison 2

## 🎬 Saison 1 : Déploiement ✅

*Scranton ouvre ses portes, puis Utica. Le domaine prend forme, les utilisateurs arrivent, et les deux sites finissent par partager leurs dossiers.*

| N1                     | 🏗️ Les fondations                                                                                                                                                                                                                                                                                                                                                                                |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Storyline**          | Scranton ouvre. Avant le premier employé, il faut que les machines sachent se trouver, que l'annuaire sache qui est qui, et qu'une horloge fasse foi pour tout le monde. Rien de spectaculaire à l'écran, et pourtant tout le reste du lab repose sur ces fondations.                                                                                                                             |
| **Concepts**           | AD DS · forêt et domaine · DNS intégré · DHCP · réplication · catalogue global · FSMO · hiérarchie de temps                                                                                                                                                                                                                                                                                       |
| **Mission panne**      | **DNS / SRV.** Trois défaillances cumulées : DNS client faussé, enregistrement SRV supprimé, redirecteur cassé. Le poste ne localise plus aucun contrôleur, un changement de mot de passe échoue. Diagnostic par `Resolve-DnsName`, `dcdiag /test:dns` et `nltest /dsgetdc`. Une seconde mission éteint DC01 : l'authentification bascule sur DC02, mais le DHCP, porté par DC01 seul, disparaît. |
| **Incidents imprévus** | `dcdiag` accuse le DNS et WMI alors que la vraie cause est un DNS client vide sur DC01, lui-même dû à une injection lancée sur la mauvaise machine et annulée par un retour arrière valable seulement en DHCP. Mauvais modèle Secure Boot à la création d'un DC.                                                                                                                                  |

[Ouvrir N1 →](./N1/)

| N2                     | 👥 L'administration                                                                                                                                                                                                                                                                                |
| ---------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Storyline**          | Dunder Mifflin recrute. Les nouveaux employés reçoivent leurs comptes et leurs droits par département. Bob Vance (Vance Refrigeration) rejoint la boîte pour un projet commun, avec un accès borné dans le temps.                                                                                  |
| **Concepts**           | OU · comptes créés par PowerShell et CSV · AGDLP · GPO · Windows LAPS · cycle de vie JML · compte prestataire                                                                                                                                                                                      |
| **Mission panne**      | **Le presta oublié.** Le compte externe `bvance.ext` reste actif après la fin de sa mission et parcourt l'annuaire. Le radar d'expiration classique ne le voit pas : il faut chercher l'absence de date. Détection par `Search-ADAccount`, désactivation, puis expiration imposée dès la création. |
| **Incidents imprévus** | LAPS ne sauvegarde rien au premier test : la stratégie visait le cloud au lieu de l'annuaire, le journal LAPS tranche là où l'intuition accusait le schéma.                                                                                                                                        |

[Ouvrir N2 →](./N2/)

| N3                     | 🧭 Réseau entreprise                                                                                                                                                                                                                                                                                                                                                                                               |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Storyline**          | Utica ouvre. La succursale distante rejoint Scranton avec son propre sous-réseau, sa réplication d'annuaire et ses postes adressés à travers le routeur                                                                                                                                                                                                                                                            |
| **Concepts**           | routeur RRAS · deux sous-réseaux · relais DHCP · Sites et services · réplication inter-site · DNS inter-site                                                                                                                                                                                                                                                                                                       |
| **Mission panne**      | **Client d'Utica affamé.** L'étendue DHCP de SITE2 est désactivée sur DC01. WIN11-B n'obtient plus d'adresse et l'ouverture de session échoue, pendant que WIN11-A sert de témoin sain sur l'autre site. Le témoin disculpe le serveur, l'état de l'étendue désigne la cause.                                                                                                                                      |
| **Incidents imprévus** | Réplication inter-site en échec (8524) après la renumérotation de DC02, plusieurs causes superposées : route de retour absente sur DC01, RRAS qui ne redémarre pas seul, enregistrements DNS de DC02 restés sur l'ancienne adresse. Un pare-feu en profil Public brouillait les tests vers le routeur. Cause au démarrage prouvée par reproduction. Le site Stamford est posé exprès ici, à décommissionner en N7. |

[Ouvrir N3 →](./N3/)

| N4                     | 🗂️ Services de fichiers                                                                                                                                                                                                                                                                                            |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Storyline**          | Scranton et Utica travaillent enfin sur les mêmes dossiers, atteints par un chemin unique quel que soit le serveur, et l'imprimante du couloir devient accessible aux services qui en ont besoin                                                                                                                    |
| **Concepts**           | SMB et NTFS · AGDLP appliqué à un partage · DFS Namespace · DFS-R · FSRM · serveur d'impression déployé par GPO                                                                                                                                                                                                     |
| **Mission panne**      | **SYSVOL / DFS-R.** Le service DFS Replication est arrêté sur DC02. Karen, sur WIN11-B, reçoit encore l'ancien avis légal à l'ouverture de session alors que `repadmin /replsummary` n'affiche aucune erreur : SYSVOL et la base AD se répliquent par deux moteurs distincts, avec deux outils de diagnostic.       |
| **Incidents imprévus** | Objet de conflit AD `CNF:` né d'une recréation de groupe DFS-R avant la convergence de la réplication. Imprimante bloquée par cinq obstacles empilés (pilote, spouleur, droits, ciblage GPO, Point and Print). Stratégie Point and Print trouvée trop permissive à la relecture, corrigée et vérifiée sur le poste. |

[Ouvrir N4 →](./N4/)

---

[← Retour au dépôt](../README.md)
