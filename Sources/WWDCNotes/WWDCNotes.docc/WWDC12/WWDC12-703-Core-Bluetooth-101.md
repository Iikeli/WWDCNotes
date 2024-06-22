# Core Bluetooth 101

The CoreBluetooth framework lets your iOS applications communicate with Bluetooth Low Energy devices over a personal area network (PAN). Learn about the Bluetooth LE technology and the APIs we provide for designing apps that connect to a Bluetooth LE peripheral and read, write, and request notification of changes to the characteristics of the peripheral.

@Metadata {
   @TitleHeading("WWDC12")
   @PageKind(sampleCode)
   @CallToAction(url: "http://developer.apple.com/wwdc12/703", purpose: link, label: "Watch Video")

   @Contributors {
      @GitHubUser(mike011)
   }
}



This is an introduction to Bluetooth Low Energy (BLE). This biggest benefit is that the Bluetooth device take sufficiently less energy compared to classic Bluetooth so it can now be powered by a cheap battery. This is accomplished by doing the following 3 things:

1. Less time on the air
2. Less energy when on the air
3. Completely new architecture

Power consumption is about 6 times less then classic, but the max amount of data transferred is over 20 worse.

## Use cases for BLE

- Health Care
- Sports & Fitness
- Security
- Home Automation
- Home Entertainment
- Kids Toys
- Pay Systems
- Time Syncing Services
- Proximity (How close things are, via RSSI)

### Key Terms

### Dual Mode vs Single Mode

- Dual mode can do both Bluetooth Classic and BLE
- Single mode can only do BLE

### Client & Server

- Server has the data
- Client is Central
- Server is Peripheral
- iOS Device can be either central or peripheral
- Peripheral can only connect to one Central
- Central can connect to many Peripherals

### Discovering the Device

- Broadcaster (Central) advertises on 40 different frequencies under 3 channels it's services
- Observer (Peripheral) scans for services
- An advertising interval is the amount of time between packets and the shorter the time the more battery that is used.

### Connecting to the device

- A connection request is sent from the Observer to the Broadcaster. The Broadcaster accepts, then data can be transferred between the two
- There is analogous connection interval which limits the Peripheral to only sending data at certain times.

### Services

- Service is a description of the set of data.
- A service contains multiple characteristics

### Characteristics

- Characteristics are specific values related to the service.

A characteristic contains the following:

- UUID
- value
- properties (read, write)
- client configuration (notifications)
- additional descriptors (you can define)

## Core Bluetooth Principles

- Simple
- Powerful
- Build on Bluetooth 4.0 standard

## Your App

- There nothing in the OS that manages BLE, so your app is responsible for:
 - Discovery
 - Connection Management
 - Data Exchange
 - Device Management

## Where is BLE supported?

 - iPhone 4S
 - Mac mini
 - iPad
 - MacBook
 - iOS Simulator

## Core Bluetooth Objects

### Main Objects

- `CBCentral`
- `CBCentralManager`
- `CBPeripheral`
- `CBPeripheralManager`

### Data Objects

 - CBService
 - CBMutableService
 - CBCharacteristic
 - CBMutableCharacteristic

### Helper Objects

 - CBUUID
 - CBATTRequest

## iOS Backgrounding Modes

1. Event-based Peripherals: something has happened, tell the user
2. Session-based Peripherals: full access to peripherals

## Heart Rate Demo

Found [here](https://developer.apple.com/library/archive/samplecode/HeartRateMonitor/).
