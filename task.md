# Samba + AD — Suivi des Tâches (Terminé à 100%)

## SPRINT 1 — Préparation et Réseau
- [x] Vérifier config VirtualBox Host-Only → ✅ `172.16.2.1/24`, Up
- [x] Vérifier IPs Ubuntu (Guest Properties) → ✅ `enp0s3=192.168.1.134`, `enp0s8=172.16.2.210`
- [x] Diagnostic liaison DC → ✅ Déplacement à chaud nic1 vers Host-Only
- [x] Valider ping bidirectionnel Ubuntu ↔ DC → ✅ 0% packet loss (<1ms)
- [x] Hostname / FQDN → ✅ `srv-samba.rs.local`
- [x] `/etc/hosts` → ✅ Mappage IP DC & Samba
- [x] Netplan durable (`00-installer-config.yaml`) → ✅ Appliqué
- [x] Synchronisation NTP (Chrony avec DC) → ✅ Synchro active
- [x] Installation paquets (samba, winbind, krb5-user, realmd, etc.) → ✅ Installés
- [x] `/etc/krb5.conf` avec enctypes AD → ✅ Configuré
- [x] Test `kinit` + `klist` → ✅ Ticket TGT obtenu (24h)

## SPRINT 2 — Jointure AD
- [x] Configurer `smb.conf` mode ADS → ✅ Validé
- [x] Résolution DNS domaine → ✅ `rs.local`, `dc.rs.local`
- [x] Jointure au domaine (`net ads join`) → ✅ Rejoint avec succès
- [x] `net ads testjoin` → ✅ "Join is OK"
- [x] Enregistrement DNS (`samba-tool dns add`) → ✅ `srv-samba.rs.local -> 172.16.2.210`

## SPRINT 3 — Samba ADS
- [x] Nettoyer `smb.conf` (suppression `password server`) → ✅ Fait
- [x] `testparm` sans erreurs → ✅ Validé
- [x] Démarrer et activer `smbd`, `nmbd`, `winbind` → ✅ Actifs et activés

## SPRINT 4 — Winbind et Authentification
- [x] NSS (`/etc/nsswitch.conf` avec winbind) → ✅ Configuré
- [x] PAM auto-création home (`pam_mkhomedir`) → ✅ Activé
- [x] `wbinfo -t` (Secret RPC) → ✅ Succès
- [x] `wbinfo -u` (Comptes AD) → ✅ `administrator`, `user1`, `user2`, `user3`, `user4`, `khaled`
- [x] `wbinfo -g` (Groupes AD) → ✅ `domain admins`, `info`, `drh`, `srv_bkp_users`, etc.
- [x] `getent passwd` / `id user1` → ✅ UID 11107, GID 10513

## SPRINT 5 — Partages Samba
- [x] Créer structure `/srv/samba/` (`commun`, `informatique`, `drh`, `backup`) → ✅ Créés
- [x] Configurer partages dans `smb.conf` → ✅ 4 partages configurés
- [x] Validation `testparm` → ✅ Validé
- [x] Test énumération `smbclient -L` → ✅ 4 partages visibles

## SPRINT 6 — Permissions et ACL
- [x] Permissions Linux de base (`chmod 2775 / 2770`) → ✅ Appliquées
- [x] ACLs POSIX par défaut et d'accès (`setfacl`) → ✅ Appliquées
- [x] Test accès positif (`user1` sur `informatique`) → ✅ Autorisé
- [x] Test accès négatif (`user4` sur `informatique`) → ✅ `NT_STATUS_ACCESS_DENIED`
- [x] Test accès négatif (`user1` sur `drh`) → ✅ `NT_STATUS_ACCESS_DENIED`

## SPRINT 7 — Quotas
- [x] Activer `usrquota,grpquota` dans `/etc/fstab` → ✅ Activé
- [x] `quotacheck` et `quotaon` sur `/dev/sda2` → ✅ Quotas actifs
- [x] Configurer quota utilisateur (`user1` : 50M soft / 60M hard) → ✅ Configuré
- [x] Configurer quota groupe (`info` : 200M soft / 250M hard) → ✅ Configuré
- [x] Vérification `quota -v` et `repquota` → ✅ Suivi en temps réel validé

## SPRINT 8 — Sécurité
- [x] Désactivation des accès anonymes (`map to guest = Bad User`, `guest ok = no`) → ✅ Validé
- [x] Protocole minimum `SMB2_02` → ✅ Validé
- [x] Pare-feu UFW activé (ports 22, 445, 139, 137, 138) → ✅ `Status: active`
- [x] Maintien strict de l'accès SSH → ✅ Validé

## SPRINT 9 — Tests Windows
- [x] Montage SMB depuis Windows (`net use \\172.16.2.210\commun`) → ✅ Réussi
- [x] Montage SMB avec compte AD (`user1`) sur `informatique` → ✅ Réussi
- [x] Création et écriture de fichiers depuis Windows → ✅ Validé
- [x] Préservation des ACLs et de la propriété AD sous Linux → ✅ Validé

## SPRINT 10 — Validation Finale
- [x] Batterie automatisée de 16 tests de conformité → ✅ **16/16 PASS (100%)**

## SPRINT 11 — Documentation
- [x] Production de la documentation technique complète → ✅ [documentation_projet_samba_ad.md](file:///C:/Users/ahmed/.gemini/antigravity-ide/brain/3f350c95-0e0b-484f-9789-ef4fc79c0c57/documentation_projet_samba_ad.md)
