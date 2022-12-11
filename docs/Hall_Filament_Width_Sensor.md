# Capteur de largeur de filament Hall

Ce document décrit le module hôte du capteur de largeur de filament. Le matériel utilisé pour développer ce module hôte est basé sur deux capteurs linéaires Hall (ss49e par exemple). Les capteurs dans le corps sont situés sur les côtés opposés. Principe de fonctionnement : deux capteurs hall fonctionnent en mode différentiel, dérive de température identique pour le capteur. Compensation de température spéciale non nécessaire.

Vous pouvez trouver des designs sur [Thingiverse](https://www.thingiverse.com/thing:4138933), une vidéo d'assemblage est également disponible sur [Youtube](https://www.youtube.com/watch?v=TDO9tME8vp4 )

Pour utiliser le capteur de largeur de filament Hall, lisez [Config Reference](Config_Reference.md#hall_filament_width_sensor) et [G-Code documentation](G-Codes.md#hall_filament_width_sensor).

## Comment ça marche?

Le capteur génère deux sorties analogiques basées sur la largeur de filament calculée. La somme de la tension de sortie est toujours égale à la largeur de filament détectée. Le module hôte surveille les changements de tension et ajuste le multiplicateur d'extrusion. J'utilise le connecteur aux2 sur les broches analogiques11 et analogiques12 de la carte de type rampes. Vous pouvez utiliser différentes broches et différentes cartes.

## Modèle pour les variables de menu

```
[menu __main __filament __width_current]
type: command
enable: {'hall_filament_width_sensor' in printer}
name: Dia: {'%.2F' % printer.hall_filament_width_sensor.Diameter}
index: 0

[menu __main __filament __raw_width_current]
type: command
enable: {'hall_filament_width_sensor' in printer}
name: Raw: {'%4.0F' % printer.hall_filament_width_sensor.Raw}
index: 1
```

## Procédure de calibrage

Pour obtenir la valeur brute du capteur, vous pouvez utiliser l'élément de menu ou la commande **QUERY_RAW_FILAMENT_WIDTH** dans le terminal.

1. Insérez la première tige d'étalonnage (taille de 1,5 mm) pour obtenir la première valeur brute du capteur

2. Insérez la deuxième tige d'étalonnage (taille de 2,0 mm) pour obtenir la deuxième valeur brute du capteur

3. Enregistrez les valeurs brutes du capteur dans les paramètres de configuration `Raw_dia1` et `Raw_dia2`

## Comment activer le capteur

Par défaut, le capteur est désactivé à la mise sous tension.

Pour activer le capteur, lancez la commande **ENABLE_FILAMENT_WIDTH_SENSOR** ou définissez le paramètre "enable" sur "true".

## Journalisation

Par défaut, l'enregistrement du diamètre est désactivé à la mise sous tension. Exécutez la commande **ENABLE_FILAMENT_WIDTH_LOG** pour démarrer la journalisation et émettre
Commande **DISABLE_FILAMENT_WIDTH_LOG** pour arrêter la journalisation. Pour activer la journalisation
à la mise sous tension, définissez le paramètre `logging` sur `true`.

Le diamètre du filament est enregistré à chaque intervalle de mesure (10 mm par défaut).
