# IPAM · Home Lab Active Directory `sabre.local`

**📌 Statut :** figé pour la topologie cible (2 sites). Les lignes marquées *(différé)* ne sont opérationnelles qu'à partir de la phase indiquée.
**🌐 Domaine :** `sabre.local` (forêt unique, domaine unique)
**🖥️ Hyperviseur :** Hyper-V (hôte 32 Go)

---

## Sommaire

1. [Principes de conception](#1-principes-de-conception)
2. [Convention d'adressage (dernier octet)](#2-convention-dadressage-dernier-octet)
3. [Sous-réseaux et commutateurs virtuels](#3-sous-réseaux-et-commutateurs-virtuels)
4. [Plan d'adressage · SITE1 `10.10.1.0/24`](#4-plan-dadressage--site1-10101024)
5. [Plan d'adressage · SITE2 `10.10.2.0/24` (différé N3)](#5-plan-dadressage--site2-10102024-différé-n3)

---

## 1. Principes de conception

| Principe                                   | Règle                                                                            | Pourquoi                                                                                                                               |
| ------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **Topologie cible dès le départ**          | On adresse pour 2 sites même si SITE2 n'est allumé qu'en N3                      | Évite le renumérotage complet à N3                                                                                                     |
| **IP statique sur les serveurs d'infra**   | DC, serveurs membres, PKI, NPS, AADC, **backup, supervision, GLPI**              | Un service que d'autres trouvent par IP/DNS ne peut pas changer d'adresse au gré d'un bail                                             |
| **DHCP réservé aux clients**               | Postes Windows 11 uniquement                                                     | L'auth AD (Kerberos, localisation du DC via SRV) passe exclusivement par le DNS AD ; deux DC = la résolution survit à la perte de l'un |
| **DNS client = les DC uniquement**         | Jamais un DNS public, jamais le routeur ; **on pousse les deux DC** (option 006) | L'auth AD (Kerberos, localisation du DC via SRV) passe exclusivement par le DNS AD ; deux DC = la résolution survit à la perte de l'un |
| **DNS d'un DC = partenaire puis loopback** | DC01 → DC02 puis `127.0.0.1` ; DC02 → DC01 puis `127.0.0.1`                      | Évite le *DNS island* : un DC qui ne pointe que sur lui-même peut s'isoler de la réplication                                           |
| **Forwarders ≠ DNS primaire**              | Résolveur public (`9.9.9.9`, `1.1.1.1`) en *forwarder* sur le DC                 | Jamais le DNS du FAI en primaire sur un DC (règle du plan maître)                                                                      |

## 2. Convention d'adressage (dernier octet)

Appliquée identiquement sur les deux sous-réseaux.

| Plage         | Usage                                                            |
| ------------- | ---------------------------------------------------------------- |
| `.1`          | Passerelle (interface RTR du site)                               |
| `.10 - .19`   | Contrôleurs de domaine / cœur                                    |
| `.20 – .29`   | Serveurs membres (fichiers, services, backup, supervision, GLPI) |
| `.30 – .39`   | Infra de sécurité (PKI, NPS, Entra Connect)                      |
| `.40 – .49`   | PAW (postes d'administration privilégiée)                        |
| `.100 – .199` | Pool DHCP (clients)                                              |
| `.200 – .254` | Réserve / statiques ponctuelles                                  |

## 3. Sous-réseaux et commutateurs virtuels

| Site | Sous-réseau | Masque | vSwitch Hyper-V | Type |
|---|---|---|---|---|
| SITE1 | `10.10.1.0/24` | 255.255.255.0 | `SITE1` | **Privé** |
| SITE2 | `10.10.2.0/24` | 255.255.255.0 | `SITE2` | **Privé** *(différé N3)* |
| - (MàJ / N10) | selon FAI | - | `EXTERNE-TEMP` | **Externe**, activé ponctuellement puis rebasculé |

## 4. Plan d'adressage · SITE1 `10.10.1.0/24`

| Machine                                          | IP                   | Attribution              | DNS configuré                                                  | Passerelle                                                                                                                                                        | Allumée à partir de  |
| ------------------------------------------------ | -------------------- | ------------------------ | -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------- |
| RTR (int. SITE1)                                 | `10.10.1.1`          | Statique                 | -                                                              | -                                                                                                                                                                 | N3 *(différé)*       |
| **DC01** (RWDC, DNS, DHCP, GC, FSMO)             | `10.10.1.10`         | **Statique**             | `127.0.0.1` *(N1-N2)* ; puis `10.10.2.10`, `127.0.0.1` *(N3+)* | Aucune passerelle par défaut (**choix Tier 0** : le DC ne peut pas être routé hors sous-réseau) ; **route statique persistante** `10.10.2.0/24 → 10.10.1.1` (N3+) | **N1, toujours**    |
| **DC02** (transitoire N1-N2, voir SITE2)        | `10.10.1.11`         | **Statique**             | `10.10.1.10`, `127.0.0.1`                                      | *(aucune, réseau isolé)*                                                                                                                                         | **N1 (réplication)** |
| SRV-FILE (fichiers, DFS, FSRM, gMSA, impression) | `10.10.1.20`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N4                   |
| SRV-BACKUP (Veeam Community)                     | `10.10.1.21`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N7                   |
| SRV-MON (supervision)                            | `10.10.1.22`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N11                  |
| SRV-GLPI (ITSM/inventaire)                       | `10.10.1.23`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N11                  |
| PKI-CA (émettrice en ligne)                      | `10.10.1.30`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N6                   |
| PKI-ROOT (CA racine offline)                     | *aucune permanente*  | Temporaire si signature  | -                                                              | -                                                                                                                                                                 | N6 (signature seule) |
| SRV-NPS (NPS/RADIUS)                             | `10.10.1.32`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N9                   |
| SRV-AADC (Entra Connect)                         | `10.10.1.33`         | Statique + externe temp. | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N10                  |
| PAW-T0 (admin AD/PKI)                            | `10.10.1.40`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N5                   |
| PAW-T1 (admin serveurs)                          | `10.10.1.41`         | Statique                 | `10.10.1.10`, `10.10.2.10`                                     | `10.10.1.1`                                                                                                                                                       | N5                   |
| **Pool DHCP clients SITE1**                      | `10.10.1.100 – .199` | DHCP                     | poussé par DHCP                                                | poussé par DHCP                                                                                                                                                   | -                    |
| WIN11-A (client site 1)                          | via DHCP             | DHCP                     | poussé par DHCP                                                | poussé par DHCP                                                                                                                                                   | N2                   |

## 5. Plan d'adressage · SITE2 `10.10.2.0/24` *(différé N3)*

| Machine                               | IP                                           | Attribution           | DNS configuré                  | Passerelle          | Allumée à partir de       |
| ------------------------------------- | -------------------------------------------- | --------------------- | ------------------------------ | ------------------- | ------------------------- |
| RTR (int. SITE2)                      | `10.10.2.1`                                  | Statique              | -                              | -                   | N3                        |
| **DC02** (RWDC → RODC, DNS, GC)       | `10.10.2.10` *(N3+ ; `10.10.1.11` en N1-N2)* | **Statique**          | `10.10.1.10`, puis `127.0.0.1` | `10.10.2.1` *(N3+)* | N1 (sur SITE1 jusqu'à N3) |
| **Pool DHCP clients SITE2**           | `10.10.2.100 – .199`                         | DHCP (relais via RTR) | poussé par DHCP                | poussé par DHCP     | -                         |
| WIN11-B (client site 2, branche RODC) | via DHCP                                     | DHCP                  | poussé par DHCP                | poussé par DHCP     | N3                        |
