# Mesurer les résonances

Klipper a un support intégré pour l'accéléromètre ADXL345, qui peut être utilisé pour mesurer les fréquences de résonance de l'imprimante pour différents axes et effectuer un réglage automatique [input shapers](Resonance_Compensation.md) pour compenser les résonances. Notez que l'utilisation d'ADXL345 nécessite un peu de soudure et de sertissage. ADXL345 peut être connecté directement à un Raspberry Pi, ou à une interface SPI d'un MCU conseil (il doit être raisonnablement rapide).

Lors de l’achat  en ADXL345, sachez qu'il existe une variété de PCB différent sconceptions de cartes et différents clones de celles-ci. Assurez-vous que la carte supporte Mode SPI (un petit nombre de cartes semblent être configurées en dur pour I2C par tirant SDO sur GND), et, s'il doit être connecté à un MCU d'imprimante 5V, qu'il a un régulateur de tension et un décaleur de niveau.


## Instructions d'installation

### Câblage

Vous devez connecter ADXL345 à votre Raspberry Pi via SPI. Notez que l'I2C
connexion, suggérée par la documentation ADXL345, a un débit trop faible
et **ne fonctionnera pas**. Le schéma de connexion recommandé :

| ADXL345 pin | RPi pin | RPi pin name |
|:--:|:--:|:--:|
| 3V3 (or VCC) | 01 | 3.3v DC power |
| GND | 06 | Ground |
| CS | 24 | GPIO08 (SPI0_CE0_N) |
| SDO | 21 | GPIO09 (SPI0_MISO) |
| SDA | 19 | GPIO10 (SPI0_MOSI) |
| SCL | 23 | GPIO11 (SPI0_SCLK) |

Une alternative à l'ADXL345 est le MPU-9250 (ou MPU-6050). Cette l'accéléromètre a été testé pour fonctionner sur I2C sur le RPi à 400kbaud.
Schéma de connexion recommandé pour I2C :

| MPU-9250 pin | RPi pin | RPi pin name |
|:--:|:--:|:--:|
| 3V3 (or VCC) | 01 | 3.3v DC power |
| GND | 09 | Ground |
| SDA | 03 | GPIO02 (SDA1) |
| SCL | 05 | GPIO03 (SCL1) |


Schémas de câblage de Fritzing pour certaines des cartes ADXL345 :

![ADXL345-Rpi](img/adxl345-fritzing.png)


Vérifiez votre câblage avant de mettre le Raspberry Pi sous tension pour éviter
l'endommager ou l'accéléromètre.

### Montage de l'accéléromètre

L'accéléromètre doit être fixé à la tête d'outil. Il faut concevoir un bon
support qui s'adapte à leur propre imprimante 3D. Il est préférable d'aligner les axes des accéléromètre avec les axes de l'imprimante (mais si c'est plus pratique,les axes peuvent être échangés - c'est-à-dire qu'il n'est pas nécessaire d'aligner l'axe X avec X et ainsi de suite – il devrait être correct même si l'axe Z de l'accéléromètre est l'axe X de l'imprimante, etc.).

Un exemple de montage ADXL345 sur le SmartEffector :

![ADXL345 on SmartEffector](img/adxl345-mount.jpg)

A noter que sur une imprimante bed slinger il faut concevoir 2 supports : un pour le tête d'outil et un pour le lit, et exécutez les mesures deux fois. Voir le [section] correspondante (#bed-slinger-printers) pour plus de détails.

**Attention :** assurez-vous que l'accéléromètre et toutes les vis qui le maintiennent ne touchez aucune partie métallique de l'imprimante. Fondamentalement, le support doit être conçu de manière à assurer l'isolation électrique de l'accéléromètre du châssis de l'imprimante. Ne pas s'assurer que cela peut créer une boucle de masse dans le système qui pourrait endommager l'électronique.

### Installation du logiciel

Notez que les mesures de résonance et l'auto-calibrage du shaper nécessitent des
dépendances logicielles non installées par défaut. Tout d'abord, exécutez sur votre Raspberry Pi les commandes suivantes :

```
sudo apt update
sudo apt install python3-numpy python3-matplotlib libatlas-base-dev
```

Ensuite, pour installer NumPy dans l'environnement Klipper, exécutez la commande :
```
~/klippy-env/bin/pip install -v numpy
```

Notez que, selon les performances du processeur, cela peut prendre *beaucoup*
de temps, jusqu'à 10-20 minutes. Soyez patient et attendez la fin de l'installation. À certaines occasions, si la carte a trop peu de RAM l'installation peut échouer et vous devrez activer le swap.

Ensuite, vérifiez et suivez les instructions du [RPi Microcontroller document](RPi_microcontroller.md) pour configurer le "linux mcu" sur le Raspberry Pi.

Assurez-vous que le pilote Linux SPI est activé en exécutant `sudo raspi-config` and permettant SPI sous le menu  "Option Interface",

Pour l'ADXL345, ajoutez ce qui suit au fichier printer.cfg :
```
[mcu rpi]
serial: /tmp/klipper_host_mcu

[adxl345]
cs_pin: rpi:None

[resonance_tester]
accel_chip: adxl345
probe_points:
    100, 100, 20  # un exemple
```
Il est conseillé de commencer avec 1 point de sonde, au milieu du lit d'impression, légèrement au-dessus.

Pour le MPU-9250, assurez-vous que le pilote Linux I2C est activé et que le débit en bauds est défini sur 400000 (voir [Activation d'I2C](RPi_microcontroller.md#optional-enabling-i2c) section pour plus de détails). Ensuite, ajoutez ce qui suit au fichier printer.cfg :
```

[mcu rpi]
serial: /tmp/klipper_host_mcu

[mpu9250]
i2c_mcu: rpi
i2c_bus: i2c.1

[resonance_tester]
accel_chip: mpu9250
probe_points:
    100, 100, 20  # un exemple
```

Redémarrez Klipper via la commande `RESTART`.

## Mesurer les résonances

### Vérifier la configuration

Vous pouvez maintenant tester une connexion.

- Pour les "non bed-slingers" (par exemple un accéléromètre), dans Octoprint, entrer `ACCELEROMETER_QUERY`
- Pour les « bed-slingers » (par exemple, plus d'un accéléromètre), entrez `ACCELEROMETER_QUERY CHIP=<puce>` où `<puce>` est le nom de la puce tel que saisi, par ex. `CHIP=bed` (voir : [bed-slinger](#bed-slinger-printers)) pour toutes les puces d'accéléromètre installées.

Vous devriez voir les mesures actuelles de l'accéléromètre, y compris le
accélération en chute libre, par ex.
```
Recv: // adxl345 values (x, y, z): 470.719200, 941.438400, 9728.196800
```

Si vous obtenez une erreur du type "ID adxl345 invalide (got xx vs e5)", où "xx"
est un autre ID, il indique le problème de connexion avec ADXL345, ou le capteur défectueux. Revérifiez l'alimentation, le câblage (qu'il corresponde les schémas, aucun fil n'est cassé ou desserré, etc.), et la qualité de la soudure.

Ensuite, essayez d'exécuter `MEASURE_AXES_NOISE` dans Octoprint, vous devriez obtenir quelques numéros de référence pour le bruit de l'accéléromètre sur les axes (devraient être quelque part dans la gamme de ~ 1-100). Bruit d'axes trop élevé (par exemple 1000 et plus) peut être révélateur de problèmes de capteur, de problèmes d'alimentation ou trop ventilateurs déséquilibrés bruyants sur une imprimante 3D.

### Mesurer les résonances

Vous pouvez maintenant exécuter des tests réels. Exécutez la commande suivante :
```
TEST_RESONANCES AXIS=X
```

Notez que cela créera des vibrations sur l'axe X. Cela désactivera également la saisie mise en forme si elle a été activée précédemment, car elle n'est pas valide pour exécuter la résonance test avec l’ input shaper activé.

**Attention !** Assurez-vous d'observer l'imprimante pour la première fois, afin de vous assurer les vibrations ne deviennent pas trop violentes (la commande 'M112' ​​peut être utilisée pour annuler le test en cas d'urgence; j'espère que cela n'en arrivera pas là). Si les vibrations deviennent trop fortes, vous pouvez essayer de spécifier une valeur inférieure à la valeur par défaut pour le paramètre `accel_per_hz` dans la section `[resonance_tester]`, par ex.
```
[resonance_tester]
accel_chip: adxl345
accel_per_hz: 50  # default is 75
probe_points: ...
```

Si cela fonctionne pour l'axe X, exécutez également pour l'axe Y:
```
TEST_RESONANCES AXIS=Y
```
Cela générera 2 fichiers CSV (`/tmp/resonances_x_*.csv` et
`/tmp/resonances_y_*.csv`). Ces fichiers peuvent être traités avec le module autonome script sur un Raspberry Pi. Pour ce faire, exécutez les commandes sND+MISO
3.3V+MOSIuivantes:
```
~/klipper/scripts/calibrate_shaper.py /tmp/resonances_x_*.csv -o /tmp/shaper_calibrate_x.png

~/klipper/scripts/calibrate_shaper.py /tmp/resonances_y_*.csv -o /tmp/shaper_calibrate_y.png
```

Ce script générera les graphiques `/tmp/shaper_calibrate_x.png` et `/tmp/shaper_calibrate_y.png` avec les réponses en fréquence. Vous obtiendrez également le fréquences suggérées pour chaque shaper d'entrée, ainsi que quel shaper d'entrée est recommandé pour votre configuration. Par exemple:

![Resonances](img/calibrate-y.png)
```
Fitted shaper 'zv' frequency = 34.4 Hz (vibrations = 4.0%, smoothing ~= 0.132)
To avoid too much smoothing with 'zv', suggested max_accel <= 4500 mm/sec^2
Fitted shaper 'mzv' frequency = 34.6 Hz (vibrations = 0.0%, smoothing ~= 0.170)
To avoid too much smoothing with 'mzv', suggested max_accel <= 3500 mm/sec^2
Fitted shaper 'ei' frequency = 41.4 Hz (vibrations = 0.0%, smoothing ~= 0.188)
To avoid too much smoothing with 'ei', suggested max_accel <= 3200 mm/sec^2
Fitted shaper '2hump_ei' frequency = 51.8 Hz (vibrations = 0.0%, smoothing ~= 0.201)
To avoid too much smoothing with '2hump_ei', suggested max_accel <= 3000 mm/sec^2
Fitted shaper '3hump_ei' frequency = 61.8 Hz (vibrations = 0.0%, smoothing ~= 0.215)
To avoid too much smoothing with '3hump_ei', suggested max_accel <= 2800 mm/sec^2
Recommended shaper is mzv @ 34.6 Hz
```

La configuration suggérée peut être ajoutée à la section `[input_shaper]` de
`printer.cfg`, par exemple :
```
[input_shaper]
shaper_freq_x: ...
shaper_type_x: ...
shaper_freq_y: 34.6
shaper_type_y: mzv

[printer]
max_accel: 3000  # ne doit pas dépasser le max_accel estimé pour les axes X et Y
```
ou vous pouvez choisir vous-même une autre configuration en fonction de la génération graphes : les pics de densité spectrale de puissance sur les graphes correspondent à les fréquences de résonance de l'imprimante.

Notez que vous pouvez également exécuter l'autocalibration du shaper d'entrée
de Klipper [directement](#input-shaper-auto-calibration), qui peut être
pratique, par exemple, pour le shaper d'entrée [re-calibration](#input-shaper-re-calibration).

### Bed-slinger imprimantes (lit)

Si votre imprimante est une imprimante bed slinger, vous devrez changer l'emplacement de l'accéléromètre entre les mesures pour les axes X et Y : mesurer la résonances de l'axe X avec l'accéléromètre attaché à la tête d'outil et le résonances de l'axe Y - au lit (la configuration habituelle du setup de lit).

Cependant, vous pouvez également connecter deux accéléromètres simultanément, bien qu'ils doit être connecté à différentes cartes (par exemple, à une carte RPi et MCU d'imprimante), ou à deux interfaces SPI physiques différentes sur la même carte (rarement disponible). Ensuite, ils peuvent être configurés de la manière suivante:

```
[adxl345 hotend]
# En supposant que la puce "hotend" est connectée à un RPi
cs_pin: rpi:None

[adxl345 bed]
# En supposant que la puce "bed" est connectée à une carte MCU d'imprimante
cs_pin: ...  # Broche de sélection de puce SPI (CS) de la carte d'imprimante

[resonance_tester]
# En supposant la configuration typique de l'imprimante de frondeur de lit
accel_chip_x: adxl345 hotend
accel_chip_y: adxl345 bed
probe_points: ...
```

Puis les commandes 'TEST_RESONANCES AXIS=X' et 'TEST_RESONANCES AXIS=Y'
utilisera le bon accéléromètre pour chaque axe.

### Max smoothing

Gardez à l'esprit que l’input shaper peut créer un lissage dans certaines parties. Réglage automatique d’input shaper effectué par `calibrate_shaper.py`
le script ou la commande `SHAPER_CALIBRATE` essaie de ne pas exacerber le lissage, mais en même temps, ils essaient de minimiser les vibrations qui en résultent. Parfois, ils peuvent faire un choix sous-optimal de la fréquence de mise en forme, ou peut-être préférez-vous simplement avoir moins de lissage dans certaines parties au détriment de une plus grande vibrations restantes. Dans ces cas, vous pouvez demander à limiter le lissage maximum du shaper d'entrée.

Considérons les résultats suivants du réglage automatique :

![Resonances](img/calibrate-x.png)
```
Fitted shaper 'zv' frequency = 57.8 Hz (vibrations = 20.3%, smoothing ~= 0.053)
To avoid too much smoothing with 'zv', suggested max_accel <= 13000 mm/sec^2
Fitted shaper 'mzv' frequency = 34.8 Hz (vibrations = 3.6%, smoothing ~= 0.168)
To avoid too much smoothing with 'mzv', suggested max_accel <= 3600 mm/sec^2
Fitted shaper 'ei' frequency = 48.8 Hz (vibrations = 4.9%, smoothing ~= 0.135)
To avoid too much smoothing with 'ei', suggested max_accel <= 4400 mm/sec^2
Fitted shaper '2hump_ei' frequency = 45.2 Hz (vibrations = 0.1%, smoothing ~= 0.264)
To avoid too much smoothing with '2hump_ei', suggested max_accel <= 2200 mm/sec^2
Fitted shaper '3hump_ei' frequency = 48.0 Hz (vibrations = 0.0%, smoothing ~= 0.356)
To avoid too much smoothing with '3hump_ei', suggested max_accel <= 1500 mm/sec^2
Recommended shaper is 2hump_ei @ 45.2 Hz
```
Notez que les valeurs de "lissage" rapportées sont des valeurs projetées abstraites. Ces valeurs permettent de comparer différentes configurations : plus le valeur, plus un shaper créera de lissage. Cependant, ces scores de lissage
ne représentent aucune mesure réelle de lissage, car le lissage réel dépend de [`max_accel`](#selecting-max-accel) et `square_corner_velocity` paramètres. Par conséquent, vous devez imprimer des impressions de test pour voir combien lissage exactement une configuration choisie crée.

Dans l'exemple ci-dessus, les paramètres de mise en forme suggérés ne sont pas mauvais, mais que se passe-t-il si vous voulez obtenir moins de lissage sur l'axe X ? Vous pouvez essayer de limiter le maximum lissage du shaper à l'aide de la commande suivante :
```
~/klipper/scripts/calibrate_shaper.py /tmp/resonances_x_*.csv -o /tmp/shaper_calibrate_x.png --max_smoothing=0.2
```
ce qui limite le lissage à un score de 0,2. Vous pouvez maintenant obtenir le résultat suivant:

![Resonances](img/calibrate-x-max-smoothing.png)
```
Fitted shaper 'zv' frequency = 55.4 Hz (vibrations = 19.7%, smoothing ~= 0.057)
To avoid too much smoothing with 'zv', suggested max_accel <= 12000 mm/sec^2
Fitted shaper 'mzv' frequency = 34.6 Hz (vibrations = 3.6%, smoothing ~= 0.170)
To avoid too much smoothing with 'mzv', suggested max_accel <= 3500 mm/sec^2
Fitted shaper 'ei' frequency = 48.2 Hz (vibrations = 4.8%, smoothing ~= 0.139)
To avoid too much smoothing with 'ei', suggested max_accel <= 4300 mm/sec^2
Fitted shaper '2hump_ei' frequency = 52.0 Hz (vibrations = 2.7%, smoothing ~= 0.200)
To avoid too much smoothing with '2hump_ei', suggested max_accel <= 3000 mm/sec^2
Fitted shaper '3hump_ei' frequency = 72.6 Hz (vibrations = 1.4%, smoothing ~= 0.155)
To avoid too much smoothing with '3hump_ei', suggested max_accel <= 3900 mm/sec^2
Recommended shaper is 3hump_ei @ 72.6 Hz
```

Si vous comparez aux paramètres suggérés précédemment, les vibrations sont un peu plus grand, mais le lissage est nettement plus petit qu'auparavant, ce qui permet accélération maximale plus importante.

Lorsque vous décidez quel paramètre `max_smoothing` choisir, vous pouvez utiliser un approche par essais et erreurs. Essayez quelques valeurs différentes et voyez quels résultats vous obtenez. Notez que le lissage réel produit par le shaper d'entrée dépend, principalement, sur la fréquence de résonance la plus basse de l'imprimante : plus la fréquence de la résonance la plus basse - plus le lissage est petit. Par conséquent, si vous demandez au script de trouver une configuration du shaper d'entrée avec le lissage irréaliste-ment petit, ce sera au détriment d'une résonance accrue aux fréquences de résonance les plus basses (qui sont, généralement, également plus en évidence visible dans les tirages). Donc, vérifiez toujours les vibrations restantes projetées signalés par le script et assurez-vous qu'ils ne sont pas trop élevés.

Notez que si vous avez choisi une bonne valeur `max_smoothing` pour vos deux axes, vous peut le stocker dans le `printer.cfg` comme
```
[resonance_tester]
accel_chip: ...
probe_points: ...
max_smoothing: 0.25  # un exemple
```
Ensuite, si vous [réexécutez] (#input-shaper-re-calibration) le réglage automatique d’input shaper en utilisant la commande Klipper `SHAPER_CALIBRATE` à l'avenir, il utilisera le `max_smoothing` valeur comme référence.

### Sélection max_accel

Étant donné que l’input shaper peut créer un certain lissage dans certaines parties, en particulier à haute accélérations, vous devrez toujours choisir la valeur `max_accel` qui ne crée pas trop de lissage dans les pièces imprimées. Un script de calibrage fournit une estimation du paramètre `max_accel` qui ne devrait pas créer trop lissage. Notez que le `max_accel` tel qu'affiché par le script de calibrage est seulement un maximum théorique auquel le façonneur respectif est encore capable de travailler sans produire trop de lissage. Il ne s'agit en aucun cas d'une recommandation de définir cette accélération pour l'impression. L'accélération maximale que votre imprimante est capable de
le maintien dépend de ses propriétés mécaniques et du couple maximal du moteur utilisé moteurs pas à pas. Par conséquent, il est suggéré de définir `max_accel` dans `[printer]` section qui ne dépasse pas les valeurs estimées pour les axes X et Y, probablement avec une certaine marge de sécurité prudente.

Sinon, suivez [this](Resonance_Compensation.md#selecting-max_accel) fait partie de le guide de réglage du shaper d'entrée et imprimez le modèle de test pour choisir `max_accel` paramètre expérimentalement.

Le même avis s'applique au shaper d'entrée [auto-calibration](#input-shaper-auto-calibration) avec Commande `SHAPER_CALIBRATE` : encore faut-il choisir le bon La valeur `max_accel` après l'auto-calibrage et l'accélération suggérée
les limites ne seront pas appliquées automatiquement.

Si vous effectuez un recalibrage du shaper et que le lissage signalé pour le
la configuration de shaper suggérée est presque la même que celle que vous avez obtenue lors de la calibrage précédent, cette étape peut être ignorée.

### Test des axes personnalisés

La commande `TEST_RESONANCES` prend en charge les axes personnalisés. Alors que ce n'est pas vraiment utile pour l'étalonnage du shaper d'entrée, il peut être utilisé pour étudier l'imprimante résonances en profondeur et pour vérifier, par exemple, la tension de la courroie.

Pour vérifier la tension de la courroie sur les imprimantes CoreXY, exécutez
```
TEST_RESONANCES AXIS=1,1 OUTPUT=raw_data
TEST_RESONANCES AXIS=1,-1 OUTPUT=raw_data
```
et utilise`graph_accelerometer.py` pour traiter les fichiers générés, par ex.
```
~/klipper/scripts/graph_accelerometer.py -c /tmp/raw_data_axis*.csv -o /tmp/resonances.png
```
qui va générer `/tmp/resonances.png` en comparant les résonances.

Pour les imprimantes Delta avec le placement de tour par défaut
(tower A ~= 210 degrees, B ~= 330 degrees, and C ~= 90 degrees), exécuter
```
TEST_RESONANCES AXIS=0,1 OUTPUT=raw_data
TEST_RESONANCES AXIS=-0.866025404,-0.5 OUTPUT=raw_data
TEST_RESONANCES AXIS=0.866025404,-0.5 OUTPUT=raw_data
```
puis utilisez la même commande
```
~/klipper/scripts/graph_accelerometer.py -c /tmp/raw_data_axis*.csv -o /tmp/resonances.png
```
générer `/tmp/resonances.png` comparant les résonances.

## Input Shaper auto-calibration

En plus de choisir manuellement les paramètres appropriés pour le shaper d'entrée fonctionnalité, il est également possible d'exécuter l'auto-réglage pour le shaper d'entrée directement de Klipper. Exécutez la commande suivante via la terminaison Octoprintl:
```
SHAPER_CALIBRATE
```

Cela exécutera le test complet pour les deux axes et générera la sortie csv
(`/tmp/calibration_data_*.csv` par défaut) pour la réponse en fréquence
et les shapers d'entrée suggérés. Vous obtiendrez également la suggestion
fréquences pour chaque shaper d'entrée, ainsi que quel shaper d'entrée est
recommandé pour votre configuration, sur la console Octoprint. Par exemple:

```
Calculating the best input shaper parameters for y axis
Fitted shaper 'zv' frequency = 39.0 Hz (vibrations = 13.2%, smoothing ~= 0.105)
To avoid too much smoothing with 'zv', suggested max_accel <= 5900 mm/sec^2
Fitted shaper 'mzv' frequency = 36.8 Hz (vibrations = 1.7%, smoothing ~= 0.150)
To avoid too much smoothing with 'mzv', suggested max_accel <= 4000 mm/sec^2
Fitted shaper 'ei' frequency = 36.6 Hz (vibrations = 2.2%, smoothing ~= 0.240)
To avoid too much smoothing with 'ei', suggested max_accel <= 2500 mm/sec^2
Fitted shaper '2hump_ei' frequency = 48.0 Hz (vibrations = 0.0%, smoothing ~= 0.234)
To avoid too much smoothing with '2hump_ei', suggested max_accel <= 2500 mm/sec^2
Fitted shaper '3hump_ei' frequency = 59.0 Hz (vibrations = 0.0%, smoothing ~= 0.235)
To avoid too much smoothing with '3hump_ei', suggested max_accel <= 2500 mm/sec^2
Recommended shaper_type_y = mzv, shaper_freq_y = 36.8 Hz
```
Si vous êtes d'accord avec les paramètres suggérés, vous pouvez exécuter `SAVE_CONFIG` maintenant pour les enregistrer et redémarrer le Klipper. Notez que cela ne sera pas mis à jour Valeur `max_accel` dans la section `[printer]`. Vous devez le mettre à jour manuellement en suivant les considérations dans [Selecting max_accel](#selecting-max_accel) section.


Si votre imprimante est une imprimante bed slinger, vous pouvez spécifier quel axe à tester, de sorte que vous pouvez changer le point de montage de l'accéléromètre entre les tests (par défaut le test est réalisé pour les deux axes) :
```
SHAPER_CALIBRATE AXIS=Y
```

Vous pouvez exécuter `SAVE_CONFIG` deux fois - après avoir calibré chaque axe.

Cependant, si vous avez connecté deux accéléromètres simultanément, vous exécutez simplement `SHAPER_CALIBRATE` sans spécifier d'axe pour calibrer le shaper d'entrée pour les deux axes en une seule fois.

### Input Shaper re-calibration

`SHAPER_CALIBRATE` La commande peut également être utilisée pour recalibrer l’input shaper dans l'avenir, surtout si certaines modifications apportées à l'imprimante peuvent affecter sa cinématique est réalisée. On peut soit relancer l'étalonnage complet en utilisant `SHAPER_CALIBRATE` commande, ou restreindre l'auto-calibrage à un seul axe enapprovisionnement `AXIS=` parameter, Comme
```
SHAPER_CALIBRATE AXIS=X
```

**Attention !** Il est déconseillé d'exécuter l'autocalibrage du shaper très
fréquemment (par exemple avant chaque impression ou tous les jours). Afin de déterminer fréquences de résonance, l'autocalibrage crée des vibrations intenses sur chacun des les axes. Généralement, les imprimantes 3D ne sont pas conçues pour résister à une utilisation prolongée exposition à des vibrations proches des fréquences de résonance. Cela peut augmenter l'usure des composants de l'imprimante et réduire leur durée de vie. Il y a aussi un risque accru que certaines pièces se dévissent ou se desserrent. Vérifiez toujours que toutes les pièces de l'imprimante (y compris celles qui ne bougent normalement pas) sont
solidement fixé en place après chaque réglage automatique.

De plus, en raison d'un certain bruit dans les mesures, il est possible que les résultats de réglage sera légèrement différente d'un cycle d'étalonnage à l'autre. Pourtant, il on ne s'attend pas à ce que le bruit affecte trop la qualité d'impression. Cependant, il est toujours conseillé de revérifier les paramètres suggérés, et imprimez quelques impressions de test avant de les utiliser pour confirmer qu'elles sont bonnes.

## Traitement hors ligne des données de l'accéléromètre

Il est possible de générer les données brutes de l'accéléromètre et de les traiter hors ligne (par exemple sur une machine hôte), par exemple pour trouver des résonances. Afin de le faire, exécutez les commandes suivantes via le terminal Octoprint:
```
SET_INPUT_SHAPER SHAPER_FREQ_X=0 SHAPER_FREQ_Y=0
TEST_RESONANCES AXIS=X OUTPUT=raw_data
```
en ignorant les erreurs pour la commande `SET_INPUT_SHAPER`. Pour `TEST_RESONANCES` commande, spécifiez l'axe de test souhaité. Les données brutes seront écrites dans le répertoire `/tmp` sur le RPi.

Les données brutes peuvent également être obtenues en exécutant la commande
Commande `ACCELEROMETER_MEASURE` deux fois pendant une imprimante normale
activité - d'abord pour démarrer les mesures, puis pour les arrêter et écrire le fichier de sortie. Reportez-vous à [G-Codes] (G-Codes.md#adxl345) pour plus
détails.

Les données peuvent être traitées ultérieurement par les scripts suivants :
`scripts/graph_accelerometer.py` et `scripts/calibrate_shaper.py`. Tous les deux
d'entre eux acceptent un ou plusieurs fichiers csv bruts en entrée selon le
mode. Le script graph_accelerometer.py prend en charge plusieurs modes de fonctionnement :

* traçant les données brutes de l'accéléromètre (utilisez le paramètre `-r`), une seule entrée est   prise en charge;
* tracer une réponse en fréquence (aucun paramètre supplémentaire requis), si plusieurs   les entrées sont spécifiées, la réponse en fréquence moyenne est calculée ;
* comparaison de la réponse en fréquence entre plusieurs entrées (utilisez `-c`   paramètre); vous pouvez en outre spécifier l'axe de l'accéléromètre à considérer via le paramètre `-a x`, `-a y` ou `-a z` (si aucun spécifié, la somme des vibrations pour tous les axes est utilisée);
* tracer le spectrogramme (utilisez le paramètre `-s`), une seule entrée est prise en charge ; vous pouvez en outre spécifier l'axe de l'accéléromètre à prendre en compte via Paramètre `-a x`, `-a y` ou `-a z` (si aucun spécifié, la somme des vibrations pour tous les axes est utilisé).

Notez que le script graph_accelerometer.py ne prend en charge que les fichiers raw_data\*.csv et non les fichiers resonances\*.csv ou calibration_data\*.csv.

Par exemple,
```
~/klipper/scripts/graph_accelerometer.py /tmp/raw_data_x_*.csv -o /tmp/resonances_x.png -c -a z
```
tracera la comparaison de plusieurs fichiers `/tmp/raw_data_x_*.csv` pour l'axe Z au fichier `/tmp/resonances_x.png`.

Le script shaper_calibrate.py accepte 1 ou plusieurs entrées et peut s'exécuter automatiquement réglage du shaper d'entrée et suggérer les meilleurs paramètres qui fonctionnent bien pour toutes les entrées fournies. Il imprime les paramètres suggérés sur la console et peut générer en plus le graphique si le paramètre `-o output.png` est fourni, ou le fichier CSV si le paramètre `-c output.csv` est spécifié.

Fournir plusieurs entrées au script shaper_calibrate.py peut être utile en cas d'exécution certains réglages avancés des shapers d'entrée, par exemple :
* Exécuter `TEST_RESONANCES AXIS=X OUTPUT=raw_data` (et `Y` axe) pour un seul   axe deux fois sur une imprimante de lit slinger avec l'accéléromètre attaché à la tête d'outil la première fois, et l'accéléromètre attaché au lit la   deuxième fois afin de détecter les résonances croisées des axes et tenter d'annuler avec des input shapers.
* Exécuter `TEST_RESONANCES AXIS=Y OUTPUT=raw_data` deux fois sur un porte-lit avec un lit en verre et une surface magnétique (qui est plus légère) pour trouver l'entrée   paramètres de mise en forme qui fonctionnent bien pour n'importe quelle configuration de surface d'impression.
* Combinaison des données de résonance de plusieurs points de test.
* Combinaison des données de résonance à partir de 2 axes (par exemple sur une imprimante de lit slinger pour configurer X-axis input_shaper à partir des résonances des axes X et Y pour annuler les vibrations du * lit * au cas où la buse "attrape" une impression lorsque se déplaçant dans la direction de l'axe X).
