# Diagnóstico & FAQ

### O XIAO não envia dados e não vejo tráfego no ar.
* Confirma que não baralhaste os pinos GPIO do modo de série no Meshtastic.  
  * Para **ESP32-S3**: RX é o pino **44** e TX é o pino **43**.
  * Para **nRF52840**: RX é o pino **7** e TX é o pino **6**.
* Garante que o cabo TX do Pico liga ao RX do XIAO (as linhas cruzam-se).

### O Pico não deteta métricas do Victron.
* Retiraste a ponte plástica preta (jumper) que vem inserida na porta do Victron de fábrica?
* Lembra-te que o fio de dados TX do Victron tem de ir ao pino **GP1 / Pin 2** do Pico.

### Outra escola pode receber ou alterar os nossos dados?
* Se mantiveres o nome de canal `VROOOM` e a chave por defeito `RA==`, todas as equipas conseguem escutar e retransmitir pacotes.
* Se quiseres um canal 100% privado:
  1. Cria um canal com outro nome (ex: `EQUIPA_01`).
  2. Gera uma chave criptográfica nova (PSK de 256 bits) na aplicação do Meshtastic.
  3. Aplica essa mesma chave no XIAO do carro e no XIAO da box.
