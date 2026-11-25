---
description: Resolução rápida dos erros mais comuns de hardware, perdas de ligação no Thonny e dúvidas frequentes em pista.
---

# Diagnóstico & FAQ

Esta página reúne as soluções práticas para os problemas mais frequentes encontrados durante a montagem, programação e testes de pista.

---

## 1. O Raspberry Pi Pico perde a ligação no Thonny ou dá "Device is busy"

Durante o desenvolvimento em MicroPython, é muito comum o Thonny perder temporariamente a comunicação com o Pico, especialmente quando está a correr um ciclo contínuo de leitura.

### Como verificar visualmente se o Pico está ligado:

Olha para a barra cinzenta no **canto inferior direito da janela do Thonny**:

```text
✅ CONECTADO COM SUCESSO:
┌────────────────────────────────────────────────────────┐
│ MicroPython (Raspberry Pi Pico) • COM3 (ou /dev/tty) ≡ │
└────────────────────────────────────────────────────────┘
A consola inferior mostra a versão e os três símbolos: >>>

❌ DESCONECTADO OU EM ERRO:
┌────────────────────────────────────────────────────────┐
│ Local Python 3 • Thonny's Python                      │ (Mudou sozinho para o Python do PC)
└────────────────────────────────────────────────────────┘
ou mensagem a vermelho: [Device is busy / Could not enter raw REPL]
```

---

### Procedimento para Restaurar a Ligação (Passo a Passo):

Segue esta ordem de resolução, do mais simples ao mais avançado:

#### Passo 1: O Botão Vermelho STOP (Interrupção por software)
No topo do Thonny, clica no botão vermelho redondo **STOP** (ou prime a tecla **`Ctrl + F2`**):
* Isto envia um sinal de paragem (*KeyboardInterrupt / Ctrl+C*) ao Pico, interrompe qualquer ciclo em execução e devolve a linha de comandos `>>>` na consola.

#### Passo 2: Reconfirmar a Porta COM
1. Clica no texto do canto inferior direito do Thonny.
2. Volta a clicar em **MicroPython (Raspberry Pi Pico)**.
3. Se aparecerem várias portas COM, escolhe a porta onde o Pico está ligado.

#### Passo 3: Reiniciar a Alimentação USB
1. Desliga o cabo USB do Pico do computador.
2. Aguarda 3 segundos.
3. Volta a ligar o cabo USB e clica novamente no botão vermelho **STOP** no Thonny.

#### Passo 4: O "Truque" do `main.py` Bloqueado
Se o ficheiro `main.py` contiver um erro grave de código ou enviar texto para a porta USB demasiado depressa, o Pico pode encravar logo no milissegundo em que arranca, impedindo o Thonny de assumir o controlo:
1. Desliga o cabo USB do Pico.
2. No Thonny, **mantém premido com o rato o botão vermelho STOP** (não largues o clique!).
3. Enquanto manténs o botão premido, **liga o cabo USB ao computador**.
4. Larga o botão STOP apenas após o cabo estar inserido: o Thonny consegue "furar" o arranque do `main.py` e retomar o controlo da placa.

{% hint style="tip" %}
**Solução Extrema (Flash Nuke):**  
Se o Pico ficar totalmente inacessível, descarrega o ficheiro oficial [flash_nuke.uf2](https://datasheets.raspberrypi.com/soft/flash_nuke.uf2) da Raspberry Pi. Entra no modo BOOTSEL, arrasta o ficheiro para o Pico para apagar toda a memória flash limpa e volta a instalar o MicroPython.
{% endhint %}

---

## 2. Porque é que o ficheiro de log local (CSV) tem muito mais dados do que o rádio transmite?

Se comparares o ficheiro de texto guardado no Pico (`dados_corrida_testing.txt`) com as mensagens que passam pelo rádio LoRa, vais notar que **o ficheiro local contém muito mais informação e variáveis**.

Esta diferença é intencional e segue uma boa prática de engenharia de telemetria:

### O que o Victron debita (Tudo):
A porta VE.Direct do Victron emite mais de 15 campos de diagnóstico a cada segundo: número de série, versão de firmware, histórico de produção diária (`H20`, `H21`), estados de erro (`ERR`), etc. O Raspberry Pi Pico tem espaço de sobra na sua memória flash para guardar todos estes registos exaustivos sem qualquer limitação.

### O que enviamos por LoRa (Apenas o Essencial Tático):
Pelo ar, o Pico envia apenas uma pequena linha com as 4 ou 5 variáveis críticas de corrida (Tensão da bateria, Corrente, Potência solar e Estado do carregador).

```text
Log Local (Flash do Pico): ──► Diagnóstico completo (15+ colunas para análise pós-corrida no Excel)
Emissão LoRa (Pelo Ar):    ──► Apenas dados vitais instantâneos (ex: 41.4,2.3,120,3)
```

### Porquê esta redução no rádio?
1. **Tempo de Emissão (*Airtime*):** O rádio LoRa tem uma largura de banda reduzida. Transmitir uma mensagem comprida ocupa o canal de rádio durante demasiado tempo, provocando atrasos de transmissão.
2. **Regulamento Europeu de 868 MHz (Duty Cycle de 1%):** Por lei, cada transmissor só pode emitir durante 36 segundos no total por hora. Mensagens muito curtas consomem apenas milissegundos de ar, cumprindo a lei.
3. **Evitar Colisões na Rede Mesh:** Mensagens compactas reduzem o risco de interferência entre carros solares que estejam a competir na mesma pista.

---

## 3. O rádio XIAO não transmite e não há sinal LoRa no ar

* **Causa mais frequente (Pinos GPIO invertidos):**  
  Reconfirma os números inseridos no menu *Module Config ➜ Serial* do Meshtastic:
  * No **XIAO ESP32-S3**: RX deve ser o **`44`** e TX deve ser o **`43`**.
  * No **XIAO nRF52840**: RX deve ser o **`7`** e TX deve ser o **`6`**.
* **Fios cruzados:** Confirma se a linha TX do Pico (**Pin 11**) liga à linha RX do XIAO (**Pin D7**).
* **Baud Rate incorreta:** Confirma se tanto o código do Pico como o Meshtastic estão configurados a **`38400`** baud.
* **Falta de Antena:** Nunca operes o rádio sem antena de 868 MHz conectada.

---

## 4. Liguei o telemóvel por Bluetooth ao rádio do carro mas não vejo mensagens

* **Comportamento normal do Meshtastic:** Quando o Pico envia texto em modo série (`TEXTMSG`), o rádio transmite o pacote diretamente pelas ondas LoRa. **A aplicação móvel ligada por Bluetooth ao próprio emissor não mostra essas mensagens no chat.**
* **Como confirmar a emissão:**
  * Abre o **Monitor de Série** no computador (`flasher.meshtastic.org`) para ver o registo de envio em direto.
  * Em alternativa, liga o telemóvel por Bluetooth ao **segundo dispositivo Meshtastic (o recetor da box)**: como o rádio da box recebe os pacotes pelo ar, as mensagens surgirão imediatamente no chat do canal `VROOOM`.

---

## 5. O Pico não deteta quaisquer leituras do Victron MPPT

* **Jumper de fábrica:** Retiraste a ponte plástica preta que vem de fábrica a fechar a porta VE.Direct do Victron?
* **Encaixe da ficha:** A ficha JST/Grove foi desbastada com um x-ato nas guias laterais para conseguir entrar até ao fundo da tomada do Victron?
* **Pinos:** O pino 3 do Victron (TX) tem de estar ligado ao **Pin 2 (GP1 / UART0 RX)** do Pico, e o pino 1 do Victron tem de ir à massa **Pin 38 (GND)**.

---

## 6. Posso ligar o cabo USB ao Pico com o carro solar ligado?

* **Sim, com toda a segurança**, desde que tenhas instalado o **díodo de proteção Schottky** entre a linha de 5V da breadboard e o **Pin 39 (VSYS)** do Pico.
* A risca prateada/branca do díodo (cátodo) tem de estar virada para o Pico. O díodo impede que a corrente vinda da porta USB do computador entre em conflito elétrico com o regulador LM2596 do carro.

---

## 7. Como tornar a telemetria da nossa escola 100% privada?

* Por omissão, todas as escolas utilizam o canal aberto `VROOOM` com a chave `RA==`. Isto permite que os carros de todas as equipas retransmitam pacotes uns dos outros através da rede em malha (*mesh*), aumentando a cobertura da pista.
* Se a tua equipa pretender guardar segredo tático das tensões e consumos:
  1. No gestor do Meshtastic, altera o nome do canal (ex: `SOLAR_ESCOLA_X`).
  2. Clica em **Generate** para criar uma nova chave segura (**PSK de 256 bits**).
  3. Aplica obrigatoriamente essa mesma chave no XIAO transmissor do carro e no XIAO recetor das boxes.
