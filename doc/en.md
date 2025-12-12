> Template version: v0.2.2

<p align="center">
  <h1 align="center"> <code>react-native-permissions</code> </h1>
</p>
<p align="center">
    <a href="https://github.com/zoontek/react-native-permissions">
        <img src="https://img.shields.io/badge/platforms-android%20|%20ios%20|%20windows%20|%20harmony%20-lightgrey.svg" alt="Supported platforms" />
    </a>
    <a href="https://github.com/zoontek/react-native-permissions/blob/master/LICENSE">
        <img src="https://img.shields.io/badge/license-MIT-green.svg" alt="License" />
    </a>
</p>

> [!TIP] [GitHub address](https://github.com/react-native-oh-library/react-native-permissions)

## Installation and Usage

Find the matching version information in the release address of a third-party library: [@react-native-oh-tpl/react-native-permissions Releases](https://github.com/react-native-oh-library/react-native-permissions/releases).For older versions that are not published to npm, please refer to the [installation guide](/en/tgz-usage-en.md) to install the tgz package.

Go to the project directory and execute the following instruction:



<!-- tabs:start -->

#### **npm**

```bash
npm install @react-native-oh-tpl/react-native-permissions
```

#### **yarn**

```bash
yarn add @react-native-oh-tpl/react-native-permissions
```

<!-- tabs:end -->

The following code shows the basic use scenario of the repository:

> [!WARNING] When using **import ... from "react-native-permissions"**, a TypeScript type error occurs because the original library does not include the HarmonyOS fields. Specifically, when trying to use **PERMISSIONS.HARMONY.ACCESS_BLUETOOTH**, TypeScript throws an error indicating that the **HARMONY** field is missing. (However, the code will still compile and run correctly.) To resolve this issue and use **PERMISSIONS.HARMONY.ACCESS_BLUETOOTH**, you should instead use **import...from "@react-native-oh-tpl/react-native-permissions"**.

```js
import { ScrollView, StyleSheet, View, Text, Button } from "react-native";
import React from "react";
import RTNPermissions, { Permission } from "@react-native-oh-tpl/react-native-permissions";

const permissionNormal: Permission[] = [
  "ohos.permission.APPROXIMATELY_LOCATION",
  "ohos.permission.CAMERA",
  "ohos.permission.MICROPHONE",
  "ohos.permission.READ_CALENDAR",
  "ohos.permission.WRITE_CALENDAR",
  "ohos.permission.ACTIVITY_MOTION",
  "ohos.permission.READ_HEALTH_DATA",
  "ohos.permission.DISTRIBUTED_DATASYNC",
  "ohos.permission.READ_MEDIA",
  "ohos.permission.MEDIA_LOCATION",
  "ohos.permission.ACCESS_BLUETOOTH",
];

export function PermissionsExample() {
  return (
    <View style={styles.sectionContainer}>
      <Button
        title="Query Camera Permissions"
        onPress={async () => {
          let check = await RTNPermissions.check("ohos.permission.CAMERA");
          console.info("RTNPermissions===== check", check);
        }}
      />
      <Button
        title="Set Camera Permissions"
        onPress={async () => {
          let request = await RTNPermissions.request("ohos.permission.CAMERA");
          console.info("RTNPermissions===== request", request);
        }}
      />
      <Button
        title={"Query Multiple Permissions"}
        onPress={async () => {
          let checkMultiple = await RTNPermissions.checkMultiple(
            permissionNormal
          );
          console.info("RTNPermissions===== checkMultiple", checkMultiple);
        }}
      />
      <Button
        title={"Set Multiple Permissions"}
        onPress={async () => {
          let requestMultiple = await RTNPermissions.requestMultiple(
            permissionNormal
          );
          console.info("RTNPermissions===== requestMultiple", requestMultiple);
        }}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  sectionContainer: {
    marginTop: 32,
    paddingHorizontal: 24,
  },
  view: {
    width: "100%",
    display: "flex",
    flexDirection: "row",
    justifyContent: "space-evenly",
    flexWrap: "wrap",
    margin: 5,
  },
});
```

## Link

Currently, HarmonyOS does not support AutoLink. Therefore, you need to manually configure the linking.

Open the `harmony` directory of the HarmonyOS project in DevEco Studio.

### 1. Adding the overrides Field to oh-package.json5 File in the Root Directory of the Project

```json
{
  ...
  "overrides": {
    "@rnoh/react-native-openharmony" : "./react_native_openharmony"
  }
}
```

### 2. Introducing Native Code

Currently, two methods are available:

Method 1 (recommended): Use the HAR file.

> [!TIP] The HAR file is stored in the `harmony` directory in the installation path of the third-party library.

Open `entry/oh-package.json5` file and add the following dependencies:

```json
"dependencies": {
    "@rnoh/react-native-openharmony": "file:../react_native_openharmony",

    "@react-native-oh-tpl/react-native-permissions": "file:../../node_modules/@react-native-oh-tpl/react-native-permissions/harmony/permissions.har"
  }
```

Click the `sync` button in the upper right corner.

Alternatively, run the following instruction on the terminal:

```bash
cd entry
ohpm install
```

Method 2: Directly link to the source code.

> [!TIP] For details, see [Directly Linking Source Code](/en/link-source-code.md).

### 3. Configuring CMakeLists and Introducing PermissionsPackage

Open `entry/src/main/cpp/CMakeLists.txt` and add the following code:

```diff
project(rnapp)
cmake_minimum_required(VERSION 3.4.1)
set(CMAKE_SKIP_BUILD_RPATH TRUE)
set(RNOH_APP_DIR "${CMAKE_CURRENT_SOURCE_DIR}")
set(NODE_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../node_modules")
+ set(OH_MODULES "${CMAKE_CURRENT_SOURCE_DIR}/../../../oh_modules")
set(RNOH_CPP_DIR "${CMAKE_CURRENT_SOURCE_DIR}/../../../../../../react-native-harmony/harmony/cpp")
set(LOG_VERBOSITY_LEVEL 1)
set(CMAKE_ASM_FLAGS "-Wno-error=unused-command-line-argument -Qunused-arguments")
set(CMAKE_CXX_FLAGS "-fstack-protector-strong -Wl,-z,relro,-z,now,-z,noexecstack -s -fPIE -pie")
set(WITH_HITRACE_SYSTRACE 1) # for other CMakeLists.txt files to use
add_compile_definitions(WITH_HITRACE_SYSTRACE)

add_subdirectory("${RNOH_CPP_DIR}" ./rn)

# RNOH_BEGIN: manual_package_linking_1
add_subdirectory("../../../../sample_package/src/main/cpp" ./sample-package)
+ add_subdirectory("${OH_MODULES}/@react-native-oh-tpl/react-native-permissions/src/main/cpp" ./permissions)
# RNOH_END: manual_package_linking_1

file(GLOB GENERATED_CPP_FILES "./generated/*.cpp")

add_library(rnoh_app SHARED
    ${GENERATED_CPP_FILES}
    "./PackageProvider.cpp"
    "${RNOH_CPP_DIR}/RNOHAppNapiBridge.cpp"
)
target_link_libraries(rnoh_app PUBLIC rnoh)

# RNOH_BEGIN: manual_package_linking_2
target_link_libraries(rnoh_app PUBLIC rnoh_sample_package)
+ target_link_libraries(rnoh_app PUBLIC rnoh_permissions)
# RNOH_END: manual_package_linking_2
```

Open `entry/src/main/cpp/PackageProvider.cpp` and add the following code:

```diff
#include "RNOH/PackageProvider.h"
#include "SamplePackage.h"
+ #include "PermissionsPackage.h"

using namespace rnoh;

std::vector<std::shared_ptr<Package>> PackageProvider::getPackages(Package::Context ctx) {
    return {
      std::make_shared<SamplePackage>(ctx),
+     std::make_shared<PermissionsPackage>(ctx)
    };
}
```

### 4.Introducing PermissionsPackage to ArkTS

Open the `entry/src/main/ets/RNPackagesFactory.ts` file and add the following code:

```diff
  ...
+ import {PermissionsPackage} from '@react-native-oh-tpl/react-native-permissions/ts';

export function createRNPackages(ctx: RNPackageContext): RNPackage[] {
  return [
+   new PermissionsPackage(ctx),
  ];
}
```

### 5.Running

Click the `sync` button in the upper right corner.

Alternatively, run the following instruction on the terminal:

```bash
cd entry
ohpm install
```

Then build and run the code.

## Constraints

### Compatibility

To use this repository, you need to use the correct React-Native and RNOH versions. In addition, you need to use DevEco Studio and the ROM on your phone.

Check the release version information in the release address of the third-party library: [@react-native-oh-tpl/react-native-permissions Releases](https://github.com/react-native-oh-library/react-native-permissions/releases)

### Applying for and Using a Permission

You need to determine whether your application needs related permissions before accessing data or performing an operation. If permissions are required, you must request the permissions in the application installation package.

Determine whether the required permissions need user authorization. If yes, display a dialog box dynamically to request user authorization.

After the user grants the permissions, the application can access the data or perform the operation.

```
 ┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
 ┃ check(ohos.permission.CAMERA) ┃
 ┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
                │                                ┌─────────────────────┐
     Is the feature available ──────── NO ─────▶ │ RESULTS.UNAVAILABLE │
        on this device ?                         └─────────────────────┘
                │
               YES
                │                                ┌─────────────────┐
        Is the permission ─────────── YES ─────▶ │ RESULTS.GRANTED │
        already granted ?                        └─────────────────┘
                │
                NO
                │
  Is the permission requestable ?
                │
               YES
                │
                ▼
       ┌────────────────┐
       │ RESULTS.DENIED │
       └────────────────┘
                │
                ▼
┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
┃ request(ohos.permission.CAMERA) ┃◀—————————————————————┐
┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛                      │
                │                                       YES
                │                                        │
      Did the user see and ──────── NO ──────── Is the permission
      accept the request ?                      still requestable ?
                │                                        │
               YES                                       NO
                │                                        │
                ▼                                        ▼
        ┌─────────────────┐                     ┌─────────────────┐
        │ RESULTS.GRANTED │                     │ RESULTS.BLOCKED │
        └─────────────────┘                     └─────────────────┘
```

### Required Permissions

You need to declare the permissions in `entry/src/main/module.json5` and create the reason string value.

```
"requestPermissions": [
      {
        "name" : "ohos.permission.CAMERA",
        "reason": "$string:reason",
        "usedScene": {
          "abilities": [
            "FormAbility"
          ],
          "when":"inuse"
        }
      },
      {
        "name" : "ohos.permission.ACCESS_BLUETOOTH",
        "reason": "$string:reason",
        "usedScene": {
          "abilities": [
            "FormAbility"
          ],
          "when":"always"
        }
      }
]
```

How to set:

    Keep the sentence concise without redundant separators.
    
    Recommended sentence pattern: Used for something/Used to do something/Used for doing something.
    
    Example: Used for code scanning and photographing.

```
    {
      "name": "reason",
      "value": "Allows an application to use the camera."
    },
```

### Levels of Permissions

The permissions available to applications vary with the APL. The permission levels include the following in ascending order of seniority.

- **normal**

  The **normal** permission allows access to common system resources beyond the default rules. Access to these resources (including data and functions) has minor risks on user privacy and other applications.

  The permissions of this level are available to applications of the **normal** or higher APL.

- **system_basic**

  The **system_basic** permission allows access to resources related to basic OS services. The basic services are basic functions provided or preconfigured by the system, such as system settings and identity authentication. Access to these resources may have considerable risks to user privacy and other applications.

  The permissions of this level are available only to applications of the **system_basic** or **system_core** APL.

```
normal permission list
    'ohos.permission.APPROXIMATELY_LOCATION',
    'ohos.permission.LOCATION_IN_BACKGROUND'
    'ohos.permission.LOCATION'
    'ohos.permission.CAMERA',
    'ohos.permission.MICROPHONE',
    'ohos.permission.READ_CALENDAR',
    'ohos.permission.WRITE_CALENDAR',
    'ohos.permission.ACTIVITY_MOTION',
    'ohos.permission.READ_HEALTH_DATA',
    'ohos.permission.DISTRIBUTED_DATASYNC',
    'ohos.permission.READ_MEDIA',
    'ohos.permission.MEDIA_LOCATION',
    'ohos.permission.ACCESS_BLUETOOTH'

system_basic permission list
    'ohos.permission.READ_WHOLE_CALENDAR'
    'ohos.permission.WRITE_WHOLE_CALENDAR'
    'ohos.permission.ANSWER_CALL'
    'ohos.permission.MANAGE_VOICEMAIL'
    'ohos.permission.READ_CONTACTS'
    'ohos.permission.WRITE_CONTACTS'
    'ohos.permission.READ_CALL_LOG'
    'ohos.permission.WRITE_CALL_LOG'
    'ohos.permission.READ_CELL_MESSAGES'
    'ohos.permission.READ_MESSAGES'
    'ohos.permission.RECEIVE_MMS'
    'ohos.permission.RECEIVE_SMS'
    'ohos.permission.RECEIVE_WAP_MESSAGES'
    'ohos.permission.SEND_MESSAGES'
    'ohos.permission.WRITE_AUDIO'
    'ohos.permission.READ_AUDIO'
    'ohos.permission.READ_DOCUMENT'
    'ohos.permission.WRITE_DOCUMENT'
    'ohos.permission.WRITE_IMAGEVIDEO'
    'ohos.permission.READ_IMAGEVIDEO'
    'ohos.permission.GET_INSTALLED_BUNDLE_LIST'

```

Note:
**ohos.permission.LOCATION_IN_BACKGROUND** allows an app running in the background to obtain the device location.
For security purposes, this permission cannot be granted to applications in a dialog box. If an application needs this permission, direct the user to manually grant this permission on the **Settings** screen.
Request process:
Request the foreground location permissions in the dialog box. You can request either of the following permissions:
Request **ohos.permission.APPROXIMATELY_LOCATION**.
Request **ohos.permission.APPROXIMATELY_LOCATION** and **ohos.permission.LOCATION**.
After the user grants the foreground location permissions, display a message to direct the user to go to the **Settings** screen to grant the **ohos.permission.LOCATION_IN_BACKGROUND** permission.
The permission is granted to the application if the user selects **Always allow** on the **Settings** screen.

## API

> [!TIP] The **Platform** column indicates the platform where the properties are supported in the original third-party library.

> [!TIP] If the value of **HarmonyOS Support** is **yes**, it means that the HarmonyOS platform supports this property; **no** means the opposite; **partially** means some capabilities of this property are supported. The usage method is the same on different platforms and the effect is the same as that of iOS or Android.

| Name                    | Description            | Type     | Required | Platform    | HarmonyOS Support                                            |
| ----------------------- | ---------------------- | -------- | -------- | ----------- | ------------------------------------------------------------ |
| check                   | Checks a single permission.          | Function | no       | iOS, Android| yes                                                          |
| checkNotifications      | Checks the notification permission.          | Function | no       | iOS, Android| yes (In return values below API 20, 'settings' is empty. API 20 added a method that can return non-empty 'settings', including permissions for sound and vibration notifications.) |
| openSettings            | Opens the settings page.            | Function | no       | iOS, Android| yes                                                          |
| request                 | Sets a single permission.          | Function | no       | iOS, Android| yes                                                          |
| requestNotifications    | Set the notification permission.          | Function | no       | iOS, Android| yes                                                          |
| checkMultiple           | Checks multiple permissions.          | Function | no       | Android     | yes                                                          |
| requestMultiple         | Sets multiple permissions.          | Function | no       | Android     | yes                                                          |
| checkLocationAccuracy   | Checks the device location permission.      | Function | no       | iOS         | no (Use **check()** to query permissions.)                                    |
| requestLocationAccuracy | Requests the permission to access the device location.| Function | no       | iOS         | no (Use **request()** to set permissions.)                                  |
| openPhotoPicker         | Opens the photo picker.          | Function | no       | iOS         | yes (For iOS, the API can be called only when the `PhotoLibrary` permission is `limited`. For HarmonyOS, the API can be directly called without any permission.)|

## Known Issues

## Others

## License

This project is licensed under [The MIT License (MIT)](https://github.com/zoontek/react-native-permissions/blob/master/LICENSE).
