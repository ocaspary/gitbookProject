# Sémaphore et Mutex

Les différences entre sémaphores et Mutex (Mutual Exclusion) sont notamment expliquées dans le tutoriel suivant : [https://fr.gadget-info.com/difference-between-semaphore](https://fr.gadget-info.com/difference-between-semaphore) .

Mutex est une technique de verrouillage qui évite la corruption des ressources partagées.

Voir aussi les explications dans : [https://circuitdigest.com/microcontroller-projects/arduino-freertos-tutorial-using-semaphore-and-mutex-in-freertos-with-arduino](https://circuitdigest.com/microcontroller-projects/arduino-freertos-tutorial-using-semaphore-and-mutex-in-freertos-with-arduino).

Pour ceux qui veulent aller plus loin, plusieurs exemples de code : [https://github.com/ShawnHymel/introduction-to-rtos](https://github.com/ShawnHymel/introduction-to-rtos).

_File : ESP32\_Mutex\_Parameter.ino_

```arduino
/**
   FreeRTOS Mutex Solution

   Pass a parameter to a task using a mutex.

   Date: January 20, 2021
   Author: Shawn Hymel
   License: 0BSD
*/

// You'll likely need this on vanilla FreeRTOS
//#include <semphr.h>

// Use only core 1 for demo purposes
#if CONFIG_FREERTOS_UNICORE
static const BaseType_t app_cpu = 0;
#else
static const BaseType_t app_cpu = 1;
#endif

// Pins (change this if your Arduino board does not have LED_BUILTIN defined)
static const int led_pin = 2;

// Globals
static SemaphoreHandle_t mutex;

//*****************************************************************************
// Tasks

// Blink LED based on rate passed by parameter
void blinkLED(void *parameters) {

  // Copy the parameter into a local variable
  int num = *(int *)parameters;

  // Release the mutex so that the creating function can finish
  xSemaphoreGive(mutex);

  // Print the parameter
  Serial.print("Received: ");
  Serial.println(num);

  // Configure the LED pin
  pinMode(led_pin, OUTPUT);

  // Blink forever and ever
  while (1) {
    digitalWrite(led_pin, HIGH);
    vTaskDelay(num / portTICK_PERIOD_MS);
    digitalWrite(led_pin, LOW);
    vTaskDelay(num / portTICK_PERIOD_MS);
  }
}

//*****************************************************************************
// Main (runs as its own task with priority 1 on core 1)

void setup() {

  long int delay_arg;

  // Configure Serial
  Serial.begin(115200);

  // Wait a moment to start (so we don't miss Serial output)
  vTaskDelay(1000 / portTICK_PERIOD_MS);
  Serial.println();
  Serial.println("---FreeRTOS Mutex Solution---");
  Serial.println("Enter a number for delay (milliseconds)");

  // Wait for input from Serial
  while (Serial.available() <= 0);

  // Read integer value
  delay_arg = Serial.parseInt();
  Serial.print("Sending: ");
  Serial.println(delay_arg);

  // Create mutex before starting tasks
  mutex = xSemaphoreCreateMutex();

  // Take the mutex
  xSemaphoreTake(mutex, portMAX_DELAY);

  // Start task 1
  xTaskCreatePinnedToCore(blinkLED,
                          "Blink LED",
                          1024,
                          (void *)&delay_arg,
                          1,
                          NULL,
                          app_cpu);

  // Do nothing until mutex has been returned (maximum delay)
  xSemaphoreTake(mutex, portMAX_DELAY);

  // Show that we accomplished our task of passing the stack-based argument
  Serial.println("Done!");
}

void loop() {

  // Do nothing but allow yielding to lower-priority tasks
  vTaskDelay(1000 / portTICK_PERIOD_MS);
}
```

Comprendre le programme ci-dessus et le faire fonctionner sur ESP32 (saisissez 1000 pour 1s).



_File : ESP32\_Mutex\_Semaphore.ino_

```arduino
// ESP32_Mutex_Semaphore

SemaphoreHandle_t xMutex;

void setup()
{
  Serial.begin(115200);

  /* create Mutex */
  xMutex = xSemaphoreCreateMutex();

  xTaskCreate(
    lowPriorityTask,          /* Task function. */
    "lowPriorityTask",        /* name of task. */
    1000,                     /* Stack size of task */
    NULL,                     /* parameter of the task */
    1,                        /* priority of the task */
    NULL);                    /* Task handle to keep track of created task */

  delay(500);

  /* let lowPriorityTask run first then create highPriorityTask */
  xTaskCreate(
    highPriorityTask,          /* Task function. */
    "highPriorityTask",        /* name of task. */
    1000,                      /* Stack size of task */
    NULL,                      /* parameter of the task */
    4,                         /* priority of the task */
    NULL);                     /* Task handle to keep track of created task */
}

void loop() {

}
void lowPriorityTask( void * parameter )
{
  Serial.println((char *)parameter);
  for (;;) {
    Serial.println("lowPriorityTask gains key");
    xSemaphoreTake( xMutex, portMAX_DELAY );
    /* even low priority task delay high priority still in Block state */
    vTaskDelay(pdMS_TO_TICKS(2000));
    Serial.println("lowPriorityTask releases key");
    xSemaphoreGive( xMutex );
  }
  vTaskDelete( NULL );
}

void highPriorityTask( void * parameter )
{
  Serial.println((char *)parameter);
  for (;;) {
    Serial.println("highPriorityTask gains key");
    /* highPriorityTask wait until lowPriorityTask release key */
    xSemaphoreTake( xMutex, portMAX_DELAY );
    Serial.println("highPriorityTask is running");
    Serial.println("highPriorityTask releases key");
    xSemaphoreGive( xMutex );
    /* delay so that lowPriorityTask has chance to run */
    vTaskDelay(pdMS_TO_TICKS(1000));
  }
  vTaskDelete( NULL );
}
```

Comprendre le programme et le faire fonctionner.



Travail à réaliser : Ecrire un programme qui vous permet de modifier en temps réel la période de clignotement de la LED bleue

#### Documentation FreeRTOS pour approfondir la partie RTOS

Il est indispensable de consulter la documentation en commençant par les fondamentaux :

[**https://www.freertos.org/implementation/a00002.html**](https://www.freertos.org/implementation/a00002.html)

{% embed url="https://www.freertos.org/Documentation/RTOS_book.html" %}

Amazon a repris à son compte FreeRTOS qui est disponible dans son offre IoT sur AWS. Il existe d'autres systèmes temps réel, par exemple Zephyr : [https://www.zephyrproject.org/](https://www.zephyrproject.org).

