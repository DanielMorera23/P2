# Count Button Presses with Interrupts

Aquest projecte està dissenyat per comptabilitzar els presses d'un botó utilitzant un interruptor extern amb un microcontrolador basat en **Arduino** o una placa compatible, com ara una **ESP32**. El botó utilitza la funcionalitat d'interrupció (Interrupt Service Routine, ISR) per incrementar un comptador de presses cada vegada que es prem el botó.

## Descripció

En aquest codi, es defineix un botó connectat a un pin específic (en aquest cas, el pin 18) i es fa servir un ISR per comptabilitzar les vegades que el botó és premut. Després, el nombre total de presses s'imprimeix a través del monitor serie. A més, després d'un minut, la interrupció es desconnecta automàticament.

### Característiques

- **Interruptor de botó:** Utilitza una interrupció per detectar quan el botó és premut.
- **Comptador de presses:** Compta quantes vegades s'ha premut el botó.
- **Desconnexió automàtica:** Desconnecta la interrupció després de 1 minut d'execució.

## Funcionament

1. Quan el botó es prem, s'atribueix una interrupció que executa la funció `isr()`, augmentant el comptador de presses i marcant el botó com a premut.
2. El codi principal al bucle (`loop()`) comprova si el botó ha estat premut i imprimeix el nombre de vegades que ha estat premut al monitor sèrie.
3. Després d'un minut, la interrupció es desconnecta automàticament.

## Connexions

- El botó ha de ser connectat al pin **18** de la placa (pot ser un pin diferent, però hauràs de modificar el valor de `button1.PIN` al codi).
- Utilitza un **resistència pull-up interna** per a la configuració del botó.

## Codi

```cpp
#include <Arduino.h>

struct Button {
    const uint8_t PIN;
    uint32_t numberKeyPresses;
    bool pressed;
};

Button button1 = {18, 0, false};

void IRAM_ATTR isr() {
    button1.numberKeyPresses += 1;
    button1.pressed = true;
}

void setup() {
    Serial.begin(115200);
    pinMode(button1.PIN, INPUT_PULLUP);
    attachInterrupt(button1.PIN, isr, FALLING);
}

void loop() {
    if (button1.pressed) {
        Serial.printf("Button 1 has been pressed %u times\n", button1.numberKeyPresses);
        button1.pressed = false;
    }

    // Detach Interrupt after 1 Minute
    static uint32_t lastMillis = 0;
    if (millis() - lastMillis > 60000) {
        lastMillis = millis();
        detachInterrupt(button1.PIN);
        Serial.println("Interrupt Detached!");
    }
}
