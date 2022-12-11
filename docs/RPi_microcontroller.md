# RPi microcontroller

Ce document décrit le processus d'exécution de Klipper sur un RPi et utilise le même RPi comme mcu secondaire.

## Pourquoi utiliser RPi comme MCU secondaire ?

Souvent les microcontrôleurs dédiés au contrôle des imprimantes 3D disposent d'un nombre limité et pré-configuré de broches exposées pour gérer les principales fonctions d'impression (résistances thermiques, extrudeuses, steppers...). L'utilisation du Rpi où Klipper est installé en tant que MCU secondaire donne la possibilité d'utiliser directement les GPIO et les bus (i2c, spi) du RPi à l'intérieur de klipper sans utiliser de plugins Octoprint (le cas échéant) ou de programmes externes donnant la possibilité de tout contrôler à l'intérieur l'impression GCODE.

**Avertissement** : Si votre plate-forme est un _Beaglebone_ et que vous avez correctement suivi les étapes d'installation, le mcu linux est déjà installé et configuré pour votre système.

## Installer le script rc

Si vous souhaitez utiliser l'hôte comme MCU secondaire, le processus klipper_mcu doit s'exécuter avant le processus klippy.

Après avoir installé Klipper, installez le script. Cours:
```
cd ~/klipper/
sudo cp "./scripts/klipper-mcu-start.sh" /etc/init.d/klipper_mcu
sudo update-rc.d klipper_mcu defaults
```

## Construction du code du microcontrôleur

Pour compiler le code du micro-contrôleur Klipper, commencez par le configurer pour le "process Linux":
```
cd ~/klipper/
make menuconfig
```

Dans le menu, définissez "Architecture du microcontrôleur" sur "Processus Linux", puis enregistrez et quittez.

Pour compiler et installer le nouveau code du microcontrôleur, exécutez :
```
sudo service klipper stop
make flash
sudo service klipper start
```

Si klippy.log signale une erreur "Autorisation refusée" lors de la tentative de connexion à `/tmp/klipper_host_mcu`, vous devez ajouter votre utilisateur au groupe tty. La commande suivante ajoutera l'utilisateur "pi" au groupe tty :
```
sudo usermod -a -G tty pi
```

## Configuration restante

Terminez l'installation en configurant le MCU secondaire Klipper en suivant les instructions de [RaspberryPi sample config](../config/sample-raspberry-pi.cfg) et [Multi MCU sample config](../config/sample-multi-mcu.cfg).

## Optionnel: Activation SPI

Assurez-vous que le pilote Linux SPI est activé en exécutant`sudo raspi-config` and permettant SPI sous le menu "Interfacing options".

## Optionnel: Activation I2C

Assurez-vous que le pilote Linux I2C est activé en exécutant `sudo raspi-config` et en activant I2C sous le menu "Interfacing options". Si vous prévoyez d'utiliser I2C pour l'accéléromètre MPU, il est également nécessaire de régler le débit en bauds sur 400000 en: adding/uncommenting  `dtparam=i2c_arm=on,i2c_arm_baudrate=400000` dans `/boot/config.txt` (or `/boot/firmware/config.txt` dans certaines distributions).

## Optionnel: Identifiez le bon gpiochip

Sur Raspberry Pi et sur de nombreux clones, les broches exposées sur le GPIO appartiennent à la première puce gpi. Ils peuvent donc être utilisés sur klipper simplement en les référant avec le nom `gpio0..n`. Cependant, il existe des cas dans lesquels les broches exposées appartiennent à des puces gpi autres que la première. Par exemple dans le cas de certains modèles OrangePi ou si un Port Expander est utilisé. Dans ces cas, il est utile d'utiliser les commandes pour accéder au _Linux GPIO character device_ pour vérifier la configuration.

Pour installer le _Linux GPIO character device - binary_ on a debian based distro like octopi run:
```
sudo apt-get install gpiod
```

Pour vérifier l'exécution gpiochip disponible:
```
gpiodetect
```

Pour vérifier le numéro de broche et la disponibilité de la broche tun:
```
gpioinfo
```

La broche choisie peut ainsi être utilisée dans la configuration comme  `gpiochip<n>/gpio<o>` où **n** est le numéro de puce vu par la commande `gpiodetect` et **o** est le numéro de ligne vu par la commande `gpioinfo`,
***Attention :*** seuls les gpio marqués comme "inutilisés" peuvent être utilisés. Il n'est pas possible pour une _line_ être utilisé par plusieurs processus simultanément.

Par exemple sur un RPi 3B+ où klipper utilise le GPIO20 pour un switch :
```
$ gpiodetect
gpiochip0 [pinctrl-bcm2835] (54 lines)
gpiochip1 [raspberrypi-exp-gpio] (8 lines)

$ gpioinfo
gpiochip0 - 54 lines:
        line   0:      unnamed       unused   input  active-high
        line   1:      unnamed       unused   input  active-high
        line   2:      unnamed       unused   input  active-high
        line   3:      unnamed       unused   input  active-high
        line   4:      unnamed       unused   input  active-high
        line   5:      unnamed       unused   input  active-high
        line   6:      unnamed       unused   input  active-high
        line   7:      unnamed       unused   input  active-high
        line   8:      unnamed       unused   input  active-high
        line   9:      unnamed       unused   input  active-high
        line  10:      unnamed       unused   input  active-high
        line  11:      unnamed       unused   input  active-high
        line  12:      unnamed       unused   input  active-high
        line  13:      unnamed       unused   input  active-high
        line  14:      unnamed       unused   input  active-high
        line  15:      unnamed       unused   input  active-high
        line  16:      unnamed       unused   input  active-high
        line  17:      unnamed       unused   input  active-high
        line  18:      unnamed       unused   input  active-high
        line  19:      unnamed       unused   input  active-high
        line  20:      unnamed    "klipper"  output  active-high [used]
        line  21:      unnamed       unused   input  active-high
        line  22:      unnamed       unused   input  active-high
        line  23:      unnamed       unused   input  active-high
        line  24:      unnamed       unused   input  active-high
        line  25:      unnamed       unused   input  active-high
        line  26:      unnamed       unused   input  active-high
        line  27:      unnamed       unused   input  active-high
        line  28:      unnamed       unused   input  active-high
        line  29:      unnamed       "led0"  output  active-high [used]
        line  30:      unnamed       unused   input  active-high
        line  31:      unnamed       unused   input  active-high
        line  32:      unnamed       unused   input  active-high
        line  33:      unnamed       unused   input  active-high
        line  34:      unnamed       unused   input  active-high
        line  35:      unnamed       unused   input  active-high
        line  36:      unnamed       unused   input  active-high
        line  37:      unnamed       unused   input  active-high
        line  38:      unnamed       unused   input  active-high
        line  39:      unnamed       unused   input  active-high
        line  40:      unnamed       unused   input  active-high
        line  41:      unnamed       unused   input  active-high
        line  42:      unnamed       unused   input  active-high
        line  43:      unnamed       unused   input  active-high
        line  44:      unnamed       unused   input  active-high
        line  45:      unnamed       unused   input  active-high
        line  46:      unnamed       unused   input  active-high
        line  47:      unnamed       unused   input  active-high
        line  48:      unnamed       unused   input  active-high
        line  49:      unnamed       unused   input  active-high
        line  50:      unnamed       unused   input  active-high
        line  51:      unnamed       unused   input  active-high
        line  52:      unnamed       unused   input  active-high
        line  53:      unnamed       unused   input  active-high
gpiochip1 - 8 lines:
        line   0:      unnamed       unused   input  active-high
        line   1:      unnamed       unused   input  active-high
        line   2:      unnamed       "led1"  output   active-low [used]
        line   3:      unnamed       unused   input  active-high
        line   4:      unnamed       unused   input  active-high
        line   5:      unnamed       unused   input  active-high
        line   6:      unnamed       unused   input  active-high
        line   7:      unnamed       unused   input  active-high
```

## Optionnel: Hardware PWM

Les Raspberry Pi ont deux canaux PWM (PWM0 et PWM1) qui sont exposés sur l'en-tête ou, sinon, peuvent être acheminés vers des broches gpio existantes. Le démon mcu Linux utilise l'interface pwmchip sysfs pour contrôler les périphériques matériels pwm sur les hôtes Linux. L'interface pwm sysfs n'est pas exposée par défaut sur un Raspberry et peut être activée en ajoutant une ligne à`/boot/config.txt`:
```
# Enable pwmchip sysfs interface
dtoverlay=pwm,pin=12,func=4
```
Cet exemple active uniquement PWM0 et l'achemine vers gpio12. Si les deux canaux PWM doivent être activés, vous pouvez utiliser `pwm-2chan`.

La superposition n'expose pas la ligne pwm sur sysfs au démarrage et doit être exportée en faisant écho au numéro du canal pwm vers `/sys/class/pwm/pwmchip0/export`:
```
echo 0 > /sys/class/pwm/pwmchip0/export
```

Cela créera le périphérique `/sys/class/pwm/pwmchip0/pwm0` dans le système de fichiers. La façon la plus simple de le faire est d'ajouter ceci à `/etc/rc.local` avant la ligne `exit 0`.

Avec le sysfs en place, vous pouvez maintenant utiliser le ou les canaux pwm en ajoutant la configuration suivante à votre `printer.cfg` :
```
[output_pin caselight]
pin: host:pwmchip0/pwm0
pwm: True
hardware_pwm: True
cycle_time: 0.000001
```
Cela ajoutera un contrôle pwm matériel à gpio12 sur le Pi (car la superposition a été configurée pour acheminer pwm0 vers la broche = 12).

PWM0 peut être acheminé vers gpio12 et gpio18, PWM1 peut être acheminé vers gpio13 et gpio19 :

| PWM | gpio PIN | Func |
| --- | -------- | ---- |
|   0 |       12 |    4 |
|   0 |       18 |    2 |
|   1 |       13 |    4 |
|   1 |       19 |    2 |
