# Les files d'attente

**Les files d’attente**

Les files d'attente (**queues**) sont très utiles pour la communication inter-tâches, permettant d'envoyer des messages d'une tâche à une autre en toute sécurité en termes de concurrence. Ils sont généralement utilisés comme **FIFO **(**First In First Out**), ce qui signifie que les nouvelles données sont insérées à l'arrière de la file d'attente et consommées à partir du front.

Les données insérées dans la file d'attente sont copiées plutôt que référencées. Cela signifie que si nous envoyons un entier à la file d'attente, sa valeur sera réellement copiée et si nous changeons la valeur d'origine après cela, aucun problème ne devrait se produire.

Un aspect de comportement important est que l'insertion dans une file d'attente pleine ou la consommation d'une file d'attente vide peut bloquer les appels pendant une durée donnée (cette durée est un **paramètre **de l'API).

Pour faire l’insertion réelle de l’élément, nous appelons la fonction **xQueueSend**. Il reçoit comme premier paramètre la référence de la file d’attente (rappelez-vous qu’elle est stockée dans une variable globale), comme second pointeur vers l’élément qui doit être inséré dans la file d’attente et enfin la durée maximale (en ticks) la tâche doit attendre si la file d’attente est déjà pleine.

Dans notre cas, étant donné que nous n’attribuerons pas un nombre d’articles supérieur à la taille de la file d’attente, ce temps mentionné précédemment n’est pas pertinent. Néanmoins, nous utiliserons la valeur **portMAX\_DELAY**, qui indique que la tâche bloquera indéfiniment jusqu’à ce qu’il y ait de l’espace dans la file d’attente pour insérer l’élément.

_Fichier : ESP32\_ProducerConsumer.ino_

```arduino
// ESP32_ProducerConsumer
QueueHandle_t queue;
int queueSize = 100;

void setup() {
  Serial.begin(9600);
  delay(1000);
  queue = xQueueCreate( queueSize, sizeof( int ) );
  if (queue == NULL) {
    Serial.println("Error creating the queue");
  }
  xTaskCreate(
    producerTask, /* Task function. */
    "Producer", /* String with name of task. */
    10000, /* Stack size in words. */
    NULL, /* Parameter passed as input of the task */
    1, /* Priority of the task. */
    NULL); /* Task handle. */
  xTaskCreate(
    consumerTask, /* Task function. */
    "Consumer", /* String with name of task. */
    10000, /* Stack size in words. */
    NULL, /* Parameter passed as input of the task */
    1, /* Priority of the task. */
    NULL); /* Task handle. */
}

void loop() {
  Serial.println("in the loop");
  delay(1000);
}

void producerTask( void * parameter )
{
  for ( int i = 0; i < queueSize; i++ ) {
    xQueueSend(queue, &i, portMAX_DELAY);
  }
  vTaskDelete( NULL );
}

void consumerTask( void * parameter)
{
  int element;
  for ( int i = 0; i < queueSize; i++ ) {
    xQueueReceive(queue, &element, portMAX_DELAY);
    Serial.print(element);
    Serial.print("|");
  }
  vTaskDelete( NULL );
}
```

_Questions :_ Quel est l’affichage en sortie ? Que se passe-t-il si queueSize = 100 ?

**Simple application IoT exécutée sur 2 cœurs**

Faire le montage suivant avec le capteur de température DHT22 et une résistance de 10 kΩ. Ce capteur est lent et fourni une valeur toutes les 2 secondes, ce qui est suffisant.

![](<../.gitbook/assets/image (4).png>)**  **![](<../.gitbook/assets/image (8).png>)****

****![](<../.gitbook/assets/image (6).png>)****

La communication entre les tâches est effectuée par le biais d’une file d’attente.

**Travail à faire : Écrire le code pour gérer une file d’attente :**

* Lire les valeurs de température et d’humidité données par le capteur DHT22. Pour cela, installer la bibliothèque DHTesp.h (à installer : [https://github.com/beegee-tokyo/DHTesp](https://github.com/beegee-tokyo/DHTesp) ) dans votre IDE Arduino. Voir les exemples : [https://github.com/beegee-tokyo/DHTesp/tree/master/examples](https://github.com/beegee-tokyo/DHTesp/tree/master/examples)
* Passer ces valeurs entre 2 tâches à l’aide d’une file d’attente (queue), chaque tâche étant associée à un cœur différent ( xTaskCreatePinnedToCore() ). Par exemple xQueueSend() sera dans la tâche sensorSendTask et xQueueReceive() dans la tâche sensorReceiveTask.
