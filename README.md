## Table of contents

* [[Setup] Create the calculator folder and the package.json](#setup)
* [[Native Module] Create the JS import](#js-import)
* [[Native Module] Create the iOS implementation](#ios-native)
* [[Native Module] Create the Android implementation](#android-native)
* [[Native Module] Test The Native Module](#test-native)
* [[TurboModule] Add the JavaScript specs](#js-spec)
* [[TurboModule] Set up CodeGen - iOS](#ios-codegen)
* [[TurboModule] Set up CodeGen - Android](#android-codegen)
* [[TurboModule] Set up podspec file](#ios-autolinking)

## Steps

### <a name="setup" />[[Setup] Create the calculator folder and the package.json](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. `mkdir calculator`
1. `touch calculator/package.json`
1. Paste the following code into the `package.json` file
```json
{
    "name": "calculator",
    "version": "0.0.1",
    "description": "Showcase Turbomodule with backward compatibility",
    "react-native": "src/index",
    "source": "src/index",
    "files": [
        "src",
        "android",
        "ios",
        "calculator.podspec",
        "!android/build",
        "!ios/build",
        "!**/__tests__",
        "!**/__fixtures__",
        "!**/__mocks__"
    ],
    "keywords": ["react-native", "ios", "android"],
    "repository": "https://github.com/<your_github_handle>/calculator",
    "author": "<Your Name> <your_email@your_provider.com> (https://github.com/<your_github_handle>)",
    "license": "MIT",
    "bugs": {
        "url": "https://github.com/<your_github_handle>/calculator/issues"
    },
    "homepage": "https://github.com/<your_github_handle>/calculator#readme",
    "devDependencies": {},
    "peerDependencies": {
        "react": "*",
        "react-native": "*"
    }
}
```

### <a name="js-import" />[[Native Module] Create the JS import](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. `mkdir calculator/src`
1. `touch calculator/src/index.js`
1. Paste the following content into the `index.js`
```js
// @flow
import { NativeModules } from 'react-native'

export default NativeModules.Calculator;
```

### <a name="ios-native" />[[Native Module] Create the iOS implementation](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. `mkdir calculator/ios`
1. Create an `ios/RNCalculator.h` file and fill it with the following code:
    ```objc
    #import <Foundation/Foundation.h>
    #import <React/RCTBridgeModule.h>

    @interface RNCalculator : NSObject <RCTBridgeModule>

    @end
    ```
1. Create an `ios/RNCalculator.m` file and replace the code with the following:
    ```objective-c
    #import "RNCalculator.h"

    @implementation RNCalculator

    RCT_EXPORT_MODULE(Calculator)

    RCT_REMAP_METHOD(add, addA:(NSInteger)a
                            andB:(NSInteger)b
                    withResolver:(RCTPromiseResolveBlock) resolve
                    withRejecter:(RCTPromiseRejectBlock) reject)
    {
        NSNumber *result = [[NSNumber alloc] initWithInteger:a+b];
        resolve(result);
    }

    @end
    ```
1. In the `calculator` folder, create a `calculator.podspec` file
1. Copy this code in the `podspec` file
```ruby
require "json"

package = JSON.parse(File.read(File.join(__dir__, "package.json")))

Pod::Spec.new do |s|
  s.name            = "calculator"
  s.version         = package["version"]
  s.summary         = package["description"]
  s.description     = package["description"]
  s.homepage        = package["homepage"]
  s.license         = package["license"]
  s.platforms       = { :ios => "11.0" }
  s.author          = package["author"]
  s.source          = { :git => package["repository"], :tag => "#{s.version}" }

  s.source_files    = "ios/**/*.{h,m,mm,swift}"

  s.dependency "React-Core"
end
```

### <a name="android-native" />[[Native Module] Create the Android implementation](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. Create a folder `calculator/android`
1. Create a file `calculator/android/build.gradle` and add this code:
    ```js
    buildscript {
        ext.safeExtGet = {prop, fallback ->
            rootProject.ext.has(prop) ? rootProject.ext.get(prop) : fallback
        }
        repositories {
            google()
            gradlePluginPortal()
        }
        dependencies {
            classpath("com.android.tools.build:gradle:7.0.4")
        }
    }

    apply plugin: 'com.android.library'

    android {
        compileSdkVersion safeExtGet('compileSdkVersion', 31)

        defaultConfig {
            minSdkVersion safeExtGet('minSdkVersion', 21)
            targetSdkVersion safeExtGet('targetSdkVersion', 31)
        }
    }

    repositories {
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url "$projectDir/../node_modules/react-native/android"
        }
        mavenCentral()
        google()
    }

    dependencies {
        implementation 'com.facebook.react:react-native:+'
    }
    ```
1. Create a file `calculator/android/src/main/AndroidManifest.xml` and add this code:
    ```xml
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
            package="com.rnnewarchitecturelibrary">
    </manifest>
    ```
1. Create a file `calculator/android/src/main/java/com/rnnewarchitecturelibrary/CalculatorModule.java` and add this code:
    ```java
    package com.rnnewarchitecturelibrary;

    import com.facebook.react.bridge.NativeModule;
    import com.facebook.react.bridge.Promise;
    import com.facebook.react.bridge.ReactApplicationContext;
    import com.facebook.react.bridge.ReactContext;
    import com.facebook.react.bridge.ReactContextBaseJavaModule;
    import com.facebook.react.bridge.ReactMethod;
    import java.util.Map;
    import java.util.HashMap;

    public class CalculatorModule extends ReactContextBaseJavaModule {
        CalculatorModule(ReactApplicationContext context) {
            super(context);
        }

        @Override
        public String getName() {
            return "Calculator";
        }

        @ReactMethod
        public void add(int a, int b, Promise promise) {
            promise.resolve(a + b);
        }
    }
    ```
1. Create a file `calculator/android/src/main/java/com/rnnewarchitecturelibrary/CalculatorPackage.java` and add this code:
    ```java
    package com.rnnewarchitecturelibrary;

    import com.facebook.react.ReactPackage;
    import com.facebook.react.bridge.NativeModule;
    import com.facebook.react.bridge.ReactApplicationContext;
    import com.facebook.react.uimanager.ViewManager;

    import java.util.ArrayList;
    import java.util.Collections;
    import java.util.List;

    public class CalculatorPackage implements ReactPackage {

        @Override
        public List<ViewManager> createViewManagers(ReactApplicationContext reactContext) {
            return Collections.emptyList();
        }

        @Override
        public List<NativeModule> createNativeModules(ReactApplicationContext reactContext) {
            List<NativeModule> modules = new ArrayList<>();
            modules.add(new CalculatorModule(reactContext));
            return modules;
        }

    }
    ```

### <a name="test-native" />[[Native Module] Test The Native Module](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. At the same level of calculator run `npx react-native init OldArchitecture --version 0.70.0-rc.0`
1. `cd OldArchitecture && yarn add ../calculator`
1. Open `OldArchitecture/App.js` file and replace the content with:
    ```js
    /**
     * Sample React Native App
     * https://github.com/facebook/react-native
     *
     * @format
     * @flow strict-local
     */
    import React from 'react';
    import {useState} from "react";
    import type {Node} from 'react';
    import {
    SafeAreaView,
    StatusBar,
    Text,
    Button,
    } from 'react-native';
    import Calculator from 'calculator/src/index'
    const App: () => Node = () => {
    const [currentResult, setResult] = useState<number | null>(null);
    return (
        <SafeAreaView>
        <StatusBar barStyle={'dark-content'}/>
        <Text style={{marginLeft:20, marginTop:20}}>3+7={currentResult ?? "??"}</Text>
        <Button title="Compute" onPress={async () => {
            const result = await Calculator.add(3, 7);
            setResult(result);
        }} />
        </SafeAreaView>
    );
    };
    export default App;
    ```
1. To run the App on iOS, install the dependencies: `cd ios && bundle install && bundle exec pod install && cd ..`
1. Run the app
    1. if using iOS: `npx react-native run-ios`
    1. if using Android: `npx react-native run-android`
1. Click on the `Compute` button and see the app working

**Note:** OldArchitecture app has not been committed not to pollute the repository.

### <a name="js-spec" />[[TurboModule] Add the JavaScript specs](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. `touch calculator/src/NativeCalculator.js`
1. Paste the following code:
    ```ts
    // @flow
    import type { TurboModule } from 'react-native/Libraries/TurboModule/RCTExport';
    import { TurboModuleRegistry } from 'react-native';

    export interface Spec extends TurboModule {
        // your module methods go here, for example:
        add(a: number, b: number): Promise<number>;
    }
    export default (TurboModuleRegistry.get<Spec>(
        'Calculator'
    ): ?Spec);
    ```

### <a name="ios-codegen" />[[TurboModule] Set up CodeGen - iOS](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. Open the `calculator/package.json`
1. Add the following snippet at the end of it:
    ```json
    ,
    "codegenConfig": {
        "libraries": [
            {
            "name": "RNCalculatorSpec",
            "type": "modules",
            "jsSrcsDir": "src"
            }
        ]
    }
    ```

### <a name="android-codegen" />[[TurboModule] Set up CodeGen - Android](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. Open the `calculator/android/build.gradle` file and update the code as follows:
    ```diff
    + def isNewArchitectureEnabled() {
    +    return project.hasProperty("newArchEnabled") && project.newArchEnabled == "true"
    +}

    apply plugin: 'com.android.library'
    +if (isNewArchitectureEnabled()) {
    +    apply plugin: 'com.facebook.react'
    +}

    // ... other parts of the build file

    dependencies {
        implementation 'com.facebook.react:react-native:+'
    }

    + if (isNewArchitectureEnabled()) {
    +     react {
    +         jsRootDir = file("../src/")
    +         libraryName = "calculator"
    +         codegenJavaPackageName = "com.rnnewarchitecturelibrary"
    +     }
    + }

### <a name="ios-autolinking" />[[TurboModule] Set up `podspec` file](https://github.com/cipolleschi/RNNewArchitectureLibraries/commit/)

1. Open the `calculator/calculator.podspec` file
1. Before the `Pod::Spec.new do |s|` add the following code:
    ```ruby
    folly_version = '2021.07.22.00'
    folly_compiler_flags = '-DFOLLY_NO_CONFIG -DFOLLY_MOBILE=1 -DFOLLY_USE_LIBCPP=1 -Wno-comma -Wno-shorten-64-to-32'
    ```
1. Before the `end ` tag, add the following code
    ```ruby
    # This guard prevent to install the dependencies when we run `pod install` in the old architecture.
    if ENV['RCT_NEW_ARCH_ENABLED'] == '1' then
        s.compiler_flags = folly_compiler_flags + " -DRCT_NEW_ARCH_ENABLED=1"
        s.pod_target_xcconfig    = {
            "HEADER_SEARCH_PATHS" => "\"$(PODS_ROOT)/boost\"",
            "CLANG_CXX_LANGUAGE_STANDARD" => "c++17"
        }

        s.dependency "React-Codegen"
        s.dependency "RCT-Folly", folly_version
        s.dependency "RCTRequired"
        s.dependency "RCTTypeSafety"
        s.dependency "ReactCommon/turbomodule/core"
    end
