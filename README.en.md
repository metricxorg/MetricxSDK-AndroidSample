## MetrixSDK Android Doc
[![CircleCI](https://circleci.com/gh/metrixorg/MetrixSDK-AndroidSample.svg?style=svg)](https://circleci.com/gh/metrixorg/MetrixSDK-AndroidSample)
[ ![Download](https://api.bintray.com/packages/metrixorg/maven/metrix-sdk-android/images/download.svg) ](https://bintray.com/metrixorg/maven/metrix-sdk-android/_latestVersion)
<div>
    
*Read this in other languages: [فارسی](README.md) , [English](README.en.md).*

<h2>Table of contents</h2>
<a href=#project_setup>1. Basic integration</a><br>
<a href=#integration>2. Add the SDK to your project</a><br>
<a style="padding-right:2em" href=#application_setup>2.1. Initial configuration in the app</a><br>
<a style="padding-right:2em" href=#about_application_class>2.2. About the application class and initialization in this class</a><br>
<a href=#methods>3. Additional features</a><br>
<a style="padding-right:2em" href=#session_event_description>3.1. Explain the concepts of event and session</a><br>
<a style="padding-right:2em" href=#enableLocationListening>3.2. Enable location listening</a><br>
<a style="padding-right:2em" href=#setEventUploadThreshold>3.3. Limitation in number of events to upload</a><br>
<a style="padding-right:2em" href=#setEventUploadMaxBatchSize>3.4. Limitation in number of events to send per request</a><br>
<a style="padding-right:2em" href=#setEventMaxCount>3.5. Limitation in number of events to buffer on the device</a><br>
<a style="padding-right:2em" href=#setEventUploadPeriodMillis>3.6. The time interval for sending events</a><br>
<a style="padding-right:2em" href=#setSessionTimeoutMillis>3.7. The session timeout</a><br>
<a style="padding-right:2em" href=#setOptOut>3.8. Disable tracking</a><br>
<a style="padding-right:2em" href=#enableLogging>3.9. Log management</a><br>
<a style="padding-right:2em" href=#setLogLevel>3.10. Set LogLevel</a><br>
<a style="padding-right:2em" href=#setOffline>3.11. Offline mode</a><br>
<a style="padding-right:2em" href=#setFlushEventsOnClose>3.12. Flush all events</a><br>
<a style="padding-right:2em" href=#getSessionNum>3.13. Current session number</a><br>
<a style="padding-right:2em" href=#newEvent>3.14. Custom event</a><br>
<a style="padding-right:2em" href=#setUserAttributes>3.15. Specify the default attributes for user</a><br>
<a style="padding-right:2em" href=#setUserMetrics>3.16. Specify the default metrics for user</a><br>
<a style="padding-right:2em" href=#setScreenFlowsAutoFill>3.17. Enable the process of storing the user flow</a><br>
<a style="padding-right:2em" href=#isScreenFlowsAutoFill>3.18. Find out the value of screenFlow</a><br>
<a style="padding-right:2em" href=#setAttributionListener>3.19. Get User attribution</a><br>
<a style="padding-right:2em" href=#setDefaultTracker>3.20. Pre-installed trackers</a><br>



<h2 id=project_setup>Basic integration</h2>

1. To start with, add the following settings in the `repositories` section of the `gradle` file:

<div dir="ltr">

    allprojects{
        repositories {

        ...

            maven {
                url 'https://dl.bintray.com/metrixorg/maven'
            }
        }
    }
</div>

2. Add the following library to the `dependencies` section of your `gradle` file:
<div dir="ltr">

    implementation 'ir.metrix:metrix:0.8.0'
</div>

3. Add the following options to the `android` block of your application's `gradle` file:

<div dir="ltr">

    compileOptions {
        targetCompatibility = "8"
        sourceCompatibility = "8"
    }
</div>


4. Add the following settings to your project's `Proguard` file:

<div dir=ltr>

    -keepattributes Signature
    -keepattributes *Annotation*
    -keepattributes EnclosingMethod
    -keepattributes InnerClasses

    -keepclassmembers enum * { *; }
    -keep class **.R$* { *; }
    -keep interface ir.metrix.sdk.NoProguard
    -keep class * implements ir.metrix.sdk.NoProguard { *; }
    -keep interface * extends ir.metrix.sdk.NoProguard { *; }

    # retrofit
    # Retain service method parameters when optimizing.
    -keepclassmembers,allowshrinking,allowobfuscation interface * {
        @retrofit2.http.* <methods>;
    }

    # Ignore JSR 305 annotations for embedding nullability information.
    -dontwarn javax.annotation.**

    # Guarded by a NoClassDefFoundError try/catch and only used when on the classpath.
    -dontwarn kotlin.Unit

    # Top-level functions that can only be used by Kotlin.
    -dontwarn retrofit2.-KotlinExtensions

    # With R8 full mode, it sees no subtypes of Retrofit interfaces since they are created with a Proxy
    # and replaces all potential values with null. Explicitly keeping the interfaces prevents this.
    -if interface * { @retrofit2.http.* <methods>; }
    -keep,allowobfuscation interface <1>

    #OkHttp
    # A resource is loaded with a relative path so the package of this class must be preserved.
    -keepnames class okhttp3.internal.publicsuffix.PublicSuffixDatabase

    # Animal Sniffer compileOnly dependency to ensure APIs are compatible with older versions of Java.
    -dontwarn org.codehaus.mojo.animal_sniffer.*

    # OkHttp platform used only on JVM and when Conscrypt dependency is available.
    -dontwarn okhttp3.internal.platform.ConscryptPlatform



    #Gson
    # Gson specific classes
    -dontwarn sun.misc.**
    #-keep class com.google.gson.stream.** { *; }

    # Prevent proguard from stripping interface information from TypeAdapterFactory,
    # JsonSerializer, JsonDeserializer instances (so they can be used in @JsonAdapter)
    -keep class * implements com.google.gson.TypeAdapterFactory
    -keep class * implements com.google.gson.JsonSerializer
    -keep class * implements com.google.gson.JsonDeserializer

    #referral
    -keep public class com.android.installreferrer.** { *; }

</div>

5. Please add the following permissions, which the Metrix SDK needs, if they are not already present in your `AndroidManifest.xml` file:

<div dir=ltr>

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" /> <!--optional-->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" /> <!--optional-->
</div>

(Two last permissions are optional)

<h2 id=integration>Implement the SDK in your project</h2>

<h3 id=application_setup>Initial configuration in the app</h3>

You need to initialize the Metrix SDK in `onCreate` method of your `Application`. If you do not already have a class `Application` in your project, create this class as below:<br>
1. Create a class that inherits from the `Application` class:<br>

<img src="https://storage.backtory.com/tapsell-server/metrix/doc/screenshots/Metrix-Application-Class.png"/>

2. Open the `AndriodManifest.xml` file and go to`<application>` tag.<br>
3. Using `Attribute` subclass, add `Application` to `AndroidManifest.xml` file:<br>

<div dir=ltr>

    <application
        android:name=“.MyApplication”
        ... >

    </application>
</div>

<img src="https://storage.backtory.com/tapsell-server/metrix/doc/screenshots/Metrix-Application-Manifest.png">

4. In `onCreate` method of your `Application` class, initialize the Metrix according to the codes below:<br>
<div dir=ltr>

    import ir.metrix.analytics.Metrix;

    public class MyApplication extends Application {

        @Override
        public void onCreate() {
            super.onCreate();
            Metrix.initialize(this, “app id”);
        }
    }
</div>

Replace `APP_ID` with your application id. You can find that in your Metrix's dashboard.

<h3 id=about_application_class>About the application class and initialization in this class</h3>

The Android gives developers the ability to run methods before the creation of any `activity` in the application class. Because counting the `session`, gathering `screen-flows` between `activities` and many other features of the SDK required them to work properly.

<h2 id=methods>Additional features</h2>

<h3 id=session_event_description>Explain the concepts of event and session</h3>
In each interaction that the user has with the app, the Metrix sends this interaction to the server as a <b>event</b>. In Metrix, <b>session</b> means that a specific timeframe that the user interacts with the app.<br>
There are three types of events in the Metrix:<br>
<b>1. Session Start:</b> The time of start a session.<br>
<b>2. Session Stop:</b> The time of stop a session.<br>
<b>3. Custom:</b> Depending on your application logic and the interactiion that the user has with your app, you can create and send custom events as below:<br>

<b>Tip:</b> To use library facilities and call the methods provided by the SDK, you must get `MetrixClient` using `getInstance` and then choose your desired method.<br>
<br>

<h3 id=enableLocationListening>Enable location listening</h3>
You can declare to Metrix to send information about the location of the user using the following functions. (In order to these method work properly, the optional permissions must be enabled) <br>
<div dir=ltr>

    Metrix.getInstance().enableLocationListening();

    Metrix.getInstance().disableLocationListening();
</div>

<h3 id=setEventUploadThreshold>Limitation in number of events to upload</h3>
Using the following function, you can specify that each time the number of your buffered events reaches the threshold, SDK should send them to the server:<br>
<div dir=ltr>

    Metrix.getInstance().setEventUploadThreshold(50);
</div>
(The default value is 30 events.)<br>

<h3 id=setEventUploadMaxBatchSize>Limitation in number of events to send per request</h3>
Using this function, you can specify the maximum number of outcoming events per request as shown below:<br>
<div dir=ltr>

    Metrix.getInstance().setEventUploadMaxBatchSize(100);
</div>
(The default value is 100 events.)<br>

<h3 id=setEventMaxCount>Limitation in number of events to buffer on the device</h3>
Using the following function, you can specify how much the maximum number of buffered events is in the SDK (for example, if the user device lost its internet connection, the events will be stored in the library as the amount of you specify) and if the number of buffered events in the library pass this amount. Old events are not kept and destroyed by SDK:<br>
<div dir=ltr>

    Metrix.getInstance().setEventMaxCount(1000);
</div>
(The default value is 100 events.)<br>

<h3 id=setEventUploadPeriodMillis>The time interval for sending events</h3>
By using this function, you can specify that how long the request for sending events should be sent: <br>
<div dir=ltr>

    Metrix.getInstance().setEventUploadPeriodMillis(30000);
</div>
(The default value is 30 seconds.)<br>

<h3 id=setSessionTimeoutMillis>The session timeout</h3>
Using this function, you can specify the limit of session lengthes in your application. For example, if the value of this value is 10,000, if the user interacts with the application in 70 seconds, the Metrix calculates this interaction in seven sessions.<br>
<div dir=ltr>

    Metrix.getInstance().setSessionTimeoutMillis(1800000);
</div>
(The default value is 30 minutes.)<br>

<h3 id=setOptOut>Disable tracking</h3>
Using this function, you can command the SDK to not record any events:<br>
<div dir=ltr>

    Metrix.getInstance().setOptOut(true);
</div>
(The default value is false.)<br>

<h3 id=enableLogging>Log management</h3>
Note that setting the value of this value to `false` during the release of your application:<br>
<div dir=ltr>

    Metrix.getInstance().enableLogging(true);
</div>
(The default value is true.)<br>

<h3 id=setLogLevel>Set LogLevel</h3>

Using this function, you can specify what level of logs to be printed in `logcat`, for example, the following command will display all logs except `VERBOSE` in `logcat`:<br>
<div dir=ltr>

    Metrix.getInstance().setLogLevel(Log.DEBUG);
</div>

(The default value is `Log.INFO`.)<br>

<h3 id=setOffline>Offline mode</h3>
Using this function, the SDK does not send events to the server, but still records events with your preferences:<br>
<div dir=ltr>

    Metrix.getInstance().setOffline(true);
</div>
(The default value is false.)<br>

<h3 id=setFlushEventsOnClose>Flush all events</h3>
Using this function, you can specify that when the application is closed, all events buffered in the device, should be sent or not:
<br>
<div dir=ltr>

    Metrix.getInstance().setFlushEventsOnClose(false);
</div>
(The default value is true.)<br>

<h3 id=getSessionNum>Current session number</h3>
By this function, you can find the current session number:<br>
<div dir=ltr>

    Metrix.getInstance().getSessionNum();
</div>

<h3 id=newEvent>Custom event</h3>

You can use Metrix to track any event in your app. Suppose you want to track every tap on a button. You would have to create a new event slug in the Events Management section of your dashboard. Let's say that event slug is `abc123`. In your button's onClick method you could then add the following lines to track the click.
<br>
You can call this function in two ways:<br>
1. Make a custom event that has only one specified name:<br>

<div dir=ltr>

    Metrix.getInstance().newEvent(“abc123");
</div>

The input of this function is String.<br>
<br>

2. Create a custom event with a specific numbers of attributes and metrics, for example, suppose you want to create a custom event in an online purchase program:<br>

<div dir=ltr>

    Map<String, String> attributes = new HashMap<>();
    attributes.put("first_name", "Ali");
    attributes.put("last_name", "Bagheri");
    attributes.put("manufacturer", "Nike");
    attributes.put("product_name", "shirt");
    attributes.put("type", "sport");
    attributes.put("size", "large");

    Map<String, Object> metrics = new HashMap<>();
    metrics.put("price", 100000);
    metrics.put("purchase_time", current_time);

    Metrix.getInstance().newEvent("purchase_event_slug", attributes, metrics);
</div>

The variables for the `newEvent` method are as follows:<br>
- <b>First variable:</b>The event slug which is String type and you get it from the Metrix dashboard.<br>
- <b>Second variable:</b> A Map `<String, String>` that specifies the attributes of an event.<br>
- <b>Third variable:</b> A Map `<String, Object>` that contains measurable metrics. Supported values in the MetriX are one of the following types:
    1. Integer
    2. Float
    3. Double
    4. Long
    5. Sting
    6. Boolean

<h3 id=setUserAttributes>Specify the default attributes for user</h3>
Using this function, you can add arbitrary `Attributes` to all events of the user:<br>
<div dir=ltr>

    Map<String, String> attributes = new HashMap<>();
    attributes.put("manufacturer", "Nike");

    Metrix.getInstance().addUserAttributes(attributes);
</div>

<h3 id=setUserMetrics>Specify the default metrics for user</h3>

Using this function, you can add arbitrary `Metrics` to all events of the user:<br>
<div dir=ltr>

    Map<String, Object> metrics = new HashMap<>();
    metrics.put("purchase_time", current_time);

    Metrix.getInstance().setUserMetrics(metrics);
</div>

<h3 id=setScreenFlowsAutoFill>Enable the process of storing the user flow</h3>

Using this function, you can inform the Metrix to gather information about user's flow in each `Activity`/`Fragment` and these details should be stored automatically:<br>
<div dir=ltr>

    Metrix.getInstance().setScreenFlowsAutoFill(true);
</div>
(The default value is false.)<br>

<h3 id=isScreenFlowsAutoFill>Find out the value of screenFlow</h3>
Using this function, you can see that what the `screenFlow` value in the Metrix is:<br>
<div dir=ltr>

    Metrix.getInstance().isScreenFlowsAutoFill();
</div>

<h3 id=setAttributionListener>Get User attribution</h3>
In case you want to access info about your user's current attribution when ever you need it, you can make a call to following method of the Metrix instance: <br>
<div dir=ltr>

    Metrix.getInstance().setOnAttributionChangedListener(new OnAttributionChangedListener() {
    @Override
      public void onAttributionChanged(AttributionModel attributionModel) {
          //TODO
        }
    });
</div>

Here is a quick summary of `AttributionModel` properties: <br>

`attributionModel.getAcquisitionAd()` : The creative/ad grouping level of the current attribution.

`attributionModel.getAcquisitionAdSet()`: The adGroup/adSet grouping level of the current attribution.

`attributionModel.getAcquisitionCampaign()`: The campaign grouping level of the current attribution.

`attributionModel.getAcquisitionSource()`: The network/source grouping level of the current attribution.

`attributionModel.getAttributionStatus()`: Specifies the status of the user in the campaign and returns only the four values below:
<br>

1- `ATTRIBUTED`

2- `NOT_ATTRIBUTED_YET`

3- `ATTRIBUTION_NOT_NEEDED`

4- `UNKNOWN`
<br>

<h3 id=setDefaultTracker>Pre-installed trackers</h3>

If you want to use the Metrix SDK to recognize users whose devices came with your app pre-installed, open your app delegate and set the default tracker of your config. Replace `trackerToken` with the tracker token you created in dashboard. Please note that the Dashboard displays a tracker URL (including http://tracker.metrix.ir/). In your source code, you should specify only the six-character token and not the entire URL. <br>
<div dir=ltr>

    Metrix.getInstance().setDefaultTracker(trackerToken);
</div>


</div>