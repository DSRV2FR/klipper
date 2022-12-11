# Nivellement manuel

Ce document décrit les outils pour calibrer une butée Z et pour effectuer des réglages sur les vis de mise à niveau du lit.

## Étalonnage d'une butée Z

Une position de butée Z précise est essentielle pour obtenir des impressions de haute qualité.

Notez, cependant, que la précision de l'interrupteur de butée Z lui-même peut être un facteur limitant. Si l'on utilise des pilotes de moteur pas à pas Trinamic, envisagez d'activer la détection [endstop phase](Endstop_Phase.md) pour améliorer la précision du commutateur.

Pour effectuer un étalonnage de la butée Z, placez l'imprimante à l'origine, commandez à la tête de se déplacer vers une position Z située à au moins cinq millimètres au-dessus du lit (si ce n'est déjà fait), commandez à la tête de se déplacer vers une position XY près du centre de le lit, puis accédez à l'onglet du terminal OctoPrint et exécutez :
```
Z_ENDSTOP_CALIBRATE
```
Suivez ensuite les étapes décrites à ["l'épreuve papier"](Bed_Level.md#the-paper-test) pour déterminer la distance réelle entre la buse et le lit à l'emplacement donné. Une fois ces étapes terminées, vous pouvez "ACCEPT" la position et enregistrer les résultats dans le fichier de configuration avec :
```
SAVE_CONFIG
```

Il est préférable d'utiliser un interrupteur de butée Z à l'extrémité opposée de l'axe Z par rapport au lit. (Retourner loin du lit est plus robuste car il est généralement toujours sûr de ramener le Z.) Cependant, si l'on doit rentrer vers le lit, il est recommandé d'ajuster la butée de manière à ce qu'elle se déclenche sur une petite distance (par exemple, 0,5 mm ) au-dessus du lit. Presque tous les interrupteurs de fin de course peuvent être enfoncés en toute sécurité sur une petite distance au-delà de leur point de déclenchement. Lorsque cela est fait, on devrait constater que la commande `Z_ENDSTOP_CALIBRATE` rapporte une petite valeur positive (par exemple, 0,5 mm) pour la position Z_endstop. Le déclenchement de la butée alors qu'elle est encore à une certaine distance du lit réduit le risque d'écrasement accidentel du lit.

Certaines imprimantes ont la capacité d'ajuster manuellement l'emplacement de l'interrupteur de butée physique. Cependant, il est recommandé d'effectuer le positionnement de la butée Z dans le logiciel avec Klipper - une fois que l'emplacement physique de la butée est à un endroit pratique, on peut faire d'autres ajustements en exécutant Z_ENDSTOP_CALIBRATE ou en mettant à jour manuellement la position Z_endstop dans le fichier de configuration.

## Réglage des vis de mise à niveau du lit

Le secret pour obtenir un bon nivellement du lit avec des vis de nivellement du lit est d'utiliser le système de mouvement de haute précision de l'imprimante pendant le processus de nivellement du lit lui-même. Cela se fait en commandant la buse à une position près de chaque vis de lit, puis en ajustant cette vis jusqu'à ce que le lit soit à une distance définie de la buse. Klipper a un outil pour vous aider. Pour utiliser l'outil, il est nécessaire de spécifier l'emplacement XY de chaque vis.

Cela se fait en créant une section de configuration `[bed_screws]`. Par exemple, cela pourrait ressembler à quelque chose de similaire à :
```
[bed_screws]
screw1: 100, 50
screw2: 100, 150
screw3: 150, 100
```

Si une vis de lit se trouve sous le lit, spécifiez la position XY directement au-dessus de la vis. Si la vis est à l'extérieur du lit, spécifiez une position XY la plus proche de la vis qui se trouve toujours dans la plage du lit.

Une fois que le fichier de configuration est prêt, exécutez "RESTART" pour charger cette configuration, puis vous pouvez démarrer l'outil en exécutant :
```
BED_SCREWS_ADJUST
```

Cet outil déplacera la buse de l'imprimante vers chaque emplacement XY de vis, puis déplacera la buse à une hauteur Z = 0. À ce stade, on peut utiliser le "test papier" pour ajuster la vis de lit directement sous la buse. Voir les informations décrites dans
["the paper test"](Bed_Level.md#the-paper-test), mais ajustez la vis du lit au lieu de commander la buse à différentes hauteurs. Ajustez la vis du lit jusqu'à ce qu'il y ait une petite friction lorsque vous poussez le papier d'avant en arrière.

Une fois que la vis est ajustée de sorte qu'une petite quantité de frottement soit ressentie, faites tourner soit le `ACCEPT` ou `ADJUSTED` commande. Utilisez la commande `ADJUSTED` si la vis du lit nécessitait un ajustement (généralement plus d'environ 1/8 de tour de vis). Utilisez la commande "ACCEPT" si aucun ajustement significatif n'est nécessaire. Les deux commandes feront passer l'outil à la vis suivante. (Lorsqu'une commande "AJUSTED" est utilisée, l'outil programme un cycle supplémentaire d'ajustements de vis d'assise ; l'outil se termine avec succès lorsque toutes les vis du lit sont vérifiées pour ne pas nécessiter d'ajustements importants.) On peut utiliser la commande "ABORT" pour quitter l'outil plus tôt.

Ce système fonctionne mieux lorsque l'imprimante a une surface d'impression plate (telle que du verre) et des rails droits. Une fois l'outil de nivellement du lit terminé avec succès, le lit doit être prêt pour l'impression.

### Réglages de la vis de lit à grain fin

Si l'imprimante utilise trois vis de lit et que les trois vis sont sous le lit, il peut être possible d'effectuer une seconde "haute précision" étape de mise à niveau du lit. Cela se fait en commandant la buse à des endroits où le lit se déplace sur une plus grande distance avec chaque réglage de vis de lit.

Par exemple, considérons un lit avec des vis aux emplacements A, B et C :

![bed_screws](img/bed_screws.svg.png)

Pour chaque réglage effectué sur la vis du lit à l'emplacement C, le lit oscillera le long d'un pendule défini par les deux vis du lit restantes (représentées ici par une ligne verte). Dans cette situation, chaque réglage de la vis du lit en C déplacera le lit en position D d'une plus grande quantité que directement en C. Il est ainsi possible d'effectuer un réglage amélioré de la vis C lorsque la buse est en position D.

Pour activer cette fonctionnalité, il faudrait déterminer les coordonnées supplémentaires de la buse et les ajouter au fichier de configuration. Par exemple, cela pourrait ressembler à :
```
[bed_screws]
screw1: 100, 50
screw1_fine_adjust: 0, 0
screw2: 100, 150
screw2_fine_adjust: 300, 300
screw3: 150, 100
screw3_fine_adjust: 0, 100
```

Lorsque cette fonctionnalité est activée, l'outil `BED_SCREWS_ADJUST` demandera d'abord des ajustements grossiers directement au-dessus de chaque position de vis, et une fois ceux-ci acceptés, il demandera des ajustements fins aux emplacements supplémentaires. Continuez à utiliser `ACCEPT` et `ADJUSTED` à chaque position.

## Réglage des vis de mise à niveau du lit à l'aide de la sonde de lit

C'est une autre façon de calibrer le niveau du lit à l'aide de la sonde de lit. Pour l'utiliser, vous devez disposer d'une sonde Z (BL Touch, capteur inductif, etc.).

Pour activer cette fonctionnalité, il faut déterminer les coordonnées de la buse de sorte que la sonde Z soit au-dessus des vis, puis les ajouter au fichier de configuration. Par exemple, cela pourrait ressembler à :
```
[screws_tilt_adjust]
screw1: -5, 30
screw1_name: front left screw
screw2: 155, 30
screw2_name: front right screw
screw3: 155, 190
screw3_name: rear right screw
screw4: -5, 190
screw4_name: rear left screw
horizontal_move_z: 10.
speed: 50.
screw_thread: CW-M3
```

La screw1 est toujours le point de référence pour les autres, donc le système suppose que la screw1 est à la bonne hauteur. Exécutez toujours `G28` en premier, puis exécutez `SCREWS_TILT_CALCULATE` - cela devrait produire une sortie similaire à:
```
Send: G28
Recv: ok
Send: SCREWS_TILT_CALCULATE
Recv: // 01:20 means 1 full turn and 20 minutes, CW=clockwise, CCW=counter-clockwise
Recv: // front left screw (base) : x=-5.0, y=30.0, z=2.48750
Recv: // front right screw : x=155.0, y=30.0, z=2.36000 : adjust CW 01:15
Recv: // rear right screw : y=155.0, y=190.0, z=2.71500 : adjust CCW 00:50
Recv: // read left screw : x=-5.0, y=190.0, z=2.47250 : adjust CW 00:02
Recv: ok
```

Cela signifie que:
- la vis avant gauche est le point de référence vous ne devez pas la changer.
- la vis avant droite doit être tournée dans le sens des aiguilles d'une montre d'un tour complet et d'un quart de tour
- la vis arrière droite doit être tournée dans le sens inverse des aiguilles d'une montre pendant 50 minutes
- la vis arrière gauche doit être tournée dans le sens horaire 2 minutes (pas besoin c'est ok)

Notez que "minutes" fait référence aux "minutes d'un cadran d'horloge". Donc pour
Par exemple, 15 minutes équivaut à un quart de tour complet.

Répétez le processus plusieurs fois jusqu'à ce que vous obteniez un bon lit de niveau - normalement lorsque tous les ajustements sont inférieurs à 6 minutes.

Si vous utilisez une sonde montée sur le côté de la hotend (c'est-à-dire qu'elle a un décalage X ou Y), notez que le réglage de l'inclinaison du lit invalidera tout étalonnage de sonde précédent effectué avec un lit incliné. Assurez-vous d'exécuter [étalonnage de la sonde] (Probe_Calibrate.md) après avoir ajusté les vis du lit.

Le paramètre `MAX_DEVIATION` est utile lorsqu'un maillage de lit enregistré est utilisé, pour s'assurer que le niveau du lit n'a pas trop dérivé de l'endroit où il se trouvait lorsque le maillage a été créé. Par exemple, `SCREWS_TILT_CALCULATE MAX_DEVIATION=0.01` peut être ajouté au gcode de démarrage personnalisé du slicer avant le chargement du maillage. Il interrompra l'impression si la limite configurée est dépassée (0,01 mm dans cet exemple), donnant à l'utilisateur une chance d'ajuster les vis et de redémarrer l'impression.

Le paramètre `DIRECTION` est utile si vous ne pouvez tourner les vis de réglage de votre lit que dans une seule direction. Par exemple, vous pourriez avoir des vis qui commencent à être serrées dans leur position la plus basse (ou la plus haute) possible, qui ne peuvent être tournées que dans une seule direction, pour élever (ou abaisser) le lit. Si vous ne pouvez tourner les vis que dans le sens des aiguilles d'une montre, exécutez `SCREWS_TILT_CALCULATE DIRECTION=CW`. Si vous ne pouvez les tourner que dans le sens inverse des aiguilles d'une montre, exécutez `SCREWS_TILT_CALCULATE DIRECTION=CCW`. Un point de référence approprié sera choisi de sorte que le lit puisse être nivelé en tournant toutes les vis dans la direction donnée.
