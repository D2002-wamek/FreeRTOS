## COMPTE RENDU TP FREERTOS 
## KITSOUKOU Debora  & DJEUNGA Daniela
## 3DSN_TP

# Objectif :
L’objectif de ce TP sur cinq séances est de mettre en place quelques applications sous FreeRTOS en utilisant la carte NUCLEO-G431RB conçue autour du microcontrôleur STM32G431RBT6.
Tout au long du TP, la fonction printf() sera beaucoup utilisée. Pour rendre celà possible, il faut ajouter les lignes suivantes, dans le fichier main.c, avant la fonction main() :

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

# 1 FreeRTOS, tâches et sémaphores
lors de l'activation du FreeRTOS, nous avons eu le choix entre deux modes d'interface: CMSIS_V1 et CMSIS_V2:
Comparaison des deux signatures :

🔹 1. void StartDefaultTask(void const * argument)
→ Utilisée avec CMSIS-RTOS V1
const void *argument : signifie que l'argument passé est constante et non modifiable
C’est la vieille convention utilisée par osThreadCreate() (CMSIS V1)
Le mot-clé const indique qu'on ne doit pas modifier les données pointées

🔹 2. void StartDefaultTask(void *argument)
→ Utilisée avec CMSIS-RTOS V2 (ou CMSIS-RTOS2)
void *argument : plus moderne, compatible avec la nouvelle API osThreadNew()
L’argument est modifiable, plus souple si tu veux passer des structures à la tâche
Utilisé avec FreeRTOS quand tu choisis CMSIS V2 dans CubeMX
 
# 1.1 Tâche simple 
1. En quoi le paramètre TOTAL_HEAP_SIZE a-t-il de l’importance ?

Il est 15360 bytes dans notre cas, il définit la quantité totale de mémoire que FreeRTOS peut utiliser pour l'allocation  dynamique. Ceci dit, la création de tâches, de   queues, de semaphores,... Une taille de tas insuffisante peut entraîner un échec de création des objets  FreeRTOS, alors qu'une taille trop grande peut gaspiller la mémoire du microcontrôleur.


Observation l’impact de notre configuration sur le fichier FreeRTOSConfig.h

configTOTAL_HEAP_SIZE = 15360 ----------->	15 Ko de RAM alloués pour créer tâches, queues, sémaphores, etc.
configUSE_PREEMPTION = 1----------------->	Système préemptif : les tâches prioritaires peuvent interrompre les moins prioritaires.
configSUPPORT_STATIC/DYNAMIC = 1--------->	on peux utiliser l’allocation statique et dynamique pour plus de flexibilité.
configTICK_RATE_HZ = 1000---------------->	Résolution de 1 ms pour les délais (vTaskDelay, etc.).
configMAX_PRIORITIES = 7----------------->	Nous pouvons créer des tâches avec 7 niveaux de priorité.
configMINIMAL_STACK_SIZE = 128----------->	Taille par défaut de la pile d’une tâche : 128 mots (≈ 512 octets).
configUSE_MUTEXES = 1-------------------->	Activation des mutex pour protéger les ressources partagées.
INCLUDE_vTaskDelay = 1, etc.------------->	les principales fonctions API utiles pour la gestion des tâches sont activés.
configASSERT(x)	Sécurité :---------------> si une erreur critique se produit (ex. mémoire insuffisante), le système s'arrête.
configLIBRARY_MAX_SYSCALL_INTERRUPT_PRIORITY = 5	Protège FreeRTOS des interruptions trop prioritaires appelant des fonctions "safe".

2. Quel est le rôle de la macro portTICK_PERIOD_MS ?
portTICK_PERIOD_MS est une macro utilisée dans FreeRTOS pour convertir une période de temps spécifiée en millisecondes au nombre de ticks du système. Cela permet d'abstraire la durée réelle entre les ticks du système, qui peut varier en fonction de la configuration de FreeRTOS et de la fréquence du processeur.
L'utilisation de cette macro garantit que le délai spécifié dans vTaskDelay() est indépendant de la configuration du système d'exploitation en temps réel et du matériel

# 1.2 Sémaphores pour la synchronisation

3., 4., 5., voir les commits du code

6. Changez les priorités. Expliquez les changements dans l’affichage
   
