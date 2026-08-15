# Windows-Security-Assessment-Active-Directory-Hardening-Lab

# Windows Security Assessment & Active Directory Hardening Lab

Laboratoire pratique consacré à l’audit et à la sécurisation d’un environnement Windows Server avec Active Directory.

Le projet couvre plusieurs dimensions de la cybersécurité : analyse de la configuration Windows, sécurité Active Directory, gestion des privilèges, séparation des responsabilités, Security Hardening, Security Logging, détection d’événements et automatisation PowerShell.

L’objectif est de reproduire une démarche proche de celle utilisée lors d’une mission de Security Assessment : identifier les surfaces d’exposition, analyser les configurations, appliquer des mesures de durcissement, vérifier les résultats et mettre en place des mécanismes élémentaires de détection.

> Ce laboratoire est un environnement personnel de formation. Il est indépendant de l’infrastructure réelle de l’entreprise dans laquelle j’effectue mon stage.

---

## 1. Objectifs du projet

Les principaux objectifs du laboratoire sont :

* comprendre et sécuriser un environnement Windows Server ;
* administrer et auditer un domaine Active Directory ;
* analyser les utilisateurs, groupes et privilèges ;
* identifier les risques liés aux comptes privilégiés ;
* appliquer le principe de Least Privilege ;
* mettre en œuvre une démarche de séparation des responsabilités ;
* analyser les services et surfaces d’administration exposés ;
* auditer Windows Defender Firewall ;
* analyser et renforcer la configuration SMB ;
* analyser WinRM et la surface d’administration distante ;
* exploiter les Security Logs Windows ;
* analyser les échecs d’authentification ;
* mettre en place une détection simple d’événements suspects ;
* automatiser certaines tâches d’analyse avec PowerShell ;
* produire des constats et recommandations de sécurité.

---

## 2. Architecture du laboratoire

Le laboratoire repose sur un environnement Windows Server utilisé comme Domain Controller.

### Composants principaux

```text
                    CyberLab
                       |
                       |
              +------------------+
              |       DC01       |
              | Windows Server   |
              +------------------+
                       |
              Active Directory
                       |
          +------------+------------+
          |            |            |
       Users         Groups       Policies
          |            |            |
          +------------+------------+
                       |
              Security Assessment
                       |
       +---------------+---------------+
       |               |               |
    Firewall          SMB            WinRM
       |               |               |
       +---------------+---------------+
                       |
                Security Logs
                       |
                 PowerShell
                       |
             Detection / Alerts
```

L’environnement Active Directory utilise le domaine :

```text
cyberlab.local
```

Le serveur principal du laboratoire est identifié comme :

```text
DC01
```

---

## 3. Environnement technique

| Technologie                      | Utilisation                                     |
| -------------------------------- | ----------------------------------------------- |
| Windows Server                   | Infrastructure serveur du laboratoire           |
| Active Directory Domain Services | Gestion des identités et du domaine             |
| PowerShell                       | Administration, audit et automatisation         |
| Windows Event Logs               | Security Logging                                |
| Windows Defender Firewall        | Contrôle du trafic entrant                      |
| SMB                              | Partage de fichiers et analyse de configuration |
| WinRM                            | Administration distante                         |
| Git / GitHub                     | Documentation et versionnement                  |

---

# 4. Active Directory & IAM

L’environnement Active Directory a été utilisé comme base du laboratoire afin de travailler sur les problématiques d’identité, de groupes et de privilèges.

L'analyse porte notamment sur les relations entre utilisateurs et groupes de sécurité.

## 4.1 Analyse des groupes

Un premier contrôle consiste à identifier les membres des groupes sensibles.

Exemple :

```powershell
Get-ADGroupMember "GG_IT_Admins" |
Select-Object Name, SamAccountName
```

Le résultat permet notamment d'identifier les comptes disposant de privilèges élevés.

Dans le laboratoire, le groupe `GG_IT_Admins` contient :

```text
Thomas Martin
Thomas.Martin
```

![Membres du groupe privilégié](docs/screenshots/01-ad-privileged-group.png)

## 4.2 Analyse des appartenances aux groupes

Les appartenances de Thomas Martin ont ensuite été vérifiées :

```powershell
Get-ADPrincipalGroupMembership "Thomas.Martin" |
Select-Object Name
```

Résultat observé :

```text
Domain Users
GG_IT
GG_IT_Admins
```

L’analyse permet de comprendre comment un utilisateur hérite de ses droits via différentes appartenances aux groupes.

![Appartenance aux groupes](docs/screenshots/02-ad-group-membership.png)

---

# 5. Privileged Access & Separation of Duties

Une partie du laboratoire porte sur le contrôle des privilèges et la séparation des responsabilités.

Le scénario met notamment en relation :

* Thomas Martin ;
* Nicolas Moreau ;
* leurs groupes respectifs ;
* une opération nécessitant une validation.

Les appartenances de Nicolas Moreau ont été vérifiées avec :

```powershell
Get-ADPrincipalGroupMembership "Nicolas.Moreau" |
Select-Object Name
```

Résultat :

```text
Domain Users
GG_Direction
```

![Analyse des rôles](docs/screenshots/03-sod-group-analysis.png)

Cette analyse permet de mettre en évidence une différence entre :

```text
Exécution d'une opération
        ≠
Validation de l'opération
```

Cette approche permet d'introduire le principe de **Separation of Duties (SoD)** et de limiter les risques liés à la concentration de privilèges.

---

# 6. Windows Security Assessment

L'audit du serveur ne se limite pas à Active Directory.

Plusieurs composants Windows exposant des fonctionnalités réseau ou d'administration ont été examinés.

Les contrôles réalisés portent notamment sur :

* Windows Defender Firewall ;
* SMB ;
* WinRM ;
* Windows Services ;
* mécanismes d'administration distante ;
* Security Logs.

L'objectif est d'identifier les services nécessaires, les surfaces d'exposition et les configurations pouvant présenter un risque.

---

# 7. Windows Defender Firewall

L'état des profils Firewall a été vérifié avec :

```powershell
Get-NetFirewallProfile |
Select-Object Name, Enabled, DefaultInboundAction, DefaultOutboundAction
```

Les trois profils Windows ont été examinés :

```text
Domain
Private
Public
```

![Windows Firewall Profiles](docs/screenshots/04-firewall-profiles.png)

Les règles entrantes autorisées ont également été analysées afin d'identifier les services accessibles et les règles présentant une exposition importante.

Une attention particulière a été portée aux règles concernant :

* SMB ;
* WMI ;
* RPC ;
* Active Directory ;
* WinRM ;
* File and Printer Sharing.

![Firewall Rules](docs/screenshots/05-firewall-rules.png)

---

# 8. SMB Security

SMB constitue un composant important de l'environnement Windows et a donc fait l'objet d'un contrôle spécifique.

La configuration a été vérifiée avec :

```powershell
Get-SmbServerConfiguration |
Select-Object EnableSMB1Protocol,
              EnableSMB2Protocol,
              EnableSecuritySignature,
              RequireSecuritySignature
```

La configuration observée est :

```text
EnableSMB1Protocol       False
EnableSMB2Protocol       True
EnableSecuritySignature  True
RequireSecuritySignature True
```

Cette configuration permet notamment de vérifier que SMBv1 est désactivé et que SMB Signing est activé et requis.

![SMB Security Configuration](docs/screenshots/06-smb-hardening.png)

### Analyse

SMBv1 est un protocole ancien présentant des risques de sécurité importants.

La désactivation de SMBv1 réduit la surface d'attaque associée à ce protocole.

La présence de SMB Signing permet également de renforcer l'intégrité des communications SMB.

---

# 9. Windows Services Security Assessment

Les services Windows en cours d'exécution ont été examinés afin d'identifier les composants présentant un intérêt pour la sécurité.

La commande utilisée est notamment :

```powershell
Get-Service |
Where-Object {
    $_.Status -eq "Running"
} |
Sort-Object DisplayName |
Select-Object DisplayName, Name, StartType
```

L'analyse a porté en particulier sur les services liés à :

* Active Directory ;
* DNS ;
* Netlogon ;
* RPC ;
* SMB ;
* Windows Management Instrumentation ;
* Windows Remote Management ;
* Print Spooler.

L'objectif n'est pas de désactiver arbitrairement les services, mais de déterminer lesquels sont nécessaires au fonctionnement du serveur et lesquels représentent une surface d'administration ou d'exposition.

---

# 10. Print Spooler

Le service Print Spooler a été examiné :

```powershell
Get-Service Spooler |
Select-Object Name, Status, StartType
```

Résultat :

```text
Name     Status   StartType
Spooler  Running  Automatic
```

La présence d'imprimantes et leur état de partage/publication ont également été contrôlés :

```powershell
Get-Printer |
Select-Object Name, Shared, Published
```

Les imprimantes présentes dans le laboratoire n'étaient pas partagées ni publiées.

![Print Spooler Assessment](docs/screenshots/07-spooler-assessment.png)

Ce contrôle permet d'illustrer une démarche de réduction de surface d'attaque : un service doit être évalué selon son utilité réelle avant toute décision de désactivation.

---

# 11. WinRM & Remote Administration

Windows Remote Management a également été analysé.

L'état du service a été vérifié :

```powershell
Get-Service WinRM |
Select-Object Name, Status, StartType
```

Le service était actif et configuré en démarrage automatique.

La configuration des listeners a ensuite été examinée :

```powershell
winrm enumerate winrm/config/listener
```

Le laboratoire présentait notamment un listener :

```text
Transport = HTTP
Port = 5985
Enabled = true
```

![WinRM Listener](docs/screenshots/08-winrm-listener.png)

L'écoute réseau a également été vérifiée :

```powershell
Get-NetTCPConnection -LocalPort 5985 -State Listen |
Select-Object LocalAddress, LocalPort, State, OwningProcess
```

Résultat :

```text
LocalAddress = ::
LocalPort    = 5985
State        = Listen
OwningProcess = 4
```

Le processus associé a ensuite été identifié :

```powershell
Get-Process -Id (
    Get-NetTCPConnection -LocalPort 5985 -State Listen
).OwningProcess |
Select-Object Id, ProcessName
```

Le processus correspond au processus système Windows.

![WinRM Listening Port](docs/screenshots/09-winrm-port.png)

---

# 12. WinRM Firewall Exposure

Les règles Windows Firewall associées à WinRM ont été examinées :

```powershell
Get-NetFirewallRule -DisplayName "*Windows Remote Management (HTTP-In)*" |
Select-Object DisplayName, Enabled, Profile, Direction, Action
```

Les règles actives ont été identifiées sur les profils concernés.

Les adresses distantes autorisées ont ensuite été examinées :

```powershell
Get-NetFirewallRule -DisplayName "*Windows Remote Management (HTTP-In)*" |
ForEach-Object {
    Get-NetFirewallAddressFilter -AssociatedNetFirewallRule $_ |
    Select-Object Name, RemoteAddress
}
```

L'analyse a notamment montré des règles utilisant :

```text
Any
LocalSubnet
```

![WinRM Firewall Rules](docs/screenshots/10-winrm-firewall.png)

Ce contrôle permet d'identifier une surface d'administration distante qui doit être limitée aux sources réellement nécessaires.

---

# 13. Security Logging

Une partie importante du laboratoire consiste à exploiter les Security Logs Windows plutôt qu'à se limiter à l'analyse de configuration.

Un répertoire dédié a été créé :

```powershell
New-Item -ItemType Directory `
    -Path "C:\CyberLab\SecurityLogs" `
    -Force
```

Les événements correspondant à l'Event ID `4625` ont ensuite été extraits :

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 100 |
Export-Csv "C:\CyberLab\SecurityLogs\FailedLogons.csv" `
-NoTypeInformation -Encoding UTF8
```

Le fichier généré a été vérifié :

```powershell
Get-Item "C:\CyberLab\SecurityLogs\FailedLogons.csv" |
Select-Object Name, Length, LastWriteTime
```

![Security Log Export](docs/screenshots/11-security-log-export.png)

---

# 14. Event ID 4625 Analysis

L'Event ID `4625` correspond à un échec d'authentification.

Les événements ont été convertis en données structurées afin de faciliter leur analyse.

Les champs étudiés comprennent notamment :

* `Time`
* `User`
* `Domain`
* `LogonType`
* `SourceIP`
* `Workstation`
* `Status`
* `SubStatus`
* `Process`

Le parsing a été réalisé directement à partir du XML des événements Windows :

```powershell
$events = Get-WinEvent -FilterHashtable @{
    LogName = 'Security'
    Id = 4625
} -MaxEvents 100

$Report = foreach ($event in $events) {

    $xml = [xml]$event.ToXml()
    $data = @{}

    foreach ($item in $xml.Event.EventData.Data) {
        $data[$item.Name] = $item.'#text'
    }

    [PSCustomObject]@{
        Time        = $event.TimeCreated
        User        = $data['TargetUserName']
        Domain      = $data['TargetDomainName']
        LogonType   = $data['LogonType']
        SourceIP    = $data['IpAddress']
        Workstation = $data['WorkstationName']
        Status      = $data['Status']
        SubStatus   = $data['SubStatus']
        Process     = $data['ProcessName']
    }
}

$Report |
Export-Csv "C:\CyberLab\SecurityLogs\FailedLogons_Analysis.csv" `
-NoTypeInformation -Encoding UTF8
```

![Event 4625 Analysis](docs/screenshots/12-event-4625-analysis.png)

---

# 15. Network Authentication Failure Detection

Une règle de détection simple a ensuite été développée.

L'objectif était d'identifier une même adresse IP générant plusieurs échecs d'authentification réseau.

La logique utilisée est :

```text
Event ID 4625
      |
      v
LogonType = 3
      |
      v
SourceIP disponible
      |
      v
Exclusion des connexions locales
      |
      v
Groupement par SourceIP
      |
      v
>= 2 échecs
      |
      v
Security Alert
```

Le script PowerShell :

```powershell
$Alerts = Import-Csv `
    "C:\CyberLab\SecurityLogs\FailedLogons_Analysis.csv" |
Where-Object {
    $_.LogonType -eq '3' -and
    $_.SourceIP -and
    $_.SourceIP -ne '127.0.0.1'
} |
Group-Object SourceIP |
Where-Object {
    $_.Count -ge 2
} |
ForEach-Object {
    [PSCustomObject]@{
        SourceIP       = $_.Name
        FailedAttempts = $_.Count
        Alert          = "Multiple network authentication failures"
    }
}
```

Le laboratoire a produit l'alerte suivante :

```text
SourceIP       FailedAttempts
192.168.100.20 2
```

avec :

```text
Alert = Multiple network authentication failures
```

![Network Authentication Detection](docs/screenshots/13-authentication-detection.png)

---

# 16. Alert Export

Les alertes ont été exportées dans un fichier CSV afin de conserver un résultat exploitable :

```powershell
$Alerts |
Export-Csv "C:\CyberLab\SecurityLogs\Alerts.csv" `
-NoTypeInformation -Encoding UTF8
```

Le contenu obtenu :

```text
SourceIP,FailedAttempts,Alert
192.168.100.20,2,Multiple network authentication failures
```

![Generated Security Alert](docs/screenshots/14-security-alert.png)

Cette étape constitue une première approche de **Security Monitoring** et de détection automatisée à partir des événements Windows.

---

# 17. PowerShell Security Automation

PowerShell a été utilisé comme outil principal d'administration et d'automatisation.

Le laboratoire utilise PowerShell pour :

* interroger Active Directory ;
* analyser les groupes ;
* examiner les services ;
* auditer le Firewall ;
* contrôler SMB ;
* analyser WinRM ;
* extraire les Security Logs ;
* parser les événements XML ;
* produire des fichiers CSV ;
* regrouper les événements ;
* générer des alertes.

Cette approche permet de passer d'une analyse manuelle à une première forme d'automatisation de Security Assessment et Security Monitoring.

---

# 18. Security Findings

Les contrôles réalisés permettent de dégager plusieurs catégories de constats.

| Domaine          | Constat                                     | Action / recommandation                                      |
| ---------------- | ------------------------------------------- | ------------------------------------------------------------ |
| Active Directory | Présence de groupes privilégiés             | Contrôler régulièrement les memberships                      |
| IAM              | Droits associés aux groupes                 | Appliquer Least Privilege                                    |
| SoD              | Séparation entre exécution et validation    | Maintenir une séparation claire des responsabilités          |
| SMB              | SMBv1 désactivé                             | Maintenir cette configuration                                |
| SMB              | SMB Signing activé et requis                | Maintenir cette protection                                   |
| Firewall         | Plusieurs règles d'administration entrantes | Vérifier régulièrement les sources autorisées                |
| WinRM            | Listener HTTP sur TCP/5985                  | Restreindre l'accès aux sources d'administration nécessaires |
| Services         | Plusieurs services actifs                   | Vérifier leur nécessité et réduire la surface inutile        |
| Authentication   | Plusieurs Event ID 4625                     | Mettre en place une surveillance des échecs répétés          |
| Monitoring       | Détection basée sur SourceIP                | Étendre la corrélation à d'autres indicateurs                |

---

# 19. Démarche de Security Assessment

Le laboratoire suit une démarche structurée :

```text
Identify
   |
   v
Assess
   |
   v
Analyze
   |
   v
Harden
   |
   v
Verify
   |
   v
Monitor
   |
   v
Detect
   |
   v
Recommend
```

Cette démarche permet de relier les différentes parties du projet au lieu de traiter chaque outil ou commande indépendamment.

---

# 20. Compétences démontrées

Ce laboratoire m'a permis de travailler concrètement sur :

### Windows Security

* Windows Server Security Assessment ;
* Windows Defender Firewall ;
* SMB Security ;
* WinRM ;
* Windows Services ;
* Security Logging.

### Active Directory & IAM

* Active Directory ;
* Users and Groups ;
* Group Membership Analysis ;
* Privileged Groups ;
* Least Privilege ;
* Separation of Duties.

### Security Monitoring

* Windows Security Events ;
* Event ID 4625 ;
* Authentication Failure Analysis ;
* Source IP Analysis ;
* Detection Logic ;
* Security Alerts.

### Automation

* PowerShell ;
* Event XML Parsing ;
* CSV Processing ;
* Automated Security Analysis ;
* Alert Generation.

### Security Assessment

* Configuration Review ;
* Attack Surface Analysis ;
* Risk Identification ;
* Security Findings ;
* Security Recommendations.

---

# 21. Technologies utilisées

```text
Windows Server
Active Directory
PowerShell
Windows Event Logs
Windows Defender Firewall
SMB
WinRM
Git
GitHub
```

---

# 22. Structure du dépôt

```text
Windows-Security-Assessment-AD-Hardening-Lab/
│
├── README.md
│
├── docs/
│   └── screenshots/
│       ├── 01-ad-privileged-group.png
│       ├── 02-ad-group-membership.png
│       ├── 03-sod-group-analysis.png
│       ├── 04-firewall-profiles.png
│       ├── 05-firewall-rules.png
│       ├── 06-smb-hardening.png
│       ├── 07-spooler-assessment.png
│       ├── 08-winrm-listener.png
│       ├── 09-winrm-port.png
│       ├── 10-winrm-firewall.png
│       ├── 11-security-log-export.png
│       ├── 12-event-4625-analysis.png
│       ├── 13-authentication-detection.png
│       └── 14-security-alert.png
│
├── scripts/
│   ├── ActiveDirectory/
│   ├── SecurityAssessment/
│   └── SecurityMonitoring/
│
└── reports/
```

---

# 23. Limites du laboratoire

Ce laboratoire constitue un environnement de formation et ne reproduit pas l'ensemble des composants d'une infrastructure d'entreprise.

Il ne constitue notamment pas une plateforme SIEM complète ni un environnement SOC industriel.

Les mécanismes de détection développés sont volontairement simples et servent à démontrer les principes de :

* Security Logging ;
* Event Analysis ;
* Detection Logic ;
* Alert Generation ;
* PowerShell Automation.

---

# 24. Perspectives d'évolution

Les prochaines évolutions du projet porteront notamment sur la sécurité des systèmes d'intelligence artificielle.

Un second laboratoire sera développé séparément afin d'étudier :

* LLM Security ;
* Prompt Injection ;
* AI Threat Analysis ;
* RAG Security ;
* Agent Security ;
* API Security ;
* AI Risk Assessment ;
* Security Monitoring ;
* AI Security Governance.

Cette séparation permet de conserver une cohérence entre les deux projets :

```text
Lab 1
Windows / Active Directory Security
        |
        | Security Assessment
        | Hardening
        | IAM
        | Monitoring
        |
        v
Lab 2
AI Security
        |
        | AI Threat Analysis
        | LLM Security
        | RAG / Agents
        | AI Governance
```

---

# Conclusion

Ce projet met en pratique une démarche progressive de cybersécurité appliquée à un environnement Windows Server et Active Directory.

L'objectif n'est pas uniquement d'exécuter des commandes d'administration, mais de suivre une démarche complète :

```text
Audit
→ Analyse
→ Hardening
→ Vérification
→ Logging
→ Detection
→ Automation
→ Recommendations
```

Le laboratoire constitue ainsi une base pratique pour approfondir les domaines de la sécurité des systèmes, de l'IAM, du Security Monitoring, de la détection et de l'audit de sécurité.
