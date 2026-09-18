# Rapport de Réalisation & Validation — Laboratoire Samba + Active Directory

Le projet d'intégration d'Ubuntu au domaine Active Directory avec Samba en mode ADS, Winbind, ACLs POSIX, Quotas et Sécurisation a été **entièrement exécuté, testé et validé à 100%**.

---

## 1. Résumé Exécutif des 11 Sprints

```
[SPRINT 1] Réseau, DNS, Chrony, Kerberos       ───► ✅ VALIDÉ (TGT 24h obtenu)
[SPRINT 2] Jointure Active Directory (ADS)     ───► ✅ VALIDÉ (SRV-SAMBA membre de rs.local)
[SPRINT 3] Configuration Samba ADS & Services  ───► ✅ VALIDÉ (testparm 0 erreur, services actifs)
[SPRINT 4] Winbind & Authentification NSS/PAM  ───► ✅ VALIDÉ (Utilisateurs et groupes AD énumérés)
[SPRINT 5] Partages Samba (4 répertoires)      ───► ✅ VALIDÉ (commun, informatique, drh, backup)
[SPRINT 6] Permissions Linux & ACLs POSIX     ───► ✅ VALIDÉ (setfacl appliqué, accès positif & négatif vérifié)
[SPRINT 7] Quotas Disques Ext4                 ───► ✅ VALIDÉ (50M soft / 60M hard pour user1)
[SPRINT 8] Sécurisation & Pare-feu UFW         ───► ✅ VALIDÉ (UFW actif, ports 22, 445, 139, 137, 138)
[SPRINT 9] Tests d'Accès depuis Windows        ───► ✅ VALIDÉ (net use, écriture de fichiers testée)
[SPRINT 10] Batterie de Validation Finale      ───► ✅ VALIDÉ (16/16 tests PASS)
[SPRINT 11] Documentation Technique Complète   ───► ✅ VALIDÉ ([documentation_projet_samba_ad.md](file:///C:/Users/ahmed/.gemini/antigravity-ide/brain/3f350c95-0e0b-484f-9789-ef4fc79c0c57/documentation_projet_samba_ad.md))
```

---

## 2. Problèmes Rencontrés & Solutions Appliquées

| Problème / Erreur | Cause Racine | Correction Appliquée |
|---|---|---|
| **ARP FAILED entre Ubuntu et Windows DC** | L'adresse statique `172.16.2.209` était sur la carte `Ethernet` liée au réseau en pont sans passerelle L2 avec Ubuntu. | Reconnexion de la carte 1 de la VM `server19` au réseau Host-Only via `VBoxManage controlvm server19 nic1 hostonly`. |
| **`E: The repository 'file:/cdrom resolute Release' no longer has a Release file`** | Dépôt CD-ROM d'installation laissé actif dans `/etc/apt/sources.list.d/cdrom.sources`. | Suppression du fichier `cdrom.sources` et mise à jour d'apt. |
| **`kinit: KDC has no support for encryption type`** | Incompatibilité de chiffrement Kerberos entre Ubuntu récent (AES SHA2) et le DC. | Configuration explicite des `permitted_enctypes` dans `/etc/krb5.conf` avec `aes256-cts-hmac-sha1-96`, `aes128-cts-hmac-sha1-96` et `rc4-hmac`. |
| **`NXDOMAIN` lors de la résolution de `srv-samba` par le DC** | L'enregistrement DNS dynamique n'avait pas été créé automatiquement lors de la jointure. | Enregistrement de l'enregistrement A dans la zone AD via `samba-tool dns add 172.16.2.209 rs.local srv-samba A 172.16.2.210`. |

---

## 3. Résultats de la Batterie de Tests Finale (Sprint 10)

```text
==================================================
    SAMBA + ACTIVE DIRECTORY FINAL VALIDATION    
==================================================
1. Testing IP Ping to DC (172.16.2.209)... [PASS] Network Ping OK
2. Testing DNS resolution of rs.local and dc.rs.local... [PASS] DNS Resolution OK
3. Testing Kerberos TGT acquisition (kinit/klist)... [PASS] Kerberos TGT OK
4. Testing Samba AD Domain Join (net ads testjoin)... [PASS] AD Domain Join OK
5. Testing Winbind RPC Trust Secret (wbinfo -t)... [PASS] Winbind Trust Secret OK
6. Testing AD Users Enumeration (wbinfo -u)... [PASS] AD Users Enumerated OK
7. Testing AD Groups Enumeration (wbinfo -g)... [PASS] AD Groups Enumerated OK
8. Testing Linux NSS AD account resolution (getent passwd)... [PASS] Linux NSS Resolution OK
9. Testing Samba Services (smbd, nmbd, winbind)... [PASS] Samba Services Active OK
10. Validating smb.conf syntax (testparm)... [PASS] smb.conf Syntax OK
11. Testing SMB Shares Listing (smbclient -L)... [PASS] Samba Shares Listing OK
12. Checking POSIX ACLs on shares... [PASS] POSIX ACLs Configured OK
13. Checking Ext4 Disk Quotas... [PASS] Disk Quotas Active OK
14. Checking UFW Firewall Status... [PASS] UFW Firewall Active OK
15. Testing positive access: user1 to /informatique... [PASS] Authorized Access Granted OK
16. Testing negative access: user4 to /informatique (must deny)... [PASS] Unauthorized Access Denied OK
==================================================
ALL 16 VALIDATION TESTS PASSED SUCCESSFULLY!
==================================================
```

---

## 4. Livrables du Projet

- **Documentation Technique Complète :** [documentation_projet_samba_ad.md](file:///C:/Users/ahmed/.gemini/antigravity-ide/brain/3f350c95-0e0b-484f-9789-ef4fc79c0c57/documentation_projet_samba_ad.md)
- **Suivi des Tâches :** [task.md](file:///C:/Users/ahmed/.gemini/antigravity-ide/brain/3f350c95-0e0b-484f-9789-ef4fc79c0c57/task.md)
- **Fichiers de Configuration Déployés sur Ubuntu :**
  - Configuration réseau : `/etc/netplan/00-installer-config.yaml`
  - Configuration Kerberos : `/etc/krb5.conf`
  - Configuration Samba : `/etc/samba/smb.conf`
  - Configuration NSS : `/etc/nsswitch.conf`
  - Système de fichiers & Quotas : `/etc/fstab`
  - Pare-feu : `/etc/ufw/user.rules`
