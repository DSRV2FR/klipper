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
4. Use the digital calipers to measure the new distance between the extruder body and the mark on the filament. Note this as `<subsequent_mark_distance>`. Then calculate: `actual_extrude_distance = <initial_mark_distance> - <subsequent_mark_distance>`
5. Calculate rotation_distance as: `rotation_distance = <previous_rotation_distance> * <actual_extrude_distance> / <requested_extrude_distance>`
   Round the new rotation_distance to three decimal places.

If the actual_extrude_distance differs from requested_extrude_distance
by more than about 2mm then it is a good idea to perform the steps
above a second time.

Note: Do *not* use a "measure and trim" type of method to calibrate x,
y, or z type axes. The "measure and trim" method is not accurate
enough for those axes and will likely lead to a worse configuration.
Instead, if needed, those axes can be determined by
[measuring the belts, pulleys, and lead screw hardware](#obtaining-rotation_distance-by-inspecting-the-hardware).

## Obtaining rotation_distance by inspecting the hardware

It's possible to calculate rotation_distance with knowledge of the
stepper motors and printer kinematics. This may be useful if the
steps_per_mm is not known or if designing a new printer.

### Belt driven axes

It is easy to calculate rotation_distance for a linear axis that uses
a belt and pulley.

First determine the type of belt. Most printers use a 2mm belt pitch
(that is, each tooth on the belt is 2mm apart). Then count the number
of teeth on the stepper motor pulley. The rotation_distance is then
calculated as:
```
rotation_distance = <belt_pitch> * <number_of_teeth_on_pulley>
```

For example, if a printer has a 2mm belt and uses a pulley with 20
teeth, then the rotation distance is 40.

### Axes with a lead screw

It is easy to calculate the rotation_distance for common lead screws
using the following formula:
```
rotation_distance = <screw_pitch> * <number_of_separate_threads>
```

For example, the common "T8 leadscrew" has a rotation distance of 8
(it has a pitch of 2mm and has 4 separate threads).

Older printers with "threaded rods" have only one "thread" on the lead
screw and thus the rotation distance is the pitch of the screw. (The
screw pitch is the distance between each groove on the screw.) So, for
example, an M6 metric rod has a rotation distance of 1 and an M8 rod
has a rotation distance of 1.25.

### Extruder

It's possible to obtain an initial rotation distance for extruders by
measuring the diameter of the "hobbed bolt" that pushes the filament
and using the following formula: `rotation_distance = <diameter> * 3.14`

If the extruder uses gears then it will also be necessary to
[determine and set the gear_ratio](#using-a-gear_ratio) for the
extruder.

The actual rotation distance on an extruder will vary from printer to
printer, because the grip of the "hobbed bolt" that engages the
filament can vary. It can even vary between filament spools. After
obtaining an initial rotation_distance, use the
[measure and trim procedure](#calibrating-rotation_distance-on-extruders)
to obtain a more accurate setting.

## Using a gear_ratio

Setting a `gear_ratio` can make it easier to configure the
`rotation_distance` on steppers that have a gear box (or similar)
attached to it. Most steppers do not have a gear box - if unsure then
do not set `gear_ratio` in the config.

When `gear_ratio` is set, the `rotation_distance` represents the
distance the axis moves with one full rotation of the final gear on
the gear box. If, for example, one is using a gearbox with a "5:1"
ratio, then one could calculate the rotation_distance with
[knowledge of the hardware](#obtaining-rotation_distance-by-inspecting-the-hardware)
and then add `gear_ratio: 5:1` to the config.

For gearing implemented with belts and pulleys, it is possible to
determine the gear_ratio by counting the teeth on the pulleys. For
example, if a stepper with a 16 toothed pulley drives the next pulley
with 80 teeth then one would use `gear_ratio: 80:16`. Indeed, one
could open a common off the shelf "gear box" and count the teeth in it
to confirm its gear ratio.

Note that sometimes a gearbox will have a slightly different gear
ratio than what it is advertised as. The common BMG extruder motor
gears are an example of this - they are advertised as "3:1" but
actually use "50:17" gearing. (Using teeth numbers without a common
denominator may improve overall gear wear as the teeth don't always
mesh the same way with each revolution.) The common "5.18:1 planetary
gearbox", is more accurately configured with `gear_ratio: 57:11`.

If several gears are used on an axis then it is possible to provide a
comma separated list to gear_ratio. For example, a "5:1" gear box
driving a 16 toothed to 80 toothed pulley could use
`gear_ratio: 5:1, 80:16`.

In most cases, gear_ratio should be defined with whole numbers as
common gears and pulleys have a whole number of teeth on them.
However, in cases where a belt drives a pulley using friction instead
of teeth, it may make sense to use a floating point number in the gear
ratio (eg, `gear_ratio: 107.237:16`).
