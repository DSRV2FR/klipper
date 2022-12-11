# TSL1401CL capteur de largeur de filament

Ce document décrit le module hôte du capteur de largeur de filament. Matériel utilisé pour développer ce module hôte est basé sur le réseau de capteurs linéaires TSL1401CL mais il peut fonctionner avec n'importe quel réseau de capteurs doté d'une sortie analogique. Tu peux trouver conceptions sur [Thingiverse](https://www.thingiverse.com/search?q=filament%20width%20sensor).

Pour utiliser un réseau de capteurs comme capteur de largeur de filament, lisez
[Référence de configuration] (Config_Reference.md#tsl1401cl_filament_width_sensor) et [Documentation G-Code] (G-Codes.md#hall_filament_width_sensor).

## Comment ça marche?

Le capteur génère une sortie analogique basée sur la largeur de filament calculée. Production la tension est toujours égale à la largeur de filament détectée (Ex. 1.65v, 1.70v, 3.0v). Le module hôte surveille les changements de tension et ajuste le multiplicateur d'extrusion.

## Note:
Lectures de capteur effectuées avec des intervalles de 10 mm par défaut. Si nécessaire, vous êtes libre de modifier ce paramètre en éditant le paramètre ***MEASUREMENT_INTERVAL_MM*** dans le fichier **filament_width_sensor.py**.
