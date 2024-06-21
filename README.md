## Desenvolvimento uma placa PCB partindo de um protótipo com o microcontrolador NodeMCU8266

### ➯ ESQUEMÁTICO

![Captura de tela 2024-06-04 211552](https://github.com/natifss/SISTEMAS-EMBARCADOS-RTOS/assets/119085630/f3ddbfbe-e025-4ca1-8df9-d396948cc4b8)


### ➯ PCB

![Captura de tela 2024-06-04 213401](https://github.com/natifss/SISTEMAS-EMBARCADOS-RTOS/assets/119085630/b4fcdf23-f987-4855-bbad-edc2caa338f2)


### ➯ 3D

![Captura de tela 2024-06-04 211723](https://github.com/natifss/SISTEMAS-EMBARCADOS-RTOS/assets/119085630/dfa7c4f1-d58d-415d-9be1-24762547a34e)

### ➯ O QUE É O MQTT

  MQTT (Message Queuing Telemetry Transport) é um protocolo de comunicação leve, projetado para transmitir mensagens entre dispositivos de forma eficiente e confiável, mesmo em redes com largura de banda limitada ou conexões instáveis. Ele é amplamente utilizado em aplicações de Internet das Coisas (IoT), onde dispositivos como sensores, atuadores e outros tipos de equipamentos precisam trocar dados de maneira eficaz. É uma escolha popular para aplicações de IoT e outros cenários que requerem comunicação eficiente entre dispositivos devido à sua leveza, flexibilidade e confiabilidade.

### ➯ COMPONENTES

O MQTT é um protocolo de comunicação leve e eficiente, projetado para dispositivos IoT. Seus principais componentes incluem:

##### ▸ Clientes
> Dispositivos ou aplicativos que enviam (publicadores) ou recebem (assinantes) mensagens

##### ▸ Broker 
> O servidor central que gerencia a comunicação entre os clientes, filtrando e encaminhando mensagens conforme os tópicos.

##### ▸ Tópicos 
> Canais hierárquicos onde as mensagens são publicadas e assinadas, como casa/sala/temperatura.

##### ▸ Mensagens 
> Dados enviados entre clientes através do broker, associados a tópicos específicos.

##### ▸ Sessões 
> Estados mantidos entre clientes e o broker, que podem ser persistentes (mantêm mensagens não entregues) ou não-persistentes.

##### ▸ Qualidade de Serviço (QoS) 
> Três níveis (QoS 0, 1 e 2) que garantem diferentes níveis de entrega de mensagens, desde nenhuma garantia até entrega exatamente uma vez.

##### ▸ Retenção de Mensagens 
> Mensagens marcadas como retidas são armazenadas pelo broker e enviadas a novos assinantes de um tópico.

##### ▸ Last Will and Testament (LWT) 
> Mensagem de última vontade definida por um cliente, publicada pelo broker em caso de desconexão inesperada.

### ➯ FUNCIONAMENTO

  No MQTT, os componentes trabalham juntos de maneira integrada para criar um sistema eficiente de comunicação entre dispositivos. Clientes, que podem ser publicadores ou assinantes, enviam e recebem mensagens sobre tópicos específicos. Publicadores enviam mensagens para o broker, especificando o tópico relevante, e o broker, por sua vez, recebe essas mensagens e as distribui para os assinantes inscritos no tópico correspondente. 
  Tópicos organizam as mensagens em canais hierárquicos, permitindo que os assinantes escolham quais dados receber. Sessões mantêm o estado entre clientes e o broker, assegurando a continuidade da comunicação mesmo após desconexões. A Qualidade de Serviço (QoS) garante diferentes níveis de entrega das mensagens, variando de "no máximo uma vez" até "exatamente uma vez". A retenção de mensagens permite que o broker armazene a última mensagem de um tópico e a envie a novos assinantes. 
  Além disso, o Last Will and Testament (LWT) fornece uma mensagem predefinida que o broker publica caso um cliente se desconecte inesperadamente, informando os demais clientes da desconexão. Esses componentes colaboram para garantir que as mensagens sejam entregues de maneira confiável e eficiente, adaptando-se a diferentes condições de rede e requisitos de aplicação.

### ➯ CÓDIGO

  O código a seguir é um exemplo de como conectar um dispositivo ESP8266 a uma rede Wi-Fi e a um servidor MQTT para controlar um relé. Ele utiliza as bibliotecas Arduino, ESP8266WiFi e PubSubClient.

#### ▸ Incluindo bibliotecas
```sh
#include <Arduino.h>
#include <ESP8266WiFi.h>
#include <PubSubClient.h>

```
Essas bibliotecas são necessárias para a funcionalidade do ESP8266, para conectar-se à Wi-Fi e para a comunicação MQTT

#### ▸ Configurações de variáveis
```sh
const char* ssid = "iPhone de Natali";
const char* password = "nati23032002";
const char* mqtt_server = "172.20.10.2";

```
Essas variáveis armazenam o SSID e a senha da rede Wi-Fi, e o endereço IP do servidor MQTT.

#### ▸ Configuração do cliente WI-FI e MQTT
```sh
WiFiClient espClient;
PubSubClient client(espClient);
const int Relay = D1;

```
Aqui, espClient é uma instância do cliente Wi-Fi e client é uma instância do cliente MQTT. Relay define o pino D1 do ESP8266 para controlar o relé.

#### ▸ Função 'setup_WIFI'
```sh
void setup_wifi() {
  delay(10);
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

```
Esta função conecta o ESP8266 à rede Wi-Fi. Ela imprime o status de conexão no monitor serial e mostra o endereço IP do dispositivo após a conexão.

#### ▸ Função Callback
```sh
void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  String message;
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
    message += (char)payload[i];
  }
  Serial.println(message);

  if (String(topic) == "ControleRelay") {
    if (message == "ON") {
      digitalWrite(Relay, LOW);
    } else if (message == "OFF") {
      digitalWrite(Relay, HIGH);
    }
  }
}

```
Esta função é chamada quando uma mensagem é recebida em um tópico subscrito. Ela imprime o tópico e a mensagem recebida no monitor serial. Se a mensagem for "ON", o relé é ativado (pino D1 é configurado como LOW), e se for "OFF", o relé é desativado (pino D1 é configurado como HIGH).

#### ▸ Função 'setup'
```sh
void setup() {
  pinMode(Relay, OUTPUT);
  Serial.begin(9600);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
}

```
A função setup configura o pino do relé como saída, inicia a comunicação serial, conecta à Wi-Fi e configura o servidor e a função de callback do MQTT.

#### ▸ Função 'Reconnect'
```sh
void reconnect() {
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    if (client.connect("ESP8266Client")) {
      Serial.println("connected");
      client.subscribe("ControleRelay");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      delay(5000);
    }
  }
}

```
Esta função tenta reconectar ao servidor MQTT caso a conexão seja perdida. Ela tenta conectar-se repetidamente até ser bem-sucedida, e após a conexão, inscreve-se no tópico "ControleRelay".

#### ▸ Função Loop
```sh
void loop() {
  if (!client.connected()) {
    reconnect();
  }
  client.loop();
}

```
A função loop verifica se a conexão MQTT está ativa e, se não estiver, chama a função reconnect para restabelecer a conexão. A função client.loop mantém a comunicação MQTT ativa.

### ➯ PRINTS DA ATIVIDADE

#### ▸ BROKER
![WhatsApp Image 2024-06-18 at 21 21 23](https://github.com/natifss/SISTEMAS-EMBARCADOS-RTOS/assets/119085630/d4e6dd8c-3a8e-4981-a6a6-fe8ae7e1029f)

#### ▸ SUB
![WhatsApp Image 2024-06-18 at 21 22 04](https://github.com/natifss/SISTEMAS-EMBARCADOS-RTOS/assets/119085630/1ab17462-521c-4023-b808-5f49b48e7153)

#### ▸ PUB
![WhatsApp Image 2024-06-18 at 21 22 04](https://github.com/natifss/SISTEMAS-EMBARCADOS-RTOS/assets/119085630/341ee5e8-d67c-4b94-a891-79d32acdc53b)

#### ▸ LED LIGANDO E DESLIGANDO NO VSCODE
![WhatsApp Image 2024-06-18 at 21 23 36](https://github.com/natifss/SISTEMAS-EMBARCADOS-RTOS/assets/119085630/44776789-fddb-4778-977f-0c4305dff5a4)

