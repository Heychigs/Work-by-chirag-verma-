# Work-by-chirag-verma-
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define SERVICE_UUID        "00000002-0000-0000-FDFD-FDFDFDFDFDFD"
#define CHARACTERISTIC_TEMP_UUID  "2A6E"
#define CHARACTERISTIC_HUM_UUID  "2A6F"

BLECharacteristic *pCharacteristicTemp;
BLECharacteristic *pCharacteristicHum;

float temperature = 25.0;
float humidity = 50.0;

void setup() {
  Serial.begin(115200);
  BLEDevice::init("ESP32_BLE_Temperature_Humidity");
  
  BLEServer *pServer = BLEDevice::createServer();
  BLEService *pService = pServer->createService(SERVICE_UUID);

  pCharacteristicTemp = pService->createCharacteristic(
                                         CHARACTERISTIC_TEMP_UUID,
                                         BLECharacteristic::PROPERTY_READ |
                                         BLECharacteristic::PROPERTY_NOTIFY
                                       );

  pCharacteristicHum = pService->createCharacteristic(
                                         CHARACTERISTIC_HUM_UUID,
                                         BLECharacteristic::PROPERTY_READ |
                                         BLECharacteristic::PROPERTY_NOTIFY
                                       );

  pCharacteristicTemp->addDescriptor(new BLE2902());
  pCharacteristicHum->addDescriptor(new BLE2902());

  pService->start();
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->addServiceUUID(SERVICE_UUID);
  pAdvertising->setScanResponse(true);
  pAdvertising->setMinPreferred(0x06);  // functions that help with iPhone connections issue
  pAdvertising->setMinPreferred(0x12);
  BLEDevice::startAdvertising();
  Serial.println("Waiting for a client connection to notify...");
}

void loop() {
  // Simulate sensor readings
  temperature = random(200, 300) / 10.0;  // Simulated temperature between 20.0 and 30.0
  humidity = random(400, 600) / 10.0;     // Simulated humidity between 40.0 and 60.0

  char tempStr[6];
  dtostrf(temperature, 4, 2, tempStr);
  pCharacteristicTemp->setValue(tempStr);
  pCharacteristicTemp->notify();

  char humStr[6];
  dtostrf(humidity, 4, 2, humStr);
  pCharacteristicHum->setValue(humStr);
  pCharacteristicHum->notify();

  Serial.print("Temperature: ");
  Serial.print(tempStr);
  Serial.print(" °C, Humidity: ");
  Serial.print(humStr);
  Serial.println(" %");

  delay(5000);  // Notify every 5 seconds
}
