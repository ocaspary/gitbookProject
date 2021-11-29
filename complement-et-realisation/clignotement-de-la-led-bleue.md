# Clignotement de la LED bleue

**La tâche clignotement de la LED bleue**

Nous avons commencé ce module par un premier exemple sur l’ESP32 qui faisait clignoter la LED bleue par défaut **** (LED\_BUILTIN) sur notre modèle d’ESP32 à GPIO2.

Reprenons ce programme en écrivant une tâche blinkLed() optimisée et qui gère les ticks.

_Fichier : ESP32\_Blinked\_Task.ino_

```arduino
// ESP32_Blinked_Task

uint8_t led_pin = GPIO_NUM_2;   // LED_BUILTIN = 2 for NodeMCU ESP32

// define taskBlinkled TaskHandle_t variable
TaskHandle_t taskBlinkled;

void blinkLed(void* param) {
  // Block for 1000 ms or 1 sec.
  const TickType_t xFrequency = 1000;
  // Initialise the xLastWakeTime variable with the current time.
  TickType_t xLastWakeTime;
  xLastWakeTime = xTaskGetTickCount();
  for (;;) {
    static uint32_t pin_val = 0;
    // toggle the pin value
    pin_val ^= 1;   // exclusive-OR (XOR) operation
    //the code below instead of function digitalWrite(led_pin, pin_val) because more fast
    if (pin_val) {
      GPIO.out_w1ts = ((uint32_t)1 << led_pin);
    }
    else {
      GPIO.out_w1tc = ((uint32_t)1 << led_pin);
    }
    Serial.printf("Led ---- %s\n" , pin_val ? "On" : "Off");
    // Simply toggle the LED every 1000ms or 1 sec., Wait for the next cycle.
    vTaskDelayUntil( &xLastWakeTime, xFrequency );
  }
  taskBlinkled = NULL;
  vTaskDelete( NULL );
}

void setup() {
  // put your setup code here, to run once:
  // set the serial port at 115200
  Serial.begin(115200);
  // set the Led pin as ouput
  pinMode(led_pin, OUTPUT);
  //create a task that will be executed in the blinkLed() function, with priority 1
  xTaskCreate(
    blinkLed,         /* Task function. */
    "blinkLed",       /* name of task. */
    1024 * 2,         /* Stack size of task */
    NULL,             /* parameter of the task */
    1,                /* priority of the task */
    &taskBlinkled);   /* Task handle to keep track of created task */
}

void loop() {}
```

Comprendre le programme. Quelle différence entre vTaskDelay() et vTaskDelayUntil() ?

Une autre manière de faire clignoter la LED en langage C dans un sketch Arduino par exemple, tout en évitant l'emploi de fonctions Arduino :

```arduino
/* Blink Example
 
   This example code is in the Public Domain (or CC0 licensed, at your option.)
 
   Unless required by applicable law or agreed to in writing, this
   software is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
   CONDITIONS OF ANY KIND, either express or implied.
*/
#include <stdio.h>
#include "freertos/FreeRTOS.h"
#include "driver/gpio.h"

#define LED_PIN GPIO_NUM_2
#define BLINK_TIME 1000
 
extern "C" void app_main()
{
    uint8_t led_value = 0;
    gpio_reset_pin(LED_PIN);
    gpio_set_direction(LED_PIN, GPIO_MODE_OUTPUT);


    while (1) {
        gpio_set_level(LED_PIN, led_value);
        led_value = !led_value;
        vTaskDelay(BLINK_TIME / portTICK_PERIOD_MS);
    }
}
```

Si vous avez des contraintes d'espace mémoire, d'optimisation (temps d'exécution), et pour avoir un code optimal et professionnel, utiliser plutôt le langage C, sinon le langage C++ déjà un peu plus "gourmand". Vous pouvez aussi être amené à travailler avec le framework ESP-IDF d'Espressif plutôt que le framework Arduino.
