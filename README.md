# PROJETO DE ENCERRAMENTO DE RESIDÊNCIA
### DISPOSITIVO IoT PARA AUTOMAÇÃO, CLIMATIZAÇÃO E MONITORAMENTO DE ESTUFAS

## 1. O Problema

Sou Produtor de Flores e Hortaliças de Alto Valor Agregado em uma cooperativa agrícola. Cultivar plantas sensíveis exige um controle rigoroso do microclima; pequenas variações na umidade do solo ou frentes frias repentinas podem destruir uma safra inteira em poucas horas. Além disso, sofremos constantemente com ataques de pequenos predadores e roedores que invadem a estufa à noite e destroem os brotos. Atualmente, o controle de irrigação e aquecimento é feito de forma manual e empírica, o que gera desperdício de água e energia.

Precisamos de uma solução embarcada para gerenciamento automatizado e inteligente da estufa. O dispositivo deve monitorar constantemente as variáveis ambientais (temperatura e umidade do ar), a umidade do próprio solo e a luminosidade. Para a proteção do cultivo, um sensor de movimento deve vigiar o perímetro para detectar a presença de invasores ou predadores. Todas as informações coletadas precisam ser enviadas para uma Dashboard de monitoramento remoto e também gravadas localmente em um cartão SD para segurança dos dados em caso de queda de internet.

O sistema controlará o ambiente e responderá a comandos através de três eixos de atuação:

* **Automação de Recursos:** O dispositivo deve possuir dois relés/MOSFET. O primeiro acionará uma bomba de água para irrigação e o segundo controlará uma lâmpada de aquecimento. Ambos os atuadores devem responder tanto a uma lógica local de automação (ex: ligar bomba se o solo estiver seco; ligar lâmpada se estiver muito frio) quanto a comandos manuais diretos vindos da Dashboard.
* **Acompanhamento do Crescimento (Time-Lapse):** Uma câmera integrada deve capturar automaticamente uma imagem da planta uma vez por dia e salvá-la no cartão SD, permitindo gerar um histórico visual completo do desenvolvimento biológico da cultura.
* **Interação em Tempo Real:** O operador, através da Dashboard na internet, deve ser capaz de enviar comandos assíncronos para acender/apagar a lâmpada, ligar/desligar a bomba de irrigação e solicitar a captura e o envio de uma imagem da planta em tempo real para verificar as condições das folhas sem precisar ir até a estufa.

## 2. Recomendações de Firmware e Conectividade

* **Conectividade e Sincronismo Temporal (NTP):** Como o dispositivo precisa registrar uma foto por dia e salvar dados temporizados no cartão SD, o firmware deve se conectar à internet e utilizar o protocolo NTP (*Network Time Protocol*) para sincronizar o relógio interno do ESP32-S3 com a hora real, garantindo que os registros de imagem e os arquivos de log tenham data e hora exatas.
* **Comunicação Bidirecional IoT:** Recomenda-se a utilização de um protocolo assíncrono (como MQTT) onde o dispositivo publica as leituras dos sensores (telemetria) e fica inscrito em tópicos de comando para reagir imediatamente às ordens de rega, aquecimento e captura de foto sob demanda.
* **Manipulação de Arquivos (SD Card):** O firmware deve inicializar o leitor de cartão SD. As fotos diárias da câmera devem ser salvas em uma pasta organizada (ex: `/imagens/foto_001.jpg`), fechando o arquivo corretamente após a gravação para evitar a corrupção do cartão.
* **Tratamento do Sensor de Movimento:** O sensor de presença/movimento deve ser configurado. Ao detectar um predador, o firmware deve acionar um buzzer e enviar um alerta para o servidor.
* **Controle de Potência dos Relés:** Ao acionar os relés, o firmware deve garantir travas de segurança (*timeouts*) para que a bomba de água não fique ligada indefinidamente por erro de software, o que poderia alagar a estufa.

## 3. Requisitos de Modelagem 3D (Gabinete)

O ambiente interno de uma estufa agrícola é altamente agressivo devido à umidade elevada, calor e respingos de água, exigindo cuidados especiais no design do case:

* **Proteção Contra Umidade e Respingos:** O gabinete principal deve ser projetado para proteger a placa de circuito (ESP32-S3), o módulo de relés e o leitor de cartão SD. O design deve evitar furos no topo que permitam a entrada de água de gotejamento ou irrigação (mimetizando uma proteção IP adequada).
* **Isolamento do Sensor de Temperatura:** Os sensores de temperatura e umidade do ar precisam de contato com o ambiente para medir o clima real, mas se ficarem dentro da caixa fechada sofrerão com o calor dissipado pelo ESP32-S3 e pelos relés. O modelo 3D deve prever um compartimento isolado externamente ou uma "gaiola de proteção" ventilada na base do case para abrigar esses sensores longe das fontes de calor interno.
* **Janelas de Interface Óptica e Cabeamento:**
    * Abertura selada ou visor transparente direcionado para a lente da câmera, garantindo que o plástico do case não cause sombras ou reflexos na foto da planta.
    * Abertura apontada para a frente do sensor de movimento.
    * Prensa-cabos ou dutos de saída na parte inferior do gabinete para a passagem dos cabos de alimentação, fios que vão para as sondas de solo externas e cabos de energia AC dos relés (bomba e lâmpada).

## 4. Entregável Inicial: Documento de Arquitetura e Viabilidade Técnico-Agrícola

Antes de iniciar o desenvolvimento prático do firmware ou a modelagem 3D do gabinete, a equipe deverá entregar um relatório técnico de arquitetura contendo obrigatoriamente:

* **4.1. Mapeamento de Hardware e Componentes:** A equipe deve mapear os componentes integrados e os periféricos externos acoplados à placa:
    * **Identificação:** Modelos exatos do sensor de umidade do solo (higrômetro capacitivo ou resistivo), sensor de temperatura/umidade do ar (ex: DHT22, AM2302), sensor de movimento (ex: PIR), leitor de cartão micro SD, módulo de relés e câmera.
    * **Pinout (Tabela de GPIOs):** Tabela detalhada mapeando todos os pinos de conexão com o ESP32-S3, destacando os pinos do barramento SPI (MISO, MOSI, SCK, CS) usados pelo cartão SD, pinos digitais dos relés e pinos analógicos (caso o sensor de solo use ADC).
    * **Consumo e Isolamento Elétrico:** Detalhar como os relés serão acionados. Como cargas indutivas (bombas de água) geram ruídos eletromagnéticos ao ligar/desligar, a equipe deve detalhar o uso de optoacopladores para isolar a parte de potência dos relés da alimentação sensível do microcontrolador.

* **4.2. Diagrama de Blocos do Sistema:** Apresentar um esquema gráfico modular do hardware. No centro deve figurar o microcontrolador ESP32-S3 e, ao redor, blocos conectados por setas indicando o fluxo de sinal:
    * **Setas de entrada (In):** Sensor de umidade de solo, Sensor de temperatura/umidade do ar, Sensor de movimento e Câmera apontando para o ESP32-S3.
    * **Setas bidirecionais ($\leftrightarrow$):** ESP32-S3 conectado ao bloco do Cartão SD (via barramento SPI).
    * **Setas de saída (Out):** ESP32-S3 apontando para os blocos do Relé 1 (Bomba) e Relé 2 (Lâmpada).
    * **Fluxo de rede:** Seta indicando a transmissão bidirecional Wi-Fi com o Servidor/Dashboard.

* **4.3. Especificação do Armazenamento Local e Protocolos IoT:**
    * **Estratégia de Arquivo e Particionamento:** Descrição de como o sistema de arquivos (FAT32) será manipulado no cartão SD e qual a estrutura dos logs textuais (padrão de colunas do CSV).
    * **Protocolo de Aplicação de Rede:** Escolha do protocolo de comunicação com o servidor e justificativa de como o firmware alternará entre o envio de telemetria leve (números textuais dos sensores) e o envio de arquivos pesados de imagem (quando houver requisição de foto in loco em tempo real).
    * **Estrutura de Payloads (JSON):** Exemplo de mensagens JSON para envio de dados e recepção de comandos (ex: `{"comando": "irrigar", "status": true}`).
