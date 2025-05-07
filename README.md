# TP FreeRTOS 3DN
SEHTEL Azzedine

BEGUIN Thomas

## Premiers pas
### Où se situe le fichier main.c ?
Le fichier `main.c` se trouve dans le chemin suivant :  
`\TP_Freertos_SEHTEL_BEGUIN\Core\Src\main.c`

### À quoi servent les commentaires indiquant BEGIN et END ?
Les commentaires `BEGIN` et `END` délimitent les sections où l'on peut écrire du code sans que l'outil de génération de code (comme CubeMX) n'écrasent ces modifications lors d'une régénération.

### Quels sont les paramètres à passer à HAL_Delay et HAL_GPIO_TogglePin ?
- `HAL_Delay`: Le paramètre est un entier non signé `uint32_t` représentant le délai en millisecondes.
- `HAL_GPIO_TogglePin`: Les paramètres sont :
  - Le port GPIO.
  - Le numéro de la broche.

### Dans quel fichier les ports d’entrée/sorties sont-ils définis ?
Les ports d'entrée/sortie sont définis dans le fichier `gpio.c`.


##  FreeRTOS, tâches et sémaphores

### Tâche simple
#### En quoi le paramètre TOTAL_HEAP_SIZE a-t-il de l’importance ?
Le paramètre `TOTAL_HEAP_SIZE` détermine la quantité de mémoire disponible pour les allocations dynamiques dans FreeRTOS, si la taille est insuffisante, ces objets ne peuvent pas être créés.

### Sémaphores pour la synchronisation
#### Créez deux tâches avec des priorités différentes (TaskGive et TaskTake), que font-elles ?
- `TaskGive`: Envoie un signal (donne un sémaphore) pour indiquer qu'une ressource est disponible.
- `TaskTake`: Attend le signal (prend le sémaphore) pour accéder à la ressource.

#### Que se passe-t-il si on ajoute 100ms au délai de TaskGive à chaque itération ?
Le délai entre chaque signal envoyé par `TaskGive` augmente progressivement, ce qui ralentit la synchronisation avec `TaskTake` , passé les 1000 ms, la fonction `NVIC_SystemReset();` est activé ce qui reset le STM32, cela agit comme un watchdog.

#### Changez les priorités. Expliquez les changements dans l’affichage.
Si `TaskGive` a une priorité plus élevée que `TaskTake`, elle sera exécutée plus fréquemment, ce qui peut entraîner un comportement plus rapide. Inversement, si `TaskTake` a une priorité plus élevée, elle peut attendre plus longtemps pour recevoir un signal.


### Queues
#### Modifiez TaskGive pour envoyer la valeur du timer dans une queue. Que doit faire TaskTake ?
- `TaskGive` : Envoie la valeur du timer `delay` dans une queue nommé QueuTask avec la fonction`xQueueSend`.
- `TaskTake` : Lit la valeur de la queue à l'aide de `xQueueReceive` et l'affiche via `printf`. Si la réception échoue (timeout), elle peut gérer l'erreur, par exemple en affichant un message ou en déclenchant un reset.

#### Réentrance et exclusion mutuelle
#### Observez attentivement la sortie dans la console. Expliquez d’où vient le problème.

#### print dans le terminal ####
||TP_FreeRTOS_SEHTEL_BEGUIN||// 
Je suis Tache 2 et je m'endors pour 2 ticks 
Je suis Tache 1 et je m'endors pour 2 ticks
Je suis Tache 2 et je m'endors pour 2 ticks 
Je suis Tache 2 et je m'endors pour 2 ticks
Je suis Tache 1 et je m'endors pour 2 ticks 
Je suis Tache 2 et je m'endors pour 2 ticks 
Je suis Tache 2 et je m'endors pour 2 ticks 
Je suis Tache 1 et je m'endors pour 2 ticks 

#### Expliquez d’où vient le problème 
Le problème vient des priorités des tâches et de la configuration des délais. Les deux tâches, `Tache 1` et `Tache 1`, ont des priorités et un délai différents.

#define TASK1_PRIORITY 1
#define TASK2_PRIORITY 2
#define TASK1_DELAY 1
#define TASK2_DELAY 2

Les deux tâches ont des délais très proches. Cela provoque des réveils fréquents et simultanés, mais Tache 2 a une priorité plus élevée sur le processeur.
Conclusion la Tache 2 s'exécute plus souvent que Tache 1 malgré le faite qu celle-ci ai un delay plus faible, ce qui déséquilibre l'affichage dans la console.

## FreeRTOS	

### On joue avec le shell
#### Quel est le nom de la zone réservée à l’allocation dynamique ?
La zone reservé s'appelle la heap

###Est-ce géré par FreeRTOS ou la HAL 
Elle est géré par FreeRTOS via son propre gestionnaire de mémoire. 

configTOTAL_HEAP_SIZE est important car il s'agit de la mémoire allouée pour FreeRTOS. 

Si elle est trop petite, il sera impossible de créer des task via xTaskCreate,
en revanche, si elle est trop grande nous gaspillons de la RAM

Les autres paramètres imortant pour FreeRTOS sont:

	- configMAX_PRIORITIES, Nombre maximal de niveaux de priorité des tâches.
	- configMINIMAL_STACK_SIZE, Taille par défaut de la pile pour une tâche simple.
	- configTICK_RATE_HZ, réquence du tick système (1000 Hz pour 1ms)

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

## Debug, gestion d’erreur et statistiques

 Toute allocation dynamique malloc, xtaskCreate utilise la HEAP 
  FreeRTOS a sont propre gestionnaire de HEAP via l'utilisation de xTaskCreate, xQueueCreate...
Nous avons donc créer une tache bidon qui essaie d'allouer un tableau d'entier de plus en plus grand
 	
  	void ErrorTask(void *arg){
	static int size = 2;
	int *buffer = NULL;

    for(;;) {
    	printf("ErrorTask Malloc buffer size %d 0x%X\r\n", size, size);
    	buffer = (int *)malloc(sizeof(int) * size);
    	if (buffer == NULL)
    		printf("ErroTask: Malloc erreur %d 0x%X\t\n", size, size);
    	else
    	{
    		size *= 2;
    		free(buffer);
    		buffer = NULL;
    	}
        vTaskDelay(pdMS_TO_TICKS(1000));
    }

Notre allocation dynamique nous retourne une erreur quand nous envoyons malloc(sizeof(int)0x1000);	
Avec TOTAL_HEAP_SIZE = 15360
Notre allocation dynamique nous retourne une erreur quand nous envoyons malloc(sizeof(int)0x4000);
Avec TOTAL_HEAP_SIZE = 61440
## Attention, ne pas utiliser malloc et free, utiliser pvPortMalloc et vPortFree

Nous pouvons observer que l'utilisation de la mémoire dans le builder analyser ne change car nous utilison des allocations dynamiques.
 	 
## Gestion des piles

Pour tester l'overflow de la pile, nous avons créer une tache recursive
	
 	void vOverflowTask(void *param)
	{
    	volatile uint8_t  buff[ configMINIMAL_STACK_SIZE / 4 ];

    	for( ;; )
    	{
        	memset((void*)buff, 0xAA, sizeof(buff));
        	vOverflowTask( NULL );
    	}
	}
Et nous avons également réecrit notre vApplicationStackOverflowHook
	
 	void vApplicationStackOverflowHook( TaskHandle_t xTask, char *pcTaskName )
	{
    	printf("STACK OVERFLOW detected in task: %s\r\n", pcTaskName);
    	Error_Handler();
	}
Afin de visualiser le fonctionnement, nous pouvons rajouter un point d'arret sur vApplicationStackOverflowHook et utiliser le debuger

Il esxiste pleins d'autres fonction de hook:
https://www.freertos.org/Documentation/02-Kernel/02-Kernel-features/12-Hook-functions

## Affichage des statistiques dans le shell
ajout d'une fonction dans notre shell
	
	int sh_stats(h_shell_t * h_shell, int argc, char **argv) 
 	{
		char buffer[512];
  		printf("=== Liste des taches ===\r\n");
		vTaskList(buffer);
		printf("%s\r\n", buffer);

		printf("=== Statistiques d'execution ===\r\n");
		vTaskGetRunTimeStats(buffer);
		printf("%s\r\n", buffer);

		return 0;
  	}
Ajout dans notre SHell

	shell_add(h_shell, 'p', sh_stats, "Affiche les statistiques");
 Attention !! Dans xTaskCreate de notre shellTask, il nous faut augmenter la taille de la stack, car nous avons un char buff[512] dans notre focntion sh_stats
Il faut également faire attention car vTaskList et vTaskGetRunTimeStats essaie d'écrire dans le buffer (si le buffer est trop petit alors il y aura un overflow)


 


 	
