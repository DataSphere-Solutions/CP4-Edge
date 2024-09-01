# Edge Computing - CP4 - Conexão Fiware e Postman via VM
Olá, bem-vindo ao nosso trabalho do Check Point 4 de Edge Computing! Nós somos a empresa Data Sphere da turma 1ESPH, e é um imenso prazer apresentar este projeto.

![Data Sphere1000x1000](https://github.com/ianmonteirom/CP2-Edge/assets/152393807/0fe80a9b-6290-417d-8367-2abe3824d0b0)
Logo da nossa equipe
## O que é a Data Sphere?
A Data Sphere Solutions é uma empresa fictícia representando a nossa equipe, formada pelos alunos: 
-  <a href="https://www.linkedin.com/in/artur-alves-tenca-b1ba862b6/">Artur Alves</a> - RM 555171 
- <a href="https://www.linkedin.com/in/giuliana-lucas-85b4532b6/">Giuliana Lucas</a> - RM 557597
- <a href="https://www.linkedin.com/in/ian-monteiro-moreira-a4543a2b7/">Ian Monteiro</a> - RM 558652 
- <a href="https://www.linkedin.com/in/igor-brunelli-ralo-39143a2b7/">Igor Brunelli</a> - RM 555035
- <a href="https://www.linkedin.com/in/matheus-estev%C3%A3o-5248b9238/">Matheus Alcântara</a> - RM 558193

## Máquina Virtual hospedada na Nuvem
Utilizando a Microsoft Azure, hospedamos e configuramos uma máquina virtual (VM) com Ubuntu Server de sistema operacional. Nela, instalamos o Docker, o Docker Compose e o Fiware Descomplicado do professor Fábrio Cabrini, e abrimos as portas necessárias para todas as comunicações neste projeto serem possíveis.
![image](https://github.com/user-attachments/assets/40755ca2-5925-4e9e-a063-1c94e3953cbb)

## Postman
Utilizando o Postman para ler a coleção do API do Fiware Descomplicado e configurando o IP público da VM, fazemos os health checks e confirmamos que está tudo comunicando corretamente.
![image](https://github.com/user-attachments/assets/ad7cfabe-54e6-4e79-991b-507691bbe501)
![image](https://github.com/user-attachments/assets/9dadf132-e51e-4b70-b94e-200baa7d275d)
![image](https://github.com/user-attachments/assets/dbb7e187-33c6-4782-98c0-e433c94b7577)
![image](https://github.com/user-attachments/assets/ee958b0e-2e61-4d4d-bdc7-c2de6d6bf75f)

## Simulação no Wokwi
Utilizando um simulador online como o Wokwi, e configurando o código corretamente para enviar e receber dados, podemos enviar os valores de luminosidade para o Postman e receber valores de led on/off do Postman.
![image](https://github.com/user-attachments/assets/50db4c79-00ae-4629-a3f4-2e7a5fbe2efb)
![image](https://github.com/user-attachments/assets/9d89a804-7c32-4255-a1b5-2c81d970f0e3)
![image](https://github.com/user-attachments/assets/d9f18839-851f-4620-9694-5b057c980aa2)
![image](https://github.com/user-attachments/assets/9665e8db-6429-4a78-b2bc-f804ba86bc24)
![image](https://github.com/user-attachments/assets/bd14148c-ca8b-417d-b9c1-8221d18e0651)
- Link do Projeto: https://wokwi.com/projects/406657416142643201

## Código do Projeto
 ```
#include <WiFi.h>
#include <PubSubClient.h>

// Configurações - variáveis editáveis
const char* default_SSID = "Wokwi-GUEST"; // Nome da rede Wi-Fi
const char* default_PASSWORD = ""; // Senha da rede Wi-Fi
const char* default_BROKER_MQTT = "20.197.230.60"; // IP do Broker MQTT
const int default_BROKER_PORT = 1883; // Porta do Broker MQTT
const char* default_TOPICO_SUBSCRIBE = "/TEF/lamp001/cmd"; // Tópico MQTT de escuta
const char* default_TOPICO_PUBLISH_1 = "/TEF/lamp001/attrs"; // Tópico MQTT de envio de informações para Broker
const char* default_TOPICO_PUBLISH_2 = "/TEF/lamp001/attrs/l"; // Tópico MQTT de envio de informações para Broker
const char* default_ID_MQTT = "fiware_001"; // ID MQTT
const int default_D4 = 2; // Pino do LED onboard
// Declaração da variável para o prefixo do tópico
const char* topicPrefix = "lamp001";

// Variáveis para configurações editáveis
char* SSID = const_cast<char*>(default_SSID);
char* PASSWORD = const_cast<char*>(default_PASSWORD);
char* BROKER_MQTT = const_cast<char*>(default_BROKER_MQTT);
int BROKER_PORT = default_BROKER_PORT;
char* TOPICO_SUBSCRIBE = const_cast<char*>(default_TOPICO_SUBSCRIBE);
char* TOPICO_PUBLISH_1 = const_cast<char*>(default_TOPICO_PUBLISH_1);
char* TOPICO_PUBLISH_2 = const_cast<char*>(default_TOPICO_PUBLISH_2);
char* ID_MQTT = const_cast<char*>(default_ID_MQTT);
int D4 = default_D4;

WiFiClient espClient;
PubSubClient MQTT(espClient);
char EstadoSaida = '0';

void initSerial() {
    Serial.begin(115200);
}

void initWiFi() {
    delay(10);
    Serial.println("------Conexao WI-FI------");
    Serial.print("Conectando-se na rede: ");
    Serial.println(SSID);
    Serial.println("Aguarde");
    reconectWiFi();
}

void initMQTT() {
    MQTT.setServer(BROKER_MQTT, BROKER_PORT);
    MQTT.setCallback(mqtt_callback);
}

void setup() {
    InitOutput();
    initSerial();
    initWiFi();
    initMQTT();
    delay(5000);
    MQTT.publish(TOPICO_PUBLISH_1, "s|on");
}

void loop() {
    VerificaConexoesWiFIEMQTT();
    EnviaEstadoOutputMQTT();
    handleLuminosity();
    MQTT.loop();
}

void reconectWiFi() {
    if (WiFi.status() == WL_CONNECTED)
        return;
    WiFi.begin(SSID, PASSWORD);
    while (WiFi.status() != WL_CONNECTED) {
        delay(100);
        Serial.print(".");
    }
    Serial.println();
    Serial.println("Conectado com sucesso na rede ");
    Serial.print(SSID);
    Serial.println("IP obtido: ");
    Serial.println(WiFi.localIP());

    // Garantir que o LED inicie desligado
    digitalWrite(D4, LOW);
}

void mqtt_callback(char* topic, byte* payload, unsigned int length) {
    String msg;
    for (int i = 0; i < length; i++) {
        char c = (char)payload[i];
        msg += c;
    }
    Serial.print("- Mensagem recebida: ");
    Serial.println(msg);

    // Forma o padrão de tópico para comparação
    String onTopic = String(topicPrefix) + "@on|";
    String offTopic = String(topicPrefix) + "@off|";

    // Compara com o tópico recebido
    if (msg.equals(onTopic)) {
        digitalWrite(D4, HIGH);
        EstadoSaida = '1';
    }

    if (msg.equals(offTopic)) {
        digitalWrite(D4, LOW);
        EstadoSaida = '0';
    }
}

void VerificaConexoesWiFIEMQTT() {
    if (!MQTT.connected())
        reconnectMQTT();
    reconectWiFi();
}

void EnviaEstadoOutputMQTT() {
    if (EstadoSaida == '1') {
        MQTT.publish(TOPICO_PUBLISH_1, "s|on");
        Serial.println("- Led Ligado");
    }

    if (EstadoSaida == '0') {
        MQTT.publish(TOPICO_PUBLISH_1, "s|off");
        Serial.println("- Led Desligado");
    }
    Serial.println("- Estado do LED onboard enviado ao broker!");
    delay(1000);
}

void InitOutput() {
    pinMode(D4, OUTPUT);
    digitalWrite(D4, HIGH);
    boolean toggle = false;

    for (int i = 0; i <= 10; i++) {
        toggle = !toggle;
        digitalWrite(D4, toggle);
        delay(200);
    }
}

void reconnectMQTT() {
    while (!MQTT.connected()) {
        Serial.print("* Tentando se conectar ao Broker MQTT: ");
        Serial.println(BROKER_MQTT);
        if (MQTT.connect(ID_MQTT)) {
            Serial.println("Conectado com sucesso ao broker MQTT!");
            MQTT.subscribe(TOPICO_SUBSCRIBE);
        } else {
            Serial.println("Falha ao reconectar no broker.");
            Serial.println("Haverá nova tentativa de conexão em 2s");
            delay(2000);
        }
    }
}

void handleLuminosity() {
    const int potPin = 34;
    int sensorValue = analogRead(potPin);
    int luminosity = map(sensorValue, 0, 4095, 0, 100);
    String mensagem = String(luminosity);
    Serial.print("Valor da luminosidade: ");
    Serial.println(mensagem.c_str());
    MQTT.publish(TOPICO_PUBLISH_2, mensagem.c_str());
}
```
