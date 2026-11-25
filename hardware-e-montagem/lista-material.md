# Lista de Material (BOM)

A tabela abaixo lista os componentes necessários para construir **uma unidade para o carro** e **uma unidade de receção para as boxes**.

| Componente | Quant. (Carro) | Quant. (Box) | Descrição |
| :--- | :---: | :---: | :--- |
| **Raspberry Pi Pico** (RP2040) | 1 | 0 | Microcontrolador central com pinos soldados. |
| **Kit Seeed XIAO + Wio-SX1262** | 1 | 1 | Rádio LoRa (versão ESP32-S3 ou nRF52840). |
| **Antena LoRa 868 MHz** | 1 | 1 | Antena obrigatória para os módulos de rádio. |
| **Conversor DC-DC Step-Down** | 1 | 0 | LM2596 (reduz a tensão da bateria para 5.0V estáveis). |
| **Díodo de Proteção** | 1 | 0 | Schottky 1N5817 (ou 1N4001/1N4007). |
| **Cabo JST-PH 2.0mm (4 pinos)** | 1 | 0 | Conector compatível com a porta VE.Direct do Victron. |
| **Mini Breadboard (400 pontos)**| 1 | 0 | Placa de prototipagem para fixação dos módulos. |
| **Cabos Jumper Dupont** | 1 kit | 0 | Ligações macho-macho e fêmea-macho. |
| **Fusível 25A Automóvel** | 1 suplente | 0 | Fusível de reserva para o MPPT Victron 100/20. |

{% hint style="danger" %}
**Aviso Crítico sobre a Antena:**  
**Nunca ligues o XIAO ou qualquer módulo LoRa à corrente sem ter a antena de 868 MHz devidamente enroscada.** Ligar o rádio sem antena queima o circuito integrado de transmissão em frações de segundo.
{% endhint %}
