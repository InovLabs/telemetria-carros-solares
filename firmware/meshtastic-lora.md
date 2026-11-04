---
description: Instalação do firmware Meshtastic no Seeed Studio XIAO e configuração via Web ou App móvel (Bluetooth).
---

# Rádio LoRa (Meshtastic)

O módulo Seeed Studio XIAO corre o software de rede em malha **Meshtastic**. Ele funciona como um emissor transparente: tudo o que o Raspberry Pi Pico injeta pela porta de série é encapsulado e transmitido por radiofrequência LoRa a **868 MHz** para a box.

---

## Passo 1: Gravar o Firmware Meshtastic

{% hint style="danger" %}
**Antena Obrigatória:**  
Garante que a antena de 868 MHz está devidamente conectada ao conector IPEX/SMA do módulo antes de ligar o cabo USB. Ligar o rádio sem antena danifica a placa.
{% endhint %}

1. Liga o Seeed Studio XIAO ao computador através de um cabo USB-C de dados.
2. No navegador Google Chrome ou Microsoft Edge, acede a **[flasher.meshtastic.org](https://flasher.meshtastic.org/)**.
3. Seleciona o teu modelo de placa:
   * **Seeed Studio XIAO ESP32-S3** ou **Seeed Studio XIAO nRF52840**.
4. Escolhe a versão de firmware estável mais recente (**Stable**) e clica em **Flash Device**.

![Interface do Meshtastic Web Flasher](../assets/meshtastic-web-flasher.png)

{% hint style="info" %}
**Se o computador não detetar a placa:**  
Carrega duas vezes rapidamente no botão minúsculo **RST (Reset)** do XIAO para forçar a entrada no modo *Bootloader*.
{% endhint %}

---

## Passo 2: Configuração dos Parâmetros

Podes configurar o módulo de duas formas: no computador (por cabo USB) ou no telemóvel (via Bluetooth).

{% tabs %}
{% tab title="Método A: No Computador (Web Serial)" %}

1. Mantém o XIAO ligado por USB e acede a **[client.meshtastic.org](https://client.meshtastic.org/)**.
2. Clica em **New Connection ➜ Serial** e seleciona a porta COM do XIAO.
3. Altera os parâmetros indicados na secção abaixo e clica em **Save** em cada menu.

{% endtab %}

{% tab title="Método B: No Telemóvel (Bluetooth / App)" %}

Este método é ideal para verificações ou ajustes rápidos na grelha de partida sem recurso a computador:

1. Instala a aplicação oficial **Meshtastic** no smartphone:
   * [Meshtastic para Android (Google Play)](https://play.google.com/store/apps/details?id=org.meshtastic.android)
   * [Meshtastic para iOS (App Store)](https://apps.apple.com/app/meshtastic/id1586432531)
2. Ativa o Bluetooth no telemóvel e abre a app.
3. Procura pelo novo dispositivo (ex: `Meshtastic_xxxx`) e clica para emparelhar.
4. Quando o telemóvel pedir o **código PIN de emparelhamento Bluetooth**, introduz:
   ```text
   123456
   ```
5. Uma vez conectado, acede ao separador das definições (roda dentada) para configurar o rádio e a porta série.

{% hint style="tip" %}
**Código Bluetooth Padrão:**  
Como as placas XIAO não têm um pequeno ecrã OLED para exibir códigos temporários, o Meshtastic utiliza o código universal pré-definido **`123456`**.
{% endhint %}

{% endtab %}
{% endtabs %}

---

## Definições Obrigatórias de Telemetria

Independentemente de usares o computador ou a app móvel, deves configurar estes três menus exatamente com os mesmos valores:

### 1. Parâmetros de Rádio (Radio Config ➜ LoRa)
* **Region:** `European Union 868MHz` *(obrigatório por lei em Portugal e na UE)*
* **Preset:** `SHORT_FAST` *(otimizado para mensagens frequentes de telemetria com menor latência)*
* **Hop Limit:** `3`
* **Ignore MQTT:** `ON`
* **OK to MQTT:** `ON`

### 2. Definição do Canal (Channels ➜ Canal 0 Primário)
* **Channel Name:** `VROOOM` *(todas as equipas e a estação base devem partilhar este nome)*
* **Pre-Shared Key (PSK):** `RA==` *(chave padrão partilhada de 8 bits)*
* **Uplink Enabled:** `ON`
* **Downlink Enabled:** `ON`

### 3. Módulo Série (Module Config ➜ Serial)
O módulo série é o responsável por receber o texto do Raspberry Pi Pico e enviá-lo pelo ar:

* **Serial Enabled:** `ON`
* **Echo Enabled:** `ON`
* **Serial Mode:** `TEXTMSG`
* **Baud Rate:** `BAUD_38400`
* **Timeout:** `2`

{% tabs %}
{% tab title="Se a placa for XIAO ESP32-S3" %}
* **RX Pin (GPIO):** `44` *(corresponde ao pino físico D7)*
* **TX Pin (GPIO):** `43` *(corresponde ao pino físico D6)*
{% endtab %}

{% tab title="Se a placa for XIAO nRF52840" %}
* **RX Pin (GPIO):** `7` *(corresponde ao pino físico D7)*
* **TX Pin (GPIO):** `6` *(corresponde ao pino físico D6)*
{% endtab %}
{% endtabs %}

{% hint style="warning" %}
**Gravar Alterações:**  
Sempre que alterares uma secção de definições, deves clicar em **Save / Enviar**. O módulo reiniciará automaticamente para carregar as novas configurações.
{% endhint %}
