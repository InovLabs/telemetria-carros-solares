---
description: Como criar canais privados de corrida, testar diferentes presets de rádio LoRa e adicionar novos sensores ao Raspberry Pi Pico.
---

# Personalização, Canais & Novos Sensores

Este sistema de telemetria foi concebido como um **ponto de partida modular**. A arquitetura aberta permite que cada escola ou equipa técnica experimente diferentes configurações de rádio ou expanda o hardware com novos sensores adaptados à sua estratégia de corrida.

---

## 1. Estratégia de Rede: Canal Partilhado vs. Canal Privado

Durante uma prova com dezenas de veículos solares em pista, a equipa pode optar por duas abordagens de comunicação:

### Opção A: A Abordagem Colaborativa (Canal `VROOOM`)
Se todas as escolas mantiverem o canal padrão `VROOOM` com a chave `RA==`:
* **Vantagem (Rede Mesh Máxima):** Todos os carros solares em pista e rádios nas boxes funcionam como repetidores automáticos (*nodes*). Se o teu carro estiver numa curva distante sem linha de vista para a tua box, o sinal pode saltar através do carro de outra escola no meio da pista e chegar à meta sem falhas.
* **Desvantagem:** Os pacotes de telemetria são públicos; qualquer equipa com um recetor pode ver a vossa tensão de bateria e potência.

### Opção B: O Canal Privado da Escola
Se a equipa quiser total confidencialidade tática:
1. No gestor do Meshtastic (**client.meshtastic.org** ou App móvel), vai a **Channels ➜ Channel 0**.
2. Altera o nome do canal para algo próprio da equipa (ex: `SOLAR_TEAM_01`).
3. Clica em **Generate** para criar uma nova chave criptográfica segura (**PSK de 256 bits**).
4. **Obrigatório:** Aplica exatamente o mesmo nome de canal e a mesma chave PSK no XIAO transmissor (do carro) e no XIAO recetor (da box).
* **Vantagem:** Apenas a vossa bancada consegue decifrar os dados transmitidos.

---

## 2. Experimentação com Presets LoRa

No menu **Radio Config ➜ LoRa**, o parâmetro **Preset** define a forma como as ondas de rádio são moduladas. Cada preset representa um compromisso diferente entre **alcance**, **velocidade** e **tempo no ar (*airtime*)**:

| Preset LoRa | Velocidade / Latência | Alcance | Recomendação para Pista |
| :--- | :---: | :---: | :--- |
| **`SHORT_FAST`** *(Padrão)* | Muito Rápida (~1s) | Médio (~1 a 2 km) | **Ideal para circuitos fechados.** Ocupa o canal durante milissegundos, permitindo envios frequentes de dados. |
| **`MEDIUM_FAST`** | Rápida (~2s) | Elevado (~3 a 5 km) | Excelente alternativa se a pista tiver muitas árvores, bancadas de betão ou zonas em desnível. |
| **`LONG_FAST`** | Lenta (~5s a 10s) | Máximo (+10 km) | Indicado para provas de estrada aberta com grandes distâncias entre a base e o carro solar. |

{% hint style="info" %}
**Regulamentação Europeia (Duty Cycle de 1%):**  
Na frequência de 868 MHz na Europa, cada dispositivo só pode ocupar o ar durante 1% do tempo por hora. O preset `SHORT_FAST` é o mais seguro para telemetria frequente porque os seus pacotes demoram muito pouco tempo a ser emitidos.
{% endhint %}

---

## 3. O Pico como Plataforma Aberta: Novos Sensores

O Raspberry Pi Pico tem **26 pinos GPIO multifunções**, conversores analógico-digitais (ADC) e barramentos **I2C, SPI e UART livres**. Podes expandir o circuito ligando sensores adicionais diretamente à breadboard:

```
                          ┌─────────────────────┐
                          │  Raspberry Pi Pico  │
                          └──────────┬──────────┘
                                     │
       ┌─────────────────────────────┼─────────────────────────────┐
       ▼                             ▼                             ▼
[Porta VE.Direct]             [Barramento I2C]              [Barramento UART]
  Victron MPPT                  Novos Sensores                Sensor GPS
  (Tensão, Corrente,             (Temperatura,                 (Velocidade real,
   Potência do Painel)            Corrente do Motor)            Tempo por volta)
```

### Exemplos de Sensores Fáceis de Integrar:

1. **Módulo GPS (ex: Neo-6M / Neo-8M via UART2):**
   * Permite calcular velocidade real de GPS em km/h, distância percorrida e cronometragem automática por setor de pista.
2. **Sensor de Corrente e Tensão I2C (ex: INA219 / INA226):**
   * Enquanto o Victron mede o que entra do painel solar para a bateria, o INA219 pode ser colocado na linha do motor elétrico para medir o consumo exato da tração.
3. **Sonda de Temperatura Digital (ex: DS18B20 1-Wire):**
   * Monitorização térmica da bateria de lítio ou do enrolamento do motor elétrico para prevenir sobreaquecimento.
4. **Sensor de Efeito Hall (Tacómetro de Roda):**
   * Um íman fixado na jante do carro e um sensor Hall num pino de interrupção do Pico permite ler a rotação real da roda com precisão milimétrica.

---

## 4. Como Adicionar as Novas Variáveis ao Código

No ficheiro `main.py` do Raspberry Pi Pico, a linha que envia os dados para o XIAO é uma simples formatação de texto em MicroPython.

Se adicionares, por exemplo, a temperatura do motor (`temp_motor`) e a velocidade do GPS (`vel_gps`), basta adicioná-las à string de envio:

```python
# Exemplo de formatação no main.py com novos dados:
mensagem = f"{volts_bat},{corrente_bat},{potencia_solar},{temp_motor},{vel_gps}\n"

# Envio pela porta série para o XIAO LoRa:
uart_xiao.write(mensagem)
```

Na estação base, o script em Python (`read_mesh_2_csv.py`) gravará essas novas colunas diretamente no ficheiro CSV, ficando imediatamente disponíveis para criar novos mostradores e gráficos no **Grafana**.
