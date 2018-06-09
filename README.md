//Alejandro Chara
//Gabriela Muñoz
//Miguel Angel guerrero

//las Librerias OneWire y DallasT para poner en funcionamiento el sensor de temperatura DS18B20
#include <OneWire.h>
#include <DallasTemperature.h>
#include <ESP8266WiFi.h>

//Librerias que necesitamos para la conexion con la pantalla lcd 16 x 2
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
LiquidCrystal_I2C lcd(0x3F, 16, 2);

#define DS18B20 D4          //El sensor de temperatura DS18B20 esta conectado al pin D4(GPIO02)


String apiKey = "X2IETOVL6GH89DWL";     //API key de ThingSpeak
const char* ssid =  "ALEJANDRO";        // Aqui ingresamos el nombre de la RED
const char* pass =  "317827398300A";        // Aqui ingresamos la contraña de la red WIFI
const char* server = "api.thingspeak.com";
float temp;  //Declaramos la variable que toma el valor de la temperatura

const int pinBuzzer = D3;  //  El pin D3(GPIO0)es  el pin donde conectamos el buzzer
const int AZUL = D6;   //El pin D0(GPIO6) es el pin de conexion al led AZUL
const int VERDE = D5;  //El pin D1(GPIO5) es el pin de conexion al led VERDE
const int ROJO = D0;   //El pin D2(GPIO4) es el pin de conexion al led ROJO
const  int numTones = 10; //numTones es elnumero de eveces que emite los sonidos del buzzer
const int tones[ ] = {261, 277, 294, 311, 330, 349, 370, 392, 415, 440, 466, 494};

//Creamos un bus ourWire donde recibe como parametro DS18B20 el cual tiene el pin de entrada del sensor de temperatura
OneWire ourWire(DS18B20);
//Creamos un bus sensor de la libreria Dallas, la cual nos servirá para calcular el tiempo,  y recibe como parametro ourWire
DallasTemperature sensor(&ourWire);
//Inicializa la librería WifiClient para la conexión a Wifi
WiFiClient client;

void setup()

{
 
  pinMode(AZUL, OUTPUT); //declaramos GPIO6 como pin de salida de led
  pinMode(VERDE, OUTPUT); //declaramos GPIO5 como pin de salida de led
  pinMode(ROJO, OUTPUT); //declaramos GPIO4 como pin de salida de led
  //Velocidad para transmitir datos00
  Serial.begin(115200);
  delay(1000);
  //Iniciamos el sensor
  sensor.begin();

  Serial.print("Conectando a: ");
  Serial.println(ssid);


//se hace la verificacion de los datos de la red wifi
  WiFi.begin(ssid, pass);

//mientras la red wifi esta en conexión 
  while (WiFi.status() != WL_CONNECTED)
  {
    delay(100);
    Serial.print("*");
  }
  Serial.println("");
  Serial.println("***WiFi connected***");


}

void loop()
{
  //inicializamos la pantalla lcd
  lcd.init();
  lcd.backlight();

  lcd.begin(16, 2);
  lcd.print("___TEMPERATURA___"); //texto que aparecerá en la pantalla lcd
  lcd.setCursor(0, 1);   //fijamos la posicion, 0 es la posicion de la columna y 1 la posicion de la fila

  sensor.requestTemperatures(); //Metodo por el cual se recibira los datos de la temperatura
  temp = sensor.getTempCByIndex(0); // la variabe tem almacena el valor de la temperatura que es un dato de float

  if (client.connect(server, 80))  //   "184.106.153.149" o api.thingspeak.com
  {
    String sendData = apiKey + "&field1=" + String(temp) + "\r\n\r\n";

    //Serial.println(envia los datos a nuestro canal thingSpeak);

    client.print("POST /update HTTP/1.1\n");
    client.print("Host: api.thingspeak.com\n");
    client.print("Connection: close\n");
    client.print("X-THINGSPEAKAPIKEY: " + apiKey + "\n");
    client.print("Content-Type: application/x-www-form-urlencoded\n");
    client.print("Content-Length: ");
    client.print(sendData.length());
    client.print("\n\n");
    client.print(sendData);
//Los datos de la temperatura se reflejaran en una grafica en thingSpeak
    Serial.print("Temperature: ");
    Serial.print(temp);
    Serial.println("deg C. Connecting to Thingspeak..");
  }

  client.stop();

  Serial.println("Sending....");

  if (sensor.getTempCByIndex(0) < 20.00) { //Si la temperatura es MENOR a 20 grados
    lcd.clear();

    digitalWrite(AZUL, HIGH); //se enciende el led AZUL
    digitalWrite(VERDE, LOW); //se mantiene apagadoel led VERDE
    digitalWrite(ROJO, LOW); //se matiene apagado el led ROJO
    lcd.setCursor(0, 0);
    lcd.print("TEMP FRIA"); //Se imprimirá este texto cuando el bombillo Azul este encendido
    String t = String ((float) temp) + "C";//la variable t almacena el valor de la temperatura como un string psra poder imprimirlo en la pantalla lcd
    lcd.setCursor (6, 1); //fijamos la posicion en el lcd, el texto se mostrara desde la columna 6 de la fila 1
    lcd.print (t); //t es la variable de texto que se imprimirá
    for (int iTones = 0; iTones < numTones; iTones++) {  //mientras el numero de tonos emitido sea menor a 10 el seguira sonando
      tone(pinBuzzer, 440); //440 es un tono del buzzer
      delay(1000); // espera en milisegundos
    }

  } else {
    //Si el valor float recibo por el sensor es mayor a 20 y menos a 29 entonces se ejecutara la siguientes intrucciones
    if (sensor.getTempCByIndex(0) > 20.00 && sensor.getTempCByIndex(0) < 29.00) {
      lcd.clear();
      digitalWrite(AZUL, LOW);
      digitalWrite(VERDE, HIGH);
      digitalWrite(ROJO, LOW);
      lcd.setCursor(0, 0);
      lcd.print("TEMP NORMAL");
      String t = String ((float) temp) + "C";
      lcd.setCursor (6, 1);
      lcd.print (t);
      noTone(pinBuzzer);

    } else {
      if (sensor.getTempCByIndex(0) > 29.00) {
        lcd.clear();
        digitalWrite(AZUL, LOW);
        digitalWrite(VERDE, LOW);
        digitalWrite(ROJO, HIGH);
        lcd.setCursor(0, 0);
        lcd.print("TEMP CALIENTE");
        String t = String ((float) temp) + "C";
        lcd.setCursor (6, 1);
        lcd.print (t);
        for (int iTones = 0; iTones < numTones; iTones++) {
          tone(pinBuzzer, tones[iTones]);
          delay(1000);
        }

      }
    }
  }

  delay(10000);
}
