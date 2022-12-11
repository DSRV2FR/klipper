# Compensation de résonance

Klipper prend en charge Input Shaping - une technique qui peut être utilisée pour réduire la résonance (également connu sous le nom d'écho, d'image fantôme ou d'ondulation) dans les impressions. La résonance est un défaut de surface d'impression lorsque, généralement, des éléments comme les bords se répètent sur une surface imprimée comme un subtil 'echo':

|![Test de sonnerie](img/ringing-test.jpg)|![3D Benchy](img/ringing-3dbenchy.jpg)|

La résonnance est causée par des vibrations mécaniques dans l'imprimante dues à des changements rapides du sens d'impression. A noter que la résonnance a généralement des origines mécaniques : cadre d'imprimante insuffisamment rigide, courroies non tendues ou trop élastiques, problèmes d’ alignement  de pièces mécaniques, masse lourde en mouvement, etc. Ceux-ci doivent être vérifiés
et fixé en premier, si possible.


[Input shaping](https://en.wikipedia.org/wiki/Input_shaping) est une boucle ouverte technique de contrôle qui crée un signal de commande qui annule ses
propres vibrations. La mise en forme des entrées nécessite quelques réglages et mesures avant d’être activé. Outre la résonnance, Input Shaping réduit généralement les vibrations et les secousses de l'imprimante en général, et peuvent également améliorer la fiabilité du mode stealthChop des pilotes pas à pas Trinamic.

## Réglage

Le réglage de base nécessite de mesurer les fréquences de résonnace de l'imprimante en imprimant un modèle de test.

Découpez le modèle de test de résonnance, qui se trouve dans
[docs/prints/ringing_tower.stl](prints/ringing_tower.stl), dans la trancheuse :

* La hauteur de couche suggérée est 0.2 or 0.25 mm.
* Les couches de remplissage et supérieures peuvent être définies sur 0.
* Utilisez 1-2 périmètres, ou encore mieux le mode vase lisse avec une base de 1-2 mm.
* Utilisez une vitesse suffisamment élevée, environ 80-100 mm/sec, pour les périmètres **externes**.
* Assurez-vous que le temps de couche minimum est ** au plus ** 3 secondes.
* Assurez-vous que tout "contrôle d'accélération dynamique" est désactivé dans le slicer.
* Ne tournez pas le modèle. Le modèle a des marques X et Y à l'arrière du modèle. Notez l'emplacement inhabituel des marques par rapport aux axes de l'imprimante – Ce n’est pas une erreur. Les marques peuvent être utilisées plus tard dans le processus de réglage comme une référence, car ils indiquent à quel axe correspondent les mesures.

### Fréquence de résonnance

Tout d'abord, mesurez la **fréquence de résonnance**.

1. Si le paramètre `square_corner_velocity` a été modifié, rétablissez-le
   à 5.0. Il n'est pas conseillé de l'augmenter lors de l'utilisation du shaper input car cela peut causer plus de lissage dans certaines parties - il est préférable d'utiliser la valeur d'accélération plus élevée à la place.
2. Augmentez `max_accel_to_decel` en exécutant la commande suivante :
   `SET_VELOCITY_LIMIT ACCEL_TO_DECEL=7000`
3. Désactiver l'avance de pression : `SET_PRESSURE_ADVANCE ADVANCE=0`
4. Si vous avez déjà ajouté la section `[input_shaper]` au fichier printer.cfg,
   exécutez la commande `SET_INPUT_SHAPER SHAPER_FREQ_X=0 SHAPER_FREQ_Y=0`. Si vous obtenez l'erreur "Commande inconnue", vous pouvez l'ignorer en toute sécurité à ce stade et continuer les mesures.
5. Exécutez la commande :
`COMMANDE TUNING_TOWER=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5`
Fondamentalement, nous essayons de rendre la résonance plus prononcée en définissant différentes grandes valeurs d'accélération. Cette commande augmentera l'accélération tous les 5 mm à partir de 1500 mm/sec^2 : 1500 mm/sec^2, 2000 mm/sec^2, 2500 mm/sec^2 et ainsi de suite jusqu'à 7000 mm/sec^2 à la dernière bande.
6. Imprimez le modèle de test tranché avec les paramètres suggérés.
7. Vous pouvez arrêter l'impression plus tôt si la résonance est clairement visible et que vous voyez que l'accélération devient trop élevée pour votre imprimante (par exemple, l'imprimante tremble beaucoup trop ou commence à sauter des étapes).
8. Utilisez les marques X et Y à l'arrière du modèle pour référence. Les mesures
   du côté avec la marque X doit être utilisé pour l'axe X *configuration*, et
   Marque Y - pour la configuration de l'axe Y. Mesurer la distance *D* (en mm) entre plusieurs oscillations sur la partie avec repère X, près des encoches, de préférence sauter la première ou les deux premières oscillations. Pour mesurer la distance entre scillations plus facilement, marquez d'abord les oscillations, puis mesurez distance entre les marques avec une règle ou des étriers :

|![Mark ringing](img/ringing-mark.jpg)|![Measure ringing](img/ringing-measure.jpg)|

9. Comptez à combien d'oscillations *N* correspond la distance mesurée *D*.
   Si vous ne savez pas comment compter les oscillations, reportez-vous à l'image ci-dessus, qui montre *N* = 6 oscillations.
10. Calculez la fréquence de résonance de l'axe X comme *V* &middot; *N* / *D* (Hz), où *V* est la vitesse pour les périmètres extérieurs (mm/sec). Pour l'exemple ci-dessus, nous avons marqué 6 oscillations, et le test a été imprimé à 100 mm/sec vitesse, donc la fréquence est 100 * 6 / 12,14 ≈ 49,4 Hz.
11. Faites (8) - (10) pour la marque Y également.

Notez que la sonnerie sur le test d'impression doit suivre le motif de la courbe
encoches, comme sur la photo ci-dessus. Si ce n'est pas le cas, alors ce défaut n'est pas vraiment une résonance et a une origine différente - soit mécanique, soit un problème d'extrudeuse. Il doit être corrigé avant d'activer et de régler les  inputs shapers.

Si les mesures ne sont pas fiables parce que, par exemple, la distance
entre les oscillations n'est pas stable, cela peut signifier que l'imprimante a
plusieurs fréquences de résonance sur le même axe. On peut essayer de suivre le
processus de réglage décrit dans [Unreliable measurements of ringing frequencies](#unreliable-measurements-of-ringing-frequencies) section à la place et toujours obtenir quelque chose de la technique de mise en forme d'entrée.

La fréquence de résonance peut dépendre de la position du modèle dans la plaque de construction et hauteur Z, *en particulier sur les imprimantes delta* ; vous pouvez vérifier si vous voyez le différences de fréquences à différentes positions le long des côtés du test modèle et à différentes hauteurs. Vous pouvez calculer la résonance moyenne fréquences sur les axes X et Y si tel est le cas.

Si la fréquence de résonance mesurée est très faible (inférieure à environ 20-25 Hz), il se peut être une bonne idée d'investir dans le renforcement de l'imprimante ou la diminution du déplacement masse - selon ce qui est applicable dans votre cas - avant de procéder à un réglage supplémentaire de la mise en forme de l'entrée et une nouvelle mesure des fréquences par la suite. Pour
de nombreux modèles d'imprimantes populaires, il existe souvent des solutions déjà disponibles.
Notez que les fréquences de résonance peuvent changer si les modifications sont apportées à l’imprimante qui affectent la masse en mouvement ou modifient la rigidité du système,par exemple:

* Certains outils sont installés, retirés ou remplacés sur la tête d'outil qui changent sa masse, par ex. un nouveau moteur pas à pas (plus lourd ou plus léger) pour extrudeuse directe ou un nouveau hotend est installé, un ventilateur lourd avec un conduit est ajouté, etc.
* Les ceintures sont serrées.
* Certains ajout pour augmenter la rigidité du cadre sont installés.
* Un lit différent est installé sur une imprimante bed-slinger, ou du verre est ajouté, etc.

Si de tels changements sont faits, c'est une bonne idée de mesurer au moins la résonance fréquences pour voir si elles ont changé.

### Formateur d'entrée de configuration

Une fois les fréquences de résonance pour les axes X et Y mesurées, vous pouvez ajouter le section suivante à votre `printer.cfg` :
```
[input_shaper]
shaper_freq_x: ... # fréquence pour la marque X du modèle de test
shaper_freq_y: ... # fréquence pour la marque Y du modèle de test
```

Pour l'exemple ci-dessus, nous obtenons shaper_freq_x/y = 49,4.

### Choisir input shaper

Klipper prend en charge plusieurs shapers d'entrée. Ils diffèrent par leur sensibilité à erreurs déterminant la fréquence de résonance et la quantité de lissage qu'elles provoquent dans les pièces imprimées. De plus, certains des shapers comme 2HUMP_EI et 3HUMP_EI ne doivent généralement pas être utilisés avec shaper_freq = fréquence de résonance - ils sont configuré à partir de différentes considérations pour réduire plusieurs résonances à la fois.

Pour la plupart des imprimantes, les shapers MZV ou EI peuvent être recommandés. Cette section décrit un processus de test pour choisir entre eux, et comprendre
quelques autres paramètres connexes.

Imprimez le modèle de test de résonance comme suit :

1. Redémarrez le micrologiciel: `RESTART`
2. Préparez-vous pour le test: `SET_VELOCITY_LIMIT ACCEL_TO_DECEL=7000`
3. Désactiver l'avance de pression: `SET_PRESSURE_ADVANCE ADVANCE=0`
4. Exécuter: `SET_INPUT_SHAPER SHAPER_TYPE=MZV`
5. Exécutez la commande:
`COMMANDE TUNING_TOWER=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5`
6. Imprimez le modèle de test tranché avec les paramètres suggérés.

Si vous ne voyez pas de résonance à ce stade, l'utilisation du shaper MZV peut être recommandée.

Si vous voyez des résonance, mesurez à nouveau les fréquences en suivant les étapes (8) à (10) décrit dans la section [Fréquence de résonance](#ringing-frequency). Si les fréquences diffèrent considérablement des valeurs que vous avez obtenues précédemment, une entrée plus complexe de configuration de l’input shaper est nécessaire. Vous pouvez vous référer aux détails techniques de 
Section [Formateurs d'entrée](#input-shapers). Sinon, passez à l'étape suivante.

N'essayez pas l’input shaper EI. Pour l'essayer, répétez les étapes (1) à (6) ci-dessus, mais exécutant à l'étape 4 la commande suivante à la place :
`SET_INPUT_SHAPER SHAPER_TYPE=EI`.

Comparez  les deux impressions MZV et EI input shaper. Si l'IE s'améliore sensiblement le résultats que MZV, utilisez EI shaper, sinon préférez MZV. Notez que le shaper EI provoquer plus de lissage dans les pièces imprimées (voir la section suivante pour plus détails). Ajoutez le paramètre `shaper_type: mzv` (ou ei) à la section [input_shaper], par exemple.:
```
[input_shaper]
shaper_freq_x : ...
shaper_freq_y : ...
shaper_type : mzv
```

Quelques notes sur la sélection du shaper :

* Le shaper EI peut être plus adapté aux imprimantes de lit slinger (si la résonance fréquence et le lissage résultant le permet) : plus le filament est déposé sur le lit mobile, la masse du lit augmente et la fréquence de résonance
 diminuera. Étant donné que le shaper EI est plus robuste à la fréquence de résonance changements, cela peut mieux fonctionner lors de l'impression de grandes pièces.
* En raison de la nature de la cinématique delta, les fréquences de résonance peuvent différer d'un lot dans différentes parties du volume de construction. Par conséquent, EI shaper peut être un mieux adapté aux imprimantes delta plutôt qu'aux MZV ou ZV, et devrait être considéré pour l'utilisation. Si la fréquence de résonance est suffisamment grande (plus de 50-60 Hz), alors on peut même tenter de tester 2HUMP_EI shaper (en exécutant le test suggéré ci-dessus avec
 `SET_INPUT_SHAPER SHAPER_TYPE=2HUMP_EI`), mais vérifiez les considérations dans
 la [section ci-dessous] (#selecting-max_accel) avant de l'activer.

### Sélection max_accel

Vous devriez avoir un test imprimé pour le shaper que vous avez choisi à l'étape précédente (si vous ne le faites pas, imprimez le modèle de test découpé avec le
[paramètres suggérés](#tuning) avec l'avance de pression désactivée `SET_PRESSURE_ADVANCE ADVANCE=0` et avec la tour de réglage activée comme
`COMMANDE TUNING_TOWER=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5`). Notez qu'à de très fortes accélérations, en fonction de la fréquence de résonance et l’input shaper que vous avez choisi (par exemple, le shaper EI crée plus de lissage que MZV), la mise en forme d'entrée peut provoquer trop de lissage et d'arrondi des pièces. Alors,
max_accel doit être choisi de manière à empêcher cela. Un autre paramètre qui peut impacter le lissage est `square_corner_velocity`, il n'est donc pas conseillé d'augmenter au-dessus de la valeur par défaut de 5 mm/s pour éviter un lissage accru.

Afin de sélectionner une valeur max_accel appropriée, inspectez le modèle pour l’input shapper. Tout d'abord, notez à quel moment la résonance d'accélération est encore faible - que vous êtes à l'aise avec cela.

Ensuite, vérifiez le lissage. Pour aider à cela, le modèle de test a un petit écart dans le mur (0,15 mm) :

![Ecart de test](img/smoothing-test.png)

À mesure que l'accélération augmente, le lissage augmente également et l'écart réel dans
l'impression s'élargit :
![Lissage du shaper](img/shaper-smoothing.jpg)

Sur cette image, l'accélération augmente de gauche à droite et l'écart commence
a croître à partir de 3500 mm/sec^2 (5ème bande à partir de la gauche). Alors la bonne valeur pour max_accel = 3000 (mm/sec^2) dans ce cas pour éviter l'excès
lissage.
Notez l'accélération lorsque l'écart est encore très faible dans votre test d'impression. Si vous voyez des renflements, mais aucun espace dans le mur, même à de fortes accélérations,cela peut être dû à une avance de pression désactivée, en particulier sur les extrudeuses Bowden. Si tel est le cas, vous devrez peut-être répéter l'impression avec le PA activé. Cela peut également être le résultat d'un flux de filament mal calibré (trop élevé), il est donc
une bonne idée de vérifier cela aussi.

Choisissez le minimum parmi les deux valeurs d'accélération (de résonance et
lissage), et mettez-le comme `max_accel` dans printer.cfg.


Notez qu'il peut arriver - en particulier à des fréquences de résonance basses - que l'EI shaper provoquera trop de lissage même à des accélérations plus faibles. Dans ce cas, MZV peut être un meilleur choix, car il peut permettre des valeurs d'accélération plus élevées.

À des fréquences de résonance très basses (~ 25 Hz et moins), même le shaper MZV peut créer trop de lissage. Si tel est le cas, vous pouvez également essayer de répéter la étapes dans la section [Choosing input shaper] (#choosing-input-shaper) avec ZV shaper, en utilisant la commande `SET_INPUT_SHAPER SHAPER_TYPE=ZV` à la place. Le façonneur ZV devrait montre encore moins de lissage que MZV, mais est plus sensible aux erreurs de mesure les fréquences de résonance.

Une autre considération est que si une fréquence de résonance est trop faible (inférieure à 20-25Hz), il peut être judicieux d'augmenter la rigidité de l'imprimante ou de réduire masse en mouvement. Sinon, l'accélération et la vitesse d'impression peuvent être limitées en raison aussi beaucoup de lissage maintenant au lieu de résonner.


### Réglage précis des fréquences de résonance

Notez que la précision des mesures de fréquences de résonance à l'aide de la
le modèle de test de résonance est suffisant pour la plupart des besoins, donc un réglage supplémentaire n'est pas informé. Si vous voulez toujours essayer de revérifier vos résultats (par exemple, si vous voyez encore une résonance après l'impression d'un modèle de test avec un input shaper de votre choix avec les mêmes fréquences que vous avez mesurées précédemment), vous pouvez
suivez les étapes de cette section. Notez que si vous voyez sonner à différents
fréquences après avoir activé [input_shaper], cette section ne vous aidera pas.

En supposant que vous avez découpé le modèle de résonance avec les
 paramètres suggéré, effectuez les étapes suivantes pour chacun des axes X et Y :

1. Préparez-vous pour le test : `SET_VELOCITY_LIMIT ACCEL_TO_DECEL=7000`
2. Assurez-vous que Pressure Advance est désactivé : `SET_PRESSURE_ADVANCE ADVANCE=0`
3. Exécutez : `SET_INPUT_SHAPER SHAPER_TYPE=ZV`
4. À partir du modèle de test de résonance existant avec le modeleur d'entrée choisi, sélectionnez l'accélération qui résonne suffisamment bien, et réglez-la avec : `SET_VELOCITY_LIMIT ACCEL=...`
5. Calculez les paramètres nécessaires pour que la commande `TUNING_TOWER` règle
paramètre `shaper_freq_x` comme suit : start = shaper_freq_x * 83 / 132 et
factor = shaper_freq_x / 66, où `shaper_freq_x` est ici la valeur actuelle
dans `printer.cfg`.
6. Exécutez la commande : 
`TUNING_TOWER COMMAND=SET_INPUT_SHAPER PARAMETER=SHAPER_FREQ_X START=start FACTOR=factor BAND=5` en utilisant les valeurs "début" et "facteur" calculées à l'étape (5).
7. Imprimez le modèle de test.
8. Réinitialisez la valeur de fréquence d'origine :
`SET_INPUT_SHAPER SHAPER_FREQ_X=...`.
9. Trouvez la bande qui résonne le moins et comptez son numéro à partir du bas à partir de 1.
10. Calculez la nouvelle valeur shaper_freq_x via l'ancienne shaper_freq_x * (39 + 5 * # numéro de bande) / 66.

Répétez ces étapes pour l'axe Y de la même manière, en remplaçant les références à X axe avec l'axe Y (par exemple, remplacer `shaper_freq_x` par `shaper_freq_y` dans les formules et dans la commande `TUNING_TOWER`).

Par exemple, supposons que vous ayez mesuré la fréquence de sonnerie d'un de l'axe égal à 45 Hz. Cela donne start = 45 * 83 / 132 = 28,30 et facteur = 45/66 = 0,6818 valeurs pour la commande `TUNING_TOWER`. Supposons maintenant qu'après l'impression du modèle de test, la quatrième bande du bas donne le moins de sonnerie. Cela donne le shaper_freq_ mis à jour? Évaluer égal à 45 * (39 + 5 * 4) / 66 ≈ 40,23.

Après que les deux nouveaux paramètres `shaper_freq_x` et `shaper_freq_y` ont été calculé, vous pouvez mettre à jour la section `[input_shaper]` dans `printer.cfg` avec les nouvelles valeurs `shaper_freq_x` et `shaper_freq_y`.

### Avance de pression

Si vous utilisez Pressure Advance, il peut être nécessaire de le réajuster. Suivre l’[instructions](Pressure_Advance.md#tuning-pressure-advance) pour trouver la nouvelle valeur, si elle diffère de la précédente. Assurez-vous de
redémarrez Klipper avant de régler Pressure Advance.

### Mesures non fiables des fréquences de résonance

Si vous ne parvenez pas à mesurer les fréquences de sonnerie, par ex. si la distance entre les oscillations n'est pas stable, vous pouvez toujours profiter
des techniques de mise en forme d'entrée, mais les résultats peuvent ne pas être aussi bons qu'avec une bonne mesures des fréquences, et nécessitera un peu plus de réglage et d'impression le modèle d'essai. Notez qu'une autre possibilité est d'acheter et d'installer un l'accéléromètre et mesurer les résonances avec (se référer au [docs](Measuring_Resonances.md) décrivant le matériel requis et la configuration processus) - mais cette option nécessite un peu de sertissage et de soudure.


Pour le réglage, ajoutez une section `[input_shaper]` vide à votre `imprimante.cfg`. Ensuite, en supposant que vous avez découpé le modèle de résonance avec les paramètres suggérés, imprimez le modèle de test 3 fois comme
suit. Première fois, avant l'impression, exécutez

1. `RESTART`
2. `SET_VELOCITY_LIMIT ACCEL_TO_DECEL=7000`
3. `SET_PRESSURE_ADVANCE AVANCE=0`
4. `SET_INPUT_SHAPER SHAPER_TYPE=2HUMP_EI SHAPER_FREQ_X=60 SHAPER_FREQ_Y=60`
5. `COMMANDE TUNING_TOWER=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5`

et imprimez le modèle. Ensuite, imprimez à nouveau le modèle, mais avant d'imprimer, exécutez à la place

1. `SET_INPUT_SHAPER SHAPER_TYPE=2HUMP_EI SHAPER_FREQ_X=50 SHAPER_FREQ_Y=50`
2. `COMMANDE TUNING_TOWER=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5`

Imprimez ensuite le modèle pour la 3ème fois, mais exécutez maintenant

1. `SET_INPUT_SHAPER SHAPER_TYPE=2HUMP_EI SHAPER_FREQ_X=40 SHAPER_FREQ_Y=40`
2. `COMMANDE TUNING_TOWER=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5`

Essentiellement, nous imprimons le modèle de test de sonnerie avec TUNING_TOWER en utilisant 2HUMP_EI shaper avec shaper_freq = 60 Hz, 50 Hz et 40 Hz.

Si aucun des modèles ne démontre d'amélioration de la sonnerie, alors, malheureusement, il ne semble pas que les techniques de mise en forme des entrées puissent vous aider dans votre cas.

Sinon, il se peut que tous les modèles n'affichent aucune résonance, ou que certains affichent la résonance et certains pas tellement. Choisissez le modèle de test avec la fréquence la plus élevée qui montre encore de bonnes améliorations dans la résonance. Par exemple, si les modèles 40 Hz et 50 Hz
ne montre presque aucune résonance, et le modèle 60 Hz montre déjà plus de sonnerie, restez avec 50 Hz.

Vérifiez maintenant si EI shaper serait assez bon dans votre cas. Choisissez le shaper EI fréquence basée sur la fréquence du shaper 2HUMP_EI que vous avez choisi :

* Pour le shaper 2HUMP_EI 60 Hz, utilisez le shaper EI avec shaper_freq = 50 Hz.
* Pour le shaper 2HUMP_EI 50 Hz, utilisez le shaper EI avec shaper_freq = 40 Hz.
* Pour le shaper 2HUMP_EI 40 Hz, utilisez le shaper EI avec shaper_freq = 33 Hz.

Maintenant, imprimez le modèle de test une fois de plus, en exécutant

1. `SET_INPUT_SHAPER SHAPER_TYPE=EI SHAPER_FREQ_X=... SHAPER_FREQ_Y=...`
2. `COMMANDE TUNING_TOWER=SET_VELOCITY_LIMIT PARAMETER=ACCEL START=1500 STEP_DELTA=500 STEP_HEIGHT=5`

fournissant le shaper_freq_x=... et shaper_freq_y=... comme déterminé précédemment.

Si le shaper EI montre de bons résultats très comparables à ceux du shaper 2HUMP_EI, restez avec EI shaper et la fréquence déterminée précédemment, sinon utiliser 2HUMP_EI shaper avec la fréquence correspondante. Ajoutez les résultats à `printer.cfg` comme, par ex.
```
[input_shaper]
shaper_freq_x : 50
shaper_freq_y : 50
shaper_type : 2hump_ei
```

Continuez le réglage avec la section [Selecting max_accel](#selecting-max_accel).


## Dépannage et FAQ

### Je n'arrive pas à obtenir des mesures fiables des fréquences de résonance

Tout d'abord, assurez-vous qu'il ne s'agit pas d'un autre problème avec l'imprimante au lieu de résonance. Si les mesures ne sont pas fiables parce que, par exemple, la distance entre les oscillations n'est pas stable, cela peut signifier que l'imprimante a  plusieurs fréquences de résonance sur le même axe. On peut essayer de suivre le processus de réglage décrit dans [Mesures non fiables des fréquences de résonance](#unreliable-measurements-of-ringing-frequencies) section et toujours obtenir quelque chose de la technique de mise en forme d'entrée. Une autre possibilité est d'installer un accéléromètre, [mesure](Measuring_Resonances.md) les résonances avec lui, et ajustez automatiquement l’input shaper en utilisant les résultats de ces mesures.

### Après avoir activé [input_shaper], j'obtiens des pièces imprimées trop lissées et les détails fins sont perdus

Vérifiez les considérations dans la section [Selecting max_accel](#selecting-max_accel). Si la fréquence de résonance est basse, il ne faut pas régler trop haut max_accel ou augmenter les paramètres square_corner_velocity. Il vaut peut-être aussi mieux choisir input Shapers MZV ou même ZV sur EI (ou shapers 2HUMP_EI et 3HUMP_EI).


### Après avoir réussi à imprimer pendant un certain temps sans sonner, il semble revenir

Il est possible qu'après un certain temps, les fréquences de résonance aient changé. Par exemple. peut-être que la tension des courroies a changé (les courroies sont devenues plus lâches), etc. C'est une bonne idée de vérifier et de remesurer les fréquences de résonance comme décrit dans Section [Fréquence de sonnerie](#ringing-frequency) et mettez à jour votre fichier de configuration
si nécessaire.

### La configuration à double chariot est-elle prise en charge avec lesinputs  shapers ?

Il n'y a pas de support dédié pour les chariots doubles avec des shapers d'entrée, mais c'est le cas ne signifie pas que cette configuration ne fonctionnera pas. Il faut exécuter le réglage deux fois pour chaque des chariots, et calculer les fréquences de sonnerie pour les axes X et Y pour chacune des voitures indépendamment. Ensuite, mettez les valeurs pour le chariot 0 dans la section [input_shaper] et modifiez les valeurs à la volée lors de la modification chariots, par ex. dans le cadre d'une macro:
```
SET_DUAL_CARRIAGE CARRIAGE=1
SET_INPUT_SHAPER SHAPER_FREQ_X=... SHAPER_FREQ_Y=...
```

Et de même lors du retour au chariot 0.

### Input_shaper affecte-t-il le temps d'impression ?

Non, la fonctionnalité `input_shaper` n'a pratiquement aucun impact sur les temps d'impression par lui-même. Cependant, la valeur de `max_accel` le fait certainement (réglage de ce paramètre décrit dans [cette section](#selecting-max_accel)).

## Détails techniques

### Façonneurs d'entrée

Les shapers d'entrée utilisés dans Klipper sont plutôt standard, et on peut trouver plus aperçu détaillé dans les articles décrivant les shapers correspondants. Cette section contient un bref aperçu de certains aspects techniques de la modeleurs d'entrée pris en charge. Le tableau ci-dessous en montre quelques-unes (généralement approximatives)paramètres de chaque shaper.

| Mise en forme <br> d'entrée | Shaper <br> durée | Réduction des vibrations 20x <br> (tolérance aux vibrations de 5 %) | Réduction des vibrations 10x <br> (tolérance aux vibrations de 10 %) |
|:--:|:--:|:--:|:--:|
| ZV | 0.5 / shaper_freq | N/A | ± 5% shaper_freq |
| MZV | 0,75 / shaper_freq | ± 4% shaper_freq | -10%...+15% shaper_freq |
| ZVD | 1 / shaper_freq | ± 15% shaper_freq | ± 22 % shaper_freq |
| IE | 1 / shaper_freq | ± 20% shaper_freq | ± 25 % shaper_freq |
| 2HUMP_EI | 1.5 / shaper_freq | ± 35 % shaper_freq | ± 40 shaper_freq |
| 3HUMP_EI | 2 / shaper_freq | -45...+50% shaper_freq | -50%...+55% shaper_freq |

Remarque sur la réduction des vibrations : les valeurs du tableau ci-dessus sont approximatives. Si le taux d'amortissement de l'imprimante est connu pour chaque axe, le shaper peut être configuré plus précisément et il réduira alors les résonances en un peu plus large gamme de fréquences. Cependant, le taux d'amortissement est généralement inconnu et est difficile pour estimer sans équipement spécial, donc Klipper utilise la valeur 0.1 par défaut, qui est une bonne valeur globale. Les gammes de fréquences du tableau couvrent une nombre de rapports d'amortissement différents possibles autour de cette valeur (environ de 0,05 à 0,2).

Notez également que EI, 2HUMP_EI et 3HUMP_EI sont réglés pour réduire les vibrations à 5 %, les valeurs de tolérance aux vibrations de 10 % sont donc fournies uniquement pour la référence.

**Comment utiliser ce tableau :**

* La durée de Shaper affecte le lissage dans les parties - plus elle est grande, plus lisser les pièces sont. Cette dépendance n'est pas linéaire, mais peut donner une idée de quels shapers "lissent" plus pour la même fréquence. La commande par le lissage est du type : ZV < MZV < ZVD ≈ EI < 2HUMP_EI < 3HUMP_EI. Alors, il est rarement pratique de définir shaper_freq = fréquence de résonance pour les shapers 2HUMP_EI et 3HUMP_EI (il faut les utiliser pour réduire les vibrations pendant plusieurs fréquences).
* On peut estimer une plage de fréquences dans laquelle le shaper réduit
vibrations. Par exemple, MZV avec shaper_freq = 35 Hz réduit les vibrations
à 5 % pour les fréquences [33,6, 36,4] Hz. 3HUMP_EI avec shaper_freq = 50 Hz
réduit les vibrations à 5 % dans la plage [27,5, 75] Hz.
* On peut utiliser ce tableau pour vérifier quel shaper ils doivent utiliser s'ils besoin de réduire les vibrations à plusieurs fréquences. Par exemple, si l'on a résonances à 35 Hz et 60 Hz sur le même axe : a) EI shaper doit avoir
shaper_freq = 35 / (1 - 0,2) = 43,75 Hz, et cela réduira les résonances jusqu'à 43,75 * (1 + 0,2) = 52,5 Hz, donc ce n'est pas suffisant ; b) 2HUMP_EI shaper doit avoir shaper_freq = 35 / (1 - 0,35) = 53,85 Hz et réduire les vibrations jusqu'à 53,85 * (1 + 0,35) = 72,7 Hz - c'est donc un configuration acceptable. Essayez toujours d'utiliser le shaper_freq le plus élevé possible pour un shaper donné (peut-être avec une certaine marge de sécurité, donc dans cet exemple
shaper_freq ≈ 50-52 Hz fonctionnerait mieux), et essayez d'utiliser un shaper avec comme petite durée de mise en forme possible.
* Si l'on a besoin de réduire les vibrations à plusieurs fréquences très différentes (par exemple, 30 Hz et 100 Hz), ils peuvent voir que le tableau ci-dessus ne fournit pas suffisamment d'informations. Dans ce cas, on peut avoir plus de chance avec [scripts/graph_shaper.py](../scripts/graph_shaper.py)
 script, which is more flexible.
