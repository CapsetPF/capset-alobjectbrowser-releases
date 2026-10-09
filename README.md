# capset-alobjectbrowser-releases

Dépôt public des installeurs signés d'**AL Object Browser**, l'outil Capset pour parcourir et explorer le code AL de Microsoft Dynamics 365 Business Central (tables, pages, codeunits, extensions, utilisations, hiérarchie d'appels).

Le code source est dans le dépôt privé `CapsetPF/alobjectbrowser`. Ce dépôt ne contient que les binaires publiés et ce README : ni code, ni secret.

## Manuel utilisateur

Le manuel, avec captures d'écran, est en ligne : [capsetpf.github.io/capset-alobjectbrowser-releases](https://capsetpf.github.io/capset-alobjectbrowser-releases/) ([version Word](docs/manuel-almaze.docx)). Il est généré depuis `docs/manuel` du dépôt source.

## Installer

Télécharger le dernier installeur depuis [Releases](https://github.com/CapsetPF/capset-alobjectbrowser-releases/releases/latest) :

| Fichier | Usage |
| --- | --- |
| `AL.Object.Browser_<version>_x64-setup.exe` | **Recommandé.** Installation par utilisateur dans `%LOCALAPPDATA%\AL Object Browser`, sans droits administrateur. Silencieux : `setup.exe /S`. |
| `AL.Object.Browser_<version>_x64_fr-FR.msi` | Déploiement centralisé. Silencieux : `msiexec /i "<fichier>.msi" /qn`. |

Prérequis : Windows 10 ou 11 x64. Le runtime WebView2, déjà présent sur Windows 11 et sur les Windows 10 à jour, est téléchargé par l'installeur s'il manque.

### Avertissement SmartScreen

Les installeurs sont signés par le certificat Capset (`CN=Capset, O=Capset, C=PF`, empreinte `058B758BE49288E841CEA3AB696C6E1D84DE5CC3`). Ce certificat est auto-signé : sans lui dans les autorités de confiance, Windows affiche « Windows a protégé votre ordinateur ». Deux possibilités :

- cliquer sur « Informations complémentaires » puis « Exécuter quand même » ;
- ou approuver une fois pour toutes le certificat public [`capset-signature.cer`](capset-signature.cer) de ce dépôt, dans une invite PowerShell **administrateur** :
  ```powershell
  Import-Certificate -FilePath .\capset-signature.cer -CertStoreLocation Cert:\LocalMachine\Root
  Import-Certificate -FilePath .\capset-signature.cer -CertStoreLocation Cert:\LocalMachine\TrustedPublisher
  ```
  Vérifier d'abord que l'empreinte du fichier est bien celle indiquée ci-dessus.

## Mise à jour automatique

Une fois installée, l'application vérifie à chaque lancement s'il existe une version plus récente sur ce dépôt (`releases/latest/download/latest.json`). Si oui, un bandeau la propose avec ses notes de version : **Installer et redémarrer** télécharge l'installeur, l'installe et relance l'application, **Plus tard** la reproposera au prochain lancement. Sans réseau, l'application fonctionne normalement.

Chaque installeur est signé par une clé **minisign** dont la clé publique est embarquée dans l'application : une mise à jour qui ne porte pas cette signature est refusée, même servie depuis ce dépôt.

## Comment les versions sont produites

Aucun binaire n'est déposé à la main. Les Releases sont créées par le workflow GitHub Actions `publier` du dépôt source :

1. La version est montée dans le dépôt source, puis un tag `v<version>` y est poussé.
2. Le workflow vérifie que le tag concorde avec la version de l'application, puis construit les installeurs avec Tauri 2 (Rust + React), sur le runner Windows de Capset ou à défaut sur un runner `windows-latest` de GitHub.
3. Il signe l'exécutable et les installeurs (Authenticode, horodatage DigiCert), signe les artefacts de mise à jour (minisign), puis crée ici la Release `v<version>` avec :
   - le setup NSIS et le MSI ;
   - leurs signatures `.sig` ;
   - `latest.json`, le manifeste lu par l'application pour se mettre à jour.
4. Les notes de version reprennent les commits depuis la version précédente.

Chaque Release correspond donc exactement à un tag du dépôt source.
