# trancheuses

Ce document fournit quelques conseils pour configurer une application "slicer" à utiliser avec Klipper. Les slicers couramment utilisés avec Klipper sont Slic3r, Cura, Simplify3D, etc.

## Définissez la saveur du G-Code sur Marlin

De nombreux slicers ont une option pour configurer le "G-Code flavor". La valeur par défaut est fréquemment "Marlin" et cela fonctionne bien avec Klipper. La "Smoothieware" le réglage fonctionne également bien avec Klipper.

## Klipper gcode_macro

Les trancheuses permettent souvent de configurer "Start G-Code" et "End G-Code" séquences. Il est souvent pratique de définir à la place des macros personnalisées dans le fichier de configuration de Klipper, telles que : `[gcode_macro START_PRINT]` et `[gcode_macro END_PRINT]`. Ensuite, on peut simplement exécuter START_PRINT et END_PRINT dans la configuration du slicer. La définition de ces actions dans la configuration de Klipper peut faciliter la modification des étapes de début et de fin de l'imprimante, car les modifications ne nécessitent pas de nouveau découpage.

Voir [sample-macros.cfg](../config/sample-macros.cfg) fou des exemples de macros START_PRINT et END_PRINT.

Voir le [config reference](Config_Reference.md#gcode_macro) pour plus de détails sur la définition d'un gcode_macro.

## Les réglages de rétraction importants peuvent nécessiter un réglage de Klipper

La vitesse et l'accélération maximales des mouvements de rétraction sont contrôlées dans Klipper par le `max_extrude_only_velocity` et `max_extrude_only_accel` paramètres de configuration. Ces paramètres ont une valeur par défaut qui devrait bien fonctionner sur de nombreuses imprimantes. Cependant, si l'on a configuré une grande rétraction dans la trancheuse (par exemple, 5 mm ou plus), on peut constater qu'ils limitent la vitesse de rétraction souhaitée.

Si vous utilisez une grande rétraction, envisagez de régler le Klipper [pressure advance](Pressure_Advance.md) Au lieu. Sinon, si l'on trouve que la tête d'outil semble "pause" pendant la rétraction et l'amorçage, envisagez alors de définir explicitement `max_extrude_only_velocity` et `max_extrude_only_accel` dans le fichier de configuration de Klipper.

## Ne pas activer le "coasting"

La fonction "coasting" est susceptible d'entraîner des impressions de mauvaise qualité avec Klipper. Envisagez d'utiliser Klipper [pressure advance](Pressure_Advance.md) au lieu.

Plus précisément, si le slicer modifie considérablement le taux d'extrusion entre les mouvements, Klipper effectuera une décélération et une accélération entre les mouvements. Cela risque d'aggraver le blobbing, pas de l'améliorer.

En revanche, il est acceptable (et souvent utile) d'utiliser une trancheuse "retract" paramètre, "wipe" setting, et/ou "wipe sur retract" paramètre.

## N'utilisez pas la "distance de redémarrage supplémentaire" sur Simplify3d

Ce paramètre peut entraîner des changements spectaculaires dans les taux d'extrusion, ce qui peut déclencher la vérification de la section d'extrusion maximale de Klipper. Envisagez d'utiliser Klipper [pressure advance](Pressure_Advance.md) ou le paramètre de retrait Simplify3d standard à la place.

## Désactiver "PreloadVE" sur KISSlicer

Si vous utilisez le logiciel de découpage KISSlicer, réglez "PreloadVE" sur zéro. Envisagez d'utiliser Klipper [pressure advance](Pressure_Advance.md) Au lieu.

## Désactivez tous les paramètres de "pression d'extrudeuse avancée"

Certaines trancheuses annoncent un "advanced extruder pressure" aptitude. Il est recommandé de garder ces options désactivées lors de l'utilisation de Klipper car elles risquent d'entraîner des impressions de mauvaise qualité. Envisagez d'utiliser Klipper [pressure advance](Pressure_Advance.md) Au lieu.

Plus précisément, ces paramètres de trancheuse peuvent demander au micrologiciel d'apporter des modifications sauvages au taux d'extrusion dans l'espoir que le micrologiciel se rapprochera de ces demandes et que l'imprimante obtiendra approximativement une pression d'extrudeuse souhaitable. Klipper, cependant, utilise des calculs cinématiques et une synchronisation précis. Lorsque Klipper reçoit l'ordre d'apporter des modifications importantes au taux d'extrusion, il planifiera les modifications correspondantes de la vitesse, de l'accélération et de l'extrudeuse.
mouvement - ce qui n'est pas l'intention de la trancheuse. La trancheuse peut même commander des taux d'extrusion excessifs au point de déclencher la vérification de la section d'extrusion maximale de Klipper.

En revanche, il est acceptable (et souvent utile) d'utiliser une trancheuse "retract" paramètre, "wipe" setting, et/ou "wipe on retract" paramètre.
