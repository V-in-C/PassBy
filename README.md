# Sistema de Contagem de Animais com Catraca e Monitoramento Remoto

> Relatório técnico do projeto cliente-servidor baseado em ESP32 (servidor) e aplicação Web (cliente).
> Instituição: PUC-PR
> Curso/Disciplina: Conectividade de Sistemas Ciberfísicos (Turma 2o A)
> Autores: Grupo 3
            Victor de Souza Maia
            Luma Felipe Area Lima
            Pedro Henrique de Andrade de Moraes
            Pedro Iago Moraes
            Rayka Souza
> Orientador: Professor Fábio Bettio
> Local e data: Curitiba, 9 de setembro de 2026

---

## Sumário

1. [Introdução](#1-introdução)
2. [Fundamentação Teórica](#2-fundamentação-teórica)
3. [Arquitetura do Sistema](#3-arquitetura-do-sistema)
4. [Desenvolvimento](#4-desenvolvimento)
5. [Testes e Validação](#5-testes-e-validação)
6. [Resultados](#6-resultados)
7. [Conclusão](#7-conclusão)
8. [Referências](#8-referências)

---

## 1. Introdução

O controle de acesso e a contagem de animais em instalações rurais — currais, mangueiros, bretes de manejo, portões de pastagens rotacionadas e centros de abate — ainda é realizado, em grande parte das propriedades, de forma manual ou por meio de contadores mecânicos simples. Esse método é suscetível a erros de contagem, não registra o sentido de deslocamento do animal (entrada ou saída do lote) e não oferece nenhuma forma de acompanhamento remoto ou de intervenção do produtor quando ele não está fisicamente no local.

Este trabalho propõe o desenvolvimento de um **sistema de contagem de animais com catraca e monitoramento remoto**, estruturado em uma arquitetura cliente-servidor, chamado Pass-By:

- **Servidor embarcado (ESP32):** responsável por ler um sensor de passagem instalado na catraca, identificar a direção do deslocamento do animal (entrada/saída), manter a contagem total e por sentido, acionar um atuador que permite ao usuário liberar, travar ou operar remotamente o mecanismo da catraca, e expor esses dados e comandos por meio de uma interface de rede (Wi-Fi/HTTP, WebSocket e/ou API REST).
- **Cliente Web:** aplicação acessada por navegador (computador, tablet ou smartphone) que consome os dados do servidor para exibir, em tempo real, a contagem de animais, o sentido do último evento registrado e o estado do atuador, além de permitir o envio de comandos de controle ao ESP32.

O sistema se enquadra no contexto de agricultura de precisão e automação rural (*smart farming*), em que sensores e microcontroladores de baixo custo, conectados à internet ou a uma rede local, permitem digitalizar processos até então manuais, reduzir erros humanos e possibilitar a tomada de decisão remota.

### 1.1 Objetivo geral

Projetar, implementar e validar um sistema embarcado de contagem bidirecional de animais integrado a uma catraca física, com monitoramento e controle remoto por meio de uma aplicação Web cliente.

### 1.2 Objetivos específicos

- Selecionar e caracterizar um sensor capaz de detectar a passagem e o sentido de deslocamento do animal na catraca;
- Implementar no ESP32 a lógica de contagem bidirecional (incremento/decremento conforme o sentido detectado);
- Implementar um atuador (relé, solenoide ou servomotor) para liberação/travamento da catraca, comandado remotamente pela rede;
- Disponibilizar os dados de contagem e o controle do atuador por meio de um servidor embarcado no ESP32, acessível por uma aplicação Web cliente;
- Testar e validar o sistema quanto à precisão da contagem, à correta identificação do sentido de passagem e à confiabilidade da comunicação cliente-servidor;
- Documentar o processo de desenvolvimento seguindo a estrutura e as normas ABNT aplicáveis a relatórios técnicos.

### 1.3 Estrutura do documento

Além desta introdução, o relatório está organizado da seguinte forma: a Seção 2 apresenta a fundamentação teórica, incluindo conceitos de sistemas embarcados, sensoriamento de passagem/direção, comunicação cliente-servidor e trabalhos correlatos usados como referência; a Seção 3 descreve a arquitetura do sistema proposto; a Seção 4 detalha o desenvolvimento do hardware e do firmware/software; a Seção 5 apresenta os testes realizados e as evidências de validação; a Seção 6 discute os resultados obtidos; a Seção 7 traz a conclusão e sugestões de trabalhos futuros; e a Seção 8 lista as referências bibliográficas, conforme a ABNT.

---

## 2. Fundamentação Teórica

### 2.1 Sistemas embarcados e o microcontrolador ESP32

O ESP32 é um microcontrolador de baixo custo, com dois núcleos de processamento, conectividade Wi-Fi e Bluetooth integradas, múltiplas interfaces de entrada/saída (GPIO digital, ADC, PWM, I2C, SPI, UART) e suporte nativo a pilhas TCP/IP, o que o torna adequado para atuar simultaneamente como controlador de sensores/atuadores e como servidor de rede — os dois papéis exigidos pelo servidor do sistema proposto neste trabalho.

### 2.2 Sensoriamento de passagem e detecção de direção

A contagem bidirecional (diferenciar entrada de saída) não é possível com um único sensor de presença, pois este apenas informa a ocorrência de uma interrupção do feixe/campo de detecção, sem indicar o sentido do movimento. A solução amplamente adotada na literatura técnica consiste em posicionar **dois sensores** (infravermelho, ultrassônico ou indutivo, conforme a aplicação) ligeiramente espaçados ao longo do trajeto de passagem. A ordem em que cada sensor é acionado — sensor 1 antes do sensor 2, ou o inverso — define o sentido do deslocamento, geralmente combinada a uma janela de tempo (*timeout*) entre os acionamentos para evitar contagens duplicadas ou falsas.

### 2.3 Comunicação cliente-servidor e protocolos Web embarcados

No modelo cliente-servidor adotado, o ESP32 hospeda um servidor HTTP (bibliotecas como `WebServer.h`, `ESPAsyncWebServer` ou, em MicroPython, módulos equivalentes) que expõe páginas HTML/CSS/JavaScript e/ou endpoints de API (REST, WebSocket ou protocolos de mensageria como MQTT) para que um cliente Web, executado em qualquer navegador da rede local ou da internet, leia o estado do sistema (contagem, sentido, status do atuador) e envie comandos (por exemplo, liberar/travar a catraca).

### 2.4 Atuadores para controle remoto

O acionamento remoto de mecanismos físicos a partir do ESP32 é tipicamente realizado por meio de módulos relé (para cargas de maior potência, como solenoides e travas elétricas) ou por servomotores/motores de passo (para movimentação mecânica direta da catraca), controlados por sinais digitais ou PWM emitidos pelos GPIOs do microcontrolador a partir de comandos recebidos pela rede.

### 2.5 Trabalhos correlatos

Foram analisados cinco projetos correlatos, todos com documentação de desenvolvimento (diagramas elétricos/eletrônicos, código-fonte e bibliotecas utilizadas), usados como referência para as decisões de arquitetura, sensoriamento e controle deste trabalho.

**a) Bidirectional Counter using IR Sensors (YOGESHWARAN, 2022).** Projeto baseado em Arduino que utiliza dois sensores infravermelhos posicionados lado a lado para detectar a ordem de interrupção dos feixes e, a partir dela, determinar o sentido de passagem (entrada ou saída). O repositório disponibiliza o diagrama elétrico completo (ligação dos módulos IR aos pinos digitais do microcontrolador) e o código-fonte em C/C++ para Arduino IDE, implementando uma máquina de estados por temporização entre os dois sensores para evitar contagem duplicada. Referência direta para a lógica de detecção de direção da catraca deste projeto.

**b) Bidirectional Visitor Counter with Automatic Light Control (CIRCUITDIGEST/HOW2ELECTRONICS, [s.d.]).** Sistema com Arduino que combina um par de sensores infravermelhos (conectados às entradas analógicas A0 e A5), um display LCD 16x2 em modo 4 bits (biblioteca `LiquidCrystal`) para exibição da contagem, e um relé (acionado pelo pino digital 2, por meio de um transistor de acionamento) que liga/desliga automaticamente a iluminação do ambiente conforme a ocupação detectada. O material apresenta esquemático completo e código comentado. Serve de referência para a integração entre lógica de contagem e acionamento de um atuador (relé), análoga ao controle da catraca, ainda que localmente (sem rede).

**c) IoT Bidirectional Visitor Counter using ESP8266 & MQTT (HOW2ELECTRONICS, [s.d.]).** Evolução do contador bidirecional para um cenário de Internet das Coisas: substitui o Arduino por um ESP8266 (SoC Wi-Fi da mesma família de aplicações do ESP32) e publica os valores de entrada, saída e ocupação atual em um broker MQTT, permitindo o acompanhamento remoto por um painel (dashboard) acessível de qualquer lugar. O projeto documenta o esquemático de ligação dos sensores IR ao ESP8266 e o firmware em C/C++ (bibliotecas `ESP8266WiFi` e `PubSubClient`). Referência principal para o requisito de monitoramento remoto do sistema proposto.

**d) ESP32/ESP8266 Relay Module Web Server (SANTOS; SANTOS, [s.d.]).** Projeto que implementa, no próprio ESP32, um servidor Web (biblioteca `WebServer.h`, com página HTML/CSS/JavaScript embutida no firmware) que expõe botões para ligar/desligar um módulo relé conectado a um GPIO (por exemplo, GPIO 26), permitindo o controle remoto de uma carga a partir de qualquer navegador na rede local. O material inclui o diagrama de ligação do relé ao ESP32 e o código-fonte completo (há também uma variante em MicroPython). Referência direta para a implementação do atuador de controle remoto da catraca deste trabalho, incluindo o padrão de servidor HTTP embarcado a ser adotado.

**e) ESP32 – HTTP Web Server – HTML – CSS – Simple Counter, como base de um "Visitors and Parking Lot Occupancy Counter" (INSTRUCTABLES, [s.d.]).** Projeto nativo em ESP32 (sem uso de placas auxiliares) que implementa um servidor HTTP com página HTML/CSS servida diretamente pelo microcontrolador, exibindo e atualizando em tempo real um contador utilizado como exemplo de aplicação para ocupação de estacionamento/ambiente. Apresenta o código-fonte completo em Arduino IDE (bibliotecas de rede nativas do ESP32) e a estrutura da página Web servida. Referência para a arquitetura cliente-servidor Web (ESP32 como servidor HTTP nativo) adotada neste projeto, sem dependência de serviços externos de terceiros.

O Quadro 1 resume a contribuição de cada trabalho correlato para o presente projeto.

| Projeto | Plataforma | Sensoriamento | Direção | Atuador/Controle remoto | Contribuição para este trabalho |
|---|---|---|---|---|---|
| a) YOGESHWARAN (2022) | Arduino | 2x IR | Sim | Não | Lógica de detecção de direção |
| b) CircuitDigest/How2Electronics | Arduino | 2x IR + LCD | Sim | Relé (local) | Integração contagem + atuador |
| c) How2Electronics (MQTT) | ESP8266 | 2x IR | Sim | Não (somente leitura remota) | Monitoramento remoto via rede |
| d) Santos; Santos (RNT) | ESP32/ESP8266 | — | — | Relé via Web | Servidor Web + controle remoto do atuador |
| e) Instructables | ESP32 | Contador simples | Não | Não | Servidor HTTP nativo no ESP32 (cliente-servidor Web) |

*Quadro 1 – Síntese dos trabalhos correlatos analisados. Fonte: elaborado pelos autores (2026).*

---

## 3. Arquitetura do Sistema



## 4. Desenvolvimento



## 5. Testes e Validação



## 6. Resultados



## 7. Conclusão



## 8. Referências

YOGESHWARAN, P. **Bidirectional Counter using IR Sensors**. Hackster.io, 2022. Disponível em: https://www.hackster.io/its_me_yogesh/diy-bidirectional-counter-using-arduino-and-ir-sensors-36ba45. Acesso em: 06 set. 2026.

YOGESHWARAN, P. **Bidirectional-Counter-using-IR-Sensors**: código-fonte e diagrama elétrico. GitHub, 2022. Disponível em: https://github.com/YogeshwaranP-05/Bidirectional-Counter-using-IR-Sensors. Acesso em: 06 set. 2026.

CIRCUITDIGEST. **How to Build a Bidirectional Visitor Counter using Arduino and IR Sensors**. CircuitDigest, [s.d.]. Disponível em: https://circuitdigest.com/microcontroller-projects/how-to-build-a-bidirectional-counter-using-arduino-and-ir-sensors. Acesso em: 06 set. 2026.

HOW2ELECTRONICS. **Bidirectional Visitor Counter with Automatic Light Control using Arduino**. How2Electronics, [s.d.]. Disponível em: https://how2electronics.com/bidirectional-visitor-counter-with-automatic-light-control-using-arduino/. Acesso em: 06 set. 2026.

HOW2ELECTRONICS. **IoT Bidirectional Visitor Counter using ESP8266 & MQTT**. How2Electronics, [s.d.]. Disponível em: https://how2electronics.com/iot-bidirectional-visitor-counter-using-espp8266-mqtt/. Acesso em: 06 set. 2026.

SANTOS, Rui; SANTOS, Sara. **ESP32/ESP8266 Relay Module Web Server using Arduino IDE**. Random Nerd Tutorials, [s.d.]. Disponível em: https://randomnerdtutorials.com/esp32-esp8266-relay-web-server/. Acesso em: 06 set. 2026.

INSTRUCTABLES. **ESP32 – HTTP Web Server – HTML – CSS – Simple Counter As the Subject of "Visitors and Parking Lot Occupancy Counter"**. Instructables, [s.d.]. Disponível em: https://www.instructables.com/ESP32-HTTP-Web-Server-HTML-CSS-Simple-Counter-As-t/. Acesso em: 06 set. 2026.

