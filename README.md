# TP FreeRTOS 3DN
SEHTEL Azzedine

BEGUIN Thomas
## Premiers pas
	Le fichier main.c se trouve dans : <project_dir>/core/src/main.c.

	Les sections délimitées par les commentaires /* USER CODE BEGIN */ et /* USER CODE END */ sont les seules zones modifiables.
	Attention : tout code écrit en dehors de ces blocs sera écrasé si CubeIDE régénère le projet.
## HAL_DELAY	
	void HAL_Delay	(uint32_t Delay)
  	Parameters:
	Delay specifies the delay time length, in milliseconds.

## HAL_GPIO_TogglePin
	void HAL_GPIO_TogglePin	( GPIO_TypeDef *GPIOx, uint16_t	GPIO_Pin)
 	Parameters:
	GPIOx	Where x can be (A..K) to select the GPIO peripheral for STM32F429X device or x can be (A..I) to select the GPIO peripheral for STM32F40XX and STM32F427X devices.
	GPIO_Pin	Specifies the pins to be toggled.
Dans STM32CubeDIE, les entrées/sortie sont configurer dans le fichier IOC qui par la suite genere du Code.

Les définition des GPIO peuvent etre retrouver dans le fichier <project_dir>/core/inc/main.c

Et la fonction d'initialisation des GPIO est dans le fichier <project_dir>/core/src/gpio.c
## FreeRTOS	
configTOTAL_HEAP_SIZE est important car il s'agit de la mémoire allouée pour FreeRTOS. 

Si elle est trop petite, il pourra etre imossible de créer des task via xTaskCreate.

Trop grande et nous gaspillons de la RAM

Les autres paramètre imortant pour FreeRTOS sont:

	configMAX_PRIORITIES, Nombre maximal de niveaux de priorité des tâches.
 	configMINIMAL_STACK_SIZE, Taille par défaut de la pile pour une tâche simple.
  	configTICK_RATE_HZ, réquence du tick système (1000 Hz pour 1ms)

La Macro portTICK_PERIOD_MS présente la durée d’un tick système en millisecondes.

	Si configTICK_RATE_HZ = 1000; et portTICK_PERIOD_MS = 1;
 	Pour attendre 500ms,
	vTaskDelay(500 / portTICK_PERIOD_MS); //soit vtaskDelay(500);

Explication changement de priorités

 Priorité de TaskGive > TaskTake
 
  	TaskGive s’exécute dès que son délai expire, même si TaskTake est prête.
	TaskTake ne peut prendre le sémaphore que quand TaskGive le relâche.
	résultat : ordre constant dans les messages, stable, sans blocage.
Priorité de TaskGive < TaskTake :
	
	TaskTake tourne en continu (surtout en attente du sémaphore).
	Si TaskGive est prête pendant que TaskTake tourne, elle peut être retardée par l’ordonnanceur.
	Résultat : l’affichage montre que TaskGive a du mal à donner le sémaphore à temps → TaskTake échoue après 1 seconde, et déclenche le reset logiciel.

## Shell


 
  
 


 	
