## COMPTE RENDU TP FREERTOS 
## KITSOUKOU Debora  & DJEUNGA Daniela
## 3DSN_TP

# Objectif :
L’objectif de ce TP sur cinq séances est de mettre en place quelques applications sous FreeRTOS en utilisant la carte NUCLEO-G431RB conçue autour du microcontrôleur STM32G431RBT6.
Tout au long du TP, la fonction printf() sera beaucoup utilisée. Pour rendre celà possible, il faut ajouter les lignes suivantes, dans le fichier main.c, avant la fonction main() :
/* USER CODE BEGIN */
int __io_putchar(int ch) {
HAL_UART_Transmit(&huart2, (uint8_t *)&ch, 1, HAL_MAX_DELAY);
return ch;
}
/* USER CODE END */

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
Réponse : Les commentaires /* USER CODE BEGIN */ et /* USER CODE END */ dans les projets STM32 (générés avec STM32CubeMX ou STM32CubeIDE) servent à protéger certaines sections de code que nous écrivons manuellement contre l’écrasement lors des regénérations automatiques du code par l’outil.

3. Quels sont les paramètres à passer à HAL_Delay et HAL_GPIO_TogglePin ?
- HAL_GPIO_TogglePin(GPIO_TypeDef* GPIOx, uint16_t GPIO_Pin);
Cette fonction inverse l’état d’un GPIO de sortie.
Dans notre code nous avons pris l'exemple suivant: HAL_GPIO_TogglePin(GPIOI, GPIO_PIN_1); // Inverse l'état de la LED
  
- HAL_Delay(uint32_t Delay);
Cette fonction met en pause le programme pendant un certain temps
exemple de notre code: HAL_Delay(500); // Attend 500 ms

4. Dans quel fichier les ports d’entrée/sorties sont-ils définis ?
  les ports dentrées/sorties sont définis dans le fichier: gpio.h

5. Écrivez un programme simple permettant de faire clignoter la LED.
   
   /* USER CODE BEGIN 2 */
HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_RESET);
/* USER CODE END 2 */

/* Infinite loop */
/* USER CODE BEGIN WHILE */
while (1)
{
    HAL_GPIO_TogglePin(GPIOI, GPIO_PIN_1);
    HAL_Delay(500);
}
/* USER CODE END WHILE */


6. Modifiez le programme pour que la LED s’allume lorsque le bouton USER est appuyé.

 while (1)
  {

   if (HAL_GPIO_ReadPin(GPIOI, GPIO_PIN_11) == GPIO_PIN_SET)
       {
           HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_SET); // Allumer la LED
       }
       else
       {
           HAL_GPIO_WritePin(GPIOI, GPIO_PIN_1, GPIO_PIN_RESET); // Éteindre la LED
       }
       HAL_Delay(10); // Anti-rebond simple
    /* USER CODE END WHILE */

    /* USER CODE BEGIN 3 */
  }

## 1 FreeRTOS, tâches et sémaphores ##
# 1.1 Tâche simple #
1. En quoi le paramètre TOTAL_HEAP_SIZE a-t-il de l’importance ?
  

