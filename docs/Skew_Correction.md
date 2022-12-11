# Correction d'inclinaison

La correction logicielle de l'inclinaison peut aider à résoudre les inexactitudes dimensionnelles résultant d'un assemblage d'imprimante qui n'est pas parfaitement d'équerre. Noter que si votre imprimante est considérablement asymétrique, il est fortement recommandé de utilisez d'abord des moyens mécaniques pour obtenir votre imprimante aussi carrée que possible avant à l'application d'une correction basée sur un logiciel.

## Imprimer un objet de calibrage

La première étape pour corriger l'inclinaison consiste à imprimer un
[objet d'étalonnage](https://www.thingiverse.com/thing:2563185/files) le long du plan que vous souhaitez corriger. Il y a aussi [objet d'étalonnage](https://www.thingiverse.com/thing:2972743) qui inclut tous les avions dans un modèle. Vous voulez l'orienté objet de sorte que le coin A soit vers l'origine du plan.

Assurez-vous qu'aucune correction d'inclinaison n'est appliquée pendant cette impression. Tu peux faites-le soit en supprimant le module `[skew_correction]` de printer.cfg ou en émettant un gcode `SET_SKEW CLEAR=1`.

## Prenez vos mensurations

Le module `[skew_correcton]` nécessite 3 mesures pour chaque plan que vous voulez corriger; la longueur du coin A au coin C, la longueur du coin B
au coin D, et la longueur du coin A au coin D. Lors de la mesure de la longueur
AD n'incluent pas les méplats sur les coins que certains objets de test fournissent.

![skew_lengths](img/skew_lengths.png)

## Configurez votre inclinaion

Assurez-vous que `[skew_correction]` est dans printer.cfg. Vous pouvez maintenant utiliser le `SET_SKEW` gcode pour configurer skew_correcton. Par exemple, si vos longueurs mesurées le long de XY sont les suivants :

```
Length AC = 140.4
Length BD = 142.8
Length AD = 99.8
```

`SET_SKEW` peut être utilisé pour configurer la correction d'inclinaison pour le plan XY.

```
SET_SKEW XY=140.4,142.8,99.8
```
Vous pouvez également ajouter des mesures pour XZ et YZ au gcode :

```
SET_SKEW XY=140.4,142.8,99.8 XZ=141.6,141.4,99.8 YZ=142.4,140.5,99.5
```

Le module `[skew_correction]` prend également en charge la gestion des profils d'une manière similaire à `[bed_mesh]`. Après avoir défini l'inclinaison à l'aide du gcode "SET_SKEW", vous pouvez utiliser le gcode `SKEW_PROFILE` pour l'enregistrer :

```
SKEW_PROFILE SAVE=my_skew_profile
```
Après cette commande, vous serez invité à émettre un gcode `SAVE_CONFIG` pour
enregistrez le profil dans un stockage persistant. Si aucun profil n'est nommé
`my_skew_profile` alors un nouveau profil sera créé. Si le profil nommé existe, il sera écrasé.

Une fois que vous avez un profil enregistré, vous pouvez le charger:
```
SKEW_PROFILE LOAD=my_skew_profile
```

Il est également possible de supprimer un profil ancien ou périmé :
```
SKEW_PROFILE REMOVE=my_skew_profile
```
Après avoir supprimé un profil, vous serez invité à émettre un "SAVE_CONFIG" pour faire perdurer ce changement.

## Vérification de votre correction

Une fois que skew_correction a été configuré, vous pouvez réimprimer le calibrage pièce avec correction activée. Utilisez le gcode suivant pour vérifier votre obliquer sur chaque plan. Les résultats devraient être inférieurs à ceux rapportés via `GET_CURRENT_SKEW`.

```
CALC_MEASURED_SKEW AC=<ac_length> BD=<bd_length> AD=<ad_length>
```

## Mises en garde

En raison de la nature de la correction de l'inclinaison, il est recommandé de configurer l'inclinaison dans votre gcode de démarrage, après la prise d'origine et tout type de mouvement qui se déplace près du bord de la zone d'impression comme une purge ou un nettoyage de buse. Tu peux utilisez les gcodes `SET_SKEW` ou `SKEW_PROFILE` pour y parvenir. Il est également recommandé d'émettre un `SET_SKEW CLEAR=1` dans votre gcode final.

Gardez à l'esprit qu'il est possible que `[skew_correction]` génère une correction qui déplace l'outil au-delà des limites de l'imprimante sur les axes X et/ou Y. Ce est recommandé d'éloigner les pièces des bords lors de l'utilisation `[skew_correction]`.
