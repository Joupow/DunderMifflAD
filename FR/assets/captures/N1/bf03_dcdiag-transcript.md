# [BF-03] Incident #5 — Le `dcdiag` qui ment : transcript de diagnostic

> **Preuve texte** (transcripts de terminal réels). Les états *cassés* n'existent plus une fois
> l'incident résolu : ces sorties sont l'unique enregistrement de la panne. Un transcript est une
> preuve plus forte qu'une capture pour un diagnostic CLI (commandes exactes, recherchables,
> reproductibles). Voir le récit complet dans [WORKFLOW § Dépannage incident #5](../../WORKFLOW.md#dépannage-incidents-de-session).

Machine : **DC01** (`10.10.1.10`), console `Administrateur`. Domaine `sabre.local`.

---

## 1. État cassé — `dcdiag /test:dns` échoue (résumé)

```
Résumé des résultats des tests DNS :
                                     Auth Basc Forw Del  Dyn  RReg Ext
     ________________________________________________________________
     Domaine : sabre.local
        DC01                         PASS FAIL PASS PASS WARN FAIL n/a
     ......................... Le test DNS de sabre.local a échoué
```

Messages accusateurs (tous trompeurs) : « can't read network adapter information through WMI »,
« The A record for this DC was not found », « Pas de connectivité LDAP ».

---

## 2. Élimination des fausses pistes — sondes indépendantes

Chaque sonde **dément** une accusation de `dcdiag`.

**WMI lit bien la carte** (contredit « can't read adapter through WMI ») :

```
PS> Get-WmiObject Win32_NetworkAdapterConfiguration -Filter "IPEnabled=True" |
      Select-Object Description, IPAddress
Description                       IPAddress
-----------                       ---------
Microsoft Hyper-V Network Adapter {10.10.1.10, fe80::836b:6824:b998:8254}
```

**L'enregistrement A existe** (contredit « A record not found ») :

```
PS> Get-DnsServerResourceRecord -ZoneName "sabre.local" -RRType A -Name "dc01"
HostName  RecordType  RecordData
--------  ----------  ----------
dc01      A           10.10.1.10
```

**Le pare-feu est ouvert sur LDAP** (contredit « Pas de connectivité LDAP ») :

```
PS> Test-NetConnection 10.10.1.10 -Port 389
ComputerName     : 10.10.1.10
RemotePort       : 389
TcpTestSucceeded : True
```

**Le dépôt WMI est sain** (contredit une corruption WMI) :

```
PS> winmgmt /verifyrepository
L'espace de stockage WMI EST cohérent.
```

Aucune carte fantôme, délégation `_msdcs` valide, forwarders valides (via `dcdiag /v`).
→ Toutes les accusations tombent : **le DNS est sain**.

---

## 3. Cause racine n°1 — le DNS *client* de DC01 est vide

Le verbose `dcdiag /test:dns /v` donne le vrai code, noyé dans le test `Dyn` :

```
[Error details: 9852 (Type: Win32 - Description:
 Aucun serveur DNS n'est configuré pour le système local.)]
```

Confirmation directe — la carte n'a **aucun** serveur DNS client renseigné :

```
PS> Get-DnsClientServerAddress -InterfaceAlias "Ethernet" -AddressFamily IPv4
InterfaceAlias   InterfaceIndex  AddressFamily  ServerAddresses
--------------   --------------  -------------  ---------------
Ethernet         6               IPv4           {}
```

> **Le mensonge décodé :** DC01 résout via son *service* DNS local, mais son *client* DNS ne pointe
> sur rien. `dcdiag` lit la config *client* de la carte → « pas de DNS » → et l'erreur WMI
> `0x80041001` n'était qu'un symptôme (impossible d'énumérer/valider une carte sans DNS client).

---

## 4. Réparation n°1 + révélation de la cause n°2

```
PS> Set-DnsClientServerAddress -InterfaceAlias "Ethernet" -ServerAddresses 127.0.0.1
PS> Clear-DnsClientCache ; ipconfig /registerdns
PS> dcdiag /test:dns
```

`Connectivity` et `Basc` passent **PASS**. `dcdiag` révèle alors le vrai problème restant,
masqué jusque-là :

```
TEST: Records registration (RReg)
   Carte réseau [00000000] Microsoft Hyper-V Network Adapter :
      Avertissement :
      Enregistrement SRV manquant au niveau du serveur DNS 10.10.1.10 :
      _ldap._tcp.dc._msdcs.sabre.local

Résumé : DC01   PASS PASS PASS PASS PASS FAIL n/a
```

L'entrée SRV de **DC01** manquait (supprimée par l'injection 2 de la mission panne, jamais
réinscrite tant que le DNS client était vide) — seule celle de DC02 subsistait :

```
PS> Get-DnsServerResourceRecord -ZoneName "_msdcs.sabre.local" -RRType SRV -Name "_ldap._tcp.dc"
HostName        RecordData
--------        ----------
_ldap._tcp.dc   [0][100][389][dc02.sabre.local.]      <-- dc01 absent
```

---

## 5. Réparation n°2 — réinscription du SRV + convergence

```
PS> Restart-Service Netlogon
PS> Start-Sleep -Seconds 60          # laisser converger la zone _msdcs
PS> Clear-DnsClientCache
```

---

## 6. État final — tout vert, prouvé de deux façons

`dcdiag /test:dns` :
```
......................... Le test DNS de sabre.local a réussi
```

Vue client — le SRV renvoie de nouveau **les deux DC** (équilibrage rétabli) :
```
PS> Resolve-DnsName -Type SRV _ldap._tcp.dc._msdcs.sabre.local
NameTarget         Priority Weight Port
----------         -------- ------ ----
dc01.sabre.local   0        100    389
dc02.sabre.local   0        100    389
```

---

## Ce que cet incident prouve

Deux pannes empilées derrière un rapport trompeur. La méthode qui les a démasquées :
lire les **codes** du verbose (pas les messages), trouver le code qui **explique** les autres
(`9852` → `0x80041001` → « A not found »), et savoir qu'une panne réparée peut en **révéler**
une seconde. Le DNS n'a jamais été cassé ; l'outil de diagnostic, si, dans son interprétation.
