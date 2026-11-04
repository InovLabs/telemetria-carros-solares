### 1. Recolha no Carro (Raspberry Pi Pico)
O Raspberry Pi Pico escuta continuamente a porta de diagnóstico VE.Direct do regulador Victron. As leituras recebidas são processadas e armazenadas de imediato num ficheiro de texto interno (`dados_corrida_testing.txt`). Em simultâneo, o Pico formata as variáveis críticas (tensão do painel, corrente de carga, tensão da bateria e potência instantânea) e envia-as para o rádio LoRa.

### 2. Transmissão Sem Fios (Meshtastic / LoRa)
O Seeed Studio XIAO (acoplado à placa Wio-SX1262) corre o firmware Meshtastic configurado em modo `TEXTMSG Serial`. Sempre que o Pico injeta uma linha de texto na porta de série, o rádio encapsula-a e emite-a na frequência europeia de **868 MHz**. Graças ao protocolo de rede em malha (*mesh*), os veículos em pista podem retransmitir os dados uns dos outros, contornando obstáculos e eliminando zonas mortas no circuito.

### 3. Receção e Análise (Estação Base & Grafana)
Na bancada das boxes, um segundo módulo LoRa ligado por USB a um computador portátil recolhe as transmissões. Um script em Python lê os pacotes e alimenta um painel no Grafana, onde a equipa consegue monitorizar a estratégia energética da corrida em direto.

---

## Especificações Técnicas do Sistema

| Característica | Especificação Técnica |
| :--- | :--- |
| **Microcontrolador Central** | Raspberry Pi Pico (RP2040 Dual-Core ARM Cortex-M0+) |
| **Linguagem do Firmware** | MicroPython v1.20+ |
| **Módulo de Rádio** | Seeed Studio XIAO (ESP32-S3 ou nRF52840) + Wio-SX1262 |
| **Frequência de Rádio** | 868 MHz (Banda ISM regulamentada para a União Europeia) |
| **Preset de Transmissão** | LoRa `SHORT_FAST` (otimizado para baixa latência de telemetria) |
| **Taxa de Leitura MPPT** | 19200 bauds (protocolo VE.Direct passivo) |
| **Taxa RPi Pico ⟷ LoRa** | 38400 bauds (UART1) |
| **Alimentação do Circuito** | 5.0 V DC (através de conversor Step-Down LM2596 da bateria do carro) |
| **Segurança Elétrica** | Díodo de bloqueio no pino VSYS contra retorno por USB |
| **Painel de Controlo** | Grafana em contentor Docker alimentado por script Python (`uv`) |

---

## Como Navegar nesta Documentação

Para montar e colocar a funcionar o teu sistema de telemetria, segue as secções pela ordem recomendada:

1. **[Lista de Material](hardware/lista-material.md):** Relação detalhada de todos os componentes necessários e ferramentas recomendadas.
2. **[Esquema Elétrico e Ligações](hardware/esquema-eletrico.md):** Como interligar o conversor step-down, o díodo de proteção, o Pico e o XIAO na breadboard.
3. **[Ligação ao MPPT Victron](hardware/victron-mppt.md):** Identificação correta dos pinos do cabo VE.Direct e remoção do jumper de curto-circuito de fábrica.
4. **[Caixa Protetora 3D](hardware/caixa-3d.md):** Ficheiros STL e recomendações para imprimir a caixa em PETG ou ABS.
5. **[Raspberry Pi Pico (MicroPython)](firmware/raspberry-pi-pico.md):** Como instalar o MicroPython, carregar o script `main.py` com o Thonny e descarregar os ficheiros de log da corrida.
6. **[Rádio LoRa (Meshtastic)](firmware/meshtastic-lora.md):** Instalação web do firmware, configuração dos canais rádio e pinos GPIO da porta de série.
7. **[Estação Base & Grafana](software/estacao-base.md):** Preparação do ambiente Python moderno com `uv` e arranque dos gráficos em Docker.
8. **[Diagnóstico & FAQ](suporte/faq.md):** Resolução rápida dos problemas mais frequentes durante os testes de pista.

---

## ⚠️ Avisos Críticos de Montagem

Antes de iniciares a montagem prática, tem sempre em conta três regras de segurança de hardware:

{% hint style="danger" %}
**1. Risco de Queima do Rádio LoRa (Sem Antena):**  
**Nunca ligues a alimentação elétrica ao Seeed Studio XIAO sem teres a respetiva antena externa de 868 MHz montada.** Transmitir sem carga de antena destrói permanentemente o andar de amplificação do chip Semtech SX1262.
{% endhint %}

{% hint style="warning" %}
**2. Atenção à Tensão de Entrada do Step-Down:**  
O módulo conversor LM2596 standard suporta uma tensão máxima de entrada de 30V a 35V DC. Verifica sempre com um multímetro a tensão da bateria do carro antes de ligar a placa e **ajusta o potenciómetro para exatamente 5.0V** antes de alimentar a breadboard.
{% endhint %}

{% hint style="warning" %}
**3. Jumper da Porta do Victron:**  
A maioria dos controladores Victron novos traz um jumper plástico preto a fechar a porta VE.Direct. Esse jumper **tem de ser removido** para ser possível encaixar a ficha do cabo de telemetria.
{% endhint %}
