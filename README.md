# Developing Packages & Plugins in Flutter

This documentation is mainly adapted from the [Official Packages & Plugins Documentation](https://flutter.dev/docs/development/packages-and-plugins/developing-packages) by Flutter. Adaptation by Ralfi Salhon.

A minimal Flutter package consists of:
- `pubspec.yaml`: A metadata file that declares the package name, version, author, etc.
- `lib`: A directory that contains the public code in the package, minimally a single `<package-name>.dart` file.

#### Package Types
- **Dart packages**: General packages written in Dart. Example: [`path`](https://pub.dev/packages/path) package.
- **Plugin packages**: A specialized Dart package which contains an API written in Dart code combined with a platform-specific implementation for Android (using Java or Kotlin), and/or for iOS (using ObjC or Swift). Example: [`battery`](https://pub.dev/packages/battery) plugin package.


## Dart Packages
#### 1) Creating the dart package
To create a Dart package, use the **--template=package** flag with **flutter create**.\
 Example: `$ flutter create --template=package hello`\
This creates a package project in the `hello/` folder with the following specialized content:
- `lib/hello.dart`: The Dart code for the package.
- `test/hello_test.dart`: The unit tests for the package.

#### 2) Implementing the package
For pure Dart packages, simply add the functionality inside the main `lib/<package name>.dart` file, or in several files in the `lib` directory.

To test the package, add unit tests in a test directory. [Flutter Unit Testing Docs](https://flutter.dev/docs/cookbook/testing/unit/introduction)



## Plugin Packages
#### 1) Creating the plugin package 
To create a plugin package, use the **--template=plugin** flag with **flutter create**.\
 Example: `flutter create --org com.example --template=plugin hello`\
This creates a plugin project in the `hello/` folder with the following specialized content:
- `lib/hello.dart`: The Dart code for the package.
- `android/src/main/java/com/example/hello/HelloPlugin.java`: The Android platform specific implementation of the plugin API.
- `ios/Classes/HelloPlugin.m`: The iOS platform specific implementation of the plugin API.
- `example/`: A Flutter app that depends on the plugin, and illustrates how to use it.

By default, the plugin project uses **Objective-C** for iOS code and **Java** for Android code. If you prefer Swift or Kotlin, you can specify the iOS language using `-i` and/or the Android language using `-a`.\
Example: `$ flutter create --template=plugin -i swift -a kotlin hello`

#### 2) Implementing the package

##### 2a) Locate your main .dart file
Open the main `hello/` folder. Locate the file `lib/hello.dart`.

##### 2b) Add Android platform code (.java/.kt)
**Test that your starter code builds:**\
Navigate to the example folder and run `flutter build apk`.\
Example: `cd hello/example && flutter build apk`

Next,
1) Launch Android Studio
2) Select **Import project** in **Welcome to Android Studio** dialog, or select **File > New > Import Project…** in the menu, and select the `hello/example/android/build.gradle` file.
3) In the **Gradle Sync** dialog, select **OK**.
4) In the **Android Gradle Plugin Update** dialog, select **Don’t remind me again for this project**.

The Android platform code of your plugin is located in `hello/java/com.example.hello/HelloPlugin`.\
You can run the example app from Android Studio by pressing the ▶ button.


##### 2c) Add iOS platform code (.h/.m/.swift)
**Test that your starter code builds:**\
Navigate to the example folder and run `flutter build apk`.\
Example: `cd hello/example; flutter build ios --no-codesign`

Next,
1) Launch Xcode
2) Select **File > Open**, and select the `hello/example/ios/Runner.xcworkspace` file.

The iOS platform code of your plugin is located in `Pods/Development Pods/hello/Classes/` in the Project Navigator.\
You can run the example app from Xcode by pressing the ▶ button.


##### 2d) Connect the API and the platform code
Adapted from [flutter.dev/docs/development/platform-integration/platform-channels](https://flutter.dev/docs/development/platform-integration/platform-channels)

Flutter’s platform-specific API support does not rely on code generation, but rather on a flexible message passing style:

- The Flutter portion of the app sends messages to its host, the iOS or Android portion of the app, over a platform channel.
- The host listens on the platform channel, and receives the message. It then calls into any number of platform-specific APIs — using the native programming language — and sends a response back to the client, the Flutter portion of the app. [Platform Channels Image](https://flutter.dev/images/PlatformChannels.png)
- Messages and responses are passed asynchronously, to ensure the user interface remains responsive.


##### Platform channel data types support
| Dart | Android | iOS |
| ------ | ------ | ------ |
|null|	null|	nil (NSNull when nested)
|bool|	java.lang.Boolean|	NSNumber numberWithBool:
|int|	java.lang.Integer|	NSNumber numberWithInt
|int|	java.lang.Long|	NSNumber numberWithLong::
|double|	java.lang.Double|	NSNumber numberWithDouble:
|String|	java.lang.String|	NSString
|Uint8List|	byte[]|	FlutterStandardTypedData typedDataWithBytes:
|Int32List|	int[]|	FlutterStandardTypedData typedDataWithInt32:
|Int64List|	long[]|	FlutterStandardTypedData typedDataWithInt64:
|Float64List|	double[]|	FlutterStandardTypedData typedDataWithFloat64:
|List|	java.util.ArrayList|	NSArray
|Map|	java.util.HashMap|	NSDictionary

- [Platform Channel Example Github](https://github.com/flutter/flutter/tree/master/examples/platform_channel)
- [Publishing Flutter Packages](https://flutter.dev/docs/development/packages-and-plugins/developing-packages#publish)
- [Writing Good Dart Documentation](https://dart.dev/guides/language/effective-dart/documentation)
- [Battery Status Example using Platform Channels](https://flutter.dev/docs/development/platform-integration/platform-channels#example-calling-platform-specific-ios-and-android-code-using-platform-channels)

##### Handling package interdependencies

If you are developing a package **hello** that depends on the Dart API exposed by another package, you need to add that package to the dependencies section of your `pubspec.yaml` file. The code below makes the Dart API of the `url_launcher` plugin available to **hello**:

In `hello/pubspec.yaml`:
```
dependencies:
  url_launcher: ^0.4.2
```
You can now import `package:url_launcher/url_launcher.dart` and launch(someUrl) in the Dart code of hello.


##### Android
In `hello/android/build.gradle`:

```java
android {
    // lines skipped
    dependencies {
        provided rootProject.findProject(":url_launcher")
    }
}
```
You can now `import io.flutter.plugins.urllauncher.UrlLauncherPlugin` and access the `UrlLauncherPlugin` class in the source code at `hello/android/src`.

##### iOS
In `hello/ios/hello.podspec`:

```python
Pod::Spec.new do |s|
  # lines skipped
  s.dependency 'url_launcher'
```
You can now `#import "UrlLauncherPlugin.h"` and access the `UrlLauncherPlugin` class in the source code at `hello/ios/Classes`.

***
Notes from [Writing a Good Flutter Plugin](https://medium.com/flutter/writing-a-good-flutter-plugin-1a561b986c9c) on Medium:

##### API design:
```js
class StoragePlugin {
  /// Reads a string
  Future<String> getString(String key) async {}
  /// Writes a string
  Future<void> setString(String key, String value) async {}
}
```

##### Avoid platform-specific API methods
```js
// Bad code 🚫
if (Platform.isIos) {
  myPlugin.doIOSThing();
} else if (Platform.isAndroid) {
  myPlugin.doAndroidThing();
} 

// Good code 🎉
myPlugin.doThing();
```

##### Avoid writing static (or global) methods
```js
// Bad code 🚫
Future<User> authenticate() async {
  // Some code
}

// Good code 🎉
class AuthenticatePlugin {
  Future<User> authenticate() async {
    // Some code
  }
}
```
***

### Publishing packages

Once you have implemented a package, you can publish it on the [Pub site](https://pub.dev/), so that other developers can easily use it.

run the dry-run command to see if everything passes analysis:
```
flutter pub pub publish --dry-run
```

Finally, run the actual publish command:
```
flutter pub pub publish
```
For details on publishing, see the [publishing docs](https://dart.dev/tools/pub/publishing) for the Pub site.
