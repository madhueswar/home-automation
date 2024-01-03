# home-automation
 This project empowers you to control various home appliances not only through Google Assistant but also via Sinric Pro and manual switches. It's a versatile setup that allows you to manage your devices remotely, even when you're miles away from home.
#include"Adafruit_MQTT.h"
#include"Adafruit_MQTT_Client.h"
#defineWIFI_SSID"E for Engineer"//Your wifi name
#defineWIFI_PASS"EforEngineer"//your wifi password
#defineMQTT_SERV"io.adafruit.com"
#defineMQTT_PORT1883
#defineMQTT_NAME"E_for_Engineer"//Your adafruit name
#defineMQTT_PASS"633551f4e1204412825a0e3b97b99fde"//Your adafruit AIO key
intled = D7;
WiFiClient client;
Adafruit_MQTT_Client mqtt(&client, MQTT_SERV, MQTT_PORT, MQTT_NAME, MQTT_PASS);
Adafruit_MQTT_Subscribe Lights = Adafruit_MQTT_Subscribe(&mqtt, MQTT_NAME "/f/Lights");
Adafruit_MQTT_Publish LightsStatus = Adafruit_MQTT_Publish(&mqtt, MQTT_NAME "/f/LightsStatus");
voidsetup()
{
  Serial.begin(9600);
  pinMode(LED_BUILTIN, OUTPUT);
  //Connect to WiFi
  Serial.print("\n\nConnecting Wifi>");
  WiFi.begin(WIFI_SSID, WIFI_PASS);
  digitalWrite(LED_BUILTIN, LOW);
  while(WiFi.status() != WL_CONNECTED)
  {
    Serial.print(">");
    delay(50);
  }
  Serial.println("OK!");
  //Subscribe to the Lights topic
  mqtt.subscribe(&Lights);
  pinMode(led, OUTPUT);
  digitalWrite(LED_BUILTIN, HIGH);
  digitalWrite(led, LOW);
}
voidloop()
{
  //Connect/Reconnect to MQTT
  MQTT_connect();
  //Read from our subscription queue until we run out, or
  //wait up to 5 seconds for subscription to update
  Adafruit_MQTT_Subscribe * subscription;
  while((subscription = mqtt.readSubscription(5000)))
  {
    //If we're in here, a subscription updated...
    if(subscription == &Lights)
    {
      //Print the new value to the serial monitor
      Serial.print("Lights: ");
      Serial.println((char*) Lights.lastread);
      //If the new value is  "ON", turn the light on.
      //Otherwise, turn it off.
      if(!strcmp((char*) Lights.lastread, "ON"))
      {
        //active low logic
        digitalWrite(led, HIGH);
        LightsStatus.publish("ON");
      }
      elseif(!strcmp((char*) Lights.lastread, "OFF"))
      {
        digitalWrite(led, LOW);
        LightsStatus.publish("OFF");
      }
      else
      {
        LightsStatus.publish("ERROR");
      }
    }
    else
    {
      //LightsStatus.publish("ERROR");
    }
  }
  //if (!mqtt.ping())
  //{
  //mqtt.disconnect();
  //}
}
voidMQTT_connect()
{
  //// Stop if already connected
  if(mqtt.connected() && mqtt.ping())
  {
    //mqtt.disconnect();
    return;
  }
  int8_tret;
  mqtt.disconnect();
  Serial.print("Connecting to MQTT... ");
  uint8_tretries = 3;
  while((ret = mqtt.connect()) != 0) //connect will return 0 for connected
  {
    Serial.println(mqtt.connectErrorString(ret));
    Serial.println("Retrying MQTT connection in 5 seconds...");
    mqtt.disconnect();
    delay(5000);  //wait 5 seconds
    retries--;
    if(retries == 0)
    {
      ESP.reset();
    }
  }
  Serial.println("MQTT Connected!");
}
