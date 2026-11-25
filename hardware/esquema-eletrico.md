---
description: Esquema detalhado de montagem na breadboard, fontes de alimentação alternativas e diagramas oficiais de pinout do Raspberry Pi Pico e Seeed XIAO.
---

# Esquema Elétrico e Ligações

Esta página explica como interligar todos os componentes na breadboard de forma segura, com especial atenção à convenção de pinos, à polaridade dos componentes e à proteção elétrica contra retorno de corrente.

![Diagrama de montagem dos componentes na breadboard](../assets/TELEMETRIA_CIRCUITO.png)

---

## 1. Circuito de Alimentação (LM2596 & Díodo de Proteção)

O circuito é habitualmente alimentado a partir da bateria principal do carro solar (normalmente entre 12V e 24V DC).

```
[Bateria do Carro (12V - 24V)]
               │
               ▼
   [Conversor LM2596 Step-Down]
               │
               ├─► Regulado rigorosamente para 5.0V DC
               │
               ├──► Barramento Positivo da Breadboard (5V)
               │      ├─► Pino 5V do Seeed XIAO
               │      └─► [Ânodo] Díodo Schottky [Cátodo com risca] ──► Pin 39 (VSYS) do Pico
               │
               └──► Barramento Negativo da Breadboard (GND comum)
```

### Passo Prévio Obrigatório: Afinar o Regulador
{% hint style="danger" %}
**Risco de queima por sobretensão:**  
O conversor LM2596 standard suporta até 30V–35V na entrada. **Antes de ligares qualquer componente à breadboard**, liga o LM2596 à bateria do carro e, com as pontas de um multímetro na saída, roda o parafuso do potenciómetro até obteres rigorosamente **5.0V DC**.
{% endhint %}

### Como e Porquê Ligar o Díodo de Proteção:
* O díodo (idealmente um **Schottky 1N5817** ou um comum 1N4001) cria uma barreira unidirecional entre a fonte do carro e o microcontrolador.
* **Polaridade do Díodo:**
  * **Ânodo (lado preto sem risca):** Liga à linha de 5.0V da breadboard.
  * **Cátodo (lado com a risca prateada/branca):** Liga ao **Pin 39 (VSYS)** do Raspberry Pi Pico.
* **Porquê esta proteção?**  
  Se precisares de ligar o cabo USB do computador ao Pico na grelha de partida (para ajustar código no Thonny) com o carro ligado, o díodo impede que a corrente do computador e a do regulador LM2596 colidam, evitando queimar as portas USB ou o microcontrolador.

---

## Formas Alternativas de Alimentação

Se o regulamento da competição não permitir efetuar ligações à bateria principal de tração, ou se pretenderes testar a telemetria na bancada da sala de aula sem o carro solar, podes recorrer a uma destas opções:

{% tabs %}
{% tab title="Opção 1: Power Bank via USB-C do XIAO (Recomendada para Bancada)" %}

Esta é a solução mais prática para testes e bancada, pois **não exige cortar nem descarnar cabos USB**:

1. Liga um cabo USB-C normal de telemóvel da *Power Bank* diretamente à porta USB-C do **Seeed Studio XIAO**.
2. O XIAO alimenta-se a si próprio e coloca 5.0V no seu pino **`5V`**.
3. Do pino **`5V`** do XIAO, a corrente segue pela breadboard através do díodo de proteção até ao **Pin 39 (VSYS)** do Raspberry Pi Pico.
4. O pino **GND** do XIAO garante a massa comum com o Pico.

```
 [Power Bank 5V]
       │ (Cabo USB-C normal)
       ▼
 [Porta USB-C do XIAO] ──► Alimenta XIAO + Rádio LoRa
       │
       └─► Pino 5V do XIAO ──► [Ânodo] Díodo [Cátodo] ──► Pin 39 (VSYS) do Pico
```

{% hint style="danger" %}
**Regra de Segurança:**  
Se estiveres a alimentar o sistema via USB-C no XIAO, **desliga o fio de 5V vindo do LM2596**. Nunca deves ter duas fontes de alimentação conectadas à breadboard em simultâneo.
{% endhint %}

{% hint style="tip" %}
**Vantagem no Consumo:**  
O consumo conjunto do XIAO a emitir em LoRa (~100 mA) e do Pico a correr o MicroPython (~40 mA) totaliza cerca de 140–160 mA. Esta corrente é normalmente suficiente para evitar que as *Power Banks* inteligentes se desliguem sozinhas por inatividade.
{% endhint %}

{% endtab %}

{% tab title="Opção 2: Bateria Auxiliar 12V (Recomendada para Corridas)" %}

Utiliza uma pequena bateria independente de 12V (ex: bateria selada de alarme de 1.2Ah ou bateria de lítio 3S de 11.1V):
* **Ligação:** Liga os terminais da bateria auxiliar diretamente à entrada do conversor LM2596 (`IN+` e `IN-`).
* **Vantagem:** Mantém o circuito elétrico idêntico ao original, isola a telemetria do ruído elétrico do motor de tração e cumpre os regulamentos de isolamento galvânico da maioria das competições escolares.

{% endtab %}

{% tab title="Opção 3: Suporte de Pilhas AA/AAA" %}

Uma solução económica com componentes comuns em contexto escolar:
* **3 Pilhas Alcalinas (1.5V x 3 = 4.5V)** ou **4 Pilhas Recarregáveis NiMH (1.2V x 4 = 4.8V)**.
* **Ligação:** O fio positivo (+4.5V a 4.8V) liga ao barramento de 5V da breadboard; o fio negativo liga ao barramento GND.
* Ambas as placas funcionam perfeitamente com tensões entre 4.0V e 5.0V e não existe qualquer risco de corte automático de energia.

{% endtab %}

{% tab title="Opção 4: Bateria LiPo 3.7V (1S / 18650)" %}

Para uma montagem ultraleve:
* **No Raspberry Pi Pico:** O regulador interno do Pico aceita tensões entre 1.8V e 5.5V, pelo que podes ligar o polo positivo da célula de 3.7V diretamente ao **Pin 39 (VSYS)** e o negativo ao **GND**.
* **No Seeed XIAO:** Liga os fios da bateria aos contactos metálicos (**pads `BAT+` e `BAT-`**) situados na face inferior da placa.

{% endtab %}
{% endtabs %}

---

## 2. Diagramas de Pinout Oficiais

Para consultar visualmente onde fica cada pino antes de encaixar os fios:

### Pinout Oficial do Raspberry Pi Pico
No Raspberry Pi Pico, identifica os pinos de comunicação **GP8 (Pin 11)**, **GP9 (Pin 12)**, o pino de leitura do Victron **GP1 (Pin 2)**, a alimentação **VSYS (Pin 39)** e a massa **GND (Pin 18 ou 38)**:

![Pinout Oficial do Raspberry Pi Pico](https://www.raspberrypi.com/documentation/microcontrollers/images/pico-pinout.svg)

---

### Pinout Oficial do Seeed Studio XIAO

Seleciona abaixo a versão da placa XIAO que a tua equipa está a utilizar para veres o mapa de pinos correspondente:

{% tabs %}
{% tab title="XIAO ESP32-S3 (Pinout)" %}

![Pinout Oficial Seeed Studio XIAO ESP32-S3](https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/XIAO_ESP32-S3_Sense_front_pinout.png)

* **Etiqueta D7 (RX de série):** Corresponde ao **GPIO 44** no software Meshtastic.
* **Etiqueta D6 (TX de série):** Corresponde ao **GPIO 43** no software Meshtastic.
* **Pinos 5V e GND:** Situados no topo esquerdo da placa.

{% endtab %}

{% tab title="XIAO nRF52840 (Pinout)" %}

![Pinout Oficial Seeed Studio XIAO nRF52840](https://files.seeedstudio.com/wiki/XIAO-BLE/XIAO_nRF52840_front_pinout.png)

* **Etiqueta D7 (RX de série):** Corresponde ao **GPIO 7** (P0.07 / P1.12) no software Meshtastic.
* **Etiqueta D6 (TX de série):** Corresponde ao **GPIO 6** (P0.06 / P1.11) no software Meshtastic.
* **Pinos 5V e GND:** Situados no topo esquerdo da placa.

{% endtab %}
{% endtabs %}

---

## 3. Convenção de Cores para os Fios Jumper

Para evitar confusões visuais na montagem, padroniza as cores dos fios na bancada da equipa:

* ⬛ **Fio Preto:** Linha de Massa comum (GND).
* 🟥 **Fio Vermelho:** Linha de Alimentação positiva (5V / VSYS).
* 🟨 **Fio Amarelo:** Envio de dados do Pico para o rádio XIAO (TX ➜ RX).
* 🟩 **Fio Verde:** Retorno de dados do rádio XIAO para o Pico (RX ⮌ TX).

---

## 4. Tabela de Interligação "De ➜ Para"

{% hint style="info" %}
Todos os pinos identificados como **GND** no Raspberry Pi Pico (Pin 3, 8, 13, 18, 23, 28 e 38) estão ligados internamente na mesma placa de circuito impresso. Podes usar qualquer um deles.
{% endhint %}

| Fio Recomendado | No RASPBERRY PI PICO<br>*(Microcontrolador Principal)* | No SEEED STUDIO XIAO<br>*(Texto impresso na placa)* | O que faz este fio? (Função Elétrica) |
| :---: | :--- | :--- | :--- |
| ⬛ **Preto** | **Pin 18 (GND)** *(ou Pin 38)* | **Pino GND** | Cria a referência de 0V (massa comum) entre as duas placas. |
| 🟨 **Amarelo** | **Pin 11** *(Pino de função GP8 / UART1 TX)* | **Pino D7** | **Envio:** O Pico envia os dados formatados para o rádio LoRa. |
| 🟩 **Verde** | **Pin 12** *(Pino de função GP9 / UART1 RX)* | **Pino D6** | **Receção:** O Pico escuta respostas ou comandos do rádio. |
| 🟥 **Vermelho** | **Pin 39 (VSYS)**<br>*(Ligado após o cátodo do díodo)* | **Pino 5V** *(ou VBUS)* | Distribui a energia estável de 5.0V a ambos os módulos. |

---

## 5. Mapeamento de Pinos para a Configuração do Meshtastic

Ao parametrizares o módulo de rádio no **Meshtastic** (no separador *Module Config ➜ Serial*), deves introduzir o número GPIO correspondente à versão da placa XIAO que a tua equipa está a usar:

{% tabs %}
{% tab title="Se usares Seeed Studio XIAO ESP32-S3" %}

* **RX Pin no software:** Introduz **`44`** *(este pino corresponde fisicamente à etiqueta D7 na placa)*.
* **TX Pin no software:** Introduz **`43`** *(este pino corresponde fisicamente à etiqueta D6 na placa)*.

{% endtab %}

{% tab title="Se usares Seeed Studio XIAO nRF52840" %}

* **RX Pin no software:** Introduz **`7`** *(este pino corresponde fisicamente à etiqueta D7 na placa)*.
* **TX Pin no software:** Introduz **`6`** *(este pino corresponde fisicamente à etiqueta D6 na placa)*.

{% endtab %}
{% endtabs %}

---

## 6. Ligação do Cabo ao Victron MPPT (Resumo)

O cabo JST-PH de telemetria vindo do regulador solar liga-se diretamente ao Pico em pinos dedicados:

* **Pino 1 do Victron (GND):** Liga ao **Pin 38 (GND)** do Raspberry Pi Pico.
* **Pino 3 do Victron (TX):** Liga ao **Pin 2 (GP1 / UART0 RX)** do Raspberry Pi Pico.
* *(Os pinos 2 e 4 do Victron devem ficar isolados e desligados).*

Para instruções completas sobre o desbaste da ficha JST e remoção do jumper de fábrica, consulta a página dedicada: **[Ligação ao MPPT Victron](victron-mppt.md)**.

---

## 7. Checklist de Segurança Antes de Ligar a Energia

Antes de ligares o interruptor do carro solar, confirma os seguintes 4 pontos críticos:

- [ ] **Antena LoRa:** A antena de 868 MHz está enroscada no conector do XIAO? *(Ligar sem antena queima o transmissor).*
- [ ] **Tensão do LM2596:** Foi confirmada com multímetro a saída em exatamente 5.0V? *(Não aplicável se usares alimentação exclusiva por USB-C/Power Bank).*
- [ ] **Polaridade do Díodo:** A risca prateada/branca do díodo está virada para o Pin 39 (VSYS) do Pico?
- [ ] **Jumper do Victron:** O jumper de plástico preto de fábrica foi retirado da porta VE.Direct?
