Projet de Chat de Groupe RPC
----------------------------

Une application de chat de groupe simple utilisant des sockets TCP avec un protocole RPC personnalisé.

Fonctionnalités
---------------

- Inscription et connexion des utilisateurs (mots de passe hachés avec SHA-256, stockés dans users.txt)
- Chat de groupe avec interface utilisateur en "bulles" dans le client
- À la connexion, le client reçoit tout l'historique du chat sans déclencher de notifications pop-up
- Les clients ne voient que les nouveaux messages comme pop-ups
- Une interface graphique serveur permet à un "admin" de diffuser des messages (préfixés avec "SERVEUR: …")
- Chaque client voit une liste en temps réel des autres utilisateurs en ligne (ne s'affiche jamais lui-même)

Fichiers
--------
### ChatProtocol.java
Définit l'interface RPC (login, register, broadcast, getHistory).

### ChatServer.java
- Écoute sur le port 12345
- Maintient les identifiants dans users.txt
- Conserve une liste synchronisée de l'historique du chat
- L'interface graphique affiche "Utilisateurs en ligne" et un "Journal du chat"
- Diffuse les événements de connexion/déconnexion et les nouveaux messages
- S'assure que l'historique est envoyé avant de diffuser le "a rejoint" de l'utilisateur pour éviter les doublons

### ChatClient.java
- Se connecte au serveur IP:12345
- Demande l'IP du serveur, puis connexion/inscription
- Lors de la connexion :
  - S'enregistre pour recevoir les lignes "MESSAGE …" entrantes
  - Charge tout l'ancien historique avec les notifications supprimées
  - Après cela, tout nouveau message ou événement de connexion/déconnexion d'autres utilisateurs déclenche un pop-up
- L'interface graphique affiche une zone de chat en "bulles" et une liste des autres utilisateurs en ligne

### users.txt
- Stocke les lignes de "nom_utilisateur:hash_sha256"
- Initialement vide. Chaque nouvelle inscription ajoute une ligne

Prérequis
---------
- Java 8 (ou plus récent) installé (javac/java dans le PATH)
- Aucune bibliothèque externe requise

Comment Compiler
----------------
Ouvrez un terminal dans ce dossier et exécutez :

```bash
javac ChatProtocol.java ChatServer.java ChatClient.java
```

Cela génère :
- ChatProtocol.class
- ChatServer.class
- ChatClient.class

Comment Exécuter
----------------
1. Démarrer le serveur

```bash
java ChatServer
```

L'interface graphique du serveur apparaît :
- Panneau gauche : "Utilisateurs en ligne" (autres noms d'utilisateur uniquement)
- Panneau central : "Journal du chat" (toutes les connexions/déconnexions et messages)
- Champ inférieur : tapez du texte et appuyez sur Entrée pour diffuser comme "SERVEUR: …"

2. Démarrer un ou plusieurs clients

Chaque client doit être démarré dans son propre terminal :

```bash
java ChatClient
```

- Un dialogue demande "Entrez l'IP du serveur :" (tapez `localhost` si sur la même machine)
- Ensuite, un dialogue "Connexion/Inscription" apparaît :
  - Entrez nom d'utilisateur/mot de passe
  - Cliquez sur "S'inscrire" pour créer un nouveau compte (ajoute à users.txt), ou "Se connecter" si déjà inscrit
- Une fois connecté :
  - L'interface graphique du client affiche :
    - Panneau gauche : liste des **autres** utilisateurs en ligne (votre propre nom est masqué)
    - Panneau droit : zone de chat de style bulles
  - Lorsque le client s'ouvre pour la première fois, il reçoit tous les messages passés mais aucune notification pop-up pour eux
  - Seuls les nouveaux messages ou événements de connexion/déconnexion d'autres utilisateurs déclenchent un dialogue pop-up "Nouveau message"

3. Comportement avec plusieurs clients

- Chaque nouveau client voit l'historique complet (pas de pop-ups pour les anciens messages)
- Quand un client envoie un message, il apparaît comme une bulle partout et déclenche un pop-up sur les autres clients
- Quand un client se connecte ou se déconnecte, tous les autres voient "SERVEUR: &lt;utilisateur&gt; a rejoint/a quitté" exactement une fois (bulle + pop-up)

Notes / Dépannage
-----------------

### users.txt
- Doit exister (même si vide). Le serveur le crée s'il manque
- Format : `nom_utilisateur:sha256_encodé_hex(mot_de_passe)`. Ne pas éditer manuellement sauf pour supprimer des comptes

### Port 12345
- Assurez-vous qu'aucun pare-feu ou autre application ne bloque le TCP 12345

### Comportement des pop-ups
- L'ancien historique n'affiche jamais de pop-ups. Seuls les nouveaux messages (et connexions/déconnexions d'autres utilisateurs) affichent des pop-ups

### Comportement de sortie
- Fermer la fenêtre du client déclenche une diffusion "SERVEUR: &lt;utilisateur&gt; a quitté"
- Fermer la fenêtre du serveur arrête le serveur et déconnecte tous les clients
