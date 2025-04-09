## COMPTE RENDU TP FREERTOS 
## KITSOUKOU Debora  & DJEUNGA Daniela
## 3DSN_TP

# Objectif :
L’objectif de ce TP sur cinq séances est de mettre en place quelques applications sous FreeRTOS en utilisant la carte NUCLEO-G431RB conçue autour du
microcontrôleur STM32G431RBT6.
Tout au long du TP, la fonction printf() sera beaucoup utilisée. Pour rendre
celà possible, il faut ajouter les lignes suivantes, dans le fichier main.c, avant la
fonction main() :

int __io_putchar(int ch) {
HAL_UART_Transmit(&huart2, (uint8_t *)&ch, 1, HAL_MAX_DELAY);
return ch;
}


# 0 (Re)prise en main

L’objectif de ce premier TP est double. D’une part vous familiariser avec la couche d’abstraction matérielle (HAL pour Hardware Abstraction Layer). 
Cette couche permet de programmer à plus haut niveau, et de perdre moins de temps à chercher les adresses des registres... 
Vous constaterez néanmoins que pour programmer les périphériques, il faut quand même comprendre leur fonctionnement.
Le second objectif est de vous apprendre à lire cette @&#$ ! de documentation (ces @&#$ !s de documentations, pour être précis).

# 0.1 Premiers pas

Création du projet

1. Où se situe le fichier main.c ?
Réponse : Core – Src

2. À quoi servent les commentaires indiquant BEGIN et END ?
Réponse :
— HAL_Delay :
— HAL_GPIO_TogglePin :



