# DHT22-Y-ULTRASONICO-CON-node-red
ESTE REPOSITORIO MUESTRA COMO PROGRAMAR CON DHT11 Y ULTRASONICO CON NODE-RED
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


2.Instalar las siguientes librerias como se muestra la siguiente imagen.

- DHT sensor library for ESPx

- ArduinoJson

- WiFi

- PubSubClient


![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/librerias%20de%20nodered.png)




4.Hacer la conexion de HC-SR04 ULTRASONIC Distance sensor y DHT11 con la ESP32 como se muestra en la siguente imagen.

![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/conexion%20ultra%20y%20dth.png)





   

## INSTRUCCIONES PARA INSTALACION DE **node-red**


### Requisitos previos

Para poder usar este programa se necesita descargar el archivo Node.js v22.16.0 previamente en [NODE-RED](https://nodejs.org/en)

### Instrucciones de preparación de entorno


1.- Instalar el programa correctamente


2.- Entrar al CMD (simbolo del sistema) en modo administrador y escribir lo siguiente:

```
npm install -g --unsafe-perm node-red

```

3.- Comprobar que funciona node-red con el siguente codigo: (con este mismo codigo podemos arrancar el programa siempre que lo necesitemos).

```
node-red
```

![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/instalacion%20de%20node%20red%20y%20arranque.png)


4.-Para abrir la aplicación nos vamos algun explorador y colocamos el siguente link: 


```
localhost:1880
```

5.-Instalar Dashboard abriendo la pestaña de opciones y elegimos *Manage palette*

6.-Seleccionamos *Install y buscamos node-red-dashboard*.

7.-Seleccionamos *node-red-dashboard*



## INSTRUCCIONES PARA LA COLOCACION DE BLOQUES Y LA CONEXION DE NODOS CORRESPONDIENTES EN  **node-red**

# Instrucciones

1.-Colocar bloque *mqqtt in*.

2.-Configurar el bloque con el puerto mqtt con el ip 52.29.87.71 como se muestra en la imagen


![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/mqtt%20in.png)



3.-Colocar el bloque *json* y configurarlo como se muestra en la imagen



![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/json.png)



4.-Colocamos 3 bloques *function* y lo configuramos con el siguente código.

```
msg.payload = msg.payload.TEMPERATURA;
msg.topic = "TEMPERATURA";
return msg
```


```
msg.payload = msg.payload.HUMEDAD;
msg.topic = "HUMEDAD";
return msg;
```

```
msg.payload = msg.payload.DISTANCIA;
msg.topic = "DISTANCIA";
return msg;
```


5.-Colocamos un bloque de *debug*.


6.-Colocamos los bloques de chart y gauge correspondientes, con sus respectivas tablas.


7.- La conexion de bloques esperada se muestra en la siguiente imagen:


![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/bloques.png)



### Instrucciónes de operación

1.Iniciar simulador.


2.Colocar la temperatura y humedad dando doble click al sensor DHT11



3. Colocar la distancia dando doble click al sensor ultrasonico


4. Ir al navegador donde este abierto nuestro node-red y apretar el boton *instanciar*




![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/INSTANCIAR.png)


5. En el *dash board* apretar en el siguiente boton



![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/ABRIR%20GRAFICAS.png)





## Resultados.

#SIMULADOR DE WOKWI

![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/RESULTADO%20NODE%20WOKWI.png)


#NODE-RED. TABLAS DE DATOS Y GRAFICOS


![](https://github.com/jaquelineriverh/DHT22-Y-ULTRASONICO-CON-node-red/blob/main/DATOS%20BIEN%20NODE.png)



### Evidencias



https://github.com/user-attachments/assets/e9c257fe-fe9d-4fcc-aa1c-9ea4ed3e137a







https://github.com/user-attachments/assets/ba7c3325-0e8a-404a-a22a-47f1612db9a8













# Créditos

Desarrollado por **RIVERA HERNANDEZ JAQUELINE**

-[GITHUB](https://github.com/jaquelineriverh)
