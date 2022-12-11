# Exclure les objets

Le module `[exclude_object]` permet à Klipper d'exclure des objets pendant qu'une impression est en cours. Pour activer cette fonctionnalité, incluez une [section de configuration d'exclude_object](Config_Reference.md#exclude_object) (voir également la [commande de référence](G-Codes.md#exclude-object) et [sample-macros.cfg](../config /sample-macros.cfg) pour une macro M486 G-Code compatible Marlin/RepRapFirmware.)

Contrairement aux autres options de micrologiciel d'imprimante 3D, une imprimante exécutant Klipper utilise une suite de composants et les utilisateurs ont le choix entre de nombreuses options. Par conséquent, afin de fournir une expérience utilisateur cohérente, le module `[exclude_object]` établira une sorte de contrat ou d'API. Le contrat porte sur le contenu de le fichier gcode, comment l'état interne du module est contrôlé et comment cela l'état est fourni aux clients.

## Présentation du flux de travail
Un flux de travail typique pour l'impression d'un fichier peut ressembler à ceci :
1. Le découpage est terminé et le fichier est téléchargé pour l'impression. Pendant le téléchargement, le fichier est traité et les marqueurs `[exclude_object]` sont ajoutés au fichier. Alternativement, les trancheurs peuvent être configurés pour préparer des marqueurs d'exclusion d'objets de manière native, ou dans sa propre étape de prétraitement.
2. Lorsque l'impression démarre, Klipper réinitialisera le `[exclude_object]` [status](Status_Reference.md#exclude_object).
3. Lorsque Klipper traite le bloc `EXCLUDE_OBJECT_DEFINE`, il met à jour le statut avec les objets connus et le transmet aux clients.
4. Le client peut utiliser ces informations pour présenter une interface utilisateur à l'utilisateur afin que les progrès puissent être suivis. Klipper mettra à jour le statut pour inclure l'objet en cours d'impression que le client peut utiliser à des fins d'affichage.
5. Si l'utilisateur demande qu'un objet soit annulé, le client enverra une commande `EXCLUDE_OBJECT NAME=<nom>` à Klipper.
6. Lorsque Klipper traite la commande, il ajoute l'objet à la liste des objets exclus et met à jour le statut du client.
7. Le client recevra le statut mis à jour de Klipper et pourra utiliser ces informations pour refléter le statut de l'objet dans l'interface utilisateur.
8. Une fois l'impression terminée, le statut `[exclude_object]` restera disponible jusqu'à ce qu'une autre action le réinitialise.

## Le fichier GCode
Le traitement gcode spécialisé nécessaire pour prendre en charge l'exclusion d'objets ne correspond pas aux principaux objectifs de conception de Klipper. Par conséquent, ce module exige que le fichier est traité avant d'être envoyé à Klipper pour impression. L'utilisation d'un script de post-traitement dans le slicer ou le fait que le middleware traite le fichier lors du téléchargement sont deux possibilités pour préparer le fichier pour Klipper. Un script de post-traitement de référence est disponible à la fois sous forme d'exécutable et de bibliothèque python, voir
[cancelobject-preprocessor](https://github.com/kageurufu/cancelobject-preprocessor).

### Définitions d'objets

La commande `EXCLUDE_OBJECT_DEFINE` est utilisée pour fournir un résumé de chaque objet dans le fichier gcode à imprimer. Fournit un résumé d'un objet dans le fichier. Les objets n'ont pas besoin d'être définis pour être référencés par d'autres commandes. L'objectif principal de cette commande est de fournir des informations à l'interface utilisateur sans avoir à analyser l'intégralité du fichier gcode.

Les définitions d'objet sont nommées, pour permettre aux utilisateurs de sélectionner facilement un objet à exclure, et des métadonnées supplémentaires peuvent être fournies pour permettre des affichages d'annulation graphique. Les métadonnées actuellement définies incluent une coordonnée X,Y `CENTER` et une liste `POLYGON` de points X,Y représentant un contour minimal de l'objet. Il peut s'agir d'une simple boîte englobante ou d'une coque compliquée pour afficher des visualisations plus détaillées des objets imprimés. Surtout lorsque les fichiers gcode incluent plusieurs parties avec des régions de délimitation qui se chevauchent, les points centraux deviennent difficiles à distinguer visuellement. `POLYGONS` doit être un tableau compatible JSON de tuples ponctuels `[X,Y]` sans espace. Des paramètres supplémentaires seront enregistrés sous forme de chaînes dans la définition de l'objet et fournis dans les mises à jour de statut.

`EXCLUDE_OBJECT_DEFINE NAME=calibration_pyramid CENTER=50,50
POLYGON=[[40,40],[50,60],[60,40]]`

Toutes les commandes G-Code disponibles sont documentées dans la [Référence G-Code](./G-Codes.md#excludeobject)

## Informations d'état
L'état de ce module est fourni aux clients par le [état de l'objet_exclude](Status_Reference.md#objet_exclude).

L'état est réinitialisé lorsque :
- Le firmware Klipper est redémarré.
- Il y a une réinitialisation de la `[virtual_sdcard]`. Notamment, cela est réinitialisé par Klipper au début d'une impression.
- Lorsqu'une commande `EXCLUDE_OBJECT_DEFINE RESET=1` est émise.

La liste des objets définis est représentée dans le champ d'état `exclude_object.objects`. Dans un fichier gcode bien défini, cela sera fait avec les commandes `EXCLUDE_OBJECT_DEFINE` au début du fichier. Cela fournira aux clients les noms et les coordonnées des objets afin que l'interface utilisateur puisse fournir une représentation graphique des objets si nécessaire.

Au fur et à mesure de la progression de l'impression, le champ d'état `exclude_object.current_object` sera mis à jour au fur et à mesure que Klipper traitera les commandes `EXCLUDE_OBJECT_START` et `EXCLUDE_OBJECT_END`. Le champ `current_object` sera défini même si l'objet a été exclu. Les objets non définis marqués d'un `EXCLUDE_OBJECT_START` seront ajoutés aux objets connus pour faciliter l'optimisation de l'interface utilisateur, sans aucune métadonnée supplémentaire.

Lorsque les commandes `EXCLUDE_OBJECT` sont émises, la liste des objets exclus est fournie dans le tableau `exclude_object.excluded_objects`. Étant donné que Klipper anticipe le traitement du gcode à venir, il peut y avoir un délai entre le moment où la commande est émise et le moment où le statut est mis à jour.
