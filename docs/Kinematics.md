# Cinématique

Ce document donne un aperçu de la façon dont Klipper implémente le mouvement du robot (sa [cinématique](https://en.wikipedia.org/wiki/Kinematics)). Le contenu peut intéresser aussi bien les développeurs intéressés à travailler sur le logiciel Klipper que les utilisateurs intéressés à mieux comprendre la mécanique de leurs machines.

## Accélération

Klipper implémente un schéma d'accélération constante chaque fois que la tête d'impression change de vitesse - la vitesse est progressivement modifiée à la nouvelle vitesse au lieu d'y être brusquement saccadée. Klipper applique toujours une accélération entre la tête de l'outil et l'impression. Le filament quittant l'extrudeuse peut être assez fragile - des secousses rapides et/ou des changements de débit de l'extrudeuse entraînent une mauvaise qualité et une mauvaise adhérence au lit. Même en l'absence d'extrusion, si la tête d'impression est au même niveau que l'impression, une secousse rapide de la tête peut provoquer une rupture du filament récemment déposé. Limiter les changements de vitesse de la tête d'impression (par rapport à l'impression) réduit les risques de perturber l'impression.

Il est également important de limiter l'accélération afin que les moteurs pas à pas ne sautent pas ou n'exercent pas de contraintes excessives sur la machine. Klipper limite le couple sur chaque stepper en limitant l'accélération de la tête d'impression. L'application d'une accélération au niveau de la tête d'impression limite également naturellement le couple des moteurs pas à pas qui déplacent la tête d'impression (l'inverse n'est pas toujours vrai).

Klipper implémente une accélération constante. La formule clé pour une accélération constante est la suivante :
```
velocity(time) = start_velocity + accel*time
```

## Générateur trapézoïdal

Klipper utilise un "générateur trapézoïdal" traditionnel pour modéliser le mouvement de chaque mouvement - chaque mouvement a une vitesse de départ, il accélère à une vitesse de croisière à accélération constante, il roule à une vitesse constante, puis décélère jusqu'à la vitesse finale en utilisant une accélération constante .

![trapezoid](img/trapezoid.svg.png)

C'est ce qu'on appelle un "générateur de trapèze" car un diagramme de vitesse du mouvement ressemble à un trapèze.

La vitesse de croisière est toujours supérieure ou égale à la fois à la vitesse de départ et à la vitesse de fin. La phase d'accélération peut être de durée nulle (si la vitesse de démarrage est égale à la vitesse de croisière), la phase de croisière peut être de durée nulle (si le mouvement commence immédiatement à décélérer après l'accélération), et/ou la phase de décélération peut être de zéro durée (si la vitesse finale est égale à la vitesse de croisière).

![trapezoids](img/trapezoids.svg.png)

## Prévision

Le système "anticipation" est utilisé pour déterminer les vitesses de virage entre les mouvements.

Considérez les deux mouvements suivants contenus sur un plan XY :

![corner](img/corner.svg.png)

Dans la situation ci-dessus, il est possible de décélérer complètement après le premier mouvement, puis d'accélérer complètement au début du mouvement suivant, mais ce n'est pas idéal car toute cette accélération et cette décélération augmenteraient considérablement le temps d'impression et les changements fréquents du débit de l'extrudeuse. entraînerait une mauvaise qualité d'impression.

Pour résoudre ce problème, le mécanisme de "prévision" met en file d'attente plusieurs mouvements entrants et analyse les angles entre les mouvements pour déterminer une vitesse raisonnable pouvant être obtenue lors de la "jonction" entre deux mouvements. Si le mouvement suivant est presque dans la même direction, la tête n'a qu'à ralentir un peu (voire pas du tout).

![lookahead](img/lookahead.svg.png)

Cependant, si le mouvement suivant forme un angle aigu (la tête va se déplacer presque dans le sens inverse lors du mouvement suivant), seule une petite vitesse de jonction est autorisée.

![lookahead](img/lookahead-slow.svg.png)
Les vitesses de jonction sont déterminées en utilisant "l'accélération centripète approchée". Meilleur [décrit par l'auteur](https://onehossshay.wordpress.com/2011/09/24/improving_grbl_cornering_algorithm/). Cependant, dans Klipper, les vitesses de jonction sont configurées en spécifiant la vitesse souhaitée qu'un coin à 90 ° devrait avoir (la "vitesse de coin carré"), et les vitesses de jonction pour les autres angles en sont dérivées.

Formule clé pour l'anticipation :
```
end_velocity^2 = start_velocity^2 + 2*accel*move_distance
```

### Anticipation lissée

Klipper implémente également un mécanisme pour lisser les mouvements de mouvements courts en "zigzag". Considérez les mouvements suivants :

![zigzag](img/zigzag.svg.png)

Dans ce qui précède, les changements fréquents de l'accélération à la décélération peuvent faire vibrer la machine, ce qui provoque une contrainte sur la machine et augmente le bruit. Pour réduire cela, Klipper suit à la fois l'accélération de mouvement régulière ainsi qu'un taux virtuel "d'accélération à décélération". Grâce à ce système, la vitesse maximale de ces mouvements courts en "zigzag" est limitée pour lisser le mouvement de l'imprimante :

![smoothed](img/smoothed.svg.png)

Plus précisément, le code calcule quelle serait la vitesse de chaque mouvement s'il était limité à ce taux virtuel "d'accélération à décélération" (la moitié du taux d'accélération normal par défaut). Dans l'image ci-dessus, les lignes grises en pointillés représentent ce taux d'accélération virtuelle pour le premier coup. Si un mouvement ne peut pas atteindre sa pleine vitesse de croisière en utilisant ce taux d'accélération virtuel, alors sa vitesse maximale est réduite à la vitesse maximale qu'il pourrait obtenir à ce taux d'accélération virtuel. Pour la plupart des mouvements, la limite sera égale ou supérieure aux limites existantes du mouvement et aucun changement de comportement n'est induit. Pour les déplacements courts en zigzag,
cependant, cette limite réduit la vitesse maximale. Notez que cela ne change pas l'accélération réelle dans le mouvement - le mouvement continue d'utiliser le schéma d'accélération normal jusqu'à sa vitesse maximale ajustée.

## Étapes de génération

Une fois le processus d'anticipation terminé, le mouvement de la tête d'impression pour le mouvement donné est entièrement connu (heure, position de départ, position finale, vitesse à chaque point) et il est possible de générer les temps de pas pour le mouvement. Ce processus est effectué dans les "classes cinématiques" du code Klipper. En dehors de ces classes cinématiques, tout est suivi en millimètres, en secondes et dans l'espace de coordonnées cartésiennes. C'est la tâche des classes cinématiques de convertir de ce système de coordonnées générique aux spécificités matérielles de l'imprimante particulière.

Les clips ne sont pas utilisés [solveur itératif](https://en.wikipedia.org/wiki/Root-finding_algorithm) pour générer les temps de pas pour chaque stepper. Le code contient les formules pour calculer les coordonnées cartésiennes idéales de la tête à chaque instant, et il contient les formules cinématiques pour calculer les positions pas à pas idéales en fonction de ces coordonnées cartésiennes. Avec ces formules, Klipper peut déterminer le moment idéal où le stepper doit être à chaque position de pas. Les étapes données sont alors ordonnancées à ces instants calculés.

La formule clé pour déterminer la distance qu'un mouvement doit parcourir sous une accélération constante est:
```
move_distance = (start_velocity + .5 * accel * move_time) * move_time
```
et la formule clé pour un mouvement à vitesse constante est :
```
move_distance = cruise_velocity * move_time
```

Les formules clés pour déterminer la coordonnée cartésienne d'un mouvement étant donné une distance de mouvement sont:
```
cartesian_x_position = start_x + move_distance * total_x_movement / total_movement
cartesian_y_position = start_y + move_distance * total_y_movement / total_movement
cartesian_z_position = start_z + move_distance * total_z_movement / total_movement
```

### Robots cartésiens

La génération d'étapes pour les imprimantes cartésiennes est le cas le plus simple. Le mouvement sur chaque axe est directement lié au mouvement dans l'espace cartésien.

Formules clés :
```
stepper_x_position = cartesian_x_position
stepper_y_position = cartesian_y_position
stepper_z_position = cartesian_z_position
```

### Robots CoreXY

La génération d'étapes sur une machine CoreXY n'est qu'un peu plus complexe que les robots cartésiens de base. Les formules clés sont :
```
stepper_a_position = cartesian_x_position + cartesian_y_position
stepper_b_position = cartesian_x_position - cartesian_y_position
stepper_z_position = cartesian_z_position
```

### Delta Robots

La génération de pas sur un robot delta est basée sur le théorème de Pythagore :
```
stepper_position = (sqrt(arm_length^2
                         - (cartesian_x_position - tower_x_position)^2
                         - (cartesian_y_position - tower_y_position)^2)
                    + cartesian_z_position)
```

### Limites d'accélération du moteur pas à pas

Avec la cinématique delta, il est possible qu'un mouvement qui s'accélère dans l'espace cartésien nécessite une accélération sur un moteur pas à pas particulier supérieure à l'accélération du mouvement. Cela peut se produire lorsqu'un bras de stepper est plus horizontal que vertical et que la ligne de mouvement passe près de la tour de ce stepper. Bien que ces mouvements puissent nécessiter une accélération du moteur pas à pas supérieure à l'accélération de mouvement maximale configurée de l'imprimante, la masse effective déplacée par ce moteur pas à pas serait plus petite. Ainsi, l'accélération pas à pas plus élevée n'entraîne pas un couple pas à pas significativement plus élevé et elle est donc considérée comme inoffensive.

Cependant, pour éviter les cas extrêmes, Klipper applique un plafond maximal sur l'accélération pas à pas de trois fois l'accélération de déplacement maximale configurée de l'imprimante. (De même, la vitesse maximale du stepper est limitée à trois fois la vitesse de déplacement maximale.) Afin d'appliquer cette limite, les mouvements à l'extrême bord de l'enveloppe de construction (où un bras stepper peut être presque horizontal) auront une valeur inférieure accélération et vitesse maximales.

### Cinématique de l'extrudeuse

Klipper implémente le mouvement de l'extrudeuse dans sa propre classe cinématique. Étant donné que la synchronisation et la vitesse de chaque mouvement de la tête d'impression sont entièrement connues pour chaque mouvement, il est possible de calculer les temps de pas pour l'extrudeuse indépendamment des calculs de temps de pas du mouvement de la tête d'impression.

Le mouvement de base de l'extrudeuse est simple à calculer. La génération de temps de pas utilise les mêmes formules que les robots cartésiens :
```
stepper_position = requested_e_position
```

### Avance de pression

L'expérimentation a montré qu'il est possible d'améliorer la modélisation de l'extrudeuse au-delà de la formule de base de l'extrudeuse. Dans le cas idéal, à mesure qu'un mouvement d'extrusion progresse, le même volume de filament doit être déposé à chaque point le long du mouvement et il ne doit y avoir aucun volume extrudé après le mouvement. Malheureusement, il est courant de constater que les formules d'extrusion de base font que trop peu de filaments sortent de l'extrudeuse au début des mouvements d'extrusion et que l'excès de filaments s'extrude après la fin de l'extrusion. Ceci est souvent appelé "ooze".

![ooze](img/ooze.svg.png)

Le système "d'avance de pression" tente de tenir compte de cela en utilisant un modèle différent pour l'extrudeuse. Au lieu de croire naïvement que chaque mm ^ 3 de filament introduit dans l'extrudeuse entraînera cette quantité de mm ^ 3 sortant immédiatement de l'extrudeuse, il utilise un modèle basé sur la pression. La pression augmente lorsque le filament est poussé dans l'extrudeuse (as in [Hooke's law](https://en.wikipedia.org/wiki/Hooke%27s_law)) et la pression nécessaire pour extruder est dominée par le débit à travers l'orifice de la buse (as in [Poiseuille's law](https://en.wikipedia.org/wiki/Poiseuille_law)). L'idée clé est que la relation entre le filament, la pression et le débit peut être modélisée à l'aide d'un coefficient linéaire :
```
pa_position = nominal_position + pressure_advance_coefficient * nominal_velocity
```

Voir le[pressure advance](Pressure_Advance.md) document pour savoir comment trouver ce coefficient d'avance de pression.

La formule d'avance de pression de base peut amener le moteur de l'extrudeuse à effectuer des changements de vitesse soudains. Outils Klipper "smoothing - lissage" du mouvement de l'extrudeuse pour éviter cela.

![pressure-advance](img/pressure-velocity.png)

Le graphique ci-dessus montre un exemple de deux mouvements d'extrusion avec une vitesse de virage non nulle entre eux. Notez que le système d'avance de pression provoque la poussée de filament supplémentaire dans l'extrudeuse lors de l'accélération. Plus le débit de filament souhaité est élevé, plus le filament doit être poussé pendant l'accélération pour tenir compte de la pression. Pendant la décélération de la tête, le filament supplémentaire est rétracté (l'extrudeuse aura une vitesse négative).

Le "lissage" est implémenté en utilisant une moyenne pondérée de la position de l'extrudeuse sur une petite période de temps (comme spécifié par le paramètre de configuration `pressure_advance_smooth_time`). Cette moyenne peut couvrir plusieurs mouvements de code g. Notez comment le moteur de l'extrudeuse commencera à se déplacer avant le début nominal du premier mouvement d'extrusion et continuera à se déplacer après la fin nominale du dernier mouvement d'extrusion.

Formule clé pour "smoothed pressure advance":
```
smooth_pa_position(t) =
    ( definitive_integral(pa_position(x) * (smooth_time/2 - abs(t - x)) * dx,
                          from=t-smooth_time/2, to=t+smooth_time/2)
     / (smooth_time/2)^2 )
```
