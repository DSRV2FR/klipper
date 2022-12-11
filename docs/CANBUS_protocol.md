# Protocole CANBUS 

Ce document décrit le protocole utilisé par Klipper pour communiquer sur
[Bus CAN](https://en.wikipedia.org/wiki/CAN_bus). Voir [CANBUS.md](CANBUS.md) pour plus d'informations sur la configuration de Klipper avec CAN bus.

## Affectation de l'identifiant du microcontrôleur

Klipper utilise uniquement des paquets de bus CAN de taille standard CAN 2.0A, qui sont limité à 8 octets de données et un identifiant de bus CAN de 11 bits. Pour supportent une communication efficace, chaque microcontrôleur est affecté à
à l'exécution un identifiant de nœud de bus CAN unique de 1 octet ("canbus_nodeid") pour Commande Klipper et trafic de réponse. Messages de commande Klipper en cours de l'hôte au microcontrôleur, utilisez l'identifiant de bus CAN de `canbus_nodeid * 2 + 256`, tandis que les messages de réponse Klipper du microcontrôleur à l'hôte utilise `canbus_nodeid * 2 + 256 + 1`.

Chaque microcontrôleur possède un identifiant de puce unique attribué en usine
qui est utilisé lors de l'attribution de l'identifiant. Cet identifiant peut dépasser le longueur d'un paquet CAN, donc une fonction de hachage est utilisée pour générer un id unique de 6 octets (`canbus_uuid`) de l'id d'usine.

## Messages d'administration

Les messages d'administration sont utilisés pour l'attribution de l'identifiant. Messages d'administration envoyés depuis l'hôte au microcontrôleur utilise l'identifiant de bus CAN "0x3f0" et les messages envoyés du microcontrôleur à l'hôte, utilisez l'identifiant de bus CAN "0x3f1". Tout les micro-contrôleurs écoutent les messages sur l'identifiant `0x3f0` ; cet identifiant peut être
considéré comme une "adresse de diffusion".

### message CMD_QUERY_UNASSIGNED 

Cette commande interroge tous les microcontrôleurs qui n'ont pas encore été
assigné un `canbus_nodeid`. Les microcontrôleurs non affectés répondront avec un message de réponse RESP_NEED_NODEID.

Le format de message CMD_QUERY_UNASSIGNED est :
`<1-byte message_id = 0x00>`

### message CMD_SET_KLIPPER_NODEID 

Cette commande attribue un `canbus_nodeid` au microcontrôleur avec un
donné `canbus_uuid`.

Le format de message CMD_SET_KLIPPER_NODEID est:
`<1-byte message_id = 0x01><6-byte canbus_uuid><1-byte canbus_nodeid>`

### messageR ESP_NEED_NODEID 

Le format du message RESP_NEED_NODEID est:
`<1-byte message_id = 0x20><6-byte canbus_uuid><1-byte set_klipper_nodeid = 0x01>`

## Data Packets

Un microcontrôleur auquel un nodeid a été attribué via La commande CMD_SET_KLIPPER_NODEID peut envoyer et recevoir des paquets de données.

Les données de paquet dans les messages utilisant l'ID de bus CAN de réception du nœud (`canbus_nodeid * 2 + 256`) sont simplement ajoutés à un tampon, et quand un [message de protocole mcu] complet (Protocol.md) est trouvé son contenu
sont analysés et traités. Les données sont traitées comme un flux d'octets - il
n'est pas nécessaire que le début d'un bloc de message Klipper s'aligne avec le début d'un paquet de bus CAN.

De même, les réponses aux messages de protocole mcu sont envoyées à partir de
microcontrôleur à héberger en copiant les données du message dans un ou plusieurs paquets avec l'identifiant de bus CAN de transmission du nœud ('canbus_nodeid * 2 + 256 + 1`).
