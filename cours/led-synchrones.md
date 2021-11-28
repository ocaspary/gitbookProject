# LED synchrones

**Exemples de 2 LED synchrones (chacune sur un cœur de l’ESP32)**

Pour un fonctionnement synchrone, c’est-à-dire liant les deux tâches, nous utiliserons un mécanisme de synchronisation des processus souvent utilisé et reposant sur les sémaphores.

Définition Wikipédia : Un **sémaphore** est une variable (ou un type de donnée abstrait) partagée par différents « acteurs », qui garantit que ceux-ci ne peuvent y accéder que de façon séquentielle à travers des opérations [atomiques](https://fr.wikipedia.org/wiki/Atomicit%C3%A9\_\(Informatique\)), et constitue la méthode utilisée couramment pour restreindre l'accès à des ressources partagées (par exemple un espace de stockage) et [synchroniser les processus](https://fr.wikipedia.org/wiki/Synchronisation\_\(multit%C3%A2ches\)) dans un environnement de [programmation concurrente](https://fr.wikipedia.org/wiki/Programmation\_concurrente).

De plus, le sémaphore binaire utilisé, qui est une exclusion mutuelle (**mutex**), évite que des ressources partagées d’un système ne soient utilisées en même temps.

Garder le montage précédent avec 2 LED chacune en série avec une résistance de 220 Ω ou 330 Ω. GPIO12 pour la LED1 (rouge) et GPIO14 pour la LED2 (verte).

**Fonctionnement :**

Dans l’exemple ci-dessous, imaginons un relais 4x100 m. La tâche qui « prend le bâton » s’exécute sur le cœur 0, puis rend le bâton qui est pris par l’autre tâche qui s’exécute à son tour sur le cœur 1, et le libère ensuite. Le compteur est partagé entre les 2 tâches et continue de s’incrémenter lors du passage d’une tâche à l’autre.

_Fichier : ESP32\_Two\_LED\_Synchron.ino_

```arduino
/*
   This sketch uses semaphores to synchronize two LEDs. The blink sketches run in two tasks and on two cores.
   As shown in the video you can either blink them sequencially or in parallel
*/
#define LED1 12
#define LED2 14

TaskHandle_t Task1, Task2;
SemaphoreHandle_t baton;

int counter = 0;

void blink(byte pin, int duration) {

  //    digitalWrite(pin, HIGH);
  GPIO.out_w1ts = ((uint32_t)1 << pin);
  delay(duration);
  //    digitalWrite(pin, LOW);
  GPIO.out_w1tc = ((uint32_t)1 << pin);
  delay(duration);
}

void codeForTask1( void * parameter )
{
  for (;;) {
    xSemaphoreTake( baton, portMAX_DELAY );
    blink(LED1, 1000);
    xSemaphoreGive( baton );
    delay(50);
    Serial.print("Counter in Task 1: ");
    Serial.println(counter);
    counter++;
  }
}

void codeForTask2( void * parameter )
{
  for (;;) {
    xSemaphoreTake( baton, portMAX_DELAY );
    //xSemaphoreGive( baton );
    blink(LED2, 1000);
    xSemaphoreGive( baton );
    delay(50);
    Serial.print("                            Counter in Task 2: ");
    Serial.println(counter);
    counter++;
  }
}

// the setup function runs once when you press reset or power the board
void setup() {
  Serial.begin(115200);
  // initialize digital pin LED as an output.
  pinMode(LED1, OUTPUT);
  pinMode(LED2, OUTPUT);

  baton = xSemaphoreCreateMutex();

  xTaskCreatePinnedToCore(
    codeForTask1,
    "led1Task",
    1000,
    NULL,
    1,
    &Task1,
    0);
  //delay(500);  // needed to start-up task1

  xTaskCreatePinnedToCore(
    codeForTask2,
    "led2Task",
    1000,
    NULL,
    1,
    &Task2,
    1);

}

void loop() {}
```
