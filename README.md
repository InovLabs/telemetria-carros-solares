---
description: Documentação técnica oficial do sistema de telemetria sem fios para carros solares.
---

# Visão Geral do Sistema

O sistema de telemetria tem como missão recolher dados elétricos em tempo real do controlador de carga do carro solar (**Victron MPPT**) e transmiti-los via rádio (**LoRa / Meshtastic**) para a bancada técnica nas boxes.

{% hint style="info" %}
**Porquê este sistema?**  
Permite que o piloto e os engenheiros na box acompanhem a potência dos painéis solares, a tensão da bateria e os consumos do motor sem recurso a redes Wi-Fi ou rede de telemóvel (3G/4G/5G).
{% endhint %}

### Arquitetura da Solução
