# Utilisation des outils PWM

Ce document décrit comment configurer un laser ou une broche contrôlé par PWM à l'aide de "output_pin" et de certaines macros.

## Comment ça marche?

En réaffectant la sortie pwm du ventilateur de la tête d'impression, vous pouvez contrôler les lasers ou les broches. Ceci est utile si vous utilisez des têtes d'impression commutables, par exemple le changeur d'outils E3D ou une solution de bricolage. Habituellement, les outils de came tels que LaserWeb peuvent être configurés pour utiliser les commandes "M3-M5", qui signifient _spindle speed CW_ (`M3 S[0-255]`), _spindle speed CCW_ (`M4 S[0-255]`) et _spindle stop_ (`M5`).

**Avertissement :** Lorsque vous conduisez un laser, respectez toutes les précautions de sécurité auxquelles vous pouvez penser ! Les lasers à diode sont généralement inversés. Cela signifie que lorsque le MCU redémarre, le laser sera _entièrement allumé_ pendant le temps nécessaire au MCU pour redémarrer. Pour faire bonne mesure, il est recommandé de porter _toujours_ des lunettes laser appropriées de la bonne longueur d'onde si le laser est alimenté ; et de déconnecter le laser lorsqu'il n'est pas nécessaire. En outre, vous devez configurer un délai d'expiration de sécurité, de sorte que lorsque votre hôte ou MCU rencontre une erreur, l'outil s'arrête.

Pour un exemple de configuration, voir  [config/sample-pwm-tool.cfg](/config/sample-pwm-tool.cfg).

## Limites actuelles

Il existe une limitation de la fréquence des mises à jour PWM. Tout en étant très précis, une mise à jour PWM ne peut se produire que toutes les 0,1 secondes, ce qui la rend presque inutile pour la gravure raster. Cependant, il existe un [experimental branch](https://github.com/Cirromulus/klipper/tree/laser_tool) avec ses propres compromis. À long terme, il est prévu d'ajouter cette fonctionnalité au klipper de la ligne principale.

## Commandes

`M3/M4 S<value>` : Set PWM duty-cycle. Values between 0 and 255.
`M5` : Stop PWM output to shutdown value.

## Configuration Laser Web

Si vous utilisez Laserweb, une configuration de travail serait :

    GCODE START:
        M5            ; Disable Laser
        G21           ; Set units to mm
        G90           ; Absolute positioning
        G0 Z0 F7000   ; Set Non-Cutting speed

    GCODE END:
        M5            ; Disable Laser
        G91           ; relative
        G0 Z+20 F4000 ;
        G90           ; absolute

    GCODE HOMING:
        M5            ; Disable Laser
        G28           ; Home all axis

    TOOL ON:
        M3 $INTENSITY

    TOOL OFF:
        M5            ; Disable Laser

    LASER INTENSITY:
        S
