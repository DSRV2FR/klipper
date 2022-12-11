# Aperçu

Bienvenue dans la documentation de Klipper. Si vous êtes nouveau sur Klipper, commencez par les [fonctionnalités](Features.md) et [installation] (Installation.md) documents.

## Informations générales

- [Features](Features.md): Une liste de haut niveau des fonctionnalités de Klipper.
- [FAQ](FAQ.md): Questions fréquemment posées.
- [Releases](Releases.md): L'histoire des versions de Klipper.
- [Config changes](Config_Changes.md): Modifications logicielles récentes qui
peut demander aux utilisateurs de mettre à jour leur fichier de configuration d'imprimante.
- [Contact](Contact.md): Informations sur les rapports de bugs et informations générales communication avec les développeurs Klipper.

## Installation et configuration

- [Installation](Installation.md): Guide d'installation de Klipper.
- [Config Reference](Config_Reference.md): Description de la configuration
  paramètres.
- [Rotation Distance](Rotation_Distance.md): Calcul de la paramètre pas à pas rotation_distance.
- [Config checks](Config_checks.md): Vérifiez les paramètres de broche de base dans le fichier de configuration.
- [Bed level](Bed_Level.md): Informations sur le "nivellement du lit" dans Klipper.
- [Delta calibrate](Delta_Calibrate.md): Étalonnage du delta
cinématique.
- [Probe calibrate](Probe_Calibrate.md): Étalonnage du Z automatique sondes.
- [BL-Touch](BLTouch.md): Configurer une sonde Z "BL-Touch".
- [Manual level](Manual_Level.md): Étalonnage des butées Z (et similaire).
- [Bed Mesh](Bed_Mesh.md): Correction de la hauteur du lit basée sur XY Emplacements.
- [Endstop phase](Endstop_Phase.md): Butée Z assistée pas à pas positionnement.
- [Resonance compensation](Resonance_Compensation.md): Un outil pour réduire la sonnerie dans les impressions.
- [Measuring resonances](Measuring_Resonances.md): Des informations sur utilisation du matériel accéléromètre adxl345 pour mesurer la résonance.
- [Pressure advance](Pressure_Advance.md): Calibrer l'extrudeuse  pression.
- [G-Codes](G-Codes.md): Informations sur les commandes prises en charge par Klipper.
- [Command Templates](Command_Templates.md): Macros G-Code et évaluation conditionnelle.
- [Status Reference](Status_Reference.md): Informations disponibles pour macros (et similaires).
- [TMC Drivers](TMC_Drivers.md): Utilisation des pilotes de moteur pas à pas Trinamic avec Klipper.
- [Multi-MCU Homing](Multi_MCU_Homing.md): Prise d'origine et sondage à l'aide de plusieurs microcontrôleurs.
- [Slicers](Slicers.md): Configurer le logiciel "slicer" pour Klipper.
- [Skew correction](Skew_Correction.md): Réglages pour axes non parfaitement carré.
- [PWM tools](Using_PWM_Tools.md): Guide d'utilisation contrôlé par PWM
  des outils tels que des lasers ou des broches.
- [Exclude Object](Exclude_Object.md) : le guide des objets Exclude la mise en oeuvre.

## Documentation pour les développeurs

- [Code overview](Code_Overview.md) : les développeurs doivent lire ceci
première.
- [Kinematics](Kinematics.md) : Détails techniques sur la façon dont Klipper
met en œuvre le mouvement.
- [Protocol](Protocol.md) : informations sur la messagerie de bas niveau protocole entre l'hôte et le microcontrôleur.
- [API Server](API_Server.md) : informations sur la commande de Klipper et API de contrôle.
- [MCU commands](MCU_Commands.md) : une description des commandes de bas niveau implémenté dans le logiciel du microcontrôleur.
- [CAN bus protocol](CANBUS_protocol.md) : message de bus CAN Klipper format.
- [Debugging](Debugging.md) : informations sur la façon de tester et de déboguer
Rochers.
- [Benchmarks](Benchmarks.md) : Informations sur le benchmark Klipper méthode.
- [Contribuer](CONTRIBUTING.md) : informations sur la façon de soumettre améliorations apportées à Klipper.
- [Packaging](Packaging.md) : informations sur la création de packages de système d'exploitation.

## Documents spécifiques à l'appareil

- [Example configs](Example_Configs.md) : informations sur l'ajout d'un exemple de fichier de configuration à Klipper.
- [SDCard Updates](SDCard_Updates.md) : Flasher un micro-contrôleur en copier un binaire sur une carte SD dans le microcontrôleur.
- [Raspberry Pi comme microcontrôleur](RPi_microcontroller.md) : Détails pour contrôler les appareils câblés aux broches GPIO d'un Raspberry Pi.
- [Beaglebone](Beaglebone.md) : Détails pour exécuter Klipper sur le Beaglebone PRU.
- [Bootloaders](Bootloaders.md) : informations pour les développeurs sur micro-contrôleur clignotant.
- [CAN bus](CANBUS.md) : informations sur l'utilisation du bus CAN avec Klipper.
- [Capteur de largeur de filament TSL1401CL] (TSL1401CL_Filament_Width_Sensor.md)
- [Capteur de largeur de filament Hall] (Hall_Filament_Width_Sensor.md)
