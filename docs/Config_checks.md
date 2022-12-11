# Contrôles de configuration

Ce document fournit une liste d'étapes pour vous aider à confirmer les paramètres de broche dans le fichier Klipper printer.cfg. C'est une bonne idée d'exécuter ces étapes après avoir suivi les étapes du [document d'installation] (Installation.md).

Au cours de ce guide, il peut être nécessaire d'apporter des modifications au fichier de configuration de Klipper. Assurez-vous d'émettre une commande RESTART après chaque modification du fichier de configuration pour vous assurer que la modification prend effet (tapez "restart" dans l'onglet du terminal Octoprint, puis cliquez sur "Envoyer"). C'est aussi une bonne idée d'émettre une commande STATUS après chaque RESTART pour vérifier que le fichier de configuration est chargé avec succès.

## Vérifier la température

Commencez par vérifier que les températures sont correctement signalées. Accédez à l'onglet de température Octoprint.

![octoprint-temperature](img/octoprint-temperature.png)

Vérifiez que la température de la buse et du lit (le cas échéant) est présente et n'augmente pas. S'il augmente, coupez l'alimentation de l'imprimante. Si les températures ne sont pas exactes, vérifiez les paramètres "sensor_type" et "sensor_pin" pour la buse et/ou le lit.

## Vérifier M112

Accédez à l'onglet du terminal Octoprint et émettez une commande M112 dans la boîte à bornes. Cette commande demande à Klipper de passer à l'état "shutdown". Cela entraînera la déconnexion d'Octoprint de Klipper - accédez à la zone de connexion et cliquez sur "Connecter" pour qu'Octoprint se reconnecte. Accédez ensuite à l'onglet de température Octoprint et vérifiez que les températures continuent de se mettre à jour et qu'elles n'augmentent pas. Si les températures augmentent, coupez l'alimentation de l'imprimante.

La commande M112 fait passer Klipper dans un état "arrêt". Pour effacer cet état, émettez une commande FIRMWARE_RESTART dans l'onglet du terminal Octoprint.

## Vérifier les radiateurs

Accédez à l'onglet de température Octoprint et tapez 50 suivi d’entrer dans la case de température "Outil". La température de l'extrudeuse dans le graphique devrait commencer à augmenter (dans environ 30 secondes environ). Alors
allez dans la liste déroulante de température "Outil" et sélectionnez "Off". Après plusieurs minutes, la température devrait commencer à revenir à sa valeur initiale valeur de la température ambiante. Si la température n'augmente pas alors vérifiez le paramètre "heater_pin" dans la configuration.

Si l'imprimante est équipée d'un lit chauffant, effectuez à nouveau le test ci-dessus avec le lit.

## Vérifiez la broche d'activation du moteur pas à pas

Vérifiez que tous les axes de l'imprimante peuvent se déplacer manuellement librement (les moteurs pas à pas sont désactivés). Si ce n'est pas le cas, émettez une commande M84 pour désactiver les moteurs. Si l'un des axes ne peut toujours pas se déplacer librement, vérifiez la configuration du stepper "enable_pin" pour l'axe donné. Sur la plupart des pilotes de moteur pas à pas de base, la broche d'activation du moteur est "actif bas" et par conséquent, la broche d'activation doit avoir un "!" avant la broche (par exemple, "enable_pin: !ar38").

## Vérifier les butées

Déplacez manuellement tous les axes de l'imprimante afin qu'aucun d'entre eux ne soit en contact avec une butée. Envoyez une commande QUERY_ENDSTOPS via l'onglet du terminal Octoprint. Il doit répondre avec l'état actuel de tous les butées configurées et ils doivent tous signaler un état "ouvert". Pour chacune des butées, réexécutez la commande QUERY_ENDSTOPS tout en déclenchant manuellement la butée. La commande QUERY_ENDSTOPS doit signaler l'endstop comme "TRIGGERED".

Si la butée apparaît inversée (elle signale "ouverte" lorsqu'elle est déclenchée et vice-versa), ajoutez un "!" à la définition de la broche (par exemple, "endstop_pin: ^!ar3"), ou supprimez le "!" s'il y en a déjà un présent.

Si la butée ne change pas du tout, cela indique généralement que la butée est connectée à une broche différente. Cependant, cela peut également nécessiter une modification du paramètre de pullup de la broche (le '^' au début du nom endstop_pin - la plupart des imprimantes utilisent une résistance de pullup et le '^' doit être présent).

## Verify stepper motors

Utilisez la commande STEPPER_BUZZ pour vérifier la connectivité de chaque moteur pas à pas. Commencez par positionner manuellement l'axe donné sur un point médian, puis exécutez `STEPPER_BUZZ STEPPER=stepper_x`. La commande STEPPER_BUZZ fera bouger le stepper donné d'un millimètre dans une direction positive, puis il reviendra à sa position de départ. (Si la butée est définie à position_endstop=0
puis au début de chaque mouvement le stepper s'éloignera de la butée.) Il effectuera cette oscillation dix fois.

Si le stepper ne bouge pas du tout, vérifiez les paramètres "enable_pin" et "step_pin" pour le stepper. Si le moteur pas à pas se déplace mais ne revient pas à sa position d'origine, vérifiez le paramètre "dir_pin". Si le moteur pas à pas oscille dans une direction incorrecte, cela indique généralement que le "dir_pin" de l'axe doit être inversé. Cela se fait en ajoutant un '!' au "dir_pin" dans le fichier de configuration de l'imprimante (ou en le supprimant s'il y en a déjà un). Si le moteur se déplace nettement plus ou nettement moins d'un millimètre, vérifiez le paramètre "rotation_distance".

Exécutez le test ci-dessus pour chaque moteur pas à pas défini dans le fichier de configuration. (Définissez le paramètre STEPPER de la commande STEPPER_BUZZ sur le nom de la section de configuration à tester.) S'il n'y a pas de filament dans l'extrudeuse, vous pouvez utiliser STEPPER_BUZZ pour vérifier la connectivité du moteur de l'extrudeuse (utilisez STEPPER=extruder). Sinon, il est préférable de tester le moteur de l'extrudeuse séparément (voir la section suivante).

Après avoir vérifié toutes les butées et vérifié tous les moteurs pas à pas, le mécanisme de prise d'origine doit être testé. Émettez une commande G28 pour référencer tous les axes. Coupez l'alimentation de l'imprimante si elle ne fonctionne pas correctement. Réexécutez les étapes de vérification de la butée et du moteur pas à pas si nécessaire.

## Vérifier le moteur de l'extrudeuse

Pour tester le moteur de l'extrudeuse, il sera nécessaire de chauffer l'extrudeuse à une température d'impression. Accédez à l'onglet de température Octoprint et sélectionnez une température cible dans la liste déroulante de température (ou entrez manuellement une température appropriée). Attendez que l'imprimante atteigne la température souhaitée. Accédez ensuite à l'onglet de contrôle Octoprint et cliquez sur le bouton "Extruder". Vérifiez que le moteur de l'extrudeuse tourne dans le bon sens. Si ce n'est pas le cas, consultez les conseils de dépannage de la section précédente pour confirmer les paramètres "enable_pin", "step_pin" et "dir_pin" pour l'extrudeuse.

## Calibrer les paramètres PID

Klipper prend en charge
[Contrôle PID](https://en.wikipedia.org/wiki/PID_controller) pour l'extrudeuse et les chauffe-lits. Afin d'utiliser ce mécanisme de contrôle, il est nécessaire de calibrer les paramètres PID sur chaque imprimante (les paramètres PID trouvés dans d'autres firmwares ou dans les exemples de fichiers de configuration fonctionnent souvent mal).

Pour calibrer l'extrudeuse, accédez à l'onglet du terminal OctoPrint et exécutez la commande PID_CALIBRATE. Par exemple : `PID_CALIBRATE HEATER=extruder TARGET=170`

À la fin du test de réglage, exécutez `SAVE_CONFIG` pour mettre à jour le fichier printer.cfg avec les nouveaux paramètres PID.

Si l'imprimante dispose d'un lit chauffant et qu'elle prend en charge le pilotage par PWM (modulation de largeur d'impulsion), il est recommandé d'utiliser le contrôle PID pour le lit. (Lorsque le chauffage du lit est contrôlé à l'aide de l'algorithme PID, il peut s'allumer et s'éteindre dix fois par seconde, ce qui peut ne pas convenir aux chauffages utilisant un interrupteur mécanique.) Une commande d'étalonnage PID de lit typique est : `PID_CALIBRATE HEATER=heater_bed TARGET= 60`

## Prochaines étapes

Ce guide est destiné à aider à la vérification de base des paramètres de broche dans le fichier de configuration de Klipper. Assurez-vous de lire le guide [bed leveling](Bed_Level.md). Consultez également le document [Slicers](Slicers.md) pour plus d'informations sur la configuration d'un slicer avec Klipper.

Après avoir vérifié que l'impression de base fonctionne, c'est une bonne idée d'envisager de calibrer [l'avance de la pression] (Pressure_Advance.md).

Il peut être nécessaire d'effectuer d'autres types de calibrage détaillé de l'imprimante - un certain nombre de guides sont disponibles en ligne pour vous aider (par exemple, effectuez une recherche sur le Web pour « calibrage de l'imprimante 3d »). Par exemple, si vous ressentez l'effet appelé sonnerie, vous pouvez essayer de suivre le guide de réglage [compensation de résonance](Resonance_Compensation.md).
