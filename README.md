# ML Kit Vision for Firebase with AutoML Vision Edge and Camera Live Streaming Support

![Pub](https://img.shields.io/pub/v/firebase_livestream_ml_vision.svg?color=orange)

A Flutter plugin to use the [ML Kit Vision for Firebase API](https://firebase.google.com/docs/ml-kit/).

For Flutter plugins for other Firebase products, see [FlutterFire.md](https://github.com/flutter/plugins/blob/master/FlutterFire.md).

*Note*: This plugin is still under development, and some APIs might not be available yet. [Feedback](https://github.com/flutter/flutter/issues) and [Pull Requests](https://github.com/flutter/plugins/pulls) are most welcome!

## Usage

To use this plugin, add `firebase_livestream_ml_vision` as a [dependency in your pubspec.yaml file](https://flutter.io/platform-plugins/). You must also configure Firebase for each platform project: Android and iOS (see the example folder or https://codelabs.developers.google.com/codelabs/flutter-firebase/#4 for step by step details).

### AutoML Vision Edge

If you plan to use AutoML Vision Edge to detect labels using a custom model, either download or host the trained model by following [these instructions](https://firebase.google.com/docs/ml-kit/train-image-labeler).

If you downloaded the file, follow the instructions below to enable the plugin.

Unzip the file downloaded, and rename the folder to reflect the model name.

Create a `assets` folder and place the previous folder within it. In `pubspec.yaml` add the appropriate paths:

```
  assets:
   - assets/<foldername>/dict.txt
   - assets/<foldername>/manifest.json
   - assets/<foldername>/model.tflite
```

### Android

Change the minimum Android sdk version to 21 (or higher) in your `android/app/build.gradle` file.

```
minSdkVersion 21
```

If you're using local AutoML VisionEdge Models, include this in your app-level build.gradle file.

```
aaptOptions {
    noCompress "tflite"
}
```

If you're using the on-device `ImageLabeler`, include the latest matching [ML Kit: Image Labeling](https://firebase.google.com/support/release-notes/android) dependency in your app-level build.gradle file.

```
android {
    dependencies {
        // ...

        api 'com.google.firebase:firebase-ml-vision-image-label-model:17.0.2'
    }
}
```

If you receive compilation errors, try an earlier version of [ML Kit: Image Labeling](https://firebase.google.com/support/release-notes/android).

Optional but recommended: If you use the on-device API, configure your app to automatically download the ML model to the device after your app is installed from the Play Store. To do so, add the following declaration to your app's AndroidManifest.xml file:

```xml
<application ...>
  ...
  <meta-data
    android:name="com.google.firebase.ml.vision.DEPENDENCIES"
    android:value="ocr" />
  <!-- To use multiple models: android:value="ocr,label,barcode,face" -->
</application>
```

### iOS

Versions `0.7.0+` use the latest ML Kit for Firebase version which requires a minimum deployment
target of 9.0. You can add the line `platform :ios, '9.0'` in your iOS project `Podfile`.

If you're using one of the on-device APIs, include the corresponding ML Kit library model in your
`Podfile`. Then run `pod update` in a terminal within the same directory as your `Podfile`.

```
pod 'Firebase/MLVisionBarcodeModel'
pod 'Firebase/MLVisionFaceModel'
pod 'Firebase/MLVisionLabelModel'
pod 'Firebase/MLVisionTextModel'
```

Add one row in `ios/Runner/Info.plist`:

The key `Privacy - Camera Usage Description` and a usage description.

In text format:

```
<key>NSCameraUsageDescription</key>
<string>Can I use the camera please?</string>

```

## Using an ML Vision Detector

### 1. Create a Camera View

```dart

import 'package:firebase_livestream_ml_vision/firebase_livestream_ml_vision.dart';
import 'package:flutter/material.dart';

void main() => runApp(MaterialApp(home: _MyHomePage()));

class _MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<_MyHomePage> {
  FirebaseVision _vision;
  dynamic _scanResults;

  @override
  void initState() {
    super.initState();
    _initializeCamera();
  }

  void _initializeCamera() async {
    List<FirebaseCameraDescription> cameras = await camerasAvailable();
    _vision = FirebaseVision(cameras[0], ResolutionSetting.high);
    _vision.initialize().then((_) {
      if (!mounted) {
        return;
      }
      setState(() {});
    });
  }
Widget _buildImage() {
    return Container(
      constraints: const BoxConstraints.expand(),
      child: _vision == null
          Stack(
              fit: StackFit.expand,
              children: <Widget>[
                FirebaseCameraPreview(_vision),
                _buildResults(),
              ],
            ),
    );
  }
  
  Widget build(BuildContext context) {
    return Scaffold(
    body: _buildImage(),
   );
}

 void dispose() {
    _vision.dispose().then((_) {
  // close all detectors
    });

    super.dispose();
  }
  }

```

See the example app to learn more on incorporating detectors in the camera app, check it out [here](https://github.com/rishab2113/firebase_livestream_ml_vision/blob/master/example/lib/main.dart).

### 2. Using detectors

### Special Instructions for using VisionEdgeImageLabeler

Get an object of `ModelManager`, and setup the local or remote model(optional, results in faster first-use)

```dart
FirebaseVision.modelManager().setupModel('<foldername(modelname)>', modelLocation);
```

#### Calling a Labeler/Detector

a. Image Labeler

```dart
_vision.addImageLabeler().then((onValue){
        onValue.listen((onData) => // do something with data
        );
      });
```

b. Cloud Image Labeler

```dart
_vision.addCloudImageLabeler().then((onValue){
        onValue.listen((onData) => // do something with data
        );
      });
```

c. Barcode Detector

```dart
_vision.addBarcodeDetector().then((onValue){
          onValue.listen((onData) => // do something with data
          );
        });
```

d. Face Detector

```dart
_vision.addFaceDetector().then((onValue){
          onValue.listen((onData) => // do something with data
          );
        });
```

e. Text Recognizer

```dart
_vision.addTextRecognizer().then((onValue){
          onValue.listen((onData) => // do something with data
          );
        });
```

f. Vision Edge Image Labeler

```dart
_vision.addVisionEdgeImageLabeler('<foldername(modelname)>', modelLocation).then((onValue){
          onValue.listen((onData) => // do something with data
          );
        });
```

You can also configure all detectors, except `TextRecognizer`, with desired options.

```dart
final ImageLabeler labeler = FirebaseVision.instance.addImageLabler(
  ImageLabelerOptions(confidenceThreshold: 0.75),
);
```

### 3. Extract data

a. Extract barcodes.

```dart
for (Barcode barcode in barcodes) {
  final Rectangle<int> boundingBox = barcode.boundingBox;
  final List<Point<int>> cornerPoints = barcode.cornerPoints;

  final String rawValue = barcode.rawValue;

  final BarcodeValueType valueType = barcode.valueType;

  // See API reference for complete list of supported types
  switch (valueType) {
    case BarcodeValueType.wifi:
      final String ssid = barcode.wifi.ssid;
      final String password = barcode.wifi.password;
      final BarcodeWiFiEncryptionType type = barcode.wifi.encryptionType;
      break;
    case BarcodeValueType.url:
      final String title = barcode.url.title;
      final String url = barcode.url.url;
      break;
  }
}
```

b. Extract faces.

```dart
for (Face face in faces) {
  final Rectangle<int> boundingBox = face.boundingBox;

  final double rotY = face.headEulerAngleY; // Head is rotated to the right rotY degrees
  final double rotZ = face.headEulerAngleZ; // Head is tilted sideways rotZ degrees

  // If landmark detection was enabled with FaceDetectorOptions (mouth, ears,
  // eyes, cheeks, and nose available):
  final FaceLandmark leftEar = face.getLandmark(FaceLandmarkType.leftEar);
  if (leftEar != null) {
    final Point<double> leftEarPos = leftEar.position;
  }

  // If classification was enabled with FaceDetectorOptions:
  if (face.smilingProbability != null) {
    final double smileProb = face.smilingProbability;
  }

  // If face tracking was enabled with FaceDetectorOptions:
  if (face.trackingId != null) {
    final int id = face.trackingId;
  }
}
```

c. Extract labels.

```dart
for (ImageLabel label in labels) {
  final String text = label.text;
  final String entityId = label.entityId;
  final double confidence = label.confidence;
}
```

c. Extract Cloud Vision Edge labels.

```dart
for (VisionEdgeImageLabel label in labels) {
  final String text = label.text;
  final double confidence = label.confidence;
}
```

d. Extract text.

```dart
String text = visionText.text;
for (TextBlock block in visionText.blocks) {
  final Rect boundingBox = block.boundingBox;
  final List<Offset> cornerPoints = block.cornerPoints;
  final String text = block.text;
  final List<RecognizedLanguage> languages = block.recognizedLanguages;

  for (TextLine line in block.lines) {
    // Same getters as TextBlock
    for (TextElement element in line.elements) {
      // Same getters as TextBlock
    }
  }
}
```

## Getting Started

See the `example` directory for a complete sample app using ML Kit Vision for Firebase with Camera Streaming.
