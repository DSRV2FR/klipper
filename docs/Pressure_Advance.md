# Pressure advance

Ce document fournit des informations sur le réglage de la variable de configuration « avance de pression » pour une buse et un filament particuliers. La fonction d'avance de pression peut être utile pour réduire le suintement. Pour plus d'informations sur la façon dont l'avance de pression est mise en œuvre, voir le document [kinematics](Kinematics.md).

## Réglage pressure advance

Pressure advance fait deux choses utiles - il réduit le suintement pendant les mouvements sans extrusion et il réduit les bavures pendant les virages. Ce guide utilise la deuxième fonctionnalité (réduction du blobbing dans les virages) comme mécanisme de réglage.

In order to calibrate pressure advance the printer must be configured and operational as the tuning test involves printing and inspecting a test object. It is a good idea to read this document in full prior to running the test.

Utilisez un trancheur pour générer le g-code pour le grand carré creux trouvé dans [docs/prints/square_tower.stl](prints/square_tower.stl). Utilisez une vitesse élevée (par exemple, 100 mm/s), un remplissage nul et une hauteur de couche grossière (la hauteur de couche doit être d'environ 75 % du diamètre de la buse). Assurez-vous que tout "contrôle d'accélération dynamique" est désactivé dans le slicer.

Préparez-vous pour le test en émettant la commande G-Code suivante:
```
SET_VELOCITY_LIMIT SQUARE_CORNER_VELOCITY=1 ACCEL=500
```
Cette commande ralentit le déplacement de la buse dans les coins pour accentuer les effets de la pression de l'extrudeuse. Ensuite, pour les imprimantes avec une extrudeuse à entraînement direct, exécutez la commande:
```
TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0 FACTOR=.005
```
Pour les extrudeuses Bowden longues, utilisez:
```
TUNING_TOWER COMMAND=SET_PRESSURE_ADVANCE PARAMETER=ADVANCE START=0 FACTOR=.020
```
Puis imprimez l'objet. Une fois entièrement imprimé, le test d'impression ressemble à :

![tuning_tower](img/tuning_tower.jpg)

La commande TUNING_TOWER ci-dessus demande à Klipper de modifier le paramètre pressure_advance sur chaque couche de l'impression. Les couches supérieures de l'impression auront une valeur d'avance de pression plus élevée. Les calques en dessous du réglage de pression_avance idéal auront des taches dans les coins, et les calques au-dessus du réglage idéal peuvent conduire à des coins arrondis et à une mauvaise extrusion menant au coin.

On peut annuler l'impression plus tôt si l'on constate que les coins ne s'impriment plus bien (et ainsi on peut éviter d'imprimer des couches dont on sait qu'elles sont au-dessus de la valeur idéale de pression_avance).

Inspectez l'impression, puis utilisez un pied à coulisse numérique pour trouver la hauteur qui a les meilleurs coins de qualité. En cas de doute, préférez une hauteur inférieure.

![tune_pa](img/tune_pa.jpg)

La valeur pressure_advance peut alors être calculée comme `pressure_advance = <start> + <measured_height> * <factor>`. (For example, `0 + 12.90 * .020` would be `.258`.)

Il est possible de choisir des réglages personnalisés pour START et FACTOR si cela permet d'identifier le meilleur réglage d'avance de pression. Ce faisant, assurez-vous d'émettre la commande TUNING_TOWER au début de chaque test d'impression.

Les valeurs d'avance de pression typiques sont comprises entre 0,050 et 1,000 (le haut de gamme généralement uniquement avec les extrudeuses Bowden). S'il n'y a pas d'amélioration significative avec une avance de pression jusqu'à 1 000, il est peu probable que l'avance de pression améliore la qualité des impressions. Retour à une configuration par défaut avec avance de pression désactivée.

Bien que cet exercice de réglage améliore directement la qualité des coins, il convient de rappeler qu'une bonne configuration d'avance de pression réduit également le suintement tout au long de l'impression.

A la fin de ce test, réglez `pressure_advance = <calculated_value>` dans la section `[extruder]` du fichier de configuration et émettez une commande RESTART. La commande RESTART effacera l'état de test et renverra l'accélération et vitesses de virage à leurs valeurs normales.

## Notes IMPORTANTES

* La valeur d'avance de pression dépend de l'extrudeuse, de la buse et du filament. Il est courant que les filaments de différents fabricants ou avec différents pigments nécessitent des valeurs d'avance de pression très différentes. Par conséquent, il faut calibrer l'avance de pression sur chaque imprimante et avec chaque bobine de filament.

* La température d'impression et les taux d'extrusion peuvent avoir un impact sur l'avance de pression. Assurez-vous d'accorder le [extruder rotation_distance](Rotation_Distance.md#calibrating-rotation_distance-on-extruders) et [nozzle temperature](http://reprap.org/wiki/Triffid_Hunter%27s_Calibration_Guide#Nozzle_Temperature) avant de régler l'avance de pression.

* L'impression de test est conçue pour fonctionner avec un débit d'extrudeuse élevé, mais sinon avec des paramètres de trancheuse "normaux". Un débit élevé est obtenu en utilisant une vitesse d'impression élevée (par exemple, 100 mm/s) et une hauteur de couche grossière (typiquement autour de 75 % du diamètre de la buse). Les autres paramètres du slicer doivent être similaires à leurs valeurs par défaut (par exemple, périmètres de 2 ou 3 lignes, quantité de rétraction normale). Il peut être utile de régler la vitesse du périmètre externe sur la même vitesse que le reste de l'impression, mais ce n'est pas obligatoire.

* Il est courant que le test d'impression montre un comportement différent sur chaque coin. Souvent, le trancheur s'arrangera pour changer les couches à un coin, ce qui peut entraîner une différence significative entre ce coin et les trois coins restants. Si cela se produit, ignorez ce coin et réglez l'avance de la pression en utilisant les trois autres coins. Il est également courant que les coins restants varient légèrement. (Cela peut se produire en raison de petites différences dans la façon dont le cadre de l'imprimante réagit aux virages dans certaines directions.) Essayez de choisir une valeur qui fonctionne bien pour tous les coins restants. En cas de doute, préférez une valeur d'avance à la pression inférieure.

* Si une valeur d'avance de pression élevée (par exemple, supérieure à 0,200) est utilisée, il se peut que l'extrudeuse saute lors du retour à l'accélération normale de l'imprimante. Le système d'avance de pression tient compte de la pression en poussant un filament supplémentaire pendant l'accélération et en rétractant ce filament pendant la décélération. Avec une accélération élevée et une avance à haute pression, l'extrudeuse peut ne pas avoir assez de couple pour pousser le filament requis. Si cela se produit, utilisez une valeur d'accélération inférieure ou désactivez l'avance de pression.

* Une fois que l'avance de pression est réglée dans Klipper, il peut toujours être utile de configurer une petite valeur de rétraction dans la trancheuse (par exemple, 0,75 mm) et d'utiliser l'option "essuyer lors de la rétraction" de la trancheuse si elle est disponible. Ces réglages de trancheuse peuvent aider à contrecarrer le suintement causé par la cohésion du filament (filament retiré de la buse en raison de l'adhérence du plastique). Il est recommandé de désactiver l'option "z-lift on retract" du slicer.

* Le système d'avance de pression ne modifie pas la synchronisation ou la trajectoire de la tête d'outil. Une impression avec l'avance de pression activée prendra le même temps qu'une impression sans avance de pression. L'avance de pression ne modifie pas non plus la quantité totale de filament extrudé lors d'une impression. L'avance de pression entraîne un mouvement supplémentaire de l'extrudeuse pendant l'accélération et la décélération du mouvement. Un réglage d'avance de pression très élevé entraînera une très grande quantité de mouvement de l'extrudeuse pendant l'accélération et la décélération, et aucun réglage de configuration ne limite la quantité de ce mouvement.
