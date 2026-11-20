# Cloner un projet GitHub avec SSH sur Windows

Guide complet, pas à pas, pour les débutants.

---

## Table des matières

1. [Comprendre SSH en 2 minutes](#1-comprendre-ssh-en-2-minutes)
2. [Prérequis](#2-prérequis)
3. [Installer Git pour Windows](#3-installer-git-pour-windows)
4. [Configurer votre identité Git](#4-configurer-votre-identité-git)
5. [Vérifier si vous avez déjà une clé SSH](#5-vérifier-si-vous-avez-déjà-une-clé-ssh)
6. [Générer une clé SSH](#6-générer-une-clé-ssh)
7. [Démarrer l'agent SSH et ajouter la clé](#7-démarrer-lagent-ssh-et-ajouter-la-clé)
8. [Ajouter la clé publique à GitHub](#8-ajouter-la-clé-publique-à-github)
9. [Tester la connexion SSH](#9-tester-la-connexion-ssh)
10. [Cloner le projet](#10-cloner-le-projet)
11. [Utilisation au quotidien](#11-utilisation-au-quotidien)
12. [Passer un dépôt HTTPS existant en SSH](#12-passer-un-dépôt-https-existant-en-ssh)
13. [Dépannage](#13-dépannage)
14. [Bonnes pratiques de sécurité](#14-bonnes-pratiques-de-sécurité)
15. [Aide-mémoire](#15-aide-mémoire)
16. [Glossaire](#16-glossaire)

---

## 1. Comprendre SSH en 2 minutes

**SSH** (Secure Shell) est un protocole qui permet de communiquer de façon sécurisée avec un serveur, ici GitHub, **sans taper votre mot de passe à chaque fois**.

Il repose sur une **paire de clés** :

| Clé | Fichier (par défaut) | Rôle | À partager ? |
|-----|----------------------|------|--------------|
| **Clé privée** | `id_ed25519` | Votre « clé de maison » secrète. Elle reste sur votre PC. | **JAMAIS** |
| **Clé publique** | `id_ed25519.pub` | Le « cadenas » que vous donnez à GitHub. | Oui, c'est fait pour ça |

**Principe** : vous placez le cadenas (clé publique) sur GitHub. Quand vous vous connectez, votre PC prouve qu'il possède la clé correspondante (clé privée), sans jamais l'envoyer.

### SSH ou HTTPS ?

| | SSH | HTTPS |
|---|-----|-------|
| URL de clonage | `git@github.com:user/repo.git` | `https://github.com/user/repo.git` |
| Authentification | Clé SSH | Jeton d'accès personnel (token) |
| Configuration initiale | Une seule fois | Gestionnaire d'identifiants |
| Pratique au quotidien | Très pratique | Pratique aussi |

> **Note** : GitHub n'accepte plus les mots de passe de compte pour Git en HTTPS. Il faut un jeton ou une clé SSH. SSH est donc un excellent choix.

---

## 2. Prérequis

- Un PC sous **Windows 10 ou Windows 11**
- Un **compte GitHub** (gratuit) : <https://github.com/signup>
- Une connexion Internet
- Les droits d'accès au dépôt à cloner (s'il est privé)

---

## 3. Installer Git pour Windows

### 3.1 Vérifier si Git est déjà installé

Ouvrez **PowerShell** (touche `Windows`, tapez `PowerShell`, Entrée) et saisissez :

```powershell
git --version
```

- Si vous voyez `git version 2.xx.x...` : Git est installé, passez à l'[étape 4](#4-configurer-votre-identité-git).
- Si vous voyez une erreur « commande introuvable » : continuez ci-dessous.

### 3.2 Installer Git

1. Rendez-vous sur <https://git-scm.com/download/win>
2. Téléchargez l'installateur (le téléchargement démarre automatiquement).
3. Lancez-le et acceptez les options par défaut en cliquant sur **Next**. Points d'attention :
   - **Default editor** : choisissez l'éditeur que vous préférez (Notepad++ ou VS Code par exemple).
   - **SSH executable** : laissez **« Use bundled OpenSSH »**.
   - **Initial branch name** : « Let Git decide » convient, ou choisissez `main`.
4. Cliquez sur **Install**, puis **Finish**.
5. **Fermez et rouvrez PowerShell**, puis vérifiez :

```powershell
git --version
ssh -V
```

Les deux commandes doivent afficher un numéro de version.

> **Astuce** : Windows 10/11 inclut aussi son propre client OpenSSH. Les commandes de ce guide fonctionnent avec les deux.

---

## 4. Configurer votre identité Git

Git signe chaque commit avec un nom et un e-mail. Configurez-les **une seule fois** :

```powershell
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@exemple.com"
```

> **Conseil** : utilisez l'e-mail associé à votre compte GitHub (ou l'adresse `noreply` fournie par GitHub dans *Settings > Emails*) pour que vos commits soient liés à votre profil.

Vérification :

```powershell
git config --global --list
```

---

## 5. Vérifier si vous avez déjà une clé SSH

Dans PowerShell :

```powershell
dir $env:USERPROFILE\.ssh
```

- Si le dossier n'existe pas ou est vide : vous n'avez pas de clé, passez à l'étape 6.
- Si vous voyez des fichiers `id_ed25519` et `id_ed25519.pub` (ou `id_rsa` et `id_rsa.pub`) : vous avez déjà une clé. Vous pouvez la réutiliser et passer à l'[étape 7](#7-démarrer-lagent-ssh-et-ajouter-la-clé).

> **Attention** : ne générez pas une nouvelle clé du même nom si vous en avez déjà une, car elle **écraserait** l'ancienne.

---

## 6. Générer une clé SSH

### 6.1 Créer la paire de clés

Dans PowerShell, remplacez l'e-mail par le vôtre :

```powershell
ssh-keygen -t ed25519 -C "votre.email@exemple.com"
```

> `ed25519` est l'algorithme recommandé : moderne, rapide et sûr.
> Si votre système est très ancien et ne le supporte pas : `ssh-keygen -t rsa -b 4096 -C "votre.email@exemple.com"`

### 6.2 Répondre aux questions

**Question 1 : l'emplacement du fichier**

```text
Enter file in which to save the key (C:\Users\VotreNom/.ssh/id_ed25519):
```

Appuyez simplement sur **Entrée** pour accepter l'emplacement par défaut.

**Question 2 : la phrase secrète (passphrase)**

```text
Enter passphrase (empty for no passphrase):
```

Une passphrase protège votre clé privée : si quelqu'un copie votre fichier, il ne pourra pas l'utiliser sans elle.

- **Recommandé** : saisissez une phrase secrète (rien ne s'affiche pendant la frappe, c'est normal), puis confirmez-la.
- Vous pouvez laisser vide (Entrée) pour ne rien avoir à taper, mais c'est moins sécurisé.

### 6.3 Résultat attendu

```text
Your identification has been saved in C:\Users\VotreNom/.ssh/id_ed25519
Your public key has been saved in C:\Users\VotreNom/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx votre.email@exemple.com
```

Deux fichiers sont créés dans `C:\Users\VotreNom\.ssh\` :

- `id_ed25519` : clé **privée** (secrète)
- `id_ed25519.pub` : clé **publique** (à donner à GitHub)

---

## 7. Démarrer l'agent SSH et ajouter la clé

L'**agent SSH** garde votre clé en mémoire pour que vous ne retapiez pas la passphrase à chaque opération.

### 7.1 Activer l'agent SSH (une seule fois)

1. Ouvrez **PowerShell en administrateur** : touche `Windows`, tapez `PowerShell`, clic droit sur **Windows PowerShell**, puis **Exécuter en tant qu'administrateur**.
2. Exécutez :

```powershell
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent
Get-Service ssh-agent
```

La colonne `Status` doit afficher **Running**.

> Si la commande échoue avec une erreur « service introuvable », installez le client OpenSSH via *Paramètres > Applications > Fonctionnalités facultatives > Ajouter une fonctionnalité > Client OpenSSH*.

3. Vous pouvez maintenant fermer cette fenêtre administrateur.

### 7.2 Ajouter votre clé à l'agent

Dans un PowerShell **normal** :

```powershell
ssh-add $env:USERPROFILE\.ssh\id_ed25519
```

Saisissez votre passphrase si demandé. Résultat attendu :

```text
Identity added: C:\Users\VotreNom\.ssh\id_ed25519 (votre.email@exemple.com)
```

Pour vérifier les clés chargées :

```powershell
ssh-add -l
```

---

## 8. Ajouter la clé publique à GitHub

### 8.1 Copier la clé publique

Dans PowerShell, cette commande copie directement la clé publique dans le presse-papiers :

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard
```

Vous pouvez aussi l'afficher pour la copier à la main :

```powershell
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

Elle ressemble à ceci (une seule ligne) :

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... votre.email@exemple.com
```

> **Important** : copiez bien le contenu du fichier **`.pub`**, jamais celui du fichier sans extension (clé privée).

### 8.2 Enregistrer la clé sur GitHub

1. Connectez-vous à <https://github.com>.
2. Cliquez sur votre **photo de profil** (en haut à droite), puis **Settings**.
3. Dans le menu de gauche, section *Access*, cliquez sur **SSH and GPG keys**.
4. Cliquez sur le bouton vert **New SSH key**.
5. Remplissez :
   - **Title** : un nom parlant, par exemple `PC Windows - Maison`
   - **Key type** : `Authentication Key`
   - **Key** : collez votre clé publique (`Ctrl + V`)
6. Cliquez sur **Add SSH key**. GitHub peut demander votre mot de passe ou un code de double authentification.

La clé apparaît maintenant dans votre liste.

---

## 9. Tester la connexion SSH

```powershell
ssh -T git@github.com
```

### 9.1 Première connexion : vérifier l'empreinte du serveur

Au premier lancement, SSH affiche :

```text
The authenticity of host 'github.com (140.82.x.x)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

1. **Comparez l'empreinte** affichée avec celle publiée par GitHub : <https://docs.github.com/fr/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints>
2. Si elles sont identiques, tapez `yes` puis Entrée.

> Cette vérification garantit que vous parlez bien au vrai GitHub. Si l'empreinte est différente, tapez `no` et ne continuez pas.

### 9.2 Résultat attendu

```text
Hi VotreNomGitHub! You've successfully authenticated, but GitHub does not provide shell access.
```

Ce message signifie que **tout fonctionne**. Le « no shell access » est normal : GitHub ne vous ouvre pas de terminal, il sert uniquement Git.

---

## 10. Cloner le projet

### 10.1 Récupérer l'URL SSH du dépôt

1. Ouvrez la page du dépôt sur GitHub.
2. Cliquez sur le bouton vert **Code**.
3. Choisissez l'onglet **SSH** (et non HTTPS).
4. Cliquez sur l'icône de copie.

L'URL ressemble à : `git@github.com:nom-utilisateur/nom-du-projet.git`

### 10.2 Choisir où cloner

Placez-vous dans le dossier qui contiendra votre projet, par exemple :

```powershell
mkdir $env:USERPROFILE\Projets
cd $env:USERPROFILE\Projets
```

### 10.3 Cloner

```powershell
git clone git@github.com:nom-utilisateur/nom-du-projet.git
```

Résultat attendu :

```text
Cloning into 'nom-du-projet'...
remote: Enumerating objects: 120, done.
remote: Counting objects: 100% (120/120), done.
Receiving objects: 100% (120/120), 45.20 KiB | 1.20 MiB/s, done.
Resolving deltas: 100% (50/50), done.
```

Pour cloner dans un dossier au nom différent :

```powershell
git clone git@github.com:nom-utilisateur/nom-du-projet.git mon-dossier
```

### 10.4 Entrer dans le projet et vérifier

```powershell
cd nom-du-projet
git status
git remote -v
```

`git remote -v` doit afficher des URL commençant par `git@github.com:`, ce qui confirme l'usage de SSH.

Pour ouvrir le dossier dans l'Explorateur Windows :

```powershell
explorer .
```

---

## 11. Utilisation au quotidien

| Action | Commande |
|--------|----------|
| Voir l'état des fichiers | `git status` |
| Récupérer les nouveautés | `git pull` |
| Ajouter vos modifications | `git add .` |
| Enregistrer un commit | `git commit -m "Description du changement"` |
| Envoyer vers GitHub | `git push` |
| Voir l'historique | `git log --oneline` |
| Créer une branche | `git switch -c nom-branche` |

Exemple de cycle complet :

```powershell
git pull
# ... vous modifiez des fichiers ...
git add .
git commit -m "Ajout de la page contact"
git push
```

---

## 12. Passer un dépôt HTTPS existant en SSH

Si vous avez déjà cloné un projet en HTTPS, inutile de le recloner :

```powershell
cd chemin\vers\le\projet
git remote set-url origin git@github.com:nom-utilisateur/nom-du-projet.git
git remote -v
```

---

## 13. Dépannage

### `Permission denied (publickey)`

La clé n'est pas reconnue par GitHub. Vérifiez dans l'ordre :

1. La clé est bien ajoutée à GitHub (*Settings > SSH and GPG keys*).
2. Vous avez collé la clé **publique** (`.pub`).
3. La clé est chargée dans l'agent : `ssh-add -l`. Sinon, relancez `ssh-add $env:USERPROFILE\.ssh\id_ed25519`.
4. Diagnostic détaillé : `ssh -vT git@github.com` (cherchez les lignes `Offering public key` et `Authentication succeeded`).
5. Vous avez bien accès au dépôt (vous êtes propriétaire ou collaborateur).

### `Could not open a connection to your authentication agent`

L'agent SSH n'est pas démarré. Voir l'[étape 7.1](#71-activer-lagent-ssh-une-seule-fois) (PowerShell en administrateur).

### `Host key verification failed`

L'empreinte de GitHub a changé ou est corrompue dans votre fichier `known_hosts`. Si vous avez vérifié que l'empreinte officielle est correcte, supprimez l'ancienne entrée :

```powershell
ssh-keygen -R github.com
```

Puis relancez `ssh -T git@github.com`.

### `Repository not found` ou `fatal: Could not read from remote repository`

- L'URL est mal copiée (faute de frappe, mauvais nom d'utilisateur).
- Le dépôt est privé et votre compte n'y a pas accès : demandez une invitation au propriétaire.

### `Bad owner or permissions on ...\.ssh\config`

Les droits du fichier sont trop ouverts. Dans PowerShell :

```powershell
icacls $env:USERPROFILE\.ssh\config /inheritance:r /grant:r "$($env:USERNAME):F"
```

### Connexion bloquée (pare-feu d'entreprise ou réseau public)

Le port 22 est parfois bloqué. GitHub propose SSH sur le port 443 :

```powershell
ssh -T -p 443 git@ssh.github.com
```

Pour le rendre permanent, créez ou éditez le fichier `C:\Users\VotreNom\.ssh\config` :

```text
Host github.com
    Hostname ssh.github.com
    Port 443
    User git
```

### La passphrase est redemandée à chaque fois

L'agent ne conserve pas la clé. Vérifiez que le service `ssh-agent` est en `Running` et relancez `ssh-add`.

### Utiliser plusieurs clés ou comptes GitHub

Créez une clé par compte, puis déclarez-les dans `C:\Users\VotreNom\.ssh\config` :

```text
Host github-perso
    Hostname github.com
    User git
    IdentityFile ~/.ssh/id_ed25519
    IdentitiesOnly yes

Host github-travail
    Hostname github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_travail
    IdentitiesOnly yes
```

Clonez ensuite avec l'alias : `git clone git@github-travail:entreprise/projet.git`

---

## 14. Bonnes pratiques de sécurité

- **Ne partagez jamais votre clé privée** (`id_ed25519`) : ni par e-mail, ni sur GitHub, ni dans un dépôt, ni dans un chat.
- **Utilisez une passphrase** pour protéger la clé.
- **Une clé par appareil** : en cas de perte d'un PC, vous révoquez uniquement sa clé.
- **Révoquez les clés inutilisées** : *Settings > SSH and GPG keys > Delete*.
- **Ne commitez jamais** le dossier `.ssh` ou vos clés dans un dépôt Git.
- Si vous pensez qu'une clé privée a fuité, supprimez-la immédiatement de GitHub et générez-en une nouvelle.

---

## 15. Aide-mémoire

```powershell
# Vérifier les outils
git --version
ssh -V

# Identité Git (une fois)
git config --global user.name "Votre Nom"
git config --global user.email "votre.email@exemple.com"

# Générer une clé
ssh-keygen -t ed25519 -C "votre.email@exemple.com"

# Agent SSH (PowerShell administrateur, une fois)
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent

# Charger la clé
ssh-add $env:USERPROFILE\.ssh\id_ed25519

# Copier la clé publique
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub | Set-Clipboard

# Tester GitHub
ssh -T git@github.com

# Cloner
git clone git@github.com:utilisateur/depot.git
```

---

## 16. Glossaire

| Terme | Définition |
|-------|------------|
| **SSH** | Protocole de communication sécurisée avec un serveur distant. |
| **Clé privée** | Fichier secret qui prouve votre identité. Ne jamais partager. |
| **Clé publique** | Fichier partageable, vérifié par GitHub à partir de votre clé privée. |
| **Passphrase** | Mot de passe qui protège votre clé privée. |
| **Agent SSH** | Programme qui garde vos clés déverrouillées en mémoire. |
| **Empreinte (fingerprint)** | Identifiant unique d'une clé, utilisé pour la vérifier. |
| **Cloner** | Copier un dépôt distant sur votre ordinateur, historique compris. |
| **Dépôt (repository)** | Projet suivi par Git. |
| **Remote / origin** | Adresse du dépôt distant (GitHub). `origin` est son nom par défaut. |
| **Commit** | Enregistrement d'un ensemble de modifications. |

---

*Guide rédigé pour Windows 10/11. Documentation officielle : <https://docs.github.com/fr/authentication/connecting-to-github-with-ssh>*
