# Packaging Klipper

Klipper est en quelque sorte une anomalie d'emballage parmi les programmes python, car il ne utilisez setuptools pour construire et installer. Quelques notes sur la meilleure façon de l'emballer sont les suivants:

## C modules

Klipper utilise un module C pour gérer plus rapidement certains calculs cinématiques. Ce module doit être compilé au moment de l'empaquetage pour éviter d'introduire un dépendance d'exécution sur un compilateur. Pour compiler le module C, exécutez `python2 klippy/chelper/__init__.py`.

## Compiler du code python

De nombreuses distributions ont pour politique de compiler tout le code python avant de l'empaqueter pour améliorer le temps de démarrage. Vous pouvez le faire en exécutant `python2 -m compileall klippy`.

## Gestion des versions

Si vous construisez un paquet de Klipper à partir de git, il est d'usage de ne pas expédier un répertoire .git, la gestion des versions doit donc être gérée sans git. Faire cela, utilisez le script fourni dans `scripts/make_version.py` qui doit être exécuté en tant que suit : `python2 scripts/make_version.py VOTRENOMDISTRO > klippy/.version`.

## Exemple de script d'emballage

klipper-git est empaqueté pour Arch Linux et possède un PKGBUILD (package build
script) disponible sur [Arch User Repositiory](https://aur.archlinux.org/cgit/aur.git/tree/PKGBUILD?h=klipper-git).
