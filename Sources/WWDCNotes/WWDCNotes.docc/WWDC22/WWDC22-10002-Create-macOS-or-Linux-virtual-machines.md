# Create macOS or Linux virtual machines

Learn how you can use the Virtualization framework to quickly create virtual machines on your Mac. We'll show you how to create a virtual Mac and quickly test changes to your app in an isolated environment. We'll also explore how you can install and run full Linux distributions on Apple silicon, and share how you can take advantage of Rosetta 2 to run x86-64 Linux binaries.

@Metadata {
   @TitleHeading("WWDC22")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc22/10002", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



> [Sample app](https://developer.apple.com/documentation/virtualization/running_gui_linux_in_a_virtual_machine_on_a_mac)

## The stack that enables virtualization

- hardware
  - Apple silicon has special hardware that enables the virtualization of CPUs and memory
  - can run multiple OSes on top of a single SoC

- macOS kernel
  - software that takes advantage of the hardware
  - no need to write kernel extensions (or KEXTs)

- [Hypervisor Framework][hy]
  - low-level API that lets you virtualize CPUs and memory
  - used to access the macOS kernel virtualization capabilities from your application

- [Virtualization Framework][vi]
  - high-level API that lets you virtualize machines
  - enables the creation of virtual machines running macOS on Apple silicon, or Linux on both Apple silicon and Intel

## Virtualization Framework

When using Virtualization framework, we'll deal with two kinds of objects: 

1. Configuration objects - which define all the properties of our virtual machines
2. Virtual machine objects - abstract virtual machines and how to interact with them

### 1. Configuration objects

- define the hardware
- creating a configuration is like configuring a Mac on the Apple Store (how many CPUs, how much memory, what kind of devices)
  - we can add a display, and we get to see the content
  - we can add a keyboard, and we can type
  - we can add a trackpad, and we can interact with the UI

- done in code:

```swift
var configuration = VZVirtualMachineConfiguration()           // 👈🏻 root object of all configurations
configuration.cpuCount = 4                                    // 👈🏻 4 CPUs
configuration.memorySize = (4 * 1024 * 1024 * 1024) as UInt64 // 👈🏻 4 GB of memory
configuration.storageDevices = [newBlockDevice()]             // 👈🏻 one storage device
configuration.pointingDevices = [newPointingDevice()]         // 👈🏻 4 GB of memory
```

### 2. Virtual machine objects

- running a virtual machine:

```swift
let virtualMachine = VZVirtualMachine(configuration: configuration) // 👈🏻 the configuration from before
try await virtualMachine.start() // 👈🏻 boot it up
```

To interact with the virtual machines, we need to use other objects from the Virtualization framework

- to show a virtual display:

```swift
let virtualMachineView = VZVirtualMachineView() // 👈🏻 just a normal NSView
virtualMachineView.virtualMachine = virtualMachine

... // integrate virtualMachineView in your app to see the content of the virtual machine
```

## Run full OSes in virtual machines

### macOS 

#### Configuration

We need to set two special properties in our configuration to make a Mac virtual machine:

1. Platform
  a. hardware model - specifies which version of the virtual Mac we want.
  b. auxiliary storage - form of non-volatile memory used by the system
  c. machine identifier - unique number representing the machine

2. boot loader - must be macOS boot loader

```swift
// 👇🏻 same as before
var configuration = VZVirtualMachineConfiguration()
configuration.cpuCount = 4
configuration.memorySize = (4 * 1024 * 1024 * 1024) as UInt64
configuration.storageDevices = [newBlockDevice()]
configuration.pointingDevices = [newPointingDevice()]

// 👇🏻 Platform properties
let platform = VZMacPlatformConfiguration() // 👈🏻 platform object

let hardwareModel = VZMacHardwareModel(dataRepresentation: savedHardwareModel)
platform.hardwareModel = hardwareModel! // 👈🏻 hardware model

let auxiliaryStorage = VZMacAuxiliaryStorage(contentsOf: auxiliaryStorageURL)
platform.auxiliaryStorage = auxiliaryStorage // 👈🏻 auxiliary storage

let machineIdentifier = VZMacMachineIdentifier(dataRepresentation: savedIdentifier)
platform.machineIdentifier = machineIdentifier! // 👈🏻 machine identifier

configuration.platform = platform // 👈🏻 we set special properties in our configuration object

//                            👇🏻 Mac Boot loader
configuration.bootLoader = VZMacOSBootLoader()
```

#### How to install macOS into a vm

Three steps:

1. download and restore image (we can choose which maoOS version we want to install)
2. create configuration (compatible with image above)
3. run installer

```swift
// 1. get and restore image

let restoreImage = try await VZMacOSRestoreImage.latestSupported // 👈🏻 gets a restores image object for the latest stable version of macOS
// alternatively we could download from the developer portal

try await download(restoreImage.url)

// 2. create configuration

let requirements = restoreImage.mostFeaturefulSupportedConfiguration
// 👆🏻 this lists us the requirements to run on the current system

guard let requirements = requirements else {
    // No compatible configuration.
    return
}

platform.hardwareModel = requirements.hardwareModel

configuration.cpuCount = requirements.minimumSupportedCPUCount
configuration.memorySize = requirements.minimumSupportedMemorySize

// 3. install macOS
let virtualMachine = VZVirtualMachine(configuration: configuration)

let installer = VZMacOSInstaller(virtualMachine: virtualMachine, restoringFromImageAt: imageURL)
try await installer.install()
```

#### Using your Mac

-  GPU acceleration ([`VZMacGraphicsDisplayConfiguration`][VZMacGraphicsDisplayConfiguration])
  - built-in graphic device that exposes the GPU capabilities to the virtual Mac
  - you can run Metal in the virtual machine, and get great graphics performance in macOS

```swift
let graphicsConfiguration = VZMacGraphicsDeviceConfiguration()
graphicsConfiguration.displays = [
  VZMacGraphicsDisplayConfiguration(widthInPixels: 1920, heightInPixels: 1200, pixelsPerInch: 80)
  //                                  👆🏻 screen size and pixel density
]

configuration.graphicsDevices = [graphicsConfiguration]
```

- virtual trackpad ([`VZMacTrackpadConfiguration`][VZMacTrackpadConfiguration])
  - requires macOS 13 on both host OS and virtual machine

```swift
let trackpad = VZMacTrackpadConfiguration()
configuration.pointingDevices = [trackpad]
```

#### Sharing files between host OS and virtual mac

- new in macOS 13 via Virtio file-system
- you can choose which folders that you want to share with the virtual machine
- any change you make from the host system is instantly reflected within the virtual machine and vice versa

```swift
//                                              👇🏻 path to the directory we want to share
let sharedDirectory = VZSharedDirectory(url: directoryURL, readOnly: false)
let share = VZSingleDirectoryShare(directory: sharedDirectory) // can use VZMultipleDirectoryShare for more directories

let tag = VZVirtioFileSystemDeviceConfiguration.macOSGuestAutomountTag // File system devices are identified by a tag
let sharingDevice = VZVirtioFileSystemDeviceConfiguration(tag: tag)
sharingDevice.share = share

configuration.directorySharingDevices = [sharingDevice]
```

### Linux

#### How to install linux into a vm

Three steps, like IRL:

1. download image
2. put into a usb drive (as a device in the vm)
3. run installer from usb drive
  - done via EFI boot loader
  - takes advantage of EFI boot discovery mechanism, which looks at each drive for one that can be booted
  - once detected, EFI will start the installer from the virtual usb drive

```swift
// 1. download image

let diskImageURL = URL(fileURLWithPath: "linux.iso")

// 2. put into a usb drive

//   👇🏻 A disk image attachment represents a piece of storage that we can attach to a device
let attachment = try! VZDiskImageStorageDeviceAttachment(url: diskImageURL, readOnly: true)
let usbDeviceConfiguration = VZUSBMassStorageDeviceConfiguration(attachment: attachment) // 👈🏻 our usb drive

configuration.storageDevices = [usbDeviceConfiguration, createBlockDevice()]

// 3. run installer from usb drive

let efi = VZEFIBootLoader()
// 👇🏻 EFI requires this non-volatile memory to store information between boots
efi.variableStore = VZEFIVariableStore(creatingVariableStoreAt: storeURL, options: [])
configuration.bootLoader = efi // set EFI boot loader
```

#### Running your vm

- graphics (Virtio GPU 2D)

```swift
let virtioGPU = VZVirtioGraphicsDeviceConfiguration()
virtioGPU.scanouts = [
  VZVirtioGraphicsScanoutConfiguration(widthInPixels: 1280, heightInPixels: 720) // 👈🏻 virtual display
]

configuration.graphicsDevices = [virtioGPU]
```

#### Rosetta 2 (macOS Ventura)

- Linux binaries support
- translates the Linux x86-64 binaries inside your virtual machine.
- useful when developing for x86-64 servers

- vm setup:

```swift
// setting up Rosetta
let rosettaDirectoryShare = try! VZLinuxRosettaDirectoryShare() // 👈🏻 special object
let directorySharingDevice = VZVirtioFileSystemDeviceConfiguration(tag: "RosettaShare")
directorySharingDevice.share = rosettaDirectoryShare

configuration.directorySharingDevices = [directorySharingDevice]
```

- in the vm, run:

```shell
mount -t virtiofs RosettaShare /mnt/Rosetta

sudo /usr/sbin/update-binfmts --install rosetta /mnt/Rosetta/rosetta \
  --magic "\x7fELF\x02\x01\x01\x00\x00\x00\x00\x00\x00\x00\x00\x00\x02\x00\x3e\x00" \
  --mask "\xff\xff\xff\xff\xff\xfe\xfe\x00\xff\xff\xff\xff\xff\xff\xff\xff\xfe\xff\xff\xff" \
  --credentials yes --preserve no --fix-binary yes
```

- `update-binfmts` tells the system to use Rosetta to handle any x86-64 binary
- after running the commands above, every x86-64 binary launched will be translated by Rosetta.

[hy]: https://developer.apple.com/documentation/hypervisor
[vi]: https://developer.apple.com/documentation/virtualization
[VZMacGraphicsDisplayConfiguration]: https://developer.apple.com/documentation/virtualization/vzmacgraphicsdisplayconfiguration
[VZMacTrackpadConfiguration]: https://developer.apple.com/documentation/virtualization/vzmactrackpadconfiguration