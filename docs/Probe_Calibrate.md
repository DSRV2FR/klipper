# Étalonnage de la sonde

Ce document décrit la méthode d'étalonnage des décalages X, Y et Z d'une "sonde z automatique" dans Klipper. Ceci est utile pour les utilisateurs qui ont une section `[probe]` ou `[bltouch]` dans leur fichier de configuration.

## Étalonnage des décalages X et Y de la sonde

TPour calibrer le décalage X et Y, accédez à l'onglet "Contrôle" d'OctoPrint, placez l'imprimante à l'origine, puis utilisez les boutons de jogging OctoPrint pour déplacer la tête vers une position proche du centre du lit.

Placez un morceau de ruban adhésif bleu (ou similaire) sur le lit sous la robe. Accédez à l'onglet "Terminal" d'OctoPrint et émettez une SONDE
commande:
```
PROBE
```
Placez une marque sur le ruban directement sous l'endroit où se trouve la sonde (ou utilisez une méthode similaire pour noter l'emplacement sur le lit).

Exécutez une commande `GET_POSITION` et enregistrez l'emplacement XY de la tête d'outil signalé par cette commande. Par exemple si l'on voit :
```
Recv: // toolhead: X:46.500000 Y:27.000000 Z:15.000000 E:0.000000
```
alors on enregistrerait une position de sonde X de 46,5 et une position de sonde Y de 27.

Après avoir enregistré la position de la sonde, émettez une série de commandes G1 jusqu'à ce que la buse soit directement au-dessus de la marque sur le lit. Par exemple, on pourrait émettre :
```
G1 F300 X57 Y30 Z15
```
pour déplacer la buse vers une position X de 57 et Y de 30. Une fois que l'on a trouvé la position directement au-dessus de la marque, utilisez la commande "GET_POSITION" pour signaler cette position. C'est la position de la buse.

Le x_offset est alors la `nozzle_x_position - probe_x_position` et y_offset est de même la `nozzle_y_position – probe_y_position`. Mettez à jour le fichier printer.cfg avec les valeurs données, retirez la bande/les marques du lit, puis émettez une commande `RESTART` pour que les nouvelles valeurs prennent effet.

## Sonde d'étalonnage Z offset

Fournir une sonde z_offset précise est essentiel pour obtenir des impressions de haute qualité. Le z_offset est la distance entre la buse et le lit lorsque la sonde se déclenche. L'outil Klipper `PROBE_CALIBRATE` peut être utilisé pour obtenir cette valeur - il exécutera une sonde automatique pour mesurer la position de déclenchement Z de la sonde, puis lancera une sonde manuelle pour obtenir la hauteur Z de la buse. La sonde z_offset sera alors calculée à partir de ces mesures.

Commencez par orienter l'imprimante, puis déplacez la tête vers une position proche du centre du lit. Accédez à l'onglet du terminal OctoPrint et exécutez la commande "PROBE_CALIBRATE" pour démarrer l'outil.

Cet outil effectuera une sonde automatique, puis soulèvera la tête, déplacera la buse sur l'emplacement du point de sonde et démarrera l'outil de sonde manuelle. Si la buse ne se déplace pas vers une position au-dessus du point de sonde automatique, alors "ABORT" l'outil de sonde manuelle et effectuez l'étalonnage de décalage de sonde XY décrit ci-dessus.

Une fois que l'outil de sonde manuelle démarre, suivez les étapes décrites dans ["the paper test"](Bed_Level.md#the-paper-test)) pour déterminer la distance réelle entre la buse et le lit à l'emplacement donné. Une fois ces étapes terminées, vous pouvez "ACCEPTER" la position et enregistrer les résultats dans le fichier de configuration avec:
```
SAVE_CONFIG
```

Notez que si une modification est apportée au système de mouvement de l'imprimante, à la position de la hotend ou à l'emplacement de la sonde, cela invalidera les résultats de PROBE_CALIBRATE.

Si la sonde a un décalage X ou Y et que l'inclinaison du lit est modifiée (par exemple, en ajustant les vis du lit, en exécutant DELTA_CALIBRATE, en exécutant Z_TILT_ADJUST, en exécutant QUAD_GANTRY_LEVEL, ou similaire), les résultats de PROBE_CALIBRATE seront invalidés. Après avoir effectué l'un des ajustements ci-dessus, il sera nécessaire d'exécuter à nouveau PROBE_CALIBRATE.

Si les résultats de PROBE_CALIBRATE sont invalidés, alors tous les résultats précédents [bed mesh](Bed_Mesh.md) qui ont été obtenus à l'aide de la sonde sont également invalidés - il sera nécessaire de réexécuter BED_MESH_CALIBRATE après avoir recalibré la sonde.

## Contrôle de répétabilité

Après avoir calibré les décalages X, Y et Z de la sonde, il est conseillé de vérifier que la sonde fournit des résultats reproductibles. Commencez par orienter l'imprimante, puis déplacez la tête vers une position proche du centre du lit. Accédez à l'onglet du terminal OctoPrint et exécutez la commande "PROBE_ACCURACY".

Cette commande exécutera la sonde dix fois et produira une sortie similaire à la suivante:
```
Recv: // probe accuracy: at X:0.000 Y:0.000 Z:10.000
Recv: // and read 10 times with speed of 5 mm/s
Recv: // probe at -0.003,0.005 is z=2.506948
Recv: // probe at -0.003,0.005 is z=2.519448
Recv: // probe at -0.003,0.005 is z=2.519448
Recv: // probe at -0.003,0.005 is z=2.506948
Recv: // probe at -0.003,0.005 is z=2.519448
Recv: // probe at -0.003,0.005 is z=2.519448
Recv: // probe at -0.003,0.005 is z=2.506948
Recv: // probe at -0.003,0.005 is z=2.506948
Recv: // probe at -0.003,0.005 is z=2.519448
Recv: // probe at -0.003,0.005 is z=2.506948
Recv: // probe accuracy results: maximum 2.519448, minimum 2.506948, range 0.012500, average 2.513198, median 2.513198, standard deviation 0.006250
```

Idéalement, l'outil rapportera une valeur maximale et minimale identique. (C'est-à-dire que, idéalement, la sonde obtient un résultat identique sur les dix sondes.) Cependant, il est normal que les valeurs minimale et maximale diffèrent d'une "distance de pas" Z ou jusqu'à 5 microns (0,005 mm). Une "distance de pas" est `distance_rotation/(full_steps_per_rotation*microsteps)`. La distance entre la valeur minimale et la valeur maximale s'appelle la plage. Ainsi, dans l'exemple ci-dessus, puisque l'imprimante utilise une distance de pas Z de 0,0125, une plage de 0,012500 serait considérée comme normale.

Si les résultats du test indiquent une valeur de plage supérieure à 25 microns (0,025 mm), la sonde n'a pas une précision suffisante pour les procédures de nivellement de lit typiques. Il peut être possible de régler la vitesse de la sonde et/ou la hauteur de départ de la sonde pour améliorer la répétabilité de la sonde. La commande `PROBE_ACCURACY` permet d'exécuter des tests avec différents paramètres pour voir leur impact - voir le [document G-Codes](G-Codes.md#probe_accuracy) pour plus de détails. Si la sonde obtient généralement des résultats reproductibles mais présente une valeur aberrante occasionnelle, il peut être possible d'en tenir compte en utilisant plusieurs échantillons sur chaque sonde - lisez la description des paramètres de configuration des « échantillons » de la sonde dans le [référence de configuration] (Config_Reference. md#probe) pour plus de détails.

Si une nouvelle vitesse de sonde, un nouveau nombre d'échantillons ou d'autres paramètres sont nécessaires, mettez à jour le fichier printer.cfg et lancez une commande `RESTART`. Si c'est le cas, c'est une bonne idée de
[calibrating the z_offset](#calibrating-probe-z-offset) à nouveau. Si des résultats reproductibles ne peuvent pas être obtenus, n'utilisez pas la sonde pour le nivellement du lit. Klipper dispose de plusieurs outils de sondage manuels qui peuvent être utilisés à la place - voir le [document Bed Level](Bed_Level.md) pour plus de détails.

## Vérification du biais de localisation

Certaines sondes peuvent avoir un biais systémique qui corrompt les résultats de la sonde à certains emplacements de la tête d'outil. Par exemple, si le support de la sonde s'incline légèrement lorsqu'il se déplace le long de l'axe Y, la sonde peut alors signaler des résultats biaisés à différentes positions Y.

Il s'agit d'un problème courant avec les sondes sur les imprimantes delta, mais il peut se produire sur toutes les imprimantes.

On peut vérifier un biais d'emplacement en utilisant la commande "PROBE_CALIBRATE" pour mesurer le décalage z_offset de la sonde à divers emplacements X et Y. Idéalement, la sonde z_offset serait une valeur constante à chaque emplacement d'imprimante.

Pour les imprimantes delta, essayez de mesurer le décalage z à une position proche de la tour A, à une position proche de la tour B et à une position proche de la tour C. Pour les imprimantes cartésiennes, corexy et similaires, essayez de mesurer le z_offset à des positions proches des quatre coins du lit.

Avant de commencer ce test, calibrez d'abord les décalages X, Y et Z de la sonde comme décrit au début de ce document. Remettez ensuite l'imprimante à l'origine et naviguez jusqu'à la première position XY. Suivez les étapes de [calibrating probe Z offset](#calibrating-probe-z-offset) pour exécuter la commande `PROBE_CALIBRATE`, les commandes `TESTZ` et la commande `ACCEPT`, mais n'exécutez pas `SAVE_CONFIG`. Notez le rapport z_offset trouvé. Naviguez ensuite vers les autres positions XY, répétez ces étapes "PROBE_CALIBRATE" et notez le décalage z signalé.

Si la différence entre le décalage z_offset minimum signalé et le décalage z_maximum signalé est supérieure à 25 microns (0,025 mm), la sonde n'est pas adaptée aux procédures de nivellement de lit typiques. Voir le [document Bed Level](Bed_Level.md) pour les alternatives de sonde manuelle.

## Biais de température

De nombreuses sondes ont un biais systémique lorsqu'elles sondent à différentes températures. Par exemple, la sonde peut se déclencher systématiquement à une hauteur inférieure lorsque la sonde est à une température plus élevée.

Il est recommandé de faire fonctionner les outils de nivellement du lit à une température constante pour tenir compte de ce biais. Par exemple, exécutez toujours les outils lorsque l'imprimante est à température ambiante ou exécutez toujours les outils une fois que l'imprimante a obtenu une température d'impression constante. Dans les deux cas, c'est une bonne idée d'attendre plusieurs minutes après que la température souhaitée est atteinte, de sorte que l'appareil d'impression soit constamment à la température souhaitée.

Pour vérifier un biais de température, commencez avec l'imprimante à température ambiante, puis placez l'imprimante à l'origine, déplacez la tête vers une position proche du centre du lit et exécutez la commande "PROBE_ACCURACY". Notez les résultats. Ensuite, sans rechercher ou désactiver les moteurs pas à pas, chauffez la buse et le lit de l'imprimante à la température d'impression, puis exécutez à nouveau la commande "PROBE_ACCURACY". Idéalement, la commande rapportera des résultats identiques. Comme ci-dessus, si la sonde a un biais de température, veillez à toujours utiliser la sonde à une température constante.
