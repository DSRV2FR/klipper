# Exemples de configuration

Tce document contient des directives pour contribuer un exemple Klipper
configuration au dépôt github de Klipper (situé dans le
[répertoire de configuration](../config/)).

Notez que le 
[Klipper Community Discourse server](https://community.klipper3d.org)
est également une ressource utile pour rechercher et partager des fichiers de configuration.

## Guidelines

1. Sélectionnez le préfixe de nom de fichier de configuration approprié:
   1. Le préfixe "imprimante" est utilisé pour les imprimantes de stock vendues par un fabricant grand public.
   2. Le préfixe "générique" est utilisé pour une carte d'imprimante 3D qui peut être utilisée dans de nombreux types d'imprimantes.
   3. Le préfixe "kit" est destiné aux imprimantes 3D qui sont assemblées selon une spécification largement utilisée. Ces imprimantes "en kit" sont généralement distinctes des "imprimantes" normales en ce sens qu'elles ne sont pas vendues par un fabricant.
   4. Le préfixe "sample" est utilisé pour les "extraits" de configuration que l'on peut copier-coller dans le fichier de configuration principal.
   5. Le préfixe "exemple" est utilisé pour décrire la cinématique de l'imprimante. Ce type de configuration n'est généralement ajouté qu'avec le code d'un nouveau type de cinématique d'imprimante.
2. Tous les fichiers de configuration doivent se terminer par un suffixe `.cfg`. Les fichiers de configuration `printer` doivent se terminer par une année suivie de `.cfg` (par exemple, `-2019.cfg`). Dans ce cas, l'année est une année approximative de vente de l'imprimante donnée.
3. N'utilisez pas d'espaces ou de caractères spéciaux dans le nom du fichier de configuration. Le nom de fichier ne doit contenir que les caractères `A-Z`, `a-z`, `0-9`, `-` et `.`.
4. Klipper doit pouvoir démarrer `printer`, `generic` et `kit`exemple de fichier de configuration sans erreur. Ces fichiers de configuration doivent être ajoutés au cas de test de régression [test/klippy/printers.test](../test/klippy/printers.test). Ajoutez de nouveaux fichiers de configuration à ce scénario de test dans la section appropriée et par ordre alphabétique dans cette section.
5. L'exemple de configuration doit être pour la configuration "stock" de l'imprimante. (Il y a trop de configurations "personnalisées" à suivre dans le référentiel principal de Klipper.) utilisation active). Envisagez d'utiliser le [serveur Klipper Community Discourse](https://community.klipper3d.org) pour d'autres configurations.
6. Spécifiez uniquement les périphériques présents sur l'imprimante ou la carte donnée. Ne spécifiez pas de paramètres spécifiques à votre configuration particulière.
   1. Pour les fichiers de configuration "génériques", seuls les périphériques de la carte mère doivent être décrits. Par exemple, cela n'aurait aucun sens d'ajouter une section de configuration d'affichage à une configuration "générique" car il n'y a aucun moyen de savoir si la carte sera attachée à ce type d'affichage. Si la carte a un port matériel spécifique pour faciliter un périphérique optionnel (par exemple, un port bltouch), alors on peut ajouter une section de configuration "commentée" pour le périphérique donné.
   2. Ne spécifiez pas `pressure_advance` dans un exemple de configuration, car cette valeur est spécifique au filament, pas au matériel de l'imprimante. De même, ne spécifiez pas les paramètres `max_extrude_only_velocity` ni `max_extrude_only_accel`.
   3. Ne spécifiez pas une section de configuration contenant un chemin d'hôte ou un matériel hôte. Par exemple, ne spécifiez pas les sections de configuration `[virtual_sdcard]` ni `[temperature_host]`.
   4. Définissez uniquement les macros qui utilisent des fonctionnalités spécifiques à l'imprimante donnée ou pour définir les g-codes qui sont couramment émis par les slicers configurés pour l'imprimante donnée.
7. Dans la mesure du possible, il est préférable d'utiliser le même libellé, la même formulation, la même indentation et le même ordre de section que les fichiers de configuration existants.
   1. Le haut de chaque fichier de configuration doit indiquer le type de microcontrôleur que l'utilisateur doit sélectionner lors de "make menuconfig". Il doit également faire référence à
      "docs/Config_Reference.md".
   2. Ne copiez pas la documentation de terrain dans les exemples de fichiers de configuration. (Cela crée un fardeau de maintenance car une mise à jour de la documentation nécessiterait alors de la modifier à de nombreux endroits.)
   3. Les exemples de fichiers de configuration ne doivent pas contenir de section "SAVE_CONFIG". Si nécessaire, copiez les champs pertinents de la section SAVE_CONFIG vers la section appropriée dans la zone de configuration principale.
   4. Utilisez la syntaxe `champ : valeur` au lieu de `champ=valeur`.
   5. Lors de l'ajout d'une extrudeuse `rotation_distance`, il est préférable de spécifier un `gear_ratio` si l'extrudeuse a un mécanisme d'engrenage. Nous nous attendons à ce que la rotation_distance dans les exemples de configuration soit en corrélation avec la circonférence de l'engrenage à la fraise dans l'extrudeuse - elle est normalement comprise entre 20 et 35 mm. Lorsque vous spécifiez un `gear_ratio`, il est préférable de spécifier les engrenages réels sur le mécanisme (par exemple, préférez `gear_ratio: 80:20` plutôt que `gear_ratio: 4:1`). Voir le [document sur la distance de rotation] (Rotation_Distance.md#using-a-gear_ratio) pour plus d'informations.
   6. Évitez de définir des valeurs de champ qui sont définies sur leur valeur par défaut. Par exemple, il ne faut pas spécifier `min_extrude_temp: 170` car c'est déjà la valeur par défaut.
   7. Dans la mesure du possible, les lignes ne doivent pas dépasser 80 colonnes.
   8. Évitez d'ajouter des messages d'attribution ou de révision aux fichiers de configuration. (Par exemple, évitez d'ajouter des lignes comme "ce fichier a été créé par ...".) Placez l'attribution et l'historique des modifications dans le message de validation git.
8. N'utilisez aucune fonctionnalité obsolète dans l'exemple de fichier de configuration.
9. Ne désactivez pas un système de sécurité par défaut dans un exemple de fichier de configuration. Par exemple, une configuration ne doit pas spécifier une `max_extrude_cross_section` personnalisée. N'activez pas les fonctionnalités de débogage. Par exemple, il ne devrait pas y avoir de section de configuration `force_move`.
10. Toutes les cartes connues prises en charge par Klipper peuvent utiliser le débit en bauds série par défaut de 250 000. Ne recommandez pas un débit en bauds différent dans un exemple de fichier de configuration. </div><div class="links-container"><ul><li><a href="https://www.google.com/m?hl=fr">Accueil Google</a></li><li><a href="https://www.google.com/tools/feedback/survey/xhtml?productId=95112&hl=fr">Envoyer des commentaires</a></li><li><a href="https://www.google.com/intl/fr/policies">Confidentialité et conditions d'utilisation</a></li><li><a href="./full">Accéder au site complet</a></li></ul>

Des exemples de fichiers de configuration sont soumis en créant une "pull request" github. Veuillez également suivre les instructions du [document de contribution](CONTRIBUTING.md).
