# Projet Samba + Active Directory

Laboratoire d'intégration d'un serveur Ubuntu au domaine **Active Directory `RS.LOCAL`** avec Samba en mode ADS. Le projet met en place un serveur de fichiers centralisé, l'authentification des comptes AD via Winbind, des ACLs POSIX, des quotas disque et un pare-feu UFW.

> **Statut :** projet réalisé et validé — **16/16 tests réussis**.

## Objectifs

- Joindre Ubuntu au domaine Active Directory.
- Authentifier les utilisateurs et groupes AD depuis Linux avec Kerberos, NSS et Winbind.
- Publier des partages SMB avec des droits distincts par service.
- Appliquer des permissions Linux et des ACLs POSIX cohérentes avec les groupes AD.
- Limiter l'utilisation du stockage avec des quotas utilisateur et groupe.
- Sécuriser les accès SMB et SSH avec UFW et un protocole SMB minimal SMB2.

## Architecture du laboratoire

| Machine | Adresse | Rôle |
| --- | --- | --- |
| Windows Server `dc.rs.local` | `172.16.2.209/24` | Contrôleur de domaine, DNS, Kerberos/KDC et NTP |
| Ubuntu Server `srv-samba.rs.local` | `172.16.2.210/24` | Membre AD, serveur Samba, Winbind, partages et quotas |
| Hôte Windows | `172.16.2.1/24` | Hyperviseur VirtualBox et poste de test |

Le réseau privé utilise le mode **Host-Only VirtualBox** (`172.16.2.0/24`). Le serveur Ubuntu dispose également d'une interface réseau en pont pour les mises à jour système.

## Fonctionnalités configurées

### Intégration Active Directory

- Domaine : `RS.LOCAL`
- Realm Kerberos : `RS.LOCAL`
- Serveur Samba : `srv-samba.rs.local`
- Mode Samba : `security = ADS`
- Winbind pour la résolution des utilisateurs et groupes AD
- Création automatique des répertoires personnels avec `pam_mkhomedir`

### Partages SMB

| Partage | Chemin Linux | Accès principal |
| --- | --- | --- |
| `commun` | `/srv/samba/commun` | Utilisateurs et administrateurs du domaine |
| `informatique` | `/srv/samba/informatique` | Groupes `info`, `grp-inf` et administrateurs |
| `drh` | `/srv/samba/drh` | Groupes `drh`, `grp-drh` et administrateurs |
| `backup` | `/srv/samba/backup` | Groupe `srv_bkp_users` et administrateurs |

Les accès anonymes sont désactivés. Les droits sont contrôlés par les groupes AD, les permissions Linux et les ACLs POSIX.

### Quotas et sécurité

- Quota utilisateur configuré pour `user1` : 50 Mo soft / 60 Mo hard.
- Quota du groupe `info` : 200 Mo soft / 250 Mo hard.
- Pare-feu UFW actif.
- Ports autorisés : SSH `22/tcp`, SMB `445/tcp`, NetBIOS `139/tcp`, `137/udp` et `138/udp`.
- Protocole SMB minimal : `SMB2_02`.
- Accès invité désactivé (`guest ok = no`).

## Structure du dépôt

```text
.
├── documentation_projet_samba_ad.md  # Documentation technique détaillée
├── task.md                           # Suivi des tâches et des sprints
├── walkthrough.md                     # Rapport de réalisation et validation
├── server19/                          # Métadonnées de la VM Windows Server
├── srv-samba/                         # Métadonnées de la VM Ubuntu Samba
└── .gitignore                         # Disques et fichiers générés exclus du dépôt
```

Les fichiers de disque virtuel (`.vdi`), journaux et fichiers temporaires VirtualBox ne sont pas versionnés.

## Prérequis

- Windows 10/11 ou un système compatible avec VirtualBox.
- VirtualBox 7.x.
- Une VM Windows Server avec les rôles AD DS, DNS, Kerberos et NTP.
- Une VM Ubuntu Server avec deux interfaces réseau :
  - une interface Host-Only sur `172.16.2.0/24` ;
  - une interface en pont ou NAT pour l'accès aux paquets.
- Accès administrateur sur le domaine de laboratoire.

Les adresses IP et noms de domaine ci-dessus sont ceux du laboratoire documenté. Adaptez-les à votre environnement avant tout déploiement.

## Installation et déploiement

1. **Configurer le réseau VirtualBox**

   Créez ou sélectionnez un réseau Host-Only et attribuez à l'adaptateur hôte l'adresse `172.16.2.1/24`. Connectez les deux VMs à ce réseau.

2. **Préparer le contrôleur de domaine**

   Configurez le contrôleur Windows avec le domaine `rs.local`, l'adresse `172.16.2.209` et le DNS local. Vérifiez que le serveur Ubuntu peut résoudre `dc.rs.local` et `rs.local`.

3. **Configurer Ubuntu**

   Définissez le hostname `srv-samba`, ajoutez les entrées DNS nécessaires dans `/etc/hosts`, configurez le DNS AD dans Netplan et synchronisez l'horloge avec le contrôleur de domaine.

4. **Installer les composants**

   Installez les paquets adaptés à votre version Ubuntu :

   ```bash
   sudo apt update
   sudo apt install samba winbind libnss-winbind libpam-winbind \
     krb5-user smbclient acl quota chrony ufw
   ```

5. **Configurer Kerberos, Samba, NSS et PAM**

   Reportez-vous à `documentation_projet_samba_ad.md` pour les exemples de `/etc/krb5.conf`, `/etc/samba/smb.conf`, `/etc/nsswitch.conf`, Netplan et les commandes de jointure AD.

6. **Joindre le domaine**

   Après avoir vérifié la résolution DNS et la synchronisation de l'heure, exécutez la jointure avec un compte administrateur du domaine :

   ```bash
   sudo net ads join -U "Administrator@RS.LOCAL"
   sudo net ads testjoin
   ```

   N'inscrivez jamais de mot de passe réel dans le dépôt ou dans l'historique Git.

7. **Créer et sécuriser les partages**

   Créez `/srv/samba/commun`, `/srv/samba/informatique`, `/srv/samba/drh` et `/srv/samba/backup`, appliquez les ACLs nécessaires, puis validez la configuration :

   ```bash
   sudo testparm -s
   smbclient -L localhost -U 'RS\Administrator'
   ```

8. **Activer les services et le pare-feu**

   ```bash
   sudo systemctl enable --now smbd nmbd winbind
   sudo ufw enable
   ```

## Validation

La validation finale couvre notamment :

- connectivité IP et résolution DNS ;
- obtention d'un ticket Kerberos ;
- jointure AD et secret de confiance Winbind ;
- résolution NSS des comptes AD ;
- état des services Samba ;
- syntaxe de `smb.conf` ;
- visibilité des quatre partages ;
- ACLs POSIX et quotas ;
- accès autorisés et refusés ;
- montage et écriture depuis Windows.

Résultat documenté : **16 tests sur 16 réussis**.

Pour le détail des commandes, résultats et incidents rencontrés, consulter :

- [`documentation_projet_samba_ad.md`](documentation_projet_samba_ad.md)
- [`task.md`](task.md)
- [`walkthrough.md`](walkthrough.md)

## Sécurité

Ce dépôt décrit un environnement de laboratoire. Remplacez toutes les adresses, comptes et secrets d'exemple avant utilisation en production. Utilisez des mots de passe gérés hors du dépôt, limitez les ports au réseau nécessaire et sauvegardez les configurations Samba et Active Directory.

## Licence

Aucune licence open source n'est actuellement déclarée pour ce projet.
