#include <BLEDevice.h>

#include <BLEUtils.h>

#include <BLEServer.h>



#define SERVICE_UUID        "0000FFE0-0000-1000-8000-00805F9B34FB"

#define CHARACTERISTIC_UUID "0000FFE1-0000-1000-8000-00805F9B34FB"



class MySecurity : public BLESecurityCallbacks {  // BLE 클래스 pin code 함수

  uint32_t onPassKeyRequest(){

    ESP_LOGI(LOG_TAG, "PassKeyRequest");

    return 123456;

  }

  void onPassKeyNotify(uint32_t pass_key){

    ESP_LOGI(LOG_TAG, "The passkey Notify number:%d", pass_key);

  }

  bool onConfirmPIN(uint32_t pass_key){

    ESP_LOGI(LOG_TAG, "The passkey YES/NO number:%d", pass_key);

    vTaskDelay(5000);

    return true;

  }

  bool onSecurityRequest(){

    ESP_LOGI(LOG_TAG, "SecurityRequest");

    return true;

  }

  void onAuthenticationComplete(esp_ble_auth_cmpl_t cmpl){

    ESP_LOGI(LOG_TAG, "Starting BLE work!");

  }

};



class MyCallbacks: public BLECharacteristicCallbacks {

    void onWrite(BLECharacteristic *pCharacteristic) {

      std::string value = pCharacteristic->getValue();

      if (value.length() > 0) {

        for (int i = 0; i < value.length(); i++)

          Serial.print(value[i]);

      }

    }

};



void setup() {

  Serial.begin(115200);

  BLEDevice::init("ESP32");

  BLEDevice::setEncryptionLevel(ESP_BLE_SEC_ENCRYPT);

  BLEDevice::setSecurityCallbacks(new MySecurity());

  BLEServer *pServer = BLEDevice::createServer();

  BLEService *pService = pServer->createService(SERVICE_UUID);
 
  BLECharacteristic *pCharacteristic = pService->createCharacteristic(

                                         CHARACTERISTIC_UUID,

                                         BLECharacteristic::PROPERTY_READ |

                                         BLECharacteristic::PROPERTY_WRITE

                                       );

  pCharacteristic->setValue("Hello World");

  pService->start();

  BLEAdvertising *pAdvertising = pServer->getAdvertising();

  pAdvertising->start();

  BLESecurity *pSecurity = new BLESecurity();  // pin code 설정

  uint8_t rsp_key = ESP_BLE_ENC_KEY_MASK | ESP_BLE_ID_KEY_MASK;

  uint32_t passkey = 123456; // PASS

  uint8_t auth_option = ESP_BLE_ONLY_ACCEPT_SPECIFIED_AUTH_DISABLE;

  esp_ble_gap_set_security_param(ESP_BLE_SM_SET_STATIC_PASSKEY, &passkey, sizeof(uint32_t));

  pSecurity->setAuthenticationMode(ESP_LE_AUTH_REQ_SC_MITM_BOND);

  pSecurity->setCapability(ESP_IO_CAP_OUT);

  pSecurity->setKeySize(16);

  esp_ble_gap_set_security_param(ESP_BLE_SM_ONLY_ACCEPT_SPECIFIED_SEC_AUTH, &auth_option, sizeof(uint8_t));

  pSecurity->setInitEncryptionKey(ESP_BLE_ENC_KEY_MASK | ESP_BLE_ID_KEY_MASK);

  esp_ble_gap_set_security_param(ESP_BLE_SM_SET_RSP_KEY, &rsp_key, sizeof(uint8_t));

  Serial.println("Characteristic defined! Now you can read it in your phone!");

}



void loop() {

  // put your main code here, to run repeatedly:

  delay(2000);

}



