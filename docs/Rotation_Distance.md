# Rotation distance

Les pilotes de moteur pas à pas sur Klipper nécessitent un `rotation_distance` paramètre dans chaque [stepper config section](Config_Reference.md#stepper). La `rotation_distance` est la distance parcourue par l'axe avec un tour complet du moteur pas à pas. Ce document décrit comment on peut configurer cette valeur.

## Obtention de rotation_distance à partir de steps_per_mm (ou step_distance)

Les concepteurs de votre imprimante 3d ont initialement calculé `steps_per_mm` à partir d'une distance de rotation. Si vous connaissez les steps_per_mm, il est alors possible d'utiliser cette formule générale pour obtenir cette distance de rotation d'origine:
```
rotation_distance = <full_steps_per_rotation> * <microsteps> / <steps_per_mm>
```

Ou, si vous avez une configuration Klipper plus ancienne et connaissez le paramètre `step_distance`, vous pouvez utiliser cette formule:
```
rotation_distance = <full_steps_per_rotation> * <microsteps> * <step_distance>
```

Le paramètre `<full_steps_per_rotation>` est déterminé à partir du type de moteur pas à pas. La plupart des moteurs pas à pas sont des "pas à pas de 1,8 degré" et ont donc 200 pas complets par rotation (360 divisé par 1,8 est 200). Certains moteurs pas à pas sont des "pas à pas de 0,9 degré" et ont donc 400 pas complets par rotation. Les autres moteurs pas à pas sont rares. En cas de doute, ne définissez pas full_steps_per_rotation dans le fichier de configuration et utilisez 200 dans la formule ci-dessus.

Le paramètre `<microsteps>` est déterminé par le pilote du moteur pas à pas. La plupart des pilotes utilisent 16 micropas. En cas de doute, définissez `microsteps : 16` dans la configuration et utilisez 16 dans la formule ci-dessus.

Presque toutes les imprimantes doivent avoir un nombre entier pour `rotation_distance` sur les axes de type X, Y et Z. Si la formule ci-dessus donne une rotation_distance inférieure à 0,01 d'un nombre entier, arrondissez la valeur finale à ce nombre entier.

## Étalonnage rotation_distance sur les extrudeuses

Sur une extrudeuse, le`rotation_distance` est la distance parcourue par le filament pour une rotation complète du moteur pas à pas. La meilleure façon d'obtenir une valeur précise pour ce paramètre est d'utiliser la procédure  "measure and trim".

Commencez d'abord par une estimation initiale de la distance de rotation. Celui-ci peut être obtenu auprès de [steps_per_mm](#obtaining-rotation_distance-from-steps_per_mm-or-step_distance) ou par [inspecting the hardware](#extruder).

Utilisez ensuite la procédure suivante pour "measure and trim":
1. Assurez-vous que l'extrudeuse contient du filament, que la hotend est chauffée à une température appropriée et que l'imprimante est prête à extruder.
2. Utilisez un marqueur pour placer une marque sur le filament à environ 70 mm de l'entrée du corps de l'extrudeur. Utilisez ensuite un pied à coulisse numérique pour mesurer la distance réelle de cette marque aussi précisément que possible. Notez ceci comme `<initial_mark_distance>`.
3. Extrudez 50 mm de filament avec la séquence de commande suivante : 'G91' suivi de 'G1 E50 F60'. Notez 50 mm comme `<requested_extrude_distance>`. Attendez que l'extrudeuse termine le mouvement (cela prendra environ 50 secondes). Il est important d'utiliser la vitesse d'extrusion lente pour ce test, car une vitesse plus rapide peut provoquer une pression élevée dans l'extrudeuse, ce qui faussera les résultats. (N'utilisez pas le "bouton d'extrusion" sur les frontaux graphiques pour ce test car ils s'extrudent à un rythme rapide.)
4. Utilisez les pieds à coulisse numériques pour mesurer la nouvelle distance entre le corps de l'extrudeuse et la marque sur le filament. Notez ceci comme `<subsequent_mark_distance>`. Calculez ensuite : `actual_extrude_distance = <initial_mark_distance> - <subsequent_mark_distance>`
5. Calculez rotation_distance comme : `rotation_distance = <previous_rotation_distance> * <actual_extrude_distance> / <requested_extrude_distance>` Arrondissez la nouvelle distance_rotation à trois décimales.

Si le actual_extrude_distance diffère de requested_extrude_distance de plus d'environ 2 mm, il est conseillé d'effectuer les étapes ci-dessus une deuxième fois.

Remarque : N'utilisez *pas* un "measure and trim" type de méthode pour calibrer les axes de type x, y ou z. La méthode "measure and trim" n'est pas assez précise pour ces axes et conduira probablement à une configuration plus mauvaise. Au lieu de cela, si nécessaire, ces axes peuvent être déterminés par [measuring the belts, pulleys, and lead screw hardware](#obtaining-rotation_distance-by-inspecting-the-hardware).

## Obtention de la distance de rotation en inspectant le matériel

Il est possible de calculer rotation_distance en connaissant les moteurs pas à pas et la cinématique de l'imprimante. Cela peut être utile si le steps_per_mm n'est pas connu ou si vous concevez une nouvelle imprimante.

### Axes entraînés par courroie

Il est facile de calculer rotation_distance pour un axe linéaire qui utilise une courroie et une poulie.

Déterminez d'abord le type de ceinture. La plupart des imprimantes utilisent un pas de courroie de 2 mm (c'est-à-dire que chaque dent de la courroie est espacée de 2 mm). Comptez ensuite le nombre de dents de la poulie du moteur pas à pas. La rotation_distance est alors calculée comme :
```
rotation_distance = <belt_pitch> * <number_of_teeth_on_pulley>
```

Par exemple, si une imprimante a une courroie de 2 mm et utilise une poulie à 20 dents, la distance de rotation est de 40.

### Axes avec une vis mère

Il est facile de calculer la rotation_distance pour les vis-mères courantes à l'aide de la formule suivante: 
```
rotation_distance = <screw_pitch> * <number_of_separate_threads>
```

Par exemple, la "vis mère T8" commune a une distance de rotation de 8 (il a un pas de 2mm et possède 4 filetages séparés).

Les imprimantes plus anciennes avec des "tiges filetées" n'ont qu'un seul "filetage" sur la vis mère et donc la distance de rotation est le pas de la vis. (Le pas de vis est la distance entre chaque rainure sur la vis.) Ainsi, par exemple, une tige métrique M6 a une distance de rotation de 1 et une tige M8 a une distance de rotation de 1,25.

### Extrudeur

Il est possible d'obtenir une distance de rotation initiale pour les extrudeuses en mesurant le diamètre du "boulon taré" qui pousse le filament et en utilisant la formule suivante: `rotation_distance = <diameter> * 3.14`

Si l'extrudeuse utilise des engrenages, il sera également nécessaire de [déterminer et définir le gear_ratio] (#using-a-gear_ratio) pour l'extrudeuse.

La distance de rotation réelle sur une extrudeuse varie d'une imprimante à l'autre, car l'adhérence du "boulon crénelé" qui engage le filament peut varier. Il peut même varier entre les bobines de filament. Après avoir obtenu une rotation_distance initiale, utilisez la [procédure de mesure et de coupe](#calibrating-rotation_distance-on-extruders) pour obtenir un réglage plus précis.

## Utilisant un gear_ratio

La définition d'un `gear_ratio` peut faciliter la configuration de la `rotation_distance` sur les steppers auxquels est attachée une boîte de vitesses (ou similaire). La plupart des steppers n'ont pas de boîte de vitesses - en cas de doute, ne définissez pas `gear_ratio` dans la configuration.

Lorsque `gear_ratio` est défini, la `rotation_distance` représente la distance parcourue par l'axe avec une rotation complète de l'engrenage final sur la boîte de vitesses. Si, par exemple, on utilise une boîte de vitesses avec un rapport "5: 1", alors on pourrait calculer la rotation_distance avec [knowledge of the hardware](#obtaining-rotation_distance-by-inspecting-the-hardware) puis ajoutez `gear_ratio: 5:1` à la configuration.

Pour les engrenages mis en œuvre avec des courroies et des poulies, il est possible de déterminer le gear_ratio en comptant les dents sur les poulies. Par exemple, si un moteur pas à pas avec une poulie à 16 dents entraîne la poulie suivante à 80 dents, on utilisera `gear_ratio: 80:16`. En effet, on pourrait ouvrir une "boîte de vitesses" courante et compter les dents qu'elle contient pour confirmer son rapport de démultiplication.

Notez que parfois une boîte de vitesses aura un rapport de démultiplication légèrement différent de celui sous lequel elle est annoncée. Les engrenages de moteur d'extrudeuse BMG courants en sont un exemple - ils sont annoncés comme "3: 1" mais utilisent en fait un engrenage "50:17". (L'utilisation de nombres de dents sans dénominateur commun peut améliorer l'usure globale des engrenages car les dents ne s'engrènent pas toujours de la même manière à chaque révolution.) `.

Si plusieurs engrenages sont utilisés sur un axe, il est alors possible de fournir une liste séparée par des virgules à gear_ratio. Par exemple, une boîte de vitesses "5: 1" entraînant une poulie de 16 à 80 dents pourrait utiliser `gear_ratio: 5: 1, 80: 16`.

Dans la plupart des cas, gear_ratio doit être défini avec des nombres entiers car les engrenages et poulies courants ont un nombre entier de dents. Cependant, dans les cas où une courroie entraîne une poulie en utilisant la friction au lieu des dents, il peut être judicieux d'utiliser un nombre à virgule flottante dans le rapport d'engrenage (par exemple, `gear_ratio: 107.237:16`).
