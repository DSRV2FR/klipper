# Maille de lit (bed mesh)

Le module Bed Mesh peut être utilisé pour compenser les irrégularités de la surface du lit afin d'obtenir une meilleure première couche sur l'ensemble du lit. Il convient de noter que la correction basée sur un logiciel n'atteindra pas des résultats parfaits, elle ne peut qu'approximer la forme du lit. Bed Mesh ne peut pas non plus compenser les problèmes mécaniques et électriques. Si un axe est de travers ou qu'un palpeur n'est pas
précis, le module bed_mesh ne recevra pas de résultats précis du processus de sondage.
 
Avant l'étalonnage du maillage, vous devez vous assurer que le décalage Z de votre sonde est calibré. Si vous utilisez une butée pour la prise d'origine Z, elle devra également être calibrée. Voir [Probe Calibrate](Probe_Calibrate.md) et Z_ENDSTOP_CALIBRATE dans [Manual Level](Manual_Level.md) pour plus d'informations.

##Configuration de base

### Lits rectangulaires
Cet exemple suppose une imprimante avec un lit rectangulaire de 250 mm x 220 mm et une sonde avec un décalage x de 24 mm et un décalage y de 5 mm.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
```

- `speed: 120`\
  _Default Value: 50_\
  La vitesse à laquelle l'outil se déplace entre les points.

- `horizontal_move_z: 5`\
  _Default Value: 5_\
  La coordonnée Z à laquelle la sonde s'élève avant de se déplacer entre les points.

- `mesh_min: 35, 6`\
  _Required_\
  La première coordonnée palpée, la plus proche de l'origine. Cette coordonnée est relative à l'emplacement de la sonde.

- `mesh_max: 240, 198`\
  _Required_\
  La coordonnée sondée la plus éloignée de l'origine. Ce n'est pas nécessairement le dernier point sondé, car le processus de sondage se déroule en zigzag. Comme avec `mesh_min`, cette coordonnée est relative à l'emplacement de la sonde.

- `probe_count: 5, 3`\
  _Default Value: 3, 3_\
  Le nombre de points à sonder sur chaque axe, spécifié sous forme de valeurs entières X, Y. Dans cet exemple, 5 points seront sondés le long de l'axe X, avec 3 points le long de l'axe Y, pour un total de 15 points sondés. Notez que si vous vouliez une grille carrée, par exemple 3x3, cela pourrait être spécifié comme une seule valeur entière qui est utilisée pour les deux axes, c'est-à-dire `probe_count: 3`.
  Notez qu'un maillage nécessite un minimum de probe_count de 3 le long de chaque axe.

L'illustration ci-dessous montre comment les options `mesh_min`, `mesh_max` et `probe_count` sont utilisées pour générer des points de sonde. Les flèches indiquent la direction de la procédure de sondage, en commençant à 'mesh_min'. Pour référence, lorsque la sonde est à "mesh_min", la buse sera à (11, 1), et lorsque la sonde est à "mesh_max", la buse sera à (206, 193).

![bedmesh_rect_basic](img/bedmesh_rect_basic.svg)

### Lits ronds
Cet exemple suppose une imprimante équipée d'un rayon de lit rond de 100 mm. Nous utiliserons les mêmes décalages de sonde que l'exemple rectangulaire, 24 mm sur X et 5 mm sur Y.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_radius: 75
mesh_origin: 0, 0
round_probe_count: 5
```

- `mesh_radius: 75`\
  _Required_\
  Le rayon du maillage sondé en mm, par rapport à `mesh_origin`. Notez que les décalages de la sonde limitent la taille du rayon du maillage. Dans cet exemple, un rayon supérieur à 76 déplacerait l'outil au-delà de la portée de l'imprimante.

- `mesh_origin: 0, 0`\
  _Default Value: 0, 0_\
  Le point central du maillage. Cette coordonnée est relative à l'emplacement de la sonde. Bien que la valeur par défaut soit 0, 0, il peut être utile d'ajuster l'origine dans le but de sonder une plus grande partie du lit. Voir l'illustration ci-dessous.

- `round_probe_count: 5`\
  _Default Value: 5_\
  Il s'agit d'une valeur entière qui définit le nombre maximal de points palpés le long des axes X et Y. Par "maximum", on entend le nombre de points sondés le long de l'origine du maillage. Cette valeur doit être un nombre impair, car il faut que le centre du maillage soit sondé.

L'illustration ci-dessous montre comment les points palpés sont générés. Comme vous pouvez le voir, définir `mesh_origin` sur (-10, 0) nous permet de spécifier un rayon de maillage plus grand de 85.

![bedmesh_round_basic](img/bedmesh_round_basic.svg)

## Configuration avancée

Ci-dessous, les options de configuration les plus avancées sont expliquées en détail. Chaque exemple s'appuiera sur la configuration de lit rectangulaire de base illustrée ci-dessus. Chacune des options avancées s'applique aux lits ronds de la même manière.

### Interpolation de maillage

Bien qu'il soit possible d'échantillonner la matrice sondée directement à l'aide d'un simple interpolation pour déterminer les valeurs Z entre les points sondés, il est souvent  utile pour interpoler des points supplémentaires à l'aide d'algorithmes d'interpolation plus avancés pour augmenter la densité du maillage. Ces algorithmes ajoutent une courbure au maillage, tentant de simuler les propriétés matérielles du lit. Bed Mesh offre une interpolation lagrange et bicubique pour y parvenir.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
mesh_pps: 2, 3
algorithm: bicubic
bicubic_tension: 0.2
```

- `mesh_pps: 2, 3`\
  _Default Value: 2, 2_\
  L'option `mesh_pps` est un raccourci pour Mesh Points Per Segment. Cette option spécifie le nombre de points à interpoler pour chaque segment le long des axes X et Y. Considérez qu'un "segment" est l'espace entre chaque point sondé. Comme `probe_count`, `mesh_pps` est spécifié comme une paire d'entiers X, Y, et peut également être spécifié comme un seul entier appliqué aux deux axes. Dans cet exemple, il y a 4 segments le long de l'axe X et 2 segments le long de l'axe Y. Cela donne 8 points interpolés le long de X, 6 points interpolés le long de Y, ce qui donne un maillage 13x8. Notez que si mesh_pps est défini sur 0, l'interpolation de maillage est désactivée et la matrice sondée sera échantillonnée directement.

- `algorithm: lagrange`\
  _Default Value: lagrange_\
  L'algorithme utilisé pour interpoler le maillage. Peut être "lagrange" ou "bicubique". L'interpolation de Lagrange est plafonnée à 6 points sondés car l'oscillation a tendance à se produire avec un plus grand nombre d'échantillons. L'interpolation bicubique nécessite un minimum de 4 points sondés le long de chaque axe, si moins de 4 points sont spécifiés, l'échantillonnage de Lagrange est forcé. Si `mesh_pps` est défini sur 0, cette valeur est ignorée car aucune interpolation de maillage n'est effectuée.

- `bicubic_tension: 0.2`\
  _Default Value: 0.2_\
  Si l'option `algorithm` est définie sur bicubique, il est possible de spécifier la valeur de tension. Plus la tension est élevée, plus la pente est interpolée. Soyez prudent lorsque vous ajustez cela, car des valeurs plus élevées créent également plus de dépassement, ce qui entraînera des valeurs interpolées supérieures ou inférieures à vos points sondés.

L'illustration ci-dessous montre comment les options ci-dessus sont utilisées pour générer un maillage interpolé.

![bedmesh_interpolated](img/bedmesh_interpolated.svg)

### Déplacer le fractionnement

Bed Mesh fonctionne en interceptant les commandes de déplacement gcode et en appliquant une transformation à leur coordonnée Z. Les mouvements longs doivent être divisés en mouvements plus petits pour suivre correctement la forme du lit. Les options ci-dessous contrôlent le comportement de fractionnement.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
move_check_distance: 5
split_delta_z: .025
```

- `move_check_distance: 5`\
  _Default Value: 5_\
  La distance minimale pour vérifier le changement souhaité en Z avant d'effectuer un fractionnement. Dans cet exemple, un déplacement supérieur à 5 mm sera parcouru par l'algorithme. Tous les 5 mm, une recherche de maille Z se produira, en la comparant à la valeur Z du mouvement précédent. Si le delta atteint le seuil défini par `split_delta_z`, le déplacement sera divisé et la traversée se poursuivra. Ce processus se répète jusqu'à la fin du mouvement, où un ajustement final sera appliqué. Les mouvements plus courts que `move_check_distance` ont le bon ajustement Z appliqué directement au mouvement sans traversée ni division.

- `split_delta_z: .025`\
  _Default Value: .025_\
  Comme mentionné ci-dessus, il s'agit de l'écart minimum requis pour déclencher une division de mouvement. Dans cet exemple, toute valeur Z avec un écart de +/- 0,025 mm déclenchera une division.

Généralement, les valeurs par défaut de ces options sont suffisantes, en fait la valeur par défaut de 5 mm pour le `move_check_distance` peut être exagérée. Cependant un l'utilisateur avancé peut souhaiter expérimenter ces options dans le but de presser la première couche optimale.

### Mesh Fade

Lorsque "fade" est activé, le réglage Z est progressivement désactivé sur une distance définie par la configuration. Ceci est accompli en appliquant de petits ajustements à la hauteur de la couche, en augmentant ou en diminuant selon la forme du lit. Lorsque le fondu est terminé, le réglage Z n'est plus appliqué, ce qui permet au haut de l'impression d'être plat plutôt que de refléter la forme du lit. Le fondu peut également avoir des caractéristiques indésirables, si vous vous fanez trop rapidement, cela peut entraîner des artefacts visibles sur l'impression. De plus, si votre lit est considérablement déformé, la décoloration peut rétrécir ou étirer la hauteur Z de l'impression. En tant que tel, le fondu est désactivé par défaut.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
fade_start: 1
fade_end: 10
fade_target: 0
```

- `fade_start: 1`\
  _Default Value: 1_\
  La hauteur Z à partir de laquelle commencer l'ajustement progressif. C'est une bonne idée d'obtenir quelques couches avant de commencer le processus de fondu.

- `fade_end: 10`\
  _Default Value: 0_\
  La hauteur Z à laquelle le fondu doit se terminer. Si cette valeur est inférieure à `fade_start`, le fondu est désactivé. Cette valeur peut être ajustée en fonction du gauchissement de la surface d'impression. Une surface fortement déformée devrait s'estomper sur une plus longue distance. Une surface presque plane peut être en mesure de réduire cette valeur pour s'éteindre plus rapidement. 10mm est une valeur raisonnable pour commencer si vous utilisez la valeur par défaut de 1 pour `fade_start`.

- `fade_target: 0`\
  _Default Value:  The average Z value of the mesh_\
  Le `fade_target` peut être considéré comme un décalage Z supplémentaire appliqué à l'ensemble du lit une fois le fondu terminé. De manière générale, nous aimerions que cette valeur soit 0, mais il y a des circonstances où elle ne devrait pas l'être. Par exemple, supposons que votre position de référence sur le lit est une valeur aberrante, ses 0,2 mm inférieure à la hauteur moyenne sondée du lit. Si le `fade_target` est 0, le fondu réduira l'impression de 0,2 mm en moyenne sur le lit. En réglant le `fade_target` sur 0,2, la zone référencée s'agrandira de 0,2 mm, mais le reste du lit aura une taille précise. En général, c'est une bonne idée de laisser `fade_target` hors de la configuration afin que la hauteur moyenne du maillage soit utilisée, mais il peut être souhaitable d'ajuster manuellement la cible de fondu si l'on veut imprimer sur une partie spécifique du lit.

### L'indice de référence relatif

La plupart des sondes sont susceptibles de dériver, c'est-à-dire des imprécisions de sonde introduites par la chaleur ou des interférences. Cela peut compliquer le calcul du décalage z de la sonde, en particulier à différentes températures de lit. Ainsi, certaines imprimantes utilisent une butée pour la prise d'origine de l'axe Z et une sonde pour calibrer le maillage. Ces imprimantes peuvent bénéficier de la configuration de l'index de référence relatif.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
relative_reference_index: 7
```

- `relative_reference_index: 7`\
  _Default Value: None (disabled)_\
  Lorsque les points sondés sont générés, un index leur est attribué. Vous pouvez rechercher cet index dans klippy.log ou en utilisant BED_MESH_OUTPUT (voir la section sur les GCodes Bed Mesh ci-dessous pour plus d'informations). Si vous affectez un index à l'option `relative_reference_index`, la valeur sondée à cette coordonnée remplacera le z_offset de la sonde. Cela fait effectivement de cette coordonnée la référence "zéro" pour le maillage.

Lors de l'utilisation de l'index de référence relatif, vous devez choisir l'index le plus proche de l'endroit du lit où l'étalonnage de la butée Z a été effectué. Notez que lorsque vous recherchez l'index à l'aide du journal ou de BED_MESH_OUTPUT, vous devez utiliser les coordonnées répertoriées sous l'en-tête "Probe" pour trouver l'index correct.

### Régions défectueuses

Il est possible que certaines zones d'un lit signalent des résultats inexacts lors du sondage en raison d'un « défaut » à des endroits spécifiques. Le meilleur exemple en est les lits avec une série d'aimants intégrés utilisés pour retenir les tôles d'acier amovibles. Le champ magnétique au niveau et autour de ces aimants peut provoquer le déclenchement d'une sonde inductive à une distance supérieure ou inférieure à ce qu'elle serait autrement, entraînant un maillage qui ne représente pas avec précision la surface à ces emplacements. **Remarque : cela ne doit pas être confondu avec le biais d'emplacement de la sonde, qui produit des résultats inexacts sur l'ensemble du lit.**

Les options `faulty_region` peuvent être configurées pour compenser cet effet. Si un point généré se trouve dans une région défectueuse, le maillage du lit tentera de sonder jusqu'à 4 points aux limites de cette région. Ces valeurs sondées seront moyennées et insérées dans le maillage en tant que valeur Z à la coordonnée (X, Y) générée.

```
[bed_mesh]
speed: 120
horizontal_move_z: 5
mesh_min: 35, 6
mesh_max: 240, 198
probe_count: 5, 3
faulty_region_1_min: 130.0, 0.0
faulty_region_1_max: 145.0, 40.0
faulty_region_2_min: 225.0, 0.0
faulty_region_2_max: 250.0, 25.0
faulty_region_3_min: 165.0, 95.0
faulty_region_3_max: 205.0, 110.0
faulty_region_4_min: 30.0, 170.0
faulty_region_4_max: 45.0, 210.0
```

- `faulty_region_{1...99}_min`\
  `faulty_region_{1..99}_max`\
  _Default Value: None (disabled)_\
  Les régions défectueuses sont définies d'une manière similaire à celle du maillage lui-même, où les coordonnées minimales et maximales (X, Y) doivent être spécifiées pour chaque région. Une région défectueuse peut s'étendre à l'extérieur d'un maillage, mais les points alternatifs générés seront toujours à l'intérieur de la limite du maillage. Deux régions ne peuvent pas se chevaucher.

L'image ci-dessous illustre comment les points de remplacement sont générés lorsqu'un point généré se trouve dans une région défectueuse. Les régions affichées correspondent à celles de l'exemple de configuration ci-dessus. Les points de remplacement et leurs coordonnées sont identifiés en vert.

![bedmesh_interpolated](img/bedmesh_faulty_regions.svg)

## Bed Mesh Gcodes

### Calibration

`BED_MESH_CALIBRATE PROFILE=<name> METHOD=[manual | automatic] [<probe_parameter>=<value>]
 [<mesh_parameter>=<value>]`\
_Default Profile:  default_\
_Default Method:  automatic if a probe is detected, otherwise manual_

Lance la procédure de sondage pour l'étalonnage du maillage du lit.

Le maillage sera enregistré dans un profil spécifié par le paramètre `PROFILE`,
ou `default` si non spécifié. Si `METHOD=manual` est sélectionné, le sondage manuel arrivera. Lors du basculement entre le palpage automatique et manuel, le
les points de maillage seront automatiquement ajustés.

Il est possible de spécifier des paramètres de maillage pour modifier la zone sondée. Les paramètres suivants sont disponibles :

- Rectangular beds (cartesian):
  - `MESH_MIN`
  - `MESH_MAX`
  - `PROBE_COUNT`
- Round beds (delta):
  - `MESH_RADIUS`
  - `MESH_ORIGIN`
  - `ROUND_PROBE_COUNT`
- All beds:
  - `RELATIVE_REFERNCE_INDEX`
  - `ALGORITHM`

Voir la documentation de configuration ci-dessus pour plus de détails sur la façon dont chaque paramètre s'applique au maillage.

### Profiles

`BED_MESH_PROFILE SAVE=<name> LOAD=<name> REMOVE=<name>`

Après l'exécution d'un BED_MESH_CALIBRATE, il est possible de sauvegarder l'état actuel du maillage dans un profil nommé. Ceci permet de charger une maille sans re-sonder le lit. Une fois qu'un profil a été enregistré à l'aide de `BED_MESH_PROFILE SAVE=<nom>`, le gcode `SAVE_CONFIG` peut être exécuté pour écrire le profil dans printer.cfg.

Les profils peuvent être chargés en exécutant`BED_MESH_PROFILE LOAD=<name>`.
 
Il convient de noter qu'à chaque fois qu'un BED_MESH_CALIBRATE se produit, l'état actuel est automatiquement enregistré dans le profil _default_. Si ce profil existe, il est automatiquement chargé au démarrage de Klipper. Si ce comportement n'est pas souhaitable, le profil _default_ peut être supprimé comme suit:

`BED_MESH_PROFILE REMOVE=default`

Tout autre profil enregistré peut être supprimé de la même manière, en remplaçant _default_ par le profil nommé que vous souhaitez supprimer.

### Output

`BED_MESH_OUTPUT PGP=[0 | 1]`

Envoie l'état actuel du maillage au terminal. Notez que le maillage lui-même est sorti

Le paramètre PGP est un raccourci pour "Imprimer les points générés". Si `PGP=1` est défini, les points palpés générés seront envoyés au terminal :

```
// bed_mesh: generated points
// Index | Tool Adjusted | Probe
// 0 | (11.0, 1.0) | (35.0, 6.0)
// 1 | (62.2, 1.0) | (86.2, 6.0)
// 2 | (113.5, 1.0) | (137.5, 6.0)
// 3 | (164.8, 1.0) | (188.8, 6.0)
// 4 | (216.0, 1.0) | (240.0, 6.0)
// 5 | (216.0, 97.0) | (240.0, 102.0)
// 6 | (164.8, 97.0) | (188.8, 102.0)
// 7 | (113.5, 97.0) | (137.5, 102.0)
// 8 | (62.2, 97.0) | (86.2, 102.0)
// 9 | (11.0, 97.0) | (35.0, 102.0)
// 10 | (11.0, 193.0) | (35.0, 198.0)
// 11 | (62.2, 193.0) | (86.2, 198.0)
// 12 | (113.5, 193.0) | (137.5, 198.0)
// 13 | (164.8, 193.0) | (188.8, 198.0)
// 14 | (216.0, 193.0) | (240.0, 198.0)
```

Les points « Outil ajusté » font référence à l'emplacement de la buse pour chaque point, et les points « Sonde » font référence à l'emplacement de la sonde. Notez que lors d'un sondage manuel, les points "Probe" se réfèrent à la fois à l'emplacement de l'outil et de la buse.

### Dégager Mesh État

`BED_MESH_CLEAR`

Ce gcode peut être utilisé pour effacer l'état interne du maillage.

### Appliquer X/Y offsets

`BED_MESH_OFFSET [X=<value>] [Y=<value>]`

Ceci est utile pour les imprimantes avec plusieurs extrudeuses indépendantes, car un décalage est nécessaire pour produire un ajustement Z correct après un changement d'outil. Les décalages doivent être spécifiés par rapport à l'extrudeuse principale. Autrement dit, un décalage X positif doit être spécifié si l'extrudeuse secondaire est montée à droite de l'extrudeuse primaire, et un décalage Y positif doit être spécifié si l'extrudeuse secondaire est montée "derrière" l'extrudeuse primaire.
