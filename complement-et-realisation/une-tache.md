# Une tâche

**Création et suppression d’une tâche**

On crée une tâche avec un appel à la fonction **xTaskCreate()**. Pour l’ESP32, nous pouvons l’attacher à un cœur avec **xTaskCreatePinnedToCore()**.

| <p>xTaskCreatePinnedToCore(</p><p>    codeForTask1,</p><p>    "led1Task",</p><p>    1000,</p><p>    NULL,</p><p>    1,</p><p>    &#x26;Task1,</p><p>    0);    // core 0</p> | <p>xTaskCreate(</p><p>    vATaskFunction, /* Task function. */</p><p>    "vATaskFunction", /* name of task. */</p><p>    10000, /* Stack size of task */</p><p>    NULL, /* parameter of the task */</p><p>    1, /* priority of the task */</p><p>    NULL); /* Task handle to keep track of created task */</p><p> </p> |
| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |

Cette fonction retourne pdPASS en cas de succès ou un code d’erreur.

Pour supprimer une tâche de son propre code, il suffit d’appeler la fonction **vTaskDelete()**. Si nous passons NULL en entrée, la tâche appelante sera supprimée, ce qui est souhaité quand nous l’appelons à partir du propre code de la tâche : **vTaskDelete(NULL) ;**

**Exemple d’une tâche supplémentaire**

_Fichier : ESP32\_OneTask.ino_

```arduino
// ESP32_OneTask

void setup() {
  Serial.begin(9600);
  /* we create a new task here */
  xTaskCreate(
    additionalTask, /* Task function. */
    "additional Task", /* name of task. */
    10000, /* Stack size of task */
    NULL, /* parameter of the task */
    1, /* priority of the task */
    NULL); /* Task handle to keep track of created task */
}
/* the forever loop() function is invoked by Arduino ESP32 loopTask */
void loop() {
  Serial.println("this is Arduino Task");
  delay(1000);
}
/* this function will be invoked when additionalTask was created */
void additionalTask( void * parameter )
{
  /* loop forever */
  for (;;) {
    Serial.println("this is additional Task");
    delay(1000);
  }
  /* delete a task when finish,
    this will never happen because this is infinity loop */
  vTaskDelete( NULL );
}
```

_Question_ : Quel est l’affichage de ce programme ?
