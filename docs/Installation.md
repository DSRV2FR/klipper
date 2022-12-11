#Mise en place

Ces instructions supposent que le logiciel fonctionnera sur un ordinateur Raspberry Pi en conjonction avec OctoPrint. Il est recommandé d'utiliser un ordinateur Raspberry Pi 2, 3 ou 4 comme machine hôte (voir la [FAQ](FAQ.md#can-i-run-klipper-on-something-other-than-a-raspberry-pi-3) pour d'autres appareils).

## Obtenir un fichier de configuration Klipper

La plupart des paramètres de Klipper sont déterminés par un "fichier de configuration d'imprimante" qui sera stocké sur le Raspberry Pi. Un fichier de configuration approprié peut souvent être trouvé en regardant dans le Klipper [config directory](../config/) pour un fichier commençant par un préfixe "printer-" qui correspond à l'imprimante cible. Le fichier de configuration Klipper contient des informations techniques sur l'imprimante qui seront nécessaires lors de l'installation.

S'il n'y a pas de fichier de configuration d'imprimante approprié dans le répertoire de configuration de Klipper, essayez de rechercher sur le site Web du fabricant de l'imprimante pour voir s'il dispose d'un fichier de configuration Klipper approprié.

Si aucun fichier de configuration pour l'imprimante ne peut être trouvé, mais que le type de carte de contrôle de l'imprimante est connu, recherchez un [config file](../config/) commençant par un préfixe "générique-". Ces exemples de fichiers de carte d'imprimante devraient permettre de réussir l'installation initiale, mais nécessiteront une certaine personnalisation pour obtenir toutes les fonctionnalités de l'imprimante.

Il est également possible de définir une nouvelle configuration d'imprimante à partir de zéro. Cependant, cela nécessite des connaissances techniques importantes sur l'imprimante et son électronique. Il est recommandé que la plupart des utilisateurs commencent avec un fichier de configuration approprié. Si vous créez un nouveau fichier de configuration d'imprimante personnalisé, commencez par l'exemple le plus proche [config file](../config/) et utilisez le Klipper [config reference](Config_Reference.md) pour plus d'informations.

## Préparation d'une image de système d'exploitation

Commencez par installer [OctoPi](https://github.com/guysoft/OctoPi) sur l'ordinateur Raspberry Pi. Utilisez OctoPi v0.17.0 ou version ultérieure - voir le [OctoPi releases](https://github.com/guysoft/OctoPi/releases) pour les informations de sortie. Il faut vérifier qu'OctoPi démarre et que le serveur Web OctoPrint fonctionne. Après vous être connecté à la page Web OctoPrint, suivez l'invite pour mettre à niveau OctoPrint vers la version 1.4.2 ou ultérieure.

Après avoir installé OctoPi et mis à niveau OctoPrint, il sera nécessaire de ssh dans la machine cible pour exécuter une poignée de commandes système. Si vous utilisez un bureau Linux ou MacOS, le logiciel "ssh" doit déjà être installé sur le bureau. Des clients ssh gratuits sont disponibles pour d'autres ordinateurs de bureau (par exemple,
[PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/)). Utilisez l'utilitaire ssh pour vous connecter au Raspberry Pi (ssh pi@octopi - le mot de passe est "raspberry") et exécutez les commandes suivantes :

```
git clone https://github.com/Klipper3d/klipper
./klipper/scripts/install-octopi.sh
```

Ce qui précède téléchargera Klipper, installera certaines dépendances du système, configurera Klipper pour qu'il s'exécute au démarrage du système et démarrera le logiciel hôte Klipper. Cela nécessitera une connexion Internet et cela peut prendre quelques minutes.

## Construire et flasher le micro-contrôleur

Pour compiler le code du microcontrôleur, commencez par exécuter ces commandes sur le Raspberry Pi :

```
cd ~/klipper/
make menuconfig
```

Les commentaires en haut du [fichier de configuration de l'imprimante] (#obtain-a-klipper-configuration-file) doivent décrire les paramètres qui doivent être définis lors de "make menuconfig". Ouvrez le fichier dans un navigateur Web ou un éditeur de texte et recherchez ces instructions en haut du fichier. Une fois que les paramètres "menuconfig" appropriés ont été configurés, appuyez sur "Q" pour quitter, puis sur "Y" pour enregistrer. Puis cours:

```
make
```

Si les commentaires en haut du [fichier de configuration de l'imprimante](#obtain-a-klipper-configuration-file) décrire les étapes personnalisées pour « flasher » l'image finale sur la carte de commande de l'imprimante, puis suivre ces étapes, puis passer à
[configuring OctoPrint](#configuring-octoprint-to-use-klipper).

Sinon, les étapes suivantes sont souvent utilisées pour « flasher » la carte de contrôle de l'imprimante. Tout d'abord, il est nécessaire de déterminer le port série connecté au microcontrôleur. Exécutez ce qui suit :

```
ls /dev/serial/by-id/*
```

Il devrait signaler quelque chose de similaire à ce qui suit :

```
/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

Il est courant que chaque imprimante ait son propre nom de port série unique. Ce nom unique sera utilisé lors du flashage du microcontrôleur. Il est possible qu'il y ait plusieurs lignes dans la sortie ci-dessus - si c'est le cas, choisissez la ligne correspondant au microcontrôleur (voir la [FAQ](FAQ.md#wheres-my-serial-port) pour plus d'informations).

Pour les microcontrôleurs courants, le code peut être flashé avec quelque chose de similar to:

```
sudo service klipper stop
make flash FLASH_DEVICE=/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
sudo service klipper start
```

Assurez-vous de mettre à jour FLASH_DEVICE avec le nom de port série unique de l'imprimante.

Lors du premier clignotement, assurez-vous qu'OctoPrint n'est pas connecté directement à l'imprimante (à partir de la page Web d'OctoPrint, sous le "Connection" section, click "Disconnect").

## Configurer OctoPrint pour utiliser Klipper

Le serveur Web OctoPrint doit être configuré pour communiquer avec le logiciel hôte Klipper. À l'aide d'un navigateur Web, connectez-vous à la page Web OctoPrint, puis configurez les éléments suivants :

Accédez à l'onglet Paramètres (l'icône de clé en haut de la page). En dessous de"Serial Connection" dans "Additional serial ports" add "/tmp/printer". Puis clique "Save".

Entrez à nouveau dans l'onglet Paramètres et sous "Serial Connection" changer "Serial Port" mise à "/tmp/printer".

Dans l'onglet Paramètres, accédez au "Behavior" sous-onglet et sélectionnez le "Cancel any ongoing prints but stay connected to the printer" option. Click "Save".

Depuis la page principale, sous le "Connection" section (en haut à gauche de la page) assurez-vous que le "Serial Port" est réglé sur"/tmp/printer" et click "Connect". (Si "/tmp/printer" n'est pas une sélection disponible, essayez de recharger la page.)

Une fois connecté, accédez au"Terminal" onglet et tapez "status" (sans les guillemets) dans la zone de saisie de commande et cliquez sur "Send". La fenêtre du terminal signalera probablement une erreur lors de l'ouverture du fichier de configuration - cela signifie qu'OctoPrint communique avec succès avec Klipper. Passez à la section suivante.

## Configuration Klipper

L'étape suivante consiste à copier le [fichier de configuration de l'imprimante](#obtain-a-klipper-configuration-file) au Raspberry Pi.

Le moyen le plus simple de définir le fichier de configuration de Klipper est sans doute d'utiliser un éditeur de bureau qui prend en charge l'édition de fichiers sur le "scp" and/or "sftp" protocoles. Il existe des outils disponibles gratuitement qui prennent en charge cela (par exemple, Notepad++, WinSCP, et Cyberduck). Chargez le fichier de configuration de l'imprimante dans l'éditeur, puis enregistrez-le sous un fichier nommé "printer.cfg" dans le répertoire personnel de l'utilisateur pi (par exemple, /home/pi/printer.cfg).

Alternativement, on peut également copier et éditer le fichier directement sur le Raspberry Pi via ssh. Cela peut ressembler à ceci (assurez-vous de mettre à jour la commande pour utiliser le nom de fichier de configuration d'imprimante approprié) :

```
cp ~/klipper/config/example-cartesian.cfg ~/printer.cfg
nano ~/printer.cfg
```

Il est courant que chaque imprimante ait son propre nom unique pour le microcontrôleur. Le nom peut changer après avoir clignoté Klipper, alors réexécutez ces étapes même si elles ont déjà été effectuées lors du clignotement. Courir:

```
ls /dev/serial/by-id/*
```

Il devrait signaler quelque chose de similaire à ce qui suit :

```
/dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

Ensuite, mettez à jour le fichier de configuration avec le nom unique. Par exemple, mettez à jour le `[mcu]` section pour ressembler à quelque chose de similaire à:

```
[mcu]
serial: /dev/serial/by-id/usb-1a86_USB2.0-Serial-if00-port0
```

Après avoir créé et édité le fichier, il sera nécessaire d'émettre un "restart" commande dans le terminal Web OctoPrint pour charger le fichier config. Un "status" la commande signalera que l'imprimante est prête si le fichier de configuration Klipper est lu avec succès et que le microcontrôleur est trouvé et configuré avec succès.

Lors de la personnalisation du fichier de configuration de l'imprimante, il n'est pas rare que Klipper signale une erreur de configuration. Si une erreur se produit, apportez les corrections nécessaires au fichier de configuration de l'imprimante et lancez "restart" jusqu'à "status" signale que l'imprimante est prête.

Klipper signale les messages d'erreur via l'onglet du terminal OctoPrint. La commande "status" peut être utilisée pour signaler à nouveau les messages d'erreur. Le script de démarrage par défaut de Klipper place également un journal dans **/tmp/klippy.log** qui fournit des informations plus détaillées.

Une fois que Klipper signale que l'imprimante est prête, passez au [document de vérification de la configuration] (Config_checks.md) pour effectuer quelques vérifications de base sur les définitions du fichier de configuration. Voir le principal [documentation reference](Overview.md) pour d'autres informations.
