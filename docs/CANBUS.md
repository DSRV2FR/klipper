# CANBUS

Ce document décrit la prise en charge du bus CAN de Klipper.

## Device Hardware

Klipper prend actuellement en charge CAN sur les puces stm32 et rp2040. En outre, la puce du microcontrôleur doit être sur une carte dotée d'un CAN
émetteur-récepteur.

Pour compiler pour CAN, lancez `make menuconfig` et sélectionnez "CAN bus" comme
Interface de Communication. Enfin, compilez le code du microcontrôleur et flashez-le sur le tableau cible.

## Matériel hôte

Pour utiliser un bus CAN, il est nécessaire d'avoir un adaptateur hôte. Il existe actuellement deux options courantes :

1. Utiliser un [Chapeau Waveshare Raspberry Pi CAN] (https://www.waveshare.com/rs485-can-hat.htm) ou l'un de ses nombreux clones.

2. Utilisez un adaptateur USB CAN (par exemple    [https://hacker-gadgets.com/product/cantact-usb-can-adapter/](https://hacker-gadgets.com/product/cantact-usb-can-adapter/)). Là de nombreux adaptateurs USB vers CAN sont disponibles - lors du choix un, nous vous recommandons de vérifier qu'il peut exécuter le [candlelight firmware]https://github.com/candle-usb/candleLight_fw). (Malheureusement, nous avons trouvé des adaptateurs USB défectueux firmware et sont verrouillés, vérifiez donc avant d'acheter.)

Il est également nécessaire de configurer le système d'exploitation hôte pour utiliser le adaptateur. Cela se fait généralement en créant un nouveau fichier nommé `/etc/network/interfaces.d/can0` avec le contenu suivant:
```
auto can0
iface can0 can static
    bitrate 500000
    up ifconfig $IFACE txqueuelen 128
```

Notez que le "chapeau Raspberry Pi CAN" nécessite également
[modifications apportées à config.txt](https://www.waveshare.com/wiki/RS485_CAN_HAT).

## Résistances de terminaison

Un bus CAN doit avoir deux résistances de 120 ohms entre CANH et CANL fils. Idéalement, une résistance située à chaque extrémité du bus.

Notez que certains appareils ont une résistance intégrée de 120 ohms (par exemple, le "Waveshare Raspberry Pi CAN hat" a une résistance soudée qui
ne peut pas être retiré facilement). Certains appareils n'incluent pas de résistance à tout. D'autres appareils ont un mécanisme pour sélectionner la résistance (généralement en connectant un "cavalier à broches"). Assurez-vous de vérifier les schémas de tous périphériques sur le bus CAN pour vérifier qu'il y en a deux et seulement deux 120 Résistances Ohm sur le bus.

Pour tester que les résistances sont correctes, on peut couper l'alimentation du
imprimante et utilisez un multimètre pour vérifier la résistance entre le CANH
et fils CANL - il devrait signaler ~ 60 ohms sur un CAN correctement câblé bus.

## Trouver le canbus_uuid pour les nouveaux microcontrôleurs

Chaque microcontrôleur sur le bus CAN se voit attribuer un identifiant unique basé sur l'identifiant de la puce d'usine encodé dans chaque microcontrôleur. À
trouver chaque identifiant de périphérique de microcontrôleur, assurez-vous que le matériel est alimenté et câblé correctement, puis exécutez :
```
~/klippy-env/bin/python ~/klipper/scripts/canbus_query.py can0
```

Si des appareils CAN non initialisés sont détectés, la commande ci-dessus lignes de rapport comme les suivantes:
```
Found canbus_uuid=11aa22bb33cc, Application: Klipper
```

Chaque appareil aura un identifiant unique. Dans l'exemple ci-dessus, `11aa22bb33cc` est le "canbus_uuid" du microcontrôleur.

Notez que l'outil `canbus_query.py` ne signalera que les fichiers non initialisés périphériques - si Klipper (ou un outil similaire) configure le périphérique, il n'apparaîtra plus dans la liste.

## Configuration Klipper

Mettez à jour le Klipper [configuration mcu](Config_Reference.md#mcu) pour utiliser le bus CAN pour communiquer avec l'appareil - par exemple:
```
[mcu my_can_mcu]
canbus_uuid: 11aa22bb33cc
```

## Mode pont USB vers bus CAN

Certains microcontrôleurs prennent en charge la sélection du mode "pont USB vers bus CAN" pendant "make menuconfig". Ce mode peut permettre d'utiliser un microcontrôleur à la fois comme "adaptateur de bus USB vers CAN" et comme Klipper nœud. 

Lorsque Klipper utilise ce mode, le microcontrôleur apparaît comme un "USB CAN
adaptateur de bus" sous Linux. Le "Klipper bridge mcu" lui-même apparaîtra comme s'il était sur ce bus CAN - il peut être identifié via `canbus_query.py` et configuré comme les autres nœuds Klipper du bus CAN. Il apparaîtra aux côtés d'autres appareils qui sont réellement sur le bus CAN.

Quelques notes importantes lors de l'utilisation de ce mode:

* Le "bridge mcu" n'est pas réellement sur le bus CAN. Messages à et de celle-ci ne consomment pas de bande passante sur le bus CAN. Le mcu ne peut pas être vu par d'autres adaptateurs qui peuvent être sur le bus CAN.

* Il est nécessaire de configurer l'interface `can0` (ou similaire) dans Linux pour communiquer avec le bus. Cependant, le bus CAN Linux les options de vitesse et de synchronisation des bits du bus CAN sont ignorées par Klipper. Actuellement, la fréquence du bus CAN est spécifiée pendant "make menuconfig" et la vitesse du bus spécifiée sous Linux est ignorée.

* Chaque fois que le "bridge mcu" est réinitialisé, Linux désactivera le l'interface `can0` correspondante. Pour assurer une bonne manipulation des commandes FIRMWARE_RESTART et RESTART, il est recommandé de remplacer `auto` avec `allow-hotplug` dans `/etc/network/interfaces.d/can0`dossier. Par exemple:
```
allow-hotplug can0
iface can0 can static
    bitrate 500000
    up ifconfig $IFACE txqueuelen 128
```
