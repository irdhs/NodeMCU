# NodeMCU #here the isting program
--------------------------------------------------------------
#define BLYNK_TEMPLATE_ID "TMPLWznvWL1_"
#define BLYNK_DEVICE_NAME "AquarIoT"
#define BLYNK_AUTH_TOKEN "1gH0Ix_LYqIrQcxd0ZJKteRUl08f6Z3E"
#include <ESP8266WiFi.h>
#include <BlynkSimpleEsp8266.h>
#define BLYNK_PRINT Serial
#include <NewPing.h>
#define TRIGGER_PIN  D0  // pin trigger ke nodemcu
#define ECHO_PIN     D1  // pin echo ke nodemcu
#define MAX_DISTANCE 200 // jarak maks sensor spy. tidak err
#define POMPA1 D2 // untuk mengisi
#define POMPA2 D3 // untuk kuras
bool kuras = true;


#include <WiFiClient.h>
unsigned long waktu_kirim = 0;
//unsigned long currentMillis;

const long interval = 30000;


unsigned long waktu_kirim_error = 0;
const long interval_error = 8000; // jika ketinggian tidak bertambah selam 8 detik, maka akan kirim notif ERR

int tinggi_terkini;
int tinggi_error;

int raw_kekeruhan, NTU;
float kekeruhan;
NewPing sonar(TRIGGER_PIN, ECHO_PIN, MAX_DISTANCE); // NewPing setup of pins and maximum distance.
BlynkTimer timer;

char auth[] = "1gH0Ix_LYqIrQcxd0ZJKteRUl08f6Z3E"; // BLYNK_AUTH_TOKEN; //"TfiB7Katm51GO9L1YA2ZIoV6IVaZwD-5";
char ssid[] = "wifi.id";
char pass[] = "12345677";

void kirim_data(){
tinggi_terkini = 20 - sonar.ping_cm();
raw_kekeruhan = analogRead(A0);
NTU = map (raw_kekeruhan, 100, 920 , 3000, 0);
float fNTU = float(NTU) / 10.0;
fNTU = fNTU + 5; // 8

digitalWrite(POMPA2, kuras); 
// fungsi mengirim notif ketika pompa on dan air tidak naik

if (tinggi_terkini == tinggi_error && kuras == false){
     unsigned long currentMillis = millis();
   
   Serial.println("Ada gejala rusak");
 
    if (currentMillis - waktu_kirim_error >= interval_error){
      
    Blynk.logEvent("ERR", "Alat Error Cek Sekarang"); //( nma event, notifnya)
    waktu_kirim_error = currentMillis;
    }
}

tinggi_error = tinggi_terkini;

if (tinggi_terkini < 6 && tinggi_terkini > 0){
digitalWrite(POMPA1, 0);
Serial.println("Pompa pengisian menyala");
}

if ( tinggi_terkini > 17 && tinggi_terkini < 20){
digitalWrite(POMPA1, 1);
Serial.println("Pompa pengisian mati");
}
Blynk.virtualWrite(V1, tinggi_terkini);
Blynk.virtualWrite(V2, fNTU);
Serial.print("Tinggi terkini: ");
Serial.println(tinggi_terkini);
Serial.print("NTU: ");
Serial.println(fNTU);

  
  if (fNTU >  20 && kuras == true){
   unsigned long currentMillis = millis();
   kuras = false;
   Serial.println("Pompa kuras menyala");
 
    if (currentMillis - waktu_kirim >= interval){
    Blynk.logEvent("KRH", "Air Keruh"); //( nma event, notifnya)
    waktu_kirim = currentMillis;
      }
    }
  if (fNTU < 20 && kuras == false){
  kuras = true;
  Serial.println("Pompa kuras mati");
  }

}
void setup() {
Serial.begin(9600);
Blynk.begin(auth, ssid, pass);
delay(2000);
pinMode(POMPA1, OUTPUT);
pinMode(POMPA2, OUTPUT);
pinMode(A0, INPUT);
Blynk.syncAll();
Serial.println("");
Serial.print("Connected to ");
Serial.println(ssid);
Serial.print("Buka Link ini untuk update: ");
Serial.print(WiFi.localIP());
Serial.println("/update");
delay(100);
timer.setInterval(120L, kirim_data);; 
}

void loop(void) {
Blynk.run();
timer.run();
delay(80);                     // Wait 50ms between pings (about 20 pings/sec). 29ms should be the shortest delay between pings.
}
