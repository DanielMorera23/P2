# Count Timer Interrupts

Aquest projecte utilitza un temporitzador per generar interrupcions a intervals específics, i cada vegada que es produeix una interrupció, es comptabilitza i es mostra el nombre total d'interrupcions a través del monitor sèrie.

## Descripció

En aquest codi, un temporitzador és configurat per generar una interrupció cada 1 segon (1000000 microsegons). Quan es produeix una interrupció, es comptabilitza i es mostra el nombre total d'interrupcions al monitor sèrie. El codi utilitza el mecanisme d'interrupcions per garantir una resposta eficient i precisa a cada temporització.

### Característiques

- **Temporitzador d'interrupció:** Genera una interrupció cada 1 segon.
- **Comptador d'interrupcions:** Compta el nombre d'interrupcions generades pel temporitzador.
- **Monitor sèrie:** Imprimeix el nombre total d'interrupcions en cada moment.

## Funcionament

1. Un temporitzador és configurat per generar interrupcions cada 1 segon (1000000 microsegons).
2. Cada vegada que el temporitzador genera una interrupció, el comptador d'interrupcions es suma en la variable `interruptCounter`.
3. Quan el codi al bucle (`loop()`) detecta que una interrupció ha tingut lloc (si `interruptCounter` és més gran que 0), el comptador es decrementa i es suma al comptador total d'interrupcions (`totalInterruptCounter`).
4. Finalment, el nombre total d'interrupcions es mostra al monitor sèrie.

## Connexions

No es necessiten connexions específiques per aquest projecte. Es basa només en la funcionalitat interna del microcontrolador.

## Codi

```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;
hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
    portENTER_CRITICAL_ISR(&timerMux);
    interruptCounter++;
    portEXIT_CRITICAL_ISR(&timerMux);
}

void setup() {
    Serial.begin(115200);
    timer = timerBegin(0, 80, true);
    timerAttachInterrupt(timer, &onTimer, true);
    timerAlarmWrite(timer, 1000000, true);
    timerAlarmEnable(timer);
}

void loop() {
    if (interruptCounter > 0) {
        portENTER_CRITICAL(&timerMux);
        interruptCounter--;
        portEXIT_CRITICAL(&timerMux);
        totalInterruptCounter++;
        Serial.print("An interrupt has occurred. Total number: ");
        Serial.println(totalInterruptCounter);
    }
}




