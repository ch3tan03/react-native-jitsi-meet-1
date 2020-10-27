# @smartness-community/react-native-jitsi-meet

Jitsi wrapper inspired by [https://github.com/skrafft/react-native-jitsi-meet](https://github.com/skrafft/react-native-jitsi-meet)

## Installation

```sh
npm install @smartness-community/react-native-jitsi-meet --registry=https://npm.pkg.github.com
```

## Android Configuration

Add the `com.ko.jitsimeet.activities.JitsiMeetActivity` in `android/app/src/main/AndroidManifest.xml` with `singleTask` launch mode.

```xml
<application
      ....
>
<activity
        android:launchMode="singleTask"
        android:name="com.ko.jitsimeet.activities.JitsiMeetActivity"
/>
</application>

```

Add the maven repository url `https://github.com/@smartness-community/jitsi-maven-repository/raw/ko-master/releases` in `android/build.gradle`

```gradle
allprojects {
    repositories {
        mavenLocal()
        maven {
            // All of React Native (JS, Obj-C sources, Android binaries) is installed from npm
            url("$rootDir/../node_modules/react-native/android")
        }
        maven {
            // Android JSC is installed from npm
            url("$rootDir/../node_modules/jsc-android/dist")
        }
        maven { // <---- Add this block
            url "https://github.com/smartness-community/jitsi-meet-android-sdk-releases/raw/master/releases"
        }

        google()
        jcenter()
        maven { url 'https://www.jitpack.io' }
    }
}
```

Update the `xmlns:tools="http://schemas.android.com/tools"` directive into the `AndroidManifest.xml` file 

```js
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
  xmlns:tools="http://schemas.android.com/tools"
  package="com.your.package">
.....
</manifest>
```

In `android/settings.gradle` add 

```groovy
include ':react-native-jitsi-meet'
project(':react-native-jitsi-meet').projectDir = new File(rootProject.projectDir, '../node_modules/@smartness-community/react-native-jitsi-meet/android')
```

In `android/app/build.gradle` some dependencies using the `variant.getRuntimeConfiguration().resolutionStrategy` gradle method 

```groovy
buildTypes {
  ...
}
// applicationVariants are e.g. debug, release
applicationVariants.all { variant ->
    variant.getRuntimeConfiguration().resolutionStrategy {
        variant.getCompileConfiguration().exclude group: 'com.facebook.react', module:'react-native-svg'
        variant.getRuntimeConfiguration().exclude group: 'com.facebook.react', module:'react-native-svg'

        variant.getCompileConfiguration().exclude group: 'com.facebook.react', module:'react-native-community-async-storage'
        variant.getRuntimeConfiguration().exclude group: 'com.facebook.react', module:'react-native-community-async-storage'

        variant.getCompileConfiguration().exclude group: 'com.facebook.react',module:'react-native-community_netinfo'
        variant.getRuntimeConfiguration().exclude group: 'com.facebook.react',module:'react-native-community_netinfo'
    }

    variant.outputs.each { output ->
```

```groovy
implementation(project(':react-native-jitsi-meet')){
  exclude group: 'com.facebook.react', module:'react-native-svg'
}
```


Add the `tools:replace="android:allowBackup"` into the `<application></application>`

```js
<application
    tools:replace="android:allowBackup"
</application>
```
Enable Hermès into `android/app/build.gradle`

```
project.ext.react = [
    enableHermes: true,  // clean and rebuild if changing
]
```

Update the MainApplication class 
```
import androidx.annotation.Nullable;
  ...
    @Override
    protected String getJSMainModuleName() {
      return "index";
    }

    @Override
    protected @Nullable String getBundleAssetName() {
      return "ko.android.bundle";
    }
```

## iOS configuration

Import `JitsiMeetSDK` into `Podfile`

```ruby
pod 'JitsiMeetSDK', :git => 'https://github.com/smartness-community/jitsi-meet-ios-sdk-releases.git'
```

## Basic Usage

```js

import * as React from 'react';
import { StyleSheet, View, Text, TouchableOpacity, TextInput } from 'react-native';
import JitsiMeet, {UserInfo} from '@smartness-community/react-native-jitsi-meet';

export default function App() {
  const [url, setUrl] = React.useState('https://meet.jit.si/exemple')
  const call = React.useCallback(() => {
    const userInfo : UserInfo = { email: 'john@ko.com', displayName: 'John Doe' }
    JitsiMeet.join(url, userInfo)
  }, [])
  return (
    <View style={styles.container}>
      <TextInput placeholder={url} defaultValue={url} onChangeText={setUrl} />
      <TouchableOpacity onPress={call}>
        <Text>Start Jitsi</Text>
      </TouchableOpacity>
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: 'center',
    justifyContent: 'center',
  },
});

```

## Jitsi API Usage

The start of a Jitsi call is done using the `joinWithFeatures()` method. This method takes as first parameter the information of the calling user and as second (optional) parameter the list of enabled features. 

```ts
import JitsiMeet, { eventEmitter, UserInfo, FeatureFlag, FeatureFlags } from '@smartness-community/react-native-jitsi-meet';

const userInfo: UserInfo = { email: 'john@ko.com', displayName: 'John Doe' }
const features : FeatureFlags = {
  [FeatureFlag.CHAT_ENABLED] : true
}
JitsiMeet.join('https://meet.jit.si/exemple', userInfo, features)
```

It is possible to listen to the events of the Jitsi call (`onConferenceJoined`, `onConferenceTerminated`) by subscribing to the corresponding events.

```js
import JitsiMeet, { eventEmitter } from '@smartness-community/react-native-jitsi-meet';

React.useEffect(() => {
    const eventListener = eventEmitter.addListener('onConferenceTerminated', (url: string, error?: string) => {
      console.log('Conference is over, see you soon !')
    })
    return () => eventListener.remove();
  }, [])
```


## Contributing

TODO

## License

MIT
