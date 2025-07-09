# Overview of HarmonyOS

As the third major mobile operating system following Android and iOS, HarmonyOS has gradually expanded its market share in China and is poised to gain international recognition.

Technically, HarmonyOS offers numerous advantages such as distributed technology, a component-based design approach, and robust ecosystem-building capabilities. These strengths enable HarmonyOS to achieve rapid device connectivity, capability collaboration, and resource sharing across diverse terminal form factors (including smartphones, tablets, PCs, smart screens, and wearables), delivering seamless full-scenario user experiences. Simultaneously, it significantly reduces development complexity and costs for application developers, enabling more efficient app creation with "write once, deploy everywhere" capabilities.

This documentation series serves as your guide and curriculum for HarmonyOS application development, assisting you in creating exceptional HarmonyOS applications!

## Advantages of HarmonyOS

### Distributed Capabilities

HarmonyOS applications fully leverage distributed technology to enable seamless cross-device collaboration. This means applications can operate beyond individual devices, sharing data, functions, and interfaces across multiple devices to deliver continuous and consistent user experiences.


For example, through distributed capabilities, files can be accessed across different devices.
```arkts
import { fileIo as fs } from '@kit.CoreFileKit';
import { common } from '@kit.AbilityKit';
import { BusinessError } from '@kit.BasicServicesKit';
let context = this.getUIContext().getHostContext() as common.UIAbilityContext; 
let pathDir: string = context.distributedFilesDir;
let filePath: string = pathDir + '/test.txt';
try {
  let file = fs.openSync(filePath, fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE);
  console.info('Succeeded in creating.');
  fs.writeSync(file.fd, 'content');
  fs.closeSync(file.fd);
} catch (error) {
  let err: BusinessError = error as BusinessError;
  console.error(`Failed to openSync / writeSync / closeSync. Code: ${err.code}, message: ${err.message}`);
} 
```

### Multi-Device Support

Unlike traditional applications that require device-specific development, HarmonyOS natively supports various device types including smartphones, tablets, TVs, and wearables. Developers can achieve cross-device adaptation and operation through a single development effort, greatly enhancing efficiency.

In the module.json file, you can declare the permitted device types for the application, enabling "develop once, deploy across multiple platforms/segments."
```
{
  "module": {
    "name": "entry",
    "type": "entry",
    "description": "$string:module_desc",
    "mainElement": "EntryAbility",
    "deviceTypes": [
      "phone",
      "tablet",
      "2in1"
    ]
  }
}
```

### Unified Development Environment

HarmonyOS provides a unified development environment where developers can use the same programming languages and toolchains for application development, reducing learning curves and technical barriers.

### Efficient Resource Scheduling

Through microkernel architecture and distributed soft bus technology, HarmonyOS enables efficient resource scheduling and sharing. The system intelligently allocates resources based on device performance and user needs, ensuring optimal application performance across scenarios.

### Rich Scenario Experiences

HarmonyOS applications support seamless multi-scenario switching, delivering personalized services tailored to actual user contexts. For example:
- In smart home scenarios, users can control appliances via smartphones for intelligent living
- In transportation scenarios, users can enjoy music and navigation services through in-vehicle devices

### Enhanced Security

HarmonyOS implements strict security controls including permission management, data encryption, and security auditing to protect user data privacy and system security.

```
{
  "module": {
    // ...
    "requestPermissions": [
      {
        "name": "ohos.permission.VIBRATE",
        "reason": "$string:vibrate_reason",
        "usedScene": {
          "abilities": [
            "EntryAbility"
          ],
          "when": "always"
        }
      }
    ]
  }
}

```

## Application Types

Beyond traditional installable applications (APPs), HarmonyOS introduces a lightweight application format called **Atomic Services**. These require no installation and emphasize "open-and-use, use-and-go" convenience for immediate access to services.

## Development Languages

HarmonyOS applications currently support development using:

* ArkTS
* Cangjie (Cangjie)
* C++

Third-party frameworks like Flutter and Uniapp are also supported.
