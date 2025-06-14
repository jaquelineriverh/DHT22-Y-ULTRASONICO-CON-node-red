# DHT22-Y-ULTRASONICO-CON-node-red
ESTE REPOSITORIO MUESTRA COMO PROGRAMAR CON DHT Y ULTRASONICO CON NODE-RED
## INTRODUCCION

### DESCRIPCION
La **NODE-RED** es una herramienta que permite interconectar hardware, software y servicios en la nube, facilitando la creacion de flujos de trabajo automatizados para diversas aplicaciones para el internet de la cosas. Recibiendo procesando, almacenando y visualizando datos  de sensores y otros dispositivos en tiempo real. En esta practica se utilizará para realizar un monitoreo de los datos (temperatura, humedad y distancia) del simulador WOKWI.
La Esp32 se utilizara para adquirir los datos, ocupando un sensor (DTH11) para otorgarnos una temperatura y humedad del entorno, al igual que un sensor ultrasonico para la medicion de  distancia.


## MATERIAL NECESARIO

Para realizar esta practica necesitas lo siguiente:

-[WOKWI](https://wokwi.com/)

-Tarjeta ESP 32

-Sensor DHT11

-HC-SR04 ULTRASONIC Distance sensor

-NODE-RED

## INSTRUCCIONES PARA WOKWI

### Requisitos previos

Para poder usar este repositorio necesitas entrar a la plataforma [WOKWI](https://wokwi.com/)

### Instrucciones de preparación de entorno

1.Abrir la terminal de programación y colocar la siguente programación:

```
#include <ArduinoJson.h>
#include <WiFi.h>
#include <PubSubClient.h>
#define BUILTIN_LED 2
#include "DHTesp.h"
DHTesp dhtSensor;
const int Trigger = 4;   //Pin digital 2 para el Trigger del sensor
const int Echo = 15;   //Pin digital 3 para el Echo del sensor
const int DHT_PIN = 16;
// Update these with values suitable for your network.

const char* ssid = "Wokwi-GUEST";
const char* password = "";
const char* mqtt_server = "52.29.87.71";
String username_mqtt="JAQUELINERH";
String password_mqtt="1234";

WiFiClient espClient;
PubSubClient client(espClient);
unsigned long lastMsg = 0;
#define MSG_BUFFER_SIZE  (50)
char msg[MSG_BUFFER_SIZE];
int value = 0;

void setup_wifi() {

  delay(10);
  // We start by connecting to a WiFi network
  Serial.println();
  Serial.print("Connecting to ");
  Serial.println(ssid);

  WiFi.mode(WIFI_STA);
  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }

  randomSeed(micros());

  Serial.println("");
  Serial.println("WiFi connected");
  Serial.println("IP address: ");
  Serial.println(WiFi.localIP());
}

void callback(char* topic, byte* payload, unsigned int length) {
  Serial.print("Message arrived [");
  Serial.print(topic);
  Serial.print("] ");
  for (int i = 0; i < length; i++) {
    Serial.print((char)payload[i]);
  }
  Serial.println();

  // Switch on the LED if an 1 was received as first character
  if ((char)payload[0] == '1') {
    digitalWrite(BUILTIN_LED, LOW);   
    // Turn the LED on (Note that LOW is the voltage level
    // but actually the LED is on; this is because
    // it is active low on the ESP-01)
  } else {
    digitalWrite(BUILTIN_LED, HIGH);  
    // Turn the LED off by making the voltage HIGH
  }

}

void reconnect() {
  // Loop until we're reconnected
  while (!client.connected()) {
    Serial.print("Attempting MQTT connection...");
    // Create a random client ID
    String clientId = "ESP8266Client-";
    clientId += String(random(0xffff), HEX);
    // Attempt to connect
    if (client.connect(clientId.c_str(), username_mqtt.c_str() , password_mqtt.c_str())) {
      Serial.println("connected");
      // Once connected, publish an announcement...
      client.publish("outTopic", "hello world");
      // ... and resubscribe
      client.subscribe("inTopic");
    } else {
      Serial.print("failed, rc=");
      Serial.print(client.state());
      Serial.println(" try again in 5 seconds");
      // Wait 5 seconds before retrying
      delay(5000);
    }
  }
}

void setup() {
  pinMode(BUILTIN_LED, OUTPUT);     // Initialize the BUILTIN_LED pin as an output
  Serial.begin(115200);
  setup_wifi();
  client.setServer(mqtt_server, 1883);
  client.setCallback(callback);
  dhtSensor.setup(DHT_PIN, DHTesp::DHT22);
  pinMode(Trigger, OUTPUT); //pin como salida
  pinMode(Echo, INPUT);  //pin como entrada
  digitalWrite(Trigger, LOW);//Inicializamos el pin con 0
}

void loop() {


delay(1000);
 long t; //timepo que demora en llegar el eco
  long d; //distancia en centimetros

  digitalWrite(Trigger, HIGH);
  delayMicroseconds(10);          //Enviamos un pulso de 10us
  digitalWrite(Trigger, LOW);
  
  t = pulseIn(Echo, HIGH); //obtenemos el ancho del pulso
  d = t/59;             //escalamos el tiempo a una distancia en cm
  Serial.print("Distancia: ");
  Serial.print(d);      //Enviamos serialmente el valor de la distancia
  Serial.print("cm");
  Serial.println();
  delay(1000);          //Hacemos una pausa de 100ms

TempAndHumidity  data = dhtSensor.getTempAndHumidity();
  if (!client.connected()) {
    reconnect();
  }
  client.loop();

  unsigned long now = millis();
  if (now - lastMsg > 2000) {
    lastMsg = now;
    //++value;
    //snprintf (msg, MSG_BUFFER_SIZE, "hello world #%ld", value);

    StaticJsonDocument<128> doc;

    doc["DEVICE"] = "ESP32";
    doc["NOMBRE"] = "JAQUELIN RIV";
    doc["TEMPERATURA"] = String(data.temperature, 1);
    doc["HUMEDAD"] = String(data.humidity, 1);
    doc["DISTANCIA"] = String(d);
   

    String output;
    
    serializeJson(doc, output);

    Serial.print("Publish message: ");
    Serial.println(output);
    Serial.println(output.c_str());
    client.publish("DiplomadoJRH", output.c_str());
  }
}
```


2.Instalar la libreria de DHT sensor library for ESPx como se muestra en la siguente imagen


![](https://github.com/jaquelineriverh/PRACTICA-ESP32-DHT11/blob/main/DHT.jpg)

3. Instalar la libreria de LiquidCrystal I2C como se muestra en la siguiente imagen



![](https://github.com/jaquelineriverh/PRACTICA-2-ESP32-CON-DHT11-Y-Lcd/blob/main/libreria%20liquid.png)


4.Hacer la conexion de HC-SR04 ULTRASONIC Distance sensor con la ESP32 como se muestra en la siguente imagen.

![](https://github.com/jaquelineriverh/-PRACTICA-ESP32-CON-DHT11-LCD-Y-ULTRASONICO/blob/main/ULTRASONICO%20CONEXION.png)



5.Hacer la conexion de LCD con la ESP32 como se muestra en la siguente imagen.


![](https://github.com/jaquelineriverh/-PRACTICA-ESP32-CON-DHT11-LCD-Y-ULTRASONICO/blob/main/ULTRASONIC%20CON%20LCD.png)


6.Hacer la conexion de DHT11 con la ESP32 como se muestra en la siguente imagen.

![](https://github.com/jaquelineriverh/-PRACTICA-ESP32-CON-DHT11-LCD-Y-ULTRASONICO/blob/main/lcd%20ultraconic%20dht.png)


### Instrucciónes de operación
1.Iniciar simulador.

2.Visualizar los datos en el LCD 16x2.

3.Colocar la temperatura y humedad dando doble click al sensor DHT11

4. Colocar la distancia dando doble click al sensor ultrasonico

## Resultados

Cuando haya funcionado, verás los valores dentro del LCD 16x2 como se muestra en la siguente imagen.


![](https://github.com/jaquelineriverh/-PRACTICA-ESP32-CON-DHT11-LCD-Y-ULTRASONICO/blob/main/resultados%20para%20distancia.png)





![](https://github.com/jaquelineriverh/-PRACTICA-ESP32-CON-DHT11-LCD-Y-ULTRASONICO/blob/main/resultados%20para%20temperatura.png)

### Evidencias


https://github.com/user-attachments/assets/da50e77f-b7f1-44fa-8e76-bdde90a2fced




# Créditos

Desarrollado por **RIVERA HERNANDEZ JAQUELINE**

-[GITHUB](https://github.com/jaquelineriverh)
