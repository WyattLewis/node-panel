# Node Panel

Un panneau dans l'esprit de Pterodactyl pour gérer plusieurs projets Node.js sur une machine, par **WyattLewisTV**.

- Tableau de bord : statut, CPU et RAM de chaque projet
- Une page par projet : Démarrer / Arrêter / Redémarrer, console en direct (couleurs comprises), envoi de commandes au processus (stdin)
- Variables d'environnement par projet (`PORT`, `NODE_ENV`…)
- Redémarrage automatique en cas de crash, démarrage automatique avec le panneau
- Scan d'un dossier pour ajouter d'un coup tous les projets qui ont un `package.json`
- **Explorateur de fichiers** par projet, avec un **éditeur de code** (coloration syntaxique, Ctrl+S, recherche) : créer, renommer, envoyer par glisser-déposer, supprimer
- Mises à jour depuis ce dépôt

Deux versions, à télécharger dans [**Releases**](https://github.com/WyattLewis/node-panel/releases/latest) :

| Système | Fichier                            | Forme                                                   |
| ------- | ---------------------------------- | ------------------------------------------------------- |
| Windows | `Node-Panel-Setup-<version>.exe`   | application installable, sans serveur web ni port       |
| Linux   | `node-panel-linux-<version>.tar.gz` | panneau web, à ouvrir dans un navigateur               |

---

## Windows

### Prérequis

- Windows 10 ou 11 (64 bits)
- [Node.js](https://nodejs.org) installé sur le PC : l'application lance tes projets avec le Node.js de ton PC

### Installation

1. Télécharge `Node-Panel-Setup-<version>.exe` depuis la [dernière release](https://github.com/WyattLewis/node-panel/releases/latest).
2. Lance-le. L'installateur n'est pas signé, donc Windows SmartScreen affiche « Windows a protégé votre ordinateur » : clique sur **Informations complémentaires**, puis **Exécuter quand même**.
3. Choisis le dossier d'installation. Les raccourcis Bureau et menu Démarrer sont créés, et l'application se lance.

### Utilisation

- **+ Nouveau projet** : un nom, le dossier du projet (bouton **Parcourir…**) et la commande de démarrage (`npm start` par défaut).
- **Scanner un dossier** : ajoute d'un coup tous les projets d'un dossier.
- Onglet **Fichiers** d'un projet : explorateur et éditeur de code. Les suppressions vont dans la corbeille de Windows.
- **Fermer la fenêtre** la réduit près de l'horloge, et les projets continuent de tourner. Pour quitter vraiment : clic droit sur l'icône, puis **Quitter (arrête les projets)**.
- **Paramètres** : lancer l'application avec Windows, démarrer réduite, comportement du bouton de fermeture.

Tes réglages et ta liste de projets sont dans `%APPDATA%\Node Panel\`.

### Mises à jour

L'application vérifie ce dépôt au démarrage, puis toutes les 4 heures. Une nouvelle version est téléchargée en arrière-plan, puis une notification te prévient. **Rien n'est installé sans ton accord** : dans **Paramètres > Mises à jour**, « Redémarrer et installer » arrête les projets, installe la version et relance l'application. Les projets réglés sur « Démarrer en même temps que le panneau » repartent tout seuls.

### Désinstallation

Paramètres Windows > Applications > **Node Panel** > Désinstaller. Tes réglages et projets restent dans `%APPDATA%\Node Panel\` : supprime ce dossier si tu n'en veux plus.

---

## Linux

### Prérequis

- Node.js 18 ou plus récent, avec npm (`node -v` pour vérifier). Sinon : paquet `nodejs` de ta distribution, [NodeSource](https://github.com/nodesource/distributions) ou [nvm](https://github.com/nvm-sh/nvm).
- `curl` et `tar`

### Installation

Dans le dossier où tu veux installer le panneau (par exemple ton dossier personnel) :

```bash
VERSION=$(curl -fsSL https://api.github.com/repos/WyattLewis/node-panel/releases/latest | sed -n 's/.*"tag_name": *"v\([^"]*\)".*/\1/p')
curl -fLO "https://github.com/WyattLewis/node-panel/releases/download/v$VERSION/node-panel-linux-$VERSION.tar.gz"
tar -xzf "node-panel-linux-$VERSION.tar.gz"
```

```bash
cd node-panel-linux
npm install --omit=dev
```

```bash
npm start
```

Ensuite, ouvre http://127.0.0.1:8420.

> Ne lance pas le panneau en root : il exécute les commandes de tes projets avec les droits de l'utilisateur qui le lance.

### Fichiers des projets

L'onglet **Fichiers** d'un projet permet de parcourir son dossier, d'éditer ses fichiers, d'en envoyer (glisser-déposer) et d'en télécharger. Sous Linux, **la suppression est définitive** (le panneau demande confirmation).

### Accès depuis un autre ordinateur

Par défaut, le panneau n'écoute que sur la machine elle-même (`127.0.0.1`). Deux façons d'y accéder à distance :

- **Tunnel SSH** (le plus sûr, rien à configurer) : depuis ton ordinateur,

  ```bash
  ssh -L 8420:127.0.0.1:8420 utilisateur@ton-serveur
  ```

  puis ouvre http://127.0.0.1:8420 sur ton ordinateur.
- **Réseau local** : dans **Paramètres**, choisis « Tout le réseau local » et définis un mot de passe (obligatoire). Le panneau est alors accessible sur `http://<ip-du-serveur>:8420`.

Avec un mot de passe, le panneau affiche sa propre page de connexion : **un seul champ, le mot de passe** (pas de nom d'utilisateur). La session dure 30 jours ; **Déconnexion** est en haut à droite. Après 5 essais ratés, l'adresse IP est bloquée 15 minutes.

> N'expose jamais le panneau directement sur Internet : il permet de lancer des commandes sur la machine. Si tu en as besoin, passe par un reverse proxy en HTTPS (Caddy, nginx…).

### Port et réglages

Le port par défaut est **8420**, pour ne pas gêner les projets qui utilisent 3000 et plus. Il se change dans **Paramètres** (immédiatement, sans arrêter les projets), dans `config.json`, ou le temps d'un lancement :

```bash
npm start -- --port 9000
```

Les réglages sont dans `config.json` et la liste des projets dans `data/projects.json`, à côté de `server.js`.

### Lancer le panneau au démarrage (systemd)

Crée `/etc/systemd/system/node-panel.service`, en remplaçant `moi` par ton utilisateur et le chemin par celui de ton installation :

```ini
[Unit]
Description=Node Panel
After=network.target

[Service]
User=moi
WorkingDirectory=/home/moi/node-panel-linux
ExecStart=/usr/bin/node server.js
Restart=on-failure
KillMode=mixed
TimeoutStopSec=15

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now node-panel
```

Pour voir les logs du panneau :

```bash
journalctl -u node-panel -f
```

Si tu utilises nvm, mets le chemin complet de `node` dans `ExecStart` (`which node`).

### Mises à jour

Le panneau vérifie ce dépôt au démarrage, puis toutes les 4 heures, et un bandeau annonce les nouvelles versions. Pour installer la dernière, dans le dossier du panneau :

```bash
npm run update
```

```bash
sudo systemctl restart node-panel
```

`config.json` et `data/` (tes projets) sont conservés. Sans `systemd`, arrête le panneau (Ctrl+C) et relance `npm start`.

---

## Bon à savoir

- Le panneau ne définit jamais `PORT` pour tes projets : ajoute-le dans les variables d'environnement de chaque projet (`PORT=3001`, par exemple).
- Un projet qui plante plus de 5 fois en 1 minute n'est plus redémarré automatiquement.
- Sous Windows, l'arrêt d'un projet tue tout son arbre de processus (`cmd`, `npm`, `node`). Sous Linux, il reçoit d'abord `SIGTERM`, puis `SIGKILL` s'il ne s'est pas arrêté au bout de 8 secondes.
