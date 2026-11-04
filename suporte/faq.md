---
description: Resolução dos erros e dúvidas mais comuns durante os testes de pista.
---

# Diagnóstico & FAQ

### 1. O rádio XIAO não transmite e não há sinal LoRa
* **Causa provável:** Pinos de série incorretos na configuração do Meshtastic.  
  * No XIAO ESP32-S3 o RX é o **GPIO 44** e o TX é o **GPIO 43**.
  * No XIAO nRF52840 o RX é o **GPIO 7** e o TX é o **GPIO 6**.
* Confirma se a linha TX do Raspberry Pi Pico (Pin 11) liga à linha RX do XIAO (Pin D7).

### 2. O Pico não regista valores nenhuns do Victron
* Retiraste a ponte plástica preta (jumper) que vem na ficha VE.Direct do Victron?
* Confirma se o fio do pino 3 do Victron (TX) está conectado ao **Pin 2 (GP1 / UART0 RX)** do Raspberry Pi Pico.

### 3. Posso ligar o cabo USB ao Pico com o carro ligado?
* **Sim**, desde que o **díodo de proteção** esteja instalado no Pin 39 (VSYS) com a risca do cátodo virada para o Pico. O díodo impede que a alimentação do USB e a do conversor LM2596 entrem em curto-circuito.

### 4. Como tornar a telemetria da nossa escola privada?
* Por omissão, todas as equipas usam o canal `VROOOM` e a chave `RA==` para que os carros possam retransmitir os dados uns dos outros em pista alargando a cobertura.
* Se quiseres um canal exclusivo da tua escola:
  1. No gestor do Meshtastic, altera o nome do canal (ex: `SOLAR_ESCOLA_A`).
  2. Gera uma nova chave PSK aleatória.
  3. Aplica essa mesma chave no módulo transmissor do carro e no módulo recetor das boxes.
