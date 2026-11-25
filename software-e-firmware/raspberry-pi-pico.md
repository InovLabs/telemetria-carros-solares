---

## 4. Checkpoint 4: Testar a Comunicação Pico ➜ XIAO (Monitor de Série)

Como podes ter a certeza absoluta de que o XIAO está a receber o que o Pico envia?

1. Mantém o Pico a correr ligado ao Victron e com os pinos interligados ao XIAO (fio amarelo no **Pin 11 / D7** e fio verde no **Pin 12 / D6**).
2. Liga o cabo USB-C do XIAO ao computador e acede a **[flasher.meshtastic.org](https://flasher.meshtastic.org/)** ou **[client.meshtastic.org](https://client.meshtastic.org/)**.
3. No canto inferior esquerdo do ecrã, clica no botão **"Abrir Monitor de Série"** (ou no separador *Debug Log*).
4. **O que deves ver no monitor:**
   * A cada segundo, deves ver surgir uma linha a dizer:
     `Incoming serial message: 41420,230,120...`
     seguida de uma mensagem do rádio a confirmar o envio do pacote:
     `Broadcasting TEXTMSG packet on channel VROOOM`.
5. **Se nada aparecer no monitor:**
   * Os fios estão invertidos? Lembra-te: o **TX do Pico (Pin 11)** tem de ir ao **RX do XIAO (Pin D7)**.
   * Configuraste o pino de RX correto no módulo Serial? (GPIO `44` no ESP32-S3 ou GPIO `7` no nRF52840).
   * A velocidade (*Baud Rate*) está definida em **`38400`**?---
description: Instalação do MicroPython no Raspberry Pi Pico, testes de diagnóstico no Thonny e validação de leitura do Victron.
---

# Raspberry Pi Pico (MicroPython)

O Raspberry Pi Pico é o cérebro da recolha de dados: escuta a porta VE.Direct do Victron, guarda uma cópia de segurança na memória flash interna e envia os dados formatados para o transmissor LoRa.

Em vez de gravares tudo "às cegas", segue este guia passo a passo com testes de validação intermédios.

---

## 1. Instalar o MicroPython no Pico

1. Com o Pico desligado do computador, mantém premido o botão branco **BOOTSEL**.
2. Enquanto manténs o botão premido, liga o cabo micro-USB ao computador (e larga o botão logo a seguir).
3. O computador vai detetar o Pico como uma vulgar pen USB com o nome **`RPI-RP2`**.
4. Descarrega o ficheiro UF2 estável em: [micropython.org/download/rp2-pico](https://micropython.org/download/rp2-pico/).
5. Arrasta o ficheiro `.uf2` descarregado para dentro da drive `RPI-RP2`.
6. A drive desaparece e o Pico reinicia de imediato já a correr MicroPython.

---

## 2. Checkpoint 1: "Smoke Test" no Thonny IDE

Antes de carregar qualquer código da corrida, vamos verificar se o Thonny e o Pico estão a comunicar perfeitamente:

1. Abre o programa **[Thonny IDE](https://thonny.org/)**.
2. No canto inferior direito, clica no seletor de interpretador e escolhe:
   * **`MicroPython (Raspberry Pi Pico)`** (certifica-te de que a porta COM correta é selecionada).

![Seleção do interpretador MicroPython no Thonny IDE](../gitbook/assets/thonny-interpretador.png)


3. Olha para a janela inferior do Thonny (**Shell / Consola**). Deves ver uma mensagem parecida com:
   ```text
   MicroPython v1.22.0 on Raspberry Pi Pico with RP2040
   Type "help()" for more information.
   >>> 
   ```
4. **Teste do LED da placa:** Na consola junto aos símbolos `>>>`, escreve o seguinte comando e carrega no Enter:
   ```python
   from machine import Pin; Pin("LED", Pin.OUT).toggle()
   ```
   * *O pequeno LED verde na placa do Pico deve acender!* Se acendeu, a placa está 100% operacional.

{% hint style="warning" %}
**Se o Thonny não detetar o Pico:**  
* Confirma se o cabo micro-USB transmite dados (muitos cabos servem apenas para carregamento de telemóveis).
* Clica no botão vermelho **STOP** no topo do Thonny para reiniciar a ligação com o chip.
{% endhint %}

---

## 3. Checkpoint 2: Testar a Leitura do Victron "Ao Vivo"

{% hint style="danger" %}
**Não graves já o ficheiro como `main.py`!**  
Testa primeiro o código em modo interativo para confirmares que o Victron está a enviar números reais.
{% endhint %}

1. Garante que os fios do Victron estão ligados ao Pico:
   * **GND do Victron** ➜ Pin 38 (GND) do Pico.
   * **TX do Victron** ➜ Pin 2 (GP1) do Pico.
2. Abre no Thonny o ficheiro de código da corrida:
   * [pico.py (Código Oficial)](https://github.com/n1ghtbyte/pico_meshtastic_testing/blob/main/rpi_pico_examples/send_uart/pico.py)
3. Clica no botão verde **Run current script (F5)** no topo do Thonny.
4. **Observa o painel inferior (Shell):**
   * ✅ **Cenário de Sucesso:** Vais ver linhas de texto a desfilarem a cada 1 segundo com números reais (tensão da bateria, corrente e potência solar, ex: `V=41420, I=230, PPV=120`).
   * ❌ **Cenário de Falha (Linhas em branco ou nada a acontecer):**
     * Retiraste a ponte plástica preta (jumper) que vinha de fábrica na porta do Victron?
     * O conector JST foi bem desbastado e encaixou até ao fundo?
     * Não baralhaste os pinos? O pino TX do Victron tem de ir obrigatoriamente ao **Pin 2 (GP1)** do Pico.

---

## 4. Checkpoint 3: Verificar o Registo Local (Datalogger)

1. Deixa o script correr durante cerca de 10 a 15 segundos no Thonny.
2. Clica no botão vermelho **STOP** no topo do Thonny.
3. No painel lateral esquerdo do Thonny (no explorador de ficheiros do Raspberry Pi Pico):
   * Confirma que foi criado um ficheiro chamado **`dados_corrida_testing.txt`**.
4. Clica duas vezes sobre ele para o abrires no Thonny.
5. Vais ver as leituras gravadas em formato tabular (separadas por vírgulas como um CSV). Isto garante que, mesmo que o rádio se avarie em prova, os dados do vosso carro ficam a salvo na memória do Pico!

---

## 5. Gravação Definitiva no Arranque (`main.py`)

Agora que comprovámos que o Pico lê o Victron e grava os logs com sucesso, vamos torná-lo autónomo para que arranque sozinho mal o carro solar receba energia:

1. Com o ficheiro `pico.py` aberto no Thonny, vai a **Ficheiro ➜ Guardar Como...**
2. Quando surgir a janela, escolhe **Raspberry Pi Pico**.
3. Escreve obrigatoriamente o nome:
   ```text
   main.py
   ```
4. Clica em **OK**.
5. Podes desligar o cabo USB do computador. Sempre que o carro solar for ligado, o Pico executa imediatamente o `main.py` e começa a transmitir dados para o rádio LoRa.
