# Esquema Elétrico e Ligações

### 1. Alimentação e Função do Díodo

* O conversor **LM2596** recebe a tensão da bateria (ex: 12V ou 24V) e deve ser regulado com um multímetro para debitar exatamente **5.0V**.
* O pino **5V** do conversor liga ao barramento positivo da breadboard.
* **Ligação do Díodo ao Raspberry Pi Pico:**
  * O díodo deve ser inserido entre a linha de 5V da breadboard e o **Pin 39 (VSYS)** do Pico.
  * **Polaridade:** O Ânodo liga aos 5V; o **Cátodo (lado que tem a risca cinzenta/branca desenhada)** liga ao **Pin 39 (VSYS)**.
  
{% hint style="info" %}
**Para que serve o díodo?**  
Impede o retorno de corrente (backfeeding) caso o carro esteja ligado e alguém ligue simultaneamente um cabo USB do computador ao Pico para ver o código no Thonny.
{% endhint %}

---

### 2. Tabela de Ligações (RPi Pico ⟷ Seeed XIAO)

{% hint style="info" %}
Todos os pinos **GND** do Raspberry Pi Pico (Pin 3, 8, 13, 18, 23, 28, 38) estão interligados internamente. Podes usar qualquer um deles para a massa comum.
{% endhint %}

{% tabs %}
{% tab title="Opção A: XIAO ESP32-S3" %}

| Sinal | Pino no Raspberry Pi Pico | Pino no XIAO ESP32-S3 | Função |
| :--- | :--- | :--- | :--- |
| **Massa (GND)** | Pin 18 (GND) | **GND** | Referência 0V comum |
| **TX ➜ RX** | Pin 11 (GP8 / UART1 TX) | **D7 (GPIO 44)** | Pico envia mensagens para o rádio |
| **RX ⮌ TX** | Pin 12 (GP9 / UART1 RX) | **D6 (GPIO 43)** | Pico recebe comandos (opcional) |
| **Alimentação** | Pin 39 (VSYS via Díodo) | **Pino 5V** | Entrada de energia regulada |

{% endtab %}

{% tab title="Opção B: XIAO nRF52840" %}

| Sinal | Pino no Raspberry Pi Pico | Pino no XIAO nRF52840 | Função |
| :--- | :--- | :--- | :--- |
| **Massa (GND)** | Pin 18 (GND) | **GND** | Referência 0V comum |
| **TX ➜ RX** | Pin 11 (GP8 / UART1 TX) | **D7 (GPIO 7)** | Pico envia mensagens para o rádio |
| **RX ⮌ TX** | Pin 12 (GP9 / UART1 RX) | **D6 (GPIO 6)** | Pico recebe comandos (opcional) |
| **Alimentação** | Pin 39 (VSYS via Díodo) | **Pino 5V** | Entrada de energia regulada |

{% endtab %}
{% endtabs %}
