# LED asynchrones

**Exemples de 2 LED asynchrones (chacune sur un cœur de l’ESP32)**

Réaliser un montage avec 2 LED chacune en série avec une résistance de 220 Ω ou 330 Ω.

Le code ci-dessous utilise GPIO12 pour la LED1 (rouge) et GPIO14 pour la LED2 (verte).

_Fichier : ESP32\_Two\_LED\_Asynchron.ino_

```arduino
/*
   This blinks two LEDs independently and not synchronized. Both have other blink frequencies.
   The blink sketches run in two tasks and on two cores.
*/

#define LED1 12
#define LED2 14

TaskHandle_t Task1, Task2;

//int counter = 0;

void blink(byte pin, int duration) {

  //    digitalWrite(pin, HIGH);
  //    delay(duration);
  //    digitalWrite(pin, LOW);
  //    delay(duration);

  GPIO.out_w1ts = ((uint32_t)1 << pin);
  delay(duration);
  GPIO.out_w1tc = ((uint32_t)1 << pin);
  delay(duration);
}

void codeForTask1( void * parameter )
{
  for (;;) {
    blink(LED1, 1000);
    //delay(50);
    Serial.println("Task 1: ");
  }
}

void codeForTask2( void * parameter )
{
  for (;;) {
    blink(LED2, 1200);
    //delay(50);
    Serial.println("             Task 2: ");
  }
}

// the setup function runs once when you press reset or power the board
void setup() {
  Serial.begin(115200);
  // initialize digital pin LED_BUILTIN as an output.

  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);

  xTaskCreatePinnedToCore(
    codeForTask1, /* Task function. */
    "led1Task",   /* name of task. */
    1000,         /* Stack size of task */
    NULL,         /* parameter of the task */
    1,            /* priority of the task */
    &Task1,       /* Task handle to keep track of created task */
    0);           /* pin task to core 0 */
  //delay(10);  // needed to start-up task1

  xTaskCreatePinnedToCore(
    codeForTask2,
    "led2Task",
    1000,
    NULL,
    1,
    &Task2,
    1);

}

void loop() {
  //delay(1000);
}
```

La tâche 1 (Task1) s’exécute sur le cœur 0 et fait clignoter la LED1 toutes les secondes (1000 ms). La tâche 2 (Task2) s’exécute sur le cœur 1 et fait clignoter la LED2 toutes les 1,2 s (1200 ms).&#x20;

Les 2 LED fonctionnent de manière indépendante, c’est pourquoi on dit qu’elles sont asynchrones. Cependant, dans notre exemple, avec les durées choisies, les 2 LED s’allument en même temps toutes les 6 secondes.

Si on met la même durée de clignotement à 1 seconde, les 2 LED clignotent simultanément mais les tâches continuent de fonctionner de manière indépendante.
