# Coller avec ⌘V dans Pi via le terminal intégré de VS Code

Ce guide explique comment coller le contenu du presse-papiers de macOS dans [Pi](https://github.com/badlogic/pi-mono) lorsqu’il s’exécute dans un conteneur Ubuntu ouvert depuis VS Code.

## Pourquoi ⌘V peut échouer

VS Code s’exécute sur le Mac, mais Pi s’exécute dans le conteneur Linux. Le processus Pi ne peut donc pas lire directement le presse-papiers macOS avec les outils de presse-papiers Linux. Il faut que **VS Code lise le presse-papiers du Mac et transmette le texte au terminal**.

Dans ce scénario, le collage depuis la palette de commandes de VS Code ou par clic droit peut fonctionner même si le raccourci clavier n’est pas correctement associé. Le réglage des raccourcis appartient à VS Code, pas à Pi.

## Configurer définitivement les raccourcis VS Code sur macOS

Le raccourci doit être enregistré dans le **fichier de raccourcis utilisateur de VS Code sur le Mac**, et non dans le conteneur Ubuntu ni dans le dépôt. Ainsi, il reste disponible après la reconnexion au conteneur et les redémarrages de VS Code.

1. Ouvrir la palette de commandes avec **⌘⇧P**.
2. Choisir **Preferences: Open Keyboard Shortcuts (JSON)**.
3. Vérifier que le fichier ouvert est bien le fichier utilisateur du profil VS Code actif, puis ajouter les raccourcis suivants au tableau JSON existant. Si le fichier ne contient pas d’autres raccourcis, son contenu peut être remplacé par cet exemple :

```json
[
  {
    "key": "cmd+v",
    "command": "workbench.action.terminal.paste",
    "when": "terminalFocus"
  },
  {
    "key": "ctrl+shift+v",
    "command": "workbench.action.terminal.paste",
    "when": "terminalFocus"
  }
]
```

4. Enregistrer avec **⌘S**.
5. Lancer **Developer: Reload Window** depuis la palette de commandes **une seule fois après la modification**. Cette commande force VS Code à recharger le fichier ; elle ne doit pas être nécessaire à chaque démarrage.
6. Cliquer dans le terminal où Pi est actif, puis essayer **⌘V**. **Ctrl+Shift+V** peut servir de raccourci de remplacement.

Sur macOS, `cmd+v` désigne **Commande+V**. N’utilisez pas `win+v`, qui correspond à Windows.

### Si le raccourci disparaît après chaque redémarrage

Cela signifie généralement que VS Code n’utilise pas le même profil ou que la synchronisation des paramètres remplace le fichier. Dans **Preferences: Open Keyboard Shortcuts (JSON)**, réouvrez le fichier après un redémarrage et vérifiez que les deux entrées sont toujours présentes. Si elles ont disparu :

- sélectionnez le profil VS Code utilisé pour le conteneur ;
- recréez les entrées dans son fichier utilisateur, pas dans un fichier du projet ;
- vérifiez que **Settings Sync** ne remplace pas les raccourcis avec une configuration venant d’un autre appareil ;
- ne placez pas ces entrées dans `~/.pi/agent/keybindings.json` : ce fichier configure Pi et est distinct du fichier de raccourcis de VS Code.

Une configuration persistante doit donc survivre à **Developer: Reload Window**, à la fermeture de VS Code et à la reconnexion au conteneur. Si le rechargement reste nécessaire à chaque ouverture, utilisez **Developer: Toggle Keyboard Shortcuts Troubleshooting** : si `cmd+v` n’apparaît pas dans le fichier utilisateur actif, le problème vient du profil ou de la synchronisation, pas de Pi.

> Si `keybindings.json` contient déjà des raccourcis, conservez-les dans le même tableau `[...]` et séparez chaque objet par une virgule. Des objets placés en dehors du tableau rendent le fichier invalide et VS Code peut les ignorer.

## Configuration côté Pi

Si Pi est configuré pour traiter lui-même le raccourci de collage, il peut essayer de lire le presse-papiers depuis Ubuntu au lieu de celui du Mac. Dans `~/.pi/agent/keybindings.json`, la configuration utilisée pour laisser VS Code gérer le collage est :

```json
{
  "app.clipboard.pasteImage": []
}
```

Ce fichier est celui de l’utilisateur qui lance Pi dans le conteneur. Une configuration dans le répertoire personnel d’un autre utilisateur (par exemple `/root`) ne s’applique pas à Pi lancé par `ubuntu`.

## Vérification et dépannage

- Vérifier que le terminal a le focus : la condition du raccourci est `terminalFocus`.
- Depuis la palette de commandes, exécuter **Terminal: Paste into Active Terminal**. Si cela colle le texte, VS Code peut transmettre le presse-papiers au terminal.
- Dans **Developer: Toggle Keyboard Shortcuts Troubleshooting**, rechercher l’appui sur `KeyV`. `No keybinding entries` signifie que VS Code n’a pas trouvé de raccourci correspondant à cette touche et à son contexte.
- Vérifier que le **fichier utilisateur de raccourcis VS Code** est un tableau JSON valide et contient encore les entrées ; ne pas le confondre avec `~/.pi/agent/keybindings.json`.
- Pour tester le chemin de collage indépendamment de Pi, lancer `cat` dans un terminal VS Code, copier du texte sur le Mac et essayer le raccourci.

## Contexte du test

La configuration a été validée avec VS Code sur macOS 14.6.1 et Pi 1.2.0 dans un conteneur Ubuntu. Le collage depuis la palette VS Code et le clic droit fonctionnait ; après correction du tableau `keybindings.json` et rechargement de la fenêtre, **⌘V** a également fonctionné.
