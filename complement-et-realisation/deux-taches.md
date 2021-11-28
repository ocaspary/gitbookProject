# Deux tâches



**Création et exécution de deux tâches**

_Fichier : ESP32\__Two_Tasks.ino_

```arduino
// ESP32_TwoTasks

void setup() {
  Serial.begin(9600);
  delay(1000);
  xTaskCreate(
    taskOne, /* Task function. */
    "TaskOne", /* String with name of task. */
    10000, /* Stack size in words. */
    NULL, /* Parameter passed as input of the task */
    1, /* Priority of the task. */
    NULL); /* Task handle. */
  xTaskCreate(
    taskTwo, /* Task function. */
    "TaskTwo", /* String with name of task. */
    10000, /* Stack size in words. */
    NULL, /* Parameter passed as input of the task */
    1, /* Priority of the task. */
    NULL); /* Task handle. */
}
void loop() {
  delay(1000);
}
void taskOne( void * parameter )
{
  for ( int i = 0; i < 5; i++ ) {
    Serial.println("Hello from task 1");
    delay(1000);
  }
  Serial.println("Ending task 1");
  vTaskDelete( NULL );
}
void taskTwo( void * parameter)
{
  for ( int i = 0; i < 5; i++ ) {
    Serial.println("Hello from task 2");
    delay(1000);
  }
  Serial.println("Ending task 2");
  vTaskDelete( NULL );
}
```

_Question :_ Quel est l’affichage ? Si vous changez la priorité de la tâche 2 de 1 à 2, que se passe-t-il ?
