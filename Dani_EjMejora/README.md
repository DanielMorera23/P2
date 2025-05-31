# Ejercicio de Mejora: Control de Frecuencia de Parpadeo con Interrupciones y Pulsadores

En este ejercicio de mejora se implementa un sistema en el ESP32 que permite controlar la **frecuencia de parpadeo de un LED** mediante interrupciones por temporizador y el uso de **dos pulsadores**:

- **GPIO18 (BTN_UP)**: Aumenta la frecuencia de parpadeo.
- **GPIO17 (BTN_DOWN)**: Disminuye la frecuencia de parpadeo.
- **GPIO4 (LED)**: LED que parpadea.

El sistema parte de una frecuencia inicial de **1Hz (500ms)** y permite ajustar este valor entre **100ms y 2000ms**. Además, se aplica **filtrado por software (debounce)** para evitar efectos indeseados al presionar los botones.

## Código `main.cpp`

```cpp
#include <Arduino.h>

const int LED_PIN = 4;       // LED en GPIO4
const int BTN_UP = 18;       // Botón para aumentar la frecuencia
const int BTN_DOWN = 17;     // Botón para disminuir la frecuencia

volatile int interruptCounter = 0;
volatile int blinkDelay = 500; // Frecuencia inicial (500ms -> 1Hz)
volatile bool ledState = false;
volatile unsigned long lastPressUp = 0;
volatile unsigned long lastPressDown = 0;
const int debounceTime = 200; // Tiempo de debounce en ms

hw_timer_t *timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

// Función de interrupción del temporizador
void IRAM_ATTR onTimer() {
    portENTER_CRITICAL_ISR(&timerMux);
    interruptCounter++;
    portEXIT_CRITICAL_ISR(&timerMux);
}

// Verificación de botones con debounce
void IRAM_ATTR checkButtons() {
    unsigned long currentMillis = millis();

    if (digitalRead(BTN_UP) == LOW && (currentMillis - lastPressUp > debounceTime)) {
        lastPressUp = currentMillis;
        if (blinkDelay > 100) {
            blinkDelay -= 50;
        }
    }

    if (digitalRead(BTN_DOWN) == LOW && (currentMillis - lastPressDown > debounceTime)) {
        lastPressDown = currentMillis;
        if (blinkDelay < 2000) {
            blinkDelay += 50;
        }
    }
}

void setup() {
    Serial.begin(115200);
    pinMode(LED_PIN, OUTPUT);
    pinMode(BTN_UP, INPUT_PULLUP);
    pinMode(BTN_DOWN, INPUT_PULLUP);

    timer = timerBegin(0, 80, true); // 1 tick = 1µs
    timerAttachInterrupt(timer, &onTimer, true);
    timerAlarmWrite(timer, 500000, true); // 500ms inicial
    timerAlarmEnable(timer);

    Serial.println("Sistema iniciado...");
}

void loop() {
    if (interruptCounter > 0) {
        portENTER_CRITICAL(&timerMux);
        interruptCounter--;
        portEXIT_CRITICAL(&timerMux);

        ledState = !ledState;
        digitalWrite(LED_PIN, ledState);

        checkButtons(); // Ajustar frecuencia si se presiona un botón

        timerAlarmWrite(timer, blinkDelay * 1000, true); // Actualizar temporizador
    }
}
