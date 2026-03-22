#include <Bluepad32.h>
//Pin-Definitionen
#define L_EN 27
#define R_EN 26
#define L_PWM 33
#define R_PWM_L 25 // R_PWM des linken BTS7960


#define R_L_EN 14
#define R_R_EN 13
#define R_L_PWM 32
#define R_R_PWM 23


// PWM-Kanal-Konfiguration (LEDC)
#define PWM_FREQ 1000 // Hz
#define PWM_RES 8 // Bit → 0-255
#define CH_L_FWD 0 // Kanal für L_PWM (vorwärts links)
#define CH_L_BWD 1 // Kanal für R_PWM_L (rückwärts links)
#define CH_R_FWD 2 // Kanal für R_L_PWM (vorwärts rechts)
#define CH_R_BWD 3 // Kanal für R_R_PWM (rückwärts rechts)


// Deadzone – hier selbst anpassen (0-511)
#define DEADZONE 10
GamepadPtr myGamepads[BP32_MAX_GAMEPADS];


// This callback gets called any time a new gamepad is connected.
// Up to 4 gamepads can be connected at the same time.
void onConnectedGamepad(GamepadPtr gp) {
bool foundEmptySlot = false;
for (int i = 0; i < BP32_MAX_GAMEPADS; i++) {
if (myGamepads[i] == nullptr) {
Serial.printf("CALLBACK: Gamepad is connected, index=%d\n", i);
// Additionally, you can get certain gamepad properties like:
// Model, VID, PID, BTAddr, flags, etc.
GamepadProperties properties = gp->getProperties();
Serial.printf("Gamepad model: %s, VID=0x%04x, PID=0x%04x\n", gp->getModelName().c_str(), properties.vendor_id,
properties.product_id);
myGamepads[i] = gp;
foundEmptySlot = true;
break;
}
}
if (!foundEmptySlot) {
Serial.println("CALLBACK: Gamepad connected, but could not found empty slot");
}
}


void onDisconnectedGamepad(GamepadPtr gp) {
bool foundGamepad = false;


for (int i = 0; i < BP32_MAX_GAMEPADS; i++) {
if (myGamepads[i] == gp) {
Serial.printf("CALLBACK: Gamepad is disconnected from index=%d\n", i);
myGamepads[i] = nullptr;
foundGamepad = true;
break;
}
}


if (!foundGamepad) {
Serial.println("CALLBACK: Gamepad disconnected, but not found in myGamepads");
}
}


// Arduino setup function. Runs in CPU 1
void setup() {
Serial.begin(115200);
Serial.printf("Firmware: %s\n", BP32.firmwareVersion());
const uint8_t* addr = BP32.localBdAddress();
Serial.printf("BD Addr: %2X:%2X:%2X:%2X:%2X:%2X\n", addr[0], addr[1], addr[2], addr[3], addr[4], addr[5]);


// Setup the Bluepad32 callbacks
BP32.setup(&onConnectedGamepad, &onDisconnectedGamepad);


// "forgetBluetoothKeys()" should be called when the user performs
// a "device factory reset", or similar.
// Calling "forgetBluetoothKeys" in setup() just as an example.
// Forgetting Bluetooth keys prevents "paired" gamepads to reconnect.
// But might also fix some connection / re-connection issues.
BP32.forgetBluetoothKeys();




// Enable-Pins dauerhaft HIGH
pinMode(L_EN, OUTPUT); digitalWrite(L_EN, HIGH);
pinMode(R_EN, OUTPUT); digitalWrite(R_EN, HIGH);
pinMode(R_L_EN, OUTPUT); digitalWrite(R_L_EN, HIGH);
pinMode(R_R_EN, OUTPUT); digitalWrite(R_R_EN, HIGH);


// LEDC-Kanäle binden
ledcSetup(CH_L_FWD, PWM_FREQ, PWM_RES);
ledcSetup(CH_L_BWD, PWM_FREQ, PWM_RES);
ledcSetup(CH_R_FWD, PWM_FREQ, PWM_RES);
ledcSetup(CH_R_BWD, PWM_FREQ, PWM_RES);


ledcAttachPin(L_PWM, CH_L_FWD);
ledcAttachPin(R_PWM_L, CH_L_BWD);
ledcAttachPin(R_L_PWM, CH_R_FWD);
ledcAttachPin(R_R_PWM, CH_R_BWD);
}


// Arduino loop function. Runs in CPU 1
void loop() {
// This call fetches all the gamepad info from the NINA (ESP32) module.
// Just call this function in your main loop.
// The gamepads pointer (the ones received in the callbacks) gets updated
// automatically.
BP32.update();


// It is safe to always do this before using the gamepad API.
// This guarantees that the gamepad is valid and connected.
for (int i = 0; i < BP32_MAX_GAMEPADS; i++) {
GamepadPtr myGamepad = myGamepads[i];


if (myGamepad && myGamepad->isConnected()) {
// There are different ways to query whether a button is pressed.
// By query each button individually:
// a(), b(), x(), y(), l1(), etc...
// ── Ersatz für den alten Block (Zeilen 79-117) im loop() ──


// Linker Motor → axisY() (negativ = Stick oben = vorwärts)
int leftY = -myGamepad->axisY();
// Rechter Motor → axisRY() (negativ = Stick oben = vorwärts)
int rightY = -myGamepad->axisRY();


// ── Linker Motor ──
if (abs(leftY) < DEADZONE) {
// Deadzone: Motor stop
ledcWrite(CH_L_FWD, 0);
ledcWrite(CH_L_BWD, 0);
} else if (leftY < 0) {
// vorwärts: -10 bis -508
int speed = map(abs(leftY), DEADZONE, 508, 0, 255);
speed = constrain(speed, 0, 255);
ledcWrite(CH_L_FWD, speed);
ledcWrite(CH_L_BWD, 0);
} else {
// rückwärts: 10 bis 512
int speed = map(leftY, DEADZONE, 512, 0, 255);
speed = constrain(speed, 0, 255);
ledcWrite(CH_L_FWD, 0);
ledcWrite(CH_L_BWD, speed);
}



// ── Rechter Motor ──
if (abs(rightY) < DEADZONE) {
ledcWrite(CH_R_FWD, 0);
ledcWrite(CH_R_BWD, 0);
} else if (rightY < 0) {
// vorwärts: -10 bis -508
int speed = map(abs(rightY), DEADZONE, 508, 0, 255);
speed = constrain(speed, 0, 255);
ledcWrite(CH_R_FWD, speed);
ledcWrite(CH_R_BWD, 0);
} else {
// rückwärts: 10 bis 512
int speed = map(rightY, DEADZONE, 512, 0, 255);
speed = constrain(speed, 0, 255);
ledcWrite(CH_R_FWD, 0);
ledcWrite(CH_R_BWD, speed);
}


// Another way to query the buttons, is by calling buttons(), or
// miscButtons() which return a bitmask.
// Some gamepads also have DPAD, axis and more.
Serial.printf(
"idx=%d, dpad: 0x%02x, buttons: 0x%04x, axis L: %4d, %4d, axis R: %4d, "
"%4d, brake: %4d, throttle: %4d, misc: 0x%02x\n",
i, // Gamepad Index
myGamepad->dpad(), // DPAD
myGamepad->buttons(), // bitmask of pressed buttons
myGamepad->axisX(), // (-511 - 512) left X Axis
myGamepad->axisY(), // (-511 - 512) left Y axis
myGamepad->axisRX(), // (-511 - 512) right X axis
myGamepad->axisRY(), // (-511 - 512) right Y axis
myGamepad->brake(), // (0 - 1023): brake button
myGamepad->throttle(), // (0 - 1023): throttle (AKA gas) button
myGamepad->miscButtons() // bitmak of pressed "misc" buttons
);


// You can query the axis and other properties as well. See Gamepad.h
// For all the available functions.
}
}


// The main loop must have some kind of "yield to lower priority task" event.
// Otherwise the watchdog will get triggered.
// If your main loop doesn't have one, just add a simple `vTaskDelay(1)`.
// Detailed info here:
// https://stackoverflow.com/questions/66278271/task-watchdog-got-tri ggered-the-tasks-did-not-reset-the-watchdog-in-time


// vTaskDelay(1);
delay(20);
}
