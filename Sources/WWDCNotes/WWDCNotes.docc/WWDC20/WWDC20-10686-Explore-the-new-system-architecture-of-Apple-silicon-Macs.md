# Explore the new system architecture of Apple silicon Macs

Discover how Macs with Apple silicon will deliver modern advantages using Apple's System-on-Chip (SoC) architecture. Leveraging a unified memory architecture for CPU and GPU tasks, Mac apps will see amazing performance benefits from Apple silicon tuned frameworks such as Metal and Accelerate.  Learn about new features and changes coming to boot and security, and how these may affect your applications.

@Metadata {
   @TitleHeading("WWDC20")
   @PageKind(sampleCode)
   @CallToAction(url: "https://developer.apple.com/wwdc20/10686", purpose: link, label: "Watch Video (23 min)")

   @Contributors {
      @GitHubUser(zntfdr)
   }
}



## Architecture: Intel vs. Mac

- Intel-based architecture is composed of several, separated components such as multicore CPUs, discrete GPUs, T2
- Apple Silicon has all its components in a SoC (System on a Chip): building everything into one chip gives the system a shared memory architecture
- SoC CPU and GPU work in the same memory, and data can be shared efficiently, without need to copy data over a PCIe bus
- By using Apple Silicon, new macs gain powerful components present on iDevices such as specialized video encoder and decoder, Neural Engine, Machine Learning Accelerators
- intel-based macs CPUs have multiple cores with similar performance (a.k.a. _symmetric_ cores)
- Apple Silicon has multiple performance cores and other power efficient cores (a.k.a. _asymmetric cores_)

### AMP (Asymmetric MultiProcessing)

All cores support the same architectural set of features, and support the same software

### How to take advantage of the new architecture

All modern frameworks have been adopted to take advantage of the new architecture, there are no API changes specifically for Apple Silicon.

- Unified memory architecture: use `Metal`
- Video encoder and decoder: use `AVFoundation` and `VideoToolbox`
- Neural Engine: use `CoreML`
- Machine Learning Accelerators: use `CoreML`, `Accelerate`, `Compression`, `simd`
- Asymmetric multiprocessing: use `QoS` and `GCD`

## Security features

Apple Silicon brings all the iDevices security features to macOS:

- [Write XOR execute (W^X)][JIT]
- Kernel Integrity Protection
- Pointer authentication
- Device isolation

### Write XOR execute (W^X)

- Memory pages cannot be both writable and executable at the same time.
- Use `pthread_jit_write_protect_np` for fast switching between RW and RX permissions
- Per-thread permission to support multi-threaded JITs

### Kernel Integrity Protection

- Apple Silicon has hardware support in the memory controller that enforces kernel immutability
- Once the kernel has been loaded into memory, Kernel Integrity Protection makes sure that Kernel pages cannot be modified, or new pages made executable

### Pointer authentication

- Pointer authentication guards against misuse of pointers and prevent memory attacks.
- Enabled for: Kernel, System applications, System services

### Device isolation

On Apple Silicon all devices use a separate [IOMMU][IOMMU], this restricts devices to memory they're only intended to (Intel macs have a shared memory for all devices)

## Rosetta

Rosetta runs:

- macOS/Catalyst applications
- games
- Web browsers
- JIT compilers
- Metal directly on Apple GPU
- Core ML with Neural Engine

There are differences between processes running on a Intel- and Apple Silicon-based Macs:

- Memory pages sizes
- TSO memory ordering
- sleep time
- Floating point NaN, denormal handling

Rosetta will make sure that Intel-apps will see the architecture they expect

## Boot and Recovery

### Boot Overview

- On Apple Silicon Macs the boot process is based on iOS and iPadOS Secure Boot
- Secure Boot ensures that each startup component is cryptographically signed by Apple and that the boot happens only after the verification of the chain of trust
- Added support to boot from multiple macOS install from both internal and external volumes
- enable booting any version of macOS signed by Apple

### Start-up and macOS Recovery

- Press and hold Touch ID or Power button to launch startup options, all existing start-up keys are replaced by UI interactions

#### Mac Sharing Mode

- replaces Target Disk Mode
- based on SMB file share
- user authentication is required to enable this service

### Protection layer

- focuses on selecting the security policy for each of the volume
- you can choose between full and reduced security mode.

- Full security mode is the same as security on iPhone (enabled by default)

- External volumes are supported in full security mode

- reduced security mode provides flexibility and configurability of your mac

- reduced security lets you run any version of macOS (including versions no longer signed by Apple)
- reduced security lets you install notarized 3rd party kernel extensions

- you can configure the security of your mac via [`csrutil(1)`][csrutil]

- Intel based macs have a system-wide security policy
- Apple Silicon macs have a per-OS security policy

- Login
 - CCID and PIV-compatible
 - VoiceOvere support

- Apple Silicon support Secure hibernation:
 -  Full at-rest protection
 - Integrity and anti-replay protection

## Recovering your Mac

At high level, the system software is composed by two components:

- macOS
- macOS Recovery

If macOS is not accessible/missing, you can use macOS Recovery to install and restore the system

What happens is even macOS Recovery is not accessible?

- In Intel-based macs you can use Internet recovery
- On Apple Silicon macs you can use System Recovery:
  - Minimal macOS environment
  - Separate hidden container
  - Lets you re-install macOS and macOS Recovery

- You can use Apple Configurator 2 when even the System Recovery is not functional

[csrutil]: https://developer.apple.com/library/archive/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html
[JIT]: https://developer.apple.com/documentation/apple_silicon/porting_just-in-time_compilers_to_apple_silicon
[IOMMU]: https://en.wikipedia.org/wiki/Input–output_memory_management_unit