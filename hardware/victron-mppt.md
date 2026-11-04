---
description: Guia de adaptação física da ficha, pinout e ligação da porta VE.Direct do regulador Victron MPPT ao Raspberry Pi Pico.
---

# Ligação ao MPPT Victron (VE.Direct)

Os controladores de carga solar da Victron Energy (modelos BlueSolar e SmartSolar MPPT, como o 75/15 ou 100/20) possuem uma interface de diagnóstico e comunicação proprietária designada por **VE.Direct**.

O Raspberry Pi Pico liga-se diretamente a esta porta para escutar passivamente as grandezas do carro solar: tensão do painel solar, corrente gerada, tensão da bateria e estado do carregador.

---

## 1. O Jumper de Curto-Circuito de Fábrica

{% hint style="warning" %}
**Passo Obrigatório:**  
Em quase todos os controladores Victron novos, a porta VE.Direct vem de fábrica com uma pequena peça plástica preta (um *jumper*) encaixada nos pinos.
{% endhint %}

Esta peça serve para manter os pinos de comunicação em curto-circuito enquanto o aparelho está em armazém. Para conseguires ligar o cabo de telemetria:

1. Localiza a porta retangular branca de 4 pinos na parte inferior do Victron.
2. Com os dedos ou com a ajuda de um pequeno alicate de pontas, **puxa a peça plástica preta para fora**.
3. Guarda esse jumper na caixa de ferramentas da equipa (caso precises de repor o Victron no estado de fábrica).

---

## 2. Preparação e Desbaste do Conector JST / Grove

A tomada fêmea VE.Direct é bastante profunda e tem guias laterais muito estreitas. Se utilizares um cabo adaptador comum do tipo **Grove (Seeed Studio)** ou uma ficha **JST-PH 2.0mm standard**, notarás que ela não entra até ao fim devido às patilhas plásticas exteriores de bloqueio.

{% hint style="danger" %}
**Cuidados de Segurança ao Cortar:**  
Pede apoio ao professor ou usa uma base de corte firme. Apoia a ficha sobre uma superfície plana e corta sempre no sentido oposto aos teus dedos.
{% endhint %}

![Desbaste da patilha plástica exterior da ficha com x-ato](../assets/corte-conector-jst.png)

### Como adaptar o conector passo a passo:
1. Com uma lâmina de x-ato afiada, apara a saliência/patilha plástica exterior da ficha de modo a torná-la lisa e retangular.
2. Faz um teste de encaixe no Victron (sem forçar): o conector deve deslizar facilmente e fazer contacto firme com os 4 pinos de metal no fundo da porta.

---

## 3. Pinout da Ficha VE.Direct

{% hint style="danger" %}
**Aviso Crítico sobre as Cores dos Fios:**  
Os cabos adaptadores comerciais **não seguem as cores elétricas convencionais**. Em muitos destes cabos, o fio vermelho não é positivo e o preto pode não ser a massa!  
**Nunca confies na cor do fio.** Orienta-te sempre pela **posição física do pino no conector do Victron**.
{% endhint %}

![Identificação das cores do cabo e pinout na porta do Victron](../assets/cabo-jst-cores.png)

```
  Porta VE.Direct no Victron (Vista Frontal)
  ┌────────────────────────────────────────┐
  │   [1]        [2]        [3]        [4] │
  └───┬──────────┬──────────┬──────────┬───┘
      │          │          │          │
     GND         RX         TX        +5V
   (Massa)    (Entrada)   (Saída)  (Potência)
```

![Pinout detalhado dos pinos da porta VE.Direct](../assets/victron-pinout-detalhe.png)

### Tabela de Ligações ao Raspberry Pi Pico

| Pino Victron | Sinal | Ligação no Raspberry Pi Pico | Descrição / Função |
| :---: | :---: | :--- | :--- |
| **Pino 1** | **GND** | **Pin 38 (GND)** | Massa de referência de sinal (0V comum). |
| **Pino 2** | **RX** | *Não Ligar (Isolar)* | Entrada de receção do Victron (não usada na leitura passiva). |
| **Pino 3** | **TX** | **Pin 2 (GP1 / UART0 RX)** | **Linha de Dados:** Envia as mensagens de telemetria a 19200 baud. |
| **Pino 4** | **+5V** | *Não Ligar (Isolar)* | Saída de alimentação de 5V do Victron. **Manter desligada!** |

---

## 4. Porquê ligar apenas 2 fios?

Para recolher a telemetria, **apenas o Pino 1 (GND) e o Pino 3 (TX) devem ser ligados**:

1. **Leitura Unidirecional (Apenas TX ➜ RX):** O Victron emite automaticamente os seus dados a cada segundo em texto simples. O Pico apenas precisa de "escutar" o sinal pelo seu pino GP1 (UART0 RX). Não é necessário enviar comandos para o Victron (dispensando o pino RX).
2. **Prevenção de Conflitos Elétricos:** O sistema de telemetria (Pico + XIAO) já é alimentado pelo conversor Step-Down LM2596 regulado a 5V. Se ligasses também o pino de +5V do Victron ao Pico, criarias circuitos de retorno de corrente (*ground loops*) e conflito de reguladores, o que danificaria o microcontrolador.
3. **Isolamento dos fios sobrantes:** Os fios dos pinos 2 e 4 que não são usados devem ser cortados rente ou devidamente isolados com manga termo-retrátil ou fita isoladora para não tocarem no chassis do carro.

---

## 5. O que é transmitido pela Porta VE.Direct?

O controlador Victron utiliza o protocolo **VE.Direct Text**. A cada 1 segundo, transmite um bloco de texto contínuo a **19200 bauds** (formato 8N1: 8 bits de dados, sem paridade, 1 stop bit).

O script em MicroPython do Raspberry Pi Pico filtra este fluxo e extrai variáveis como:

* **`V` (mV):** Tensão da bateria do carro em milivolts.
* **`I` (mA):** Corrente fornecida à bateria em miliamperes.
* **`VPV` (mV):** Tensão nos terminais dos painéis solares.
* **`PPV` (W):** Potência instantânea gerada pelo painel solar.
* **`CS` (Estado da Carga):** Modo do MPPT (*Off*, *Fault*, *Bulk*, *Absorption*, *Float*).
* **`ERR` (Código de Erro):** Diagnóstico de anomalias elétricas.

---

## 6. Cuidados com o Fusível de Proteção

O modelo **Victron SmartSolar / BlueSolar MPPT 100/20** vem equipado com um fusível automóvel plano standard de **25 Amperes** alojado na base do aparelho.

{% hint style="info" %}
**Dica para os Dias de Corrida:**  
Se durante a montagem do veículo solar houver uma inversão acidental de polaridade na bateria, este fusível queima de imediato para proteger a eletrónica interna. Tem sempre na caixa da equipa **pelo menos dois fusíveis suplentes de 25A**.
{% endhint %}
