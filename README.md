# Paystack Flutter SDK

A [Paystack](https://paystack.com) plugin for accepting payments in your Flutter application.

## Support 
| Android |   iOS   |  MacOS  |   Web   |  Linux  | Windows |
|  :---:  |  :---:  |  :---:  |  :---:  |  :---:  |  :---:  |
| &check; | &check; | &cross; | &check; | &cross; | &cross; |

## Requirements
Paystack Flutter SDK builds upon the recent patterns in the Android and iOS, thus your app should target:
- Flutter >= 3.4.0
- iOS >= 15
- Android 
  - Minimum SDK: 24
  - Compile SDK: 36

> [!IMPORTANT]
>
> Flutter (3.3.0 below, at the moment) extends the `FlutterActivity` as the base class for Android. The `FlutterActivity` doesn't have the `ComponentActivity`, a compulsory necessity for loading the payment views with the SDK, in its ancestral tree. To fix this, change the `FlutterActivity` to `FlutterFragmentActivity` in the `MainActivity` in the `android` folder of your project.

## Platform Support

#### Android Support
Please add the following snippet to your `android/app/proguard-rules.pro` file
```
# --- Keep annotation + inner class attributes ---
-keepattributes Annotation, InnerClasses, Signature

# --- Kotlinx Serialization ---
-dontnote kotlinx.serialization.AnnotationsKt

-keepclassmembers class kotlinx.serialization.json.** {
    *** Companion;
}

-keepclasseswithmembers class kotlinx.serialization.json.** {
    kotlinx.serialization.KSerializer serializer(...);
}

# --- Kotlin Standard + kotlinx packages ---
-keep class kotlin.** { *; }
-keep class kotlinx.** { *; }

-keepclassmembers class **$WhenMappings {
    *;
}

-keepclassmembers class kotlin.Metadata {
    public *;
}

# --- Parcelable Support ---
-keep class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

-keepnames class * implements android.os.Parcelable {
    public static final ** CREATOR;
}

-keepclassmembers class * implements android.os.Parcelable {
    public static final android.os.Parcelable$Creator *;
}

# --- Retrofit (Square or standard Retrofit2) ---
-keep interface retrofit2.Call
-keep class retrofit2.** { *; }
-keep interface retrofit2.* { *; }

# Keep Retrofit annotations
-keepclasseswithmembers class * {
    @retrofit2.http.* <methods>;
}

# --- Retrofit – Square package (optional but safe) ---
-keep class com.squareup.retrofit2.** { *; }
-keep interface com.squareup.retrofit2.* { *; }

# --- Paystack SDK ---
-keep class com.paystack.android.core.api.** { *; }
-keep interface com.paystack.android.core.api.* { *; }
-keep class com.paystack.android.core.api.models.* { *; }

# Already defined above, but safe to repeat (R8 ignores duplicates)
-keepattributes Annotation
```

#### Flutter Web Support
Please add the following snippet to your `web/index.html` inside `<body></body>` in your Flutter project.
```
<script src="https://js.paystack.co/v2/inline.js"></script>
```

## Getting Started
- Install the dependency in your project
```sh
flutter pub get paystack_flutter_sdk
```
- Import the paystack_flutter into your `.dart` file
```dart
import 'package:paystack_flutter_sdk/paystack_flutter_sdk.dart';
```
- Use the plugin
```dart
final _publicKey = "pk_domain_xxxxxx";
final _accessCode = "67joTry7t1jz2o";
final _paystack = Paystack();

 initialize(String publicKey) async {
    try {
      final response = await _paystack.initialize(publicKey, true);
      if (response) {
        log("Sucessfully initialised the SDK");
      } else {
        log("Unable to initialise the SDK");
      }
    } on PlatformException catch (e) {
      log(e.message!);
    }
  }

  launch() async {
    String reference = "";
    try {
      final response = await _paystack.launch(_accessCode);
      if (response.status) {
        reference = response.reference;
        log(reference);
      } else {
        log(response.message);
      }
    } on PlatformException catch (e) {
      log(e.message!);
    }

    setState(() {
      _reference = reference;
    });
  }
```

## Handling responses
The SDK returns two categories of responses:
- **Transaction**: This returned when you've successfully intergated the SDK and able to get the payment UI. This response 
is tied to the transaction lifecylce and has the following model:

  | Parameter |  Type  | Description |
  |   :---   |  :---: |    :---    |
  | `status` | String | Indicates the state of the transaction. Possible values are: `success`, `cancelled`, `failed` |
  | `message` | String | This is a short detail about the status of the transaction |
  | `reference` | String | This is a unique identifier used to manage post-payment processes. It is only returned when the `status` is `success` |

- **Exception**: Exceptions are thrown during your integration process. There are mostly issues surronding the development process. Exceptions make use of the platform-specific `FlutterError` model which has the following parameters:
following model:
  | Parameter |  Type  | Description |
  |   :---   |  :---: |    :---    |
  | `code` | String | This indicates the category of error |
  | `message` | String | This is a short description of the error |
  | `details` | String | This current returns `nil`. In a future update, we can use it to provide details about fixing the error |


The following are the error codes that can be returned when an exception occurs:

  | Error Code | Description |
  |    :---    |    :---     |
  | `INVALID_ARGUMENT` | This occurs when you aren't passing the required paramter(s) (public key) to set up the SDK |
  | `INITIALIZATION_ERROR` | This occurs when the SDK cannot be initialized with the parameters passed |
  | `UNSUPPORTED_VERSION` | This occurs when your projects doesn't conform to the SDK requirements |
  | `MISSING_VIEW` | This occur when the SDK cannot find a view controller (iOS) or Activity (Android) to attach to |
  | `LAUNCH_ERROR` | This occurs when the payment UI cannot be loaded |