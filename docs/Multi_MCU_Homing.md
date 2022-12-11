# Multiple Micro-controller Homing et Probing

Klipper prend en charge un mécanisme de prise d'origine avec une butée fixée à
un microcontrôleur tandis que ses moteurs pas à pas sont sur un autre microcontrôleur. Ce support est appelé "multi-mcu prise d'origine". Cette fonction est également utilisée lorsqu'une sonde Z est sur un autre
microcontrôleur que les moteurs pas à pas Z.

Cette fonctionnalité peut être utile pour simplifier le câblage, car il peut être plus pratique pour fixer une butée ou une sonde à un microcontrôleur plus proche. Cependant, l'utilisation de cette fonctionnalité peut entraîner un "dépassement" du stepper moteurs pendant les opérations de prise d'origine et de palpage.

Le dépassement se produit en raison d'éventuels retards de transmission des messages entre le microcontrôleur surveillant la butée et le micro-contrôleurs déplaçant les moteurs pas à pas. Le code Klipper est conçu pour limiter ce retard à pas plus de 25 ms. (Lorsque multi-mcu le homing est activé, les micro-contrôleurs envoient des états périodiques messages et vérifier que les messages d'état correspondants sont reçus dans les 25 ms.)

Ainsi, par exemple, si la prise d'origine à 10 mm/s, il est possible pour un
dépassement jusqu'à 0,250 mm (10 mm/s * 0,025 s == 0,250 mm). Les soins doivent être prises lors de la configuration de l'hébergement multi-mcu pour tenir compte de ce type de dépasser. L'utilisation de vitesses de prise d'origine ou de palpage plus lentes peut réduire la dépasser.

Le dépassement du moteur pas à pas ne doit pas nuire à la précision de la procédure de prise d'origine et de sondage. Le code Klipper détectera le dépasser et en tenir compte dans ses calculs. Cependant, il est important que la conception matérielle soit capable de gérer les dépassements sans endommager la machine.

Si Klipper détecte un problème de communication entre les microcontrôleurs
pendant le référencement multi-mcu, il déclenchera un "Délai d'attente de communication lors de la prise d'origine".

Notez qu'un axe avec plusieurs steppers (eg, `stepper_z` et
`stepper_z1`) doivent être sur le même microcontrôleur pour pouvoir utiliser
référencement multi-mcu. Par exemple, si une butée se trouve sur un autre
micro-contrôleur de `stepper_z` puis `stepper_z1` doit être sur le même micro-contrôleur que `stepper_z`.
