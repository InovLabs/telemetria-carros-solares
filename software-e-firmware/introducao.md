---
description: Visão geral da arquitetura de software, fluxo de dados entre os 3 componentes e configuração do rádio recetor nas boxes.
---

# Introdução à Programação & Fluxo de Dados

Antes de avançares para a programação individual de cada placa, é fundamental compreender a arquitetura global do sistema e como os três componentes de software cooperam entre si.

Nenhum dos componentes faz tudo sozinho: dividimos o sistema em **recolha local**, **transporte rádio** e **estação de receção**.

---

## 1. O Mapa dos 3 Componentes

```
┌────────────────────────────────────────────────────────────────────────┐
│                             NO CARRO SOLAR                             │
│                                                                        │
│   [Victron MPPT]                                                       │
│         │ (Emite texto VE.Direct a 19200 baud)                         │
│         ▼                                                              │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │ 1. RASPBERRY PI PICO (MicroPython)                           │      │
│  │    • Lê e descodifica as métricas da porta de série          │      │
│  │    • Grava no datalogger interno (dados_corrida_testing.txt) │      │
│  │    • Resume e envia uma linha CSV limpa pela UART1           │      │
│  └──────────────────────────────┬───────────────────────────────┘      │
│                                 │ (Série a 38400 baud)                 │
│                                 ▼                                      │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │ 2. SEEED STUDIO XIAO #1 (Firmware Meshtastic)                │      │
│  │    • Funciona em modo transparente (Serial TEXTMSG)          │      │
│  │    • Pega na linha de texto recebida e difunde-a por rádio   │      │
│  └──────────────────────────────┬───────────────────────────────┘      │
└─────────────────────────────────┼──────────────────────────────────────┘
                                  │
                                  │ Ondas de Rádio LoRa (868 MHz / Canal VROOOM)
                                  ▼
┌────────────────────────────────────────────────────────────────────────┐
│                              NAS BOXES                                 │
│                                                                        │
│  ┌──────────────────────────────────────────────────────────────┐      │
│  │ 3. A ESTAÇÃO BASE                                            │      │
│  │    a) Seeed Studio XIAO #2 (Rádio Recetor Meshtastic)        │      │
│  │       • Escuta os pacotes no ar e envia-os por USB ao PC     │      │
│  │                                                              │      │
│  │    b) Computador da Equipa                                   │      │
│  │       • Script Python: extrai o texto e cria telemetria.csv  │      │
│  │       • Servidor Docker (Grafana): desenha os gráficos live  │      │
│  └──────────────────────────────────────────────────────────────┘      │
└────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Como os Componentes se Relacionam

### Componente 1: Raspberry Pi Pico (O "Cérebro" de Aquisição)
* **Linguagem:** MicroPython (`main.py`).
* **Missão:** Conversar diretamente com o Victron MPPT.
* O Pico não precisa de saber o que é rádio LoRa nem antenas. Ele apenas "escuta" a ficha VE.Direct a 19200 baud, extrai valores como a tensão da bateria e a potência dos painéis, e guarda uma cópia na sua memória flash para segurança caso o rádio falhe. Em seguida, injeta uma linha de texto simples (ex: `41.4,2.3,120`) na sua segunda porta série (UART1) a 38400 baud.

### Componente 2: Seeed Studio XIAO no Carro (O "Emissor" LoRa)
* **Firmware:** Meshtastic oficial (modo *Serial TEXTMSG*).
* **Missão:** Ser uma ponte (*bridge*) transparente entre o Pico e o ar.
* O XIAO não altera os dados nem faz cálculos elétricos. Sempre que deteta que o Pico enviou uma linha de texto pelos pinos de série, empacota essa mensagem e dispara-a pelas ondas de rádio a 868 MHz com o protocolo de rede em malha (*mesh*).

### Componente 3: A Estação Base nas Boxes
* **Tecnologia:** Python com `uv` + Grafana em Docker.
* **Missão:** Receber, armazenar e exibir visualmente as variáveis de corrida.

---

## 3. O Segundo Dispositivo Meshtastic (Como Funciona o Recetor)

Uma das dúvidas mais comuns é: **o que é o recetor na box e como deve ser programado?**

{% hint style="info" %}
**O segundo módulo de rádio é idêntico ao do carro:**  
Não precisas de comprar hardware diferente. O dispositivo da box é outro **Seeed Studio XIAO com shield Wio-SX1262** (ou qualquer nó comercial compatível com Meshtastic a 868 MHz).
{% endhint %}

### O que o Segundo XIAO faz nas Boxes:
1. **Não precisa de Raspberry Pi Pico:** O rádio da box é ligado **diretamente por cabo USB-C ao computador portátil** da equipa técnica.
2. **Receção Contínua:** Ele escuta o ar na mesma frequência (868 MHz). Quando um pacote do carro chega, ele descodifica o texto e entrega-o imediatamente pela porta USB (como se fosse uma porta COM de série).
3. **Alimentação do Grafana:** O computador portátil corre um pequeno script (`read_mesh_2_csv.py`) que apanha essas mensagens de texto na porta USB e vai adicionando linhas ao ficheiro `telemetria.csv`, que o Grafana lê a cada 5 segundos para atualizar os gráficos.

### Como Parametrizar o Segundo XIAO:
A programação deste rádio é praticamente igual à do carro, com apenas **uma diferença fundamental no módulo série**:

| Parâmetro | XIAO do Carro (Emissor) | XIAO da Box (Recetor) |
| :--- | :--- | :--- |
| **Firmware** | Meshtastic Stable | Meshtastic Stable *(exatamente o mesmo)* |
| **Região LoRa** | `European Union 868MHz` | `European Union 868MHz` |
| **Preset de Rádio** | `SHORT_FAST` | `SHORT_FAST` *(tem de ser idêntico)* |
| **Canal Primário** | `VROOOM` (PSK: `RA==`) | `VROOOM` (PSK: `RA==`) *(tem de ser idêntico)* |
| **Módulo Serial** | **Ativado (TEXTMSG)** *(para ler pinos D6/D7 do Pico)* | **Desativado (Default)** *(comunica por USB nativo com o PC)* |

{% hint style="tip" %}
**Regra de Ouro:**  
Para o rádio da box comunicar com o do carro, basta que **a Região, o Preset e o Nome do Canal (com a chave PSK) sejam rigorosamente iguais** em ambos os aparelhos.
{% endhint %}

---

## 4. Ordem Recomendada de Programação

Para colocares o ecossistema a funcionar sem confusões, segue os manuais das próximas páginas por esta ordem:

1. **[Raspberry Pi Pico (MicroPython)](raspberry-pi-pico.md):** Gravação do script de leitura e teste do ficheiro de log local.
2. **[Rádio LoRa (Meshtastic)](meshtastic-lora.md):** Gravação do firmware e configuração do módulo emissor do carro solar.
3. **[Estação Base & Grafana](estacao-base.md):** Ligação do segundo rádio ao computador por USB, arranque do script Python e visualização dos gráficos.
