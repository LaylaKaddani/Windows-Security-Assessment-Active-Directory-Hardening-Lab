# Windows Security Assessment, Active Directory & Hardening Lab

## Présentation

Ce projet consiste en la construction, la configuration, la sécurisation et la supervision d'un environnement Windows Server basé sur Active Directory.

L'objectif est de reproduire dans un environnement de laboratoire contrôlé plusieurs activités rencontrées en cybersécurité des systèmes Windows : administration sécurisée, IAM, gestion des privilèges, hardening, sécurité réseau, analyse des journaux, détection et automatisation PowerShell.

La démarche suivie est :

**Build → Configure → Secure → Harden → Validate → Monitor → Detect → Automate**

Ce Lab met particulièrement l'accent sur les compétences directement pertinentes pour des missions de cybersécurité junior, notamment :

- Windows Server Security
- Active Directory Security
- IAM et gestion des privilèges
- Windows Server Hardening
- Network Security
- Security Monitoring
- Log Analysis
- Detection Engineering
- PowerShell Automation

---

## 1. Lab Environment

### Domaine Active Directory

```text
Domain: cyberlab.local
Domain Controller: DC01
Client: CLIENT01
```

### Architecture

```text
                         cyberlab.local
                                |
                         +--------------+
                         |     DC01     |
                         | Windows      |
                         | Server      |
                         | AD DS / DNS |
                         +--------------+
                                |
                         +--------------+
                         |   CLIENT01   |
                         |   Windows    |
                         |    Client   |
                         +--------------+
```

### Technologies utilisées

- Windows Server
- Active Directory Domain Services (AD DS)
- Active Directory
- DNS
- Windows Defender Firewall
- SMB
- WinRM
- Windows Event Log
- PowerShell
- Group Policy / Windows security configuration

---

## 2. Active Directory Configuration

Une partie importante du Lab a consisté à construire et configurer l'environnement Active Directory avant d'effectuer les contrôles de sécurité.

Les travaux réalisés comprennent :

- configuration du domaine Active Directory ;
- création et organisation des comptes utilisateurs ;
- création et utilisation de groupes de sécurité ;
- organisation des objets dans Active Directory ;
- mise en place d'une séparation entre utilisateurs standards et comptes disposant de privilèges ;
- vérification des appartenances aux groupes ;
- contrôle des privilèges effectivement accordés aux utilisateurs.

### Exemple de gestion des privilèges

Le compte `Thomas.Martin` appartient au groupe :

```text
GG_IT_Admins
```

La vérification PowerShell a permis de confirmer :

```text
GG_IT_Admins
    └── Thomas Martin
```

et les appartenances du compte :

```text
Thomas.Martin
    ├── Domain Users
    ├── GG_IT
    └── GG_IT_Admins
```

Le compte `Nicolas.Moreau` possède quant à lui :

```text
Nicolas.Moreau
    ├── Domain Users
    └── GG_Direction
```

Cette étape a permis de vérifier concrètement les relations entre utilisateurs, groupes et privilèges dans Active Directory.

### Commandes utilisées

```powershell
Get-ADGroupMember "GG_IT_Admins" |
Select-Object Name, SamAccountName
```

```powershell
Get-ADPrincipalGroupMembership "Thomas.Martin" |
Select-Object Name
```

```powershell
Get-ADPrincipalGroupMembership "Nicolas.Moreau" |
Select-Object Name
```

### Capture

![Active Directory Groups](screenshots/01_ActiveDirectory_Groups.png)

Insérer ici une capture montrant les groupes et les appartenances vérifiées avec PowerShell.

---

## 3. IAM and Privileged Access Review

Le Lab a ensuite permis de travailler sur les principes fondamentaux de l'IAM :

- identification des utilisateurs ;
- appartenance aux groupes ;
- gestion des privilèges ;
- distinction entre utilisateurs standards et utilisateurs privilégiés ;
- vérification des accès effectifs ;
- contrôle des groupes à privilèges.

L'objectif n'était pas uniquement d'observer la configuration : les comptes et groupes ont été configurés dans le Lab, puis leur niveau de privilège a été vérifié.

Cette démarche permet de reproduire une partie du travail réalisé dans des activités de :

- IAM
- Access Management
- Identity Security
- Active Directory Security
- Privileged Access Management

---

## 4. Windows Server Hardening

Une phase importante du projet a été consacrée au durcissement du serveur.

Le hardening a été réalisé avant les phases de validation et de monitoring afin de pouvoir vérifier ensuite que les mesures de sécurité étaient effectivement appliquées.

Les contrôles réalisés portent notamment sur :

- services Windows ;
- Windows Defender Firewall ;
- SMB ;
- SMB signing ;
- protocoles réseau ;
- services d'administration distante ;
- Print Spooler ;
- règles de pare-feu ;
- surfaces d'exposition réseau.

L'objectif était d'appliquer des mesures de sécurité pertinentes pour un serveur Windows tout en conservant les fonctionnalités nécessaires au fonctionnement d'Active Directory.

---

## 5. SMB Security

Le protocole SMB a été étudié et sécurisé dans le cadre du hardening.

### Configuration vérifiée

```powershell
Get-SmbServerConfiguration |
Select-Object EnableSMB1Protocol,
              EnableSMB2Protocol,
              EnableSecuritySignature,
              RequireSecuritySignature
```

Résultat obtenu :

```text
EnableSMB1Protocol        False
EnableSMB2Protocol        True
EnableSecuritySignature   True
RequireSecuritySignature  True
```

Cette configuration montre notamment :

- SMBv1 désactivé ;
- SMBv2 activé ;
- SMB signing activé ;
- SMB signing requis.

Le SMB signing permet d'améliorer l'intégrité des communications SMB en empêchant notamment qu'un attaquant puisse modifier silencieusement certains échanges.

### Capture

![SMB Security Configuration](screenshots/02_SMB_Security_Configuration.png)

---

## 6. Windows Defender Firewall

Les profils Windows Defender Firewall ont été vérifiés :

```powershell
Get-NetFirewallProfile |
Select-Object Name,
              Enabled,
              DefaultInboundAction,
              DefaultOutboundAction
```

Les trois profils suivants étaient actifs :

```text
Domain
Private
Public
```

Les règles entrantes autorisées ont ensuite été étudiées afin d'identifier les services accessibles et les surfaces réseau exposées.

### Analyse des règles

Une attention particulière a été portée aux règles concernant :

- SMB / File and Printer Sharing ;
- WMI ;
- RPC ;
- Active Directory ;
- DNS ;
- Kerberos ;
- WinRM ;
- DFS ;
- services de gestion distante.

Cette analyse permet de relier la configuration du firewall aux rôles réellement assurés par le serveur.

### Capture

![Windows Defender Firewall](screenshots/03_Windows_Defender_Firewall.png)

---

## 7. Windows Services Review

Les services Windows en fonctionnement ont été recensés afin d'identifier les composants actifs sur le serveur.

Commande utilisée :

```powershell
Get-Service |
Where-Object {
    $_.Status -eq "Running"
} |
Sort-Object DisplayName |
Select-Object DisplayName, Name, StartType
```

Cette étape a notamment permis d'identifier les composants liés à :

- Active Directory Domain Services ;
- DNS ;
- Kerberos ;
- Netlogon ;
- RPC ;
- SMB ;
- Windows Defender ;
- Windows Event Log ;
- WinRM ;
- Print Spooler ;
- DFS ;
- WMI.

Le principe appliqué est de comparer les services actifs avec les fonctions réellement nécessaires au serveur, afin de réduire autant que possible la surface d'attaque sans casser les dépendances d'Active Directory.

### Capture

![Windows Services Review](screenshots/04_Windows_Services_Review.png)

---

## 8. Print Spooler Security Review

Le service Print Spooler a été vérifié :

```powershell
Get-Service Spooler |
Select-Object Name, Status, StartType
```

Résultat :

```text
Name      Status   StartType
Spooler   Running  Automatic
```

Les imprimantes présentes sur le serveur ont également été contrôlées :

```powershell
Get-Printer |
Select-Object Name,
              Shared,
              Published
```

Résultat :

```text
Microsoft XPS Document Writer
Microsoft Print to PDF
```

Aucune de ces imprimantes n'était partagée ou publiée.

Cette vérification permet de distinguer la présence du service de l'existence effective de ressources d'impression partagées.

### Capture

![Print Spooler Review](screenshots/05_Print_Spooler_Review.png)

---

## 9. WinRM Security Review

WinRM a également été étudié car il constitue un mécanisme important d'administration distante Windows.

Le service a été vérifié :

```powershell
Get-Service WinRM |
Select-Object Name, Status, StartType
```

Résultat :

```text
Name   Status   StartType
WinRM  Running  Automatic
```

Le serveur WinRM écoute sur le port :

```text
5985/TCP
```

La présence du listener a été vérifiée avec :

```powershell
winrm enumerate winrm/config/listener
```

Configuration observée :

```text
Transport = HTTP
Port      = 5985
Enabled   = true
```

Le port a également été vérifié au niveau réseau :

```powershell
Get-NetTCPConnection -LocalPort 5985 -State Listen |
Select-Object LocalAddress, LocalPort, State, OwningProcess
```

Résultat :

```text
LocalAddress  LocalPort  State   OwningProcess
::            5985       Listen  4
```

Le processus associé était :

```text
System
```

### Firewall WinRM

Les règles associées ont été contrôlées :

```powershell
Get-NetFirewallRule -DisplayName "*Windows Remote Management (HTTP-In)*" |
Select-Object DisplayName, Enabled, Profile, Direction, Action
```

Le port associé a été confirmé :

```text
TCP 5985
```

Les adresses distantes autorisées par les règles ont également été vérifiées.

Cette phase a permis d'identifier précisément :

- le service ;
- le port ;
- le listener ;
- le processus ;
- les règles Firewall ;
- les profils concernés ;
- la surface d'administration distante exposée.

### Capture

![WinRM Security Review](screenshots/06_WinRM_Security_Review.png)

---

## 10. Security Event Logging

Après la configuration et le hardening, le Lab a été étendu avec une partie dédiée au monitoring et à l'analyse des événements de sécurité.

Un répertoire dédié a été créé :

```powershell
New-Item -ItemType Directory `
-Path "C:\CyberLab\SecurityLogs" `
-Force
```

Les événements Windows Security correspondant à l'Event ID `4625` ont ensuite été collectés.

L'Event ID 4625 correspond à un échec d'ouverture de session.

### Export initial

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 100 |
Export-Csv "C:\CyberLab\SecurityLogs\FailedLogons.csv" `
-NoTypeInformation `
-Encoding UTF8
```

Le fichier généré a été vérifié :

```text
C:\CyberLab\SecurityLogs\FailedLogons.csv
```

### Capture

![Security Event ID 4625](screenshots/07_Security_Events_4625.png)

---

## 11. Security Event Analysis with PowerShell

Une analyse plus structurée des événements `4625` a ensuite été développée.

Les informations extraites comprennent notamment :

- date et heure ;
- utilisateur ;
- domaine ;
- Logon Type ;
- adresse IP source ;
- workstation ;
- status ;
- substatus ;
- processus.

Le parsing XML de l'événement permet d'extraire les champs présents dans `EventData`.

Le rapport a été exporté vers :

```text
C:\CyberLab\SecurityLogs\FailedLogons_Analysis.csv
```

### Logon Type 3

Le `LogonType = 3` correspond à une ouverture de session de type **Network**.

Dans le contexte du Lab, cela permet notamment d'identifier des tentatives d'authentification provenant d'une autre machine ou d'un accès réseau à une ressource Windows.

Cela est particulièrement intéressant pour l'analyse de :

- SMB ;
- accès réseau ;
- authentification Windows ;
- mouvements latéraux ;
- comportements suspects sur le réseau.

### Exemple observé

```text
SourceIP       LogonType
192.168.100.20 3
```

Une autre information observée concernait des tentatives locales sur `DC01` avec :

```text
127.0.0.1
```

Ces événements ont été distingués des authentifications réseau afin de ne pas mélanger les deux types de comportements.

### Capture

![Failed Logons Analysis](screenshots/08_Failed_Logons_Analysis.png)

---

## 12. Detection Logic

Une première logique de détection a été développée avec PowerShell.

L'objectif était d'identifier plusieurs échecs d'authentification réseau provenant de la même adresse IP.

La logique appliquée :

```text
Event ID = 4625
        |
        v
LogonType = 3
        |
        v
SourceIP != 127.0.0.1
        |
        v
Group by SourceIP
        |
        v
Count >= 2
        |
        v
Security Alert
```

Le script a généré l'alerte suivante :

```text
SourceIP       FailedAttempts   Alert
192.168.100.20 2                Multiple network authentication failures
```

Cette logique constitue une première approche de détection permettant de transformer des événements bruts en information exploitable par un analyste.

---

## 13. Automated Security Alert

Les alertes ont été exportées automatiquement :

```powershell
$Alerts |
Export-Csv "C:\CyberLab\SecurityLogs\Alerts.csv" `
-NoTypeInformation `
-Encoding UTF8
```

Le fichier obtenu :

```text
C:\CyberLab\SecurityLogs\Alerts.csv
```

Contenu :

```text
SourceIP,FailedAttempts,Alert
192.168.100.20,2,Multiple network authentication failures
```

Cette étape démontre le passage :

**Windows Event Logs → PowerShell Parsing → Filtering → Aggregation → Detection → Alert**

### Capture

![PowerShell Security Alert](screenshots/09_PowerShell_Security_Alert.png)

---

## 14. Security Monitoring Workflow

Le workflow final du Lab peut être résumé ainsi :

```text
Windows Security Events
          |
          v
      Event ID 4625
          |
          v
      PowerShell
          |
          v
      XML Parsing
          |
          v
   Structured CSV Data
          |
          v
 Filtering / Grouping
          |
          v
 Detection Logic
          |
          v
     Security Alert
          |
          v
       Alerts.csv
```

Cette partie permet de mettre en pratique plusieurs concepts utilisés dans les environnements SOC :

- event collection ;
- log parsing ;
- event filtering ;
- authentication monitoring ;
- source IP analysis ;
- detection logic ;
- alert generation ;
- basic detection engineering ;
- PowerShell automation.

---

## 15. Skills Demonstrated

### Windows & Active Directory

- Windows Server administration
- Active Directory Domain Services
- Active Directory users and groups
- Group membership management
- Privileged group analysis
- IAM fundamentals
- Windows authentication

### Security Engineering

- Windows Server Hardening
- SMB security
- SMB signing
- Windows Defender Firewall
- Network exposure analysis
- Remote administration security
- Service security review

### SOC & Detection

- Windows Security Event Logs
- Event ID 4625 analysis
- Logon Type analysis
- Network authentication monitoring
- Source IP analysis
- Detection logic
- Alert generation

### Automation

- PowerShell
- Windows Event Log querying
- XML event parsing
- CSV reporting
- Automated filtering and aggregation
- Security alert generation

---

## 16. Repository Structure

```text
Windows-Server-AD-Security-Lab/
│
├── README.md
│
├── screenshots/
│   ├── 01_ActiveDirectory_Groups.png
│   ├── 02_SMB_Security_Configuration.png
│   ├── 03_Windows_Defender_Firewall.png
│   ├── 04_Windows_Services_Review.png
│   ├── 05_Print_Spooler_Review.png
│   ├── 06_WinRM_Security_Review.png
│   ├── 07_Security_Events_4625.png
│   ├── 08_Failed_Logons_Analysis.png
│   └── 09_PowerShell_Security_Alert.png
│
└── SecurityLogs/
    ├── FailedLogons.csv
    ├── FailedLogons_Analysis.csv
    └── Alerts.csv
```

---

## 17. Project Outcome

Ce Lab montre une démarche complète allant de la construction d'un environnement Windows à la mise en place de contrôles de sécurité et d'une première capacité de détection.

Le projet combine :

```text
Infrastructure
      ↓
Active Directory
      ↓
IAM / Privileges
      ↓
Hardening
      ↓
Network Security
      ↓
Security Logging
      ↓
PowerShell Analysis
      ↓
Detection
      ↓
Alerting
```

L'intérêt principal du projet est de montrer une approche pratique de la cybersécurité : les composants ont été configurés et manipulés dans le Lab, puis vérifiés et analysés afin de comprendre leur niveau de sécurité et leur comportement.

---

## 18. Relevance to Cybersecurity Roles

Les compétences développées dans ce projet sont directement transposables à plusieurs missions junior en cybersécurité :

- Security Analyst
- Windows Security
- Active Directory Security
- IAM Analyst
- Vulnerability Management
- Security Operations
- Security Engineering
- Detection Engineering
- SOC Analyst
- Cybersecurity Consulting

