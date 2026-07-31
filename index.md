# Hexapod
My project is the hexapod, and it involves 18 total servos with an acryllic frame. The hexapod can be controlled remotely using wireless modules connected to both it and the controller. A big issue was the fact that the oversized battery would not fit in the battry holder. In order to solve this issue, I had to solder a direct connection for the battery.

You should comment out all portions of your portfolio that you have not completed yet, as well as any instructions:
```HTML 
<!--- This is an HTML comment in Markdown -->
<!--- Anything between these symbols will not render on the published site -->
```

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Brandon K | Valley Christian | Electrical Engineering | Incoming Sophmore |

**Replace the BlueStamp logo below with an image of yourself and your completed project. Follow the guide [here](https://tomcam.github.io/least-github-pages/adding-images-github-pages-site.html) if you need help.**

<img src="IMG_1493.jpeg" style="width: 50%; height: 50%; margin: 0 auto; display: block;"/>
  
# Final Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/QxdbVuQXoKA?si=DdLuYz5-GujQg2SR" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For your final milestone, explain the outcome of your project. Key details to include are:

In the previous milestones, I focused on the base hexapods as well as QOL changes, such as a battery holder. In this milestone, I decided to incorporate a claw that utilizes a micro servo and an ESP32 board. The servo is controlled by the bluetooth connection between my phone and the ESP32 board. The Arduino Nano is powered by a battery pack to make sure it does not weaken the hexapod's servos. Some challenges I faced were the screws and the WLAN module. The screws were tough to put into place, but it worked out in the end. The WLAN module was an issue since every hexapod had the same address. I got around this by making my own hexadecimal address to make sure nobody else could control my hexapod.


# Second Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**
<iframe width="560" height="315" src="https://www.youtube.com/embed/KT_2A_sWXmQ?si=uwqKORgyJwTBR19D" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

For my second milestone, I finished the controller for the hexapod. It functions by using a wireless module to communicate with the hexapod. The hexapod can move in virtually any direction, including up and down. Some challenges I overcame were the lack of nuts. However, I was able to subsititute for these missing parts. For my final milestone, I am planning to use a micro/mini servo to power a small claw. This claw would be strong enough to hold things while the hexapod is in motion.

# First Milestone

**Don't forget to replace the text below with the embedding for your milestone video. Go to Youtube, click Share -> Embed, and copy and paste the code to replace what's below.**

<iframe width="560" height="315" src="https://www.youtube.com/embed/UMam9Bwb8Vw?si=Nnax9ykTupBcM2Op" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

My project consists of 18 servos, 3 of them going onto each leg. It also contains an acryllic frame at the top and bottom to make the base of the hexapod. Some problems I faced so far were with the screws as they were self tapping. This means I had to brute force my way through a lot of them, which took up a big chunk of my time. Another problem I faced was the fact that my battery was not a standard battery, meaning it did not fit in the standard battery holder. To get around this, I directly soldered on the battery. After making sure that the voltage was going through properly, I CADed a holder for the battery pack that would be mounted on top of the hexapod. In the future, I plan to complete the controller as well as to add a claw as my modification.

# Schematics 
<img src="Screenshot2026-07-06 091700.png" style="width: 50%; height: 50%; margin: 0 auto; display: block;"/>


# Code

```c++
#ifndef ARDUINO_AVR_UNO
#error Wrong board. Please choose "Arduino/Genuino Uno"
#endif

// Include FNHR (Freenove Hexapod Robot) library
#include <FNHR.h>

FNHRRemote remote;

void setup() {
  remote.Set(0xB2, 0xC3, 0xD4, 0xE5, 0xA1);

  // Start remote
  remote.Start();

}

void loop() {
  // Update remote
  remote.Update();
}

#include <ESP32Servo.h>
#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

Servo myServo;
const int servoPin = 38; // Physical D11 on Nano ESP32

// Standard Nordic UART BLE UUIDs used by the Bluefruit App
#define SERVICE_UUID           "6E400001-B5A3-F393-E0A9-E50E24DCCA9E"
#define CHARACTERISTIC_UUID_RX "6E400002-B5A3-F393-E0A9-E50E24DCCA9E"
#define CHARACTERISTIC_UUID_TX "6E400003-B5A3-F393-E0A9-E50E24DCCA9E"

BLECharacteristic *pTxCharacteristic;
bool deviceConnected = false;

// Custom callbacks to track Bluetooth connection status
class MyServerCallbacks: public BLEServerCallbacks {
    void onConnect(BLEServer* pServer) {
      deviceConnected = true;
    };
    void onDisconnect(BLEServer* pServer) {
      deviceConnected = false;
      // Restart advertising so you can reconnect if it drops
      BLEDevice::startAdvertising(); 
    }
};

// Processes incoming data packets from the Bluefruit App
class MyCallbacks: public BLECharacteristicCallbacks {
    void onWrite(BLECharacteristic *pCharacteristic) {
      std::string rxValue = pCharacteristic->getValue();
      
      if (rxValue.length() > 0) {
        // Validate the Bluefruit Control Pad packet format: !B [ButtonNum] [State]
        if (rxValue[0] == '!' && rxValue[1] == 'B') {
          char buttonNum = rxValue[2]; // '1' through '8'
          char isPressed = rxValue[3]; // '1' = Pressed, '0' = Released
          
          // Target Button 1 (The '1' button or Up Arrow depending on the UI)
          if (buttonNum == '1') { 
            if (isPressed == '1') {
              myServo.write(60); // Move claw to active/closed position
            } else {
              myServo.write(0);   // Return claw to default/open position
            }
          }
        }
      }
}
};

void setup() {
  Serial.begin(115200);

  // Allocate timers and setup servo
  ESP32PWM::allocateTimer(0);
  myServo.setPeriodHertz(50);
  myServo.attach(servoPin, 500, 2400);
  myServo.write(0); // Start in default open position

  // Initialize ESP32 Bluetooth
  BLEDevice::init("ESP32-Claw");

  // Create BLE Server and register connection status callbacks
  BLEServer *pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  // Create Nordic UART Service
  BLEService *pService = pServer->createService(SERVICE_UUID);

  // Create TX Characteristic (For sending data back to phone if needed)
  pTxCharacteristic = pService->createCharacteristic(
                        CHARACTERISTIC_UUID_TX,
                        BLECharacteristic::PROPERTY_NOTIFY
                      );
  pTxCharacteristic->addDescriptor(new BLE2902());

  // Create RX Characteristic (For receiving button presses from phone)
  BLECharacteristic *pRxCharacteristic = pService->createCharacteristic(
                                           CHARACTERISTIC_UUID_RX,
                                           BLECharacteristic::PROPERTY_WRITE
                                         );
  pRxCharacteristic->setCallbacks(new MyCallbacks());

  // Start the service and start broadcasting
  pService->start();
  BLEDevice::startAdvertising();
  Serial.println("Bluetooth active. Awaiting connection from Bluefruit app...");
}

void loop() {
  // Bluetooth relies on the background interrupts and callbacks.
  // Keep the main loop clear so execution doesn't block incoming data.
  delay(10); 
}

}```



# Bill of Materials
Here's where you'll list the parts in your project. To add more rows, just copy and paste the example rows below.
Don't forget to place the link of where to buy each component inside the quotation marks in the corresponding row after href =. Follow the guide [here]([url](https://www.markdownguide.org/extended-syntax/)) to learn how to customize this to your project needs.

| **Part** | **Note** | **Price** | **Link** |
|:--:|:--:|:--:|:--:|z
| Hexapod Kit | Base project | $126.95 | <a href="https://www.amazon.com/Freenove-Raspberry-Crawling-Detailed-Tutorial/dp/B07FLVZ2DN?th=1"> Link </a> |
| Tenergy 7.5 Volt Battery | Used to power the whole hexapod | $39.49 | <a href="https://power.tenergy.com/tenergy-nimh-7-2v-3800mah-battery-pack-w-tamiya-connector-for-rc-cars/?gad_source=1&gad_campaignid=17180860516&gbraid=0AAAAAD_fnYGTDtGje6QeOpAs5nEXq4u6k&gclid=CjwKCAjwmJjSBhB-EiwAkZgxi6lMLvEO2wPKv8e2Ecv_Wq2niAyCdd9d6MZQEQ0SFw0eMbnKqPrliRoCRtEQAvD_BwE"> Link </a> |
