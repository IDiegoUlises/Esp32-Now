# Esp32-Now

### Codigo para obtener direccion mac wifi
```c++
#include <WiFi.h>
 
void setup()
{
  Serial.begin(115200);
  WiFi.mode(WIFI_MODE_STA);
  Serial.println(WiFi.macAddress());
}
 
void loop(){

}
```

### Remitente
```c++
#include <esp_now.h>
#include <WiFi.h>

//Remplazar con la direccion MAC del dispositivo receptor
uint8_t broadcastAddress[] = {0xA0, 0xB7, 0x65, 0x4A, 0x8F, 0xC4};

//Ejemplo de estructura para enviar datos
//Debe coincidir con la estructura del receptor
typedef struct struct_message {
  char a[32];
  int b;
  float c;
  bool d;
} struct_message;

//Crea un struct_message llamado myData
struct_message myData;

esp_now_peer_info_t peerInfo;

//devolucion de llamada cuando se envían datos
void OnDataSent(const uint8_t *mac_addr, esp_now_send_status_t status)
{
  Serial.print("\r\nLast Packet Send Status:\t");
  Serial.println(status == ESP_NOW_SEND_SUCCESS ? "Delivery Success" : "Delivery Fail");
}

void setup()
{
  //Inicializa el puerto serial
  Serial.begin(115200);

  //Establece el dispositovo como Wi-Fi modo estacion
  WiFi.mode(WIFI_STA);

  //Inicializa ESP-NOW
  if (esp_now_init() != ESP_OK) 
  {
    Serial.println("Error initializing ESP-NOW");
    return;
  }

  //Una vez que ESPNow se inicie con éxito, nos registraremos para Enviar CB a
  //obtener el estado del paquete transmitido
  esp_now_register_send_cb(OnDataSent);

  //Registrar peer
  memcpy(peerInfo.peer_addr, broadcastAddress, 6);
  peerInfo.channel = 0;
  peerInfo.encrypt = false;

  //Agregar peer
  if (esp_now_add_peer(&peerInfo) != ESP_OK) 
  {
    Serial.println("Failed to add peer");
    return;
  }
}

void loop()
{
  //Establece valores para enviar
  strcpy(myData.a, "THIS IS A CHAR");
  myData.b = random(1, 20);
  myData.c = 1.2;
  myData.d = false;

  //Envia mensaje mediante ESP-NOW
  esp_err_t result = esp_now_send(broadcastAddress, (uint8_t *) &myData, sizeof(myData));

  if (result == ESP_OK) 
  {
    Serial.println("Sent with success");
  }
  
  else 
  {
    Serial.println("Error sending the data");
  }
  delay(2000);
}
```
### Receptor
```c++
#include <esp_now.h>
#include <WiFi.h>

//Estructura para recibir datos
//Debe coincidir con la estructura del remitente
typedef struct struct_message {
  char a[32];
  int b;
  float c;
  bool d;
} struct_message;

//Crea un struct_message llamado myData
struct_message myData;

//funcion de devolucion de llamada que se ejecutara cuando se reciban los datos
void OnDataRecv(const uint8_t * mac, const uint8_t *incomingData, int len)
{
  memcpy(&myData, incomingData, sizeof(myData));
  Serial.print("Bytes received: ");
  Serial.println(len);
  Serial.print("Char: ");
  Serial.println(myData.a);
  Serial.print("Int: ");
  Serial.println(myData.b);
  Serial.print("Float: ");
  Serial.println(myData.c);
  Serial.print("Bool: ");
  Serial.println(myData.d);
  Serial.println();
}

void setup() 
{
  //Inicializa el puerto serial
  Serial.begin(115200);

  //Establece el dispositivo como modo Wi-Fi estacion
  WiFi.mode(WIFI_STA);

  //Inicializa ESP-NOW
  if (esp_now_init() != ESP_OK) 
  {
    Serial.println("Error initializing ESP-NOW");
    return;
  }

  //Una vez que ESPNow se inicie con éxito, nos registraremos para recv CB para
  //obtener información del empaquetador recv
  esp_now_register_recv_cb(OnDataRecv);
}

void loop() 
{

}
```
