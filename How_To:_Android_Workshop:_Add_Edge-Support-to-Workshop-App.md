Title: How To: Android Workshop: Add Edge Support to Workshop App

## Introduction
This workshop demonstrates how to add Edge support to the Workshop app. We will also introduce you to MobiledgeX's Distributed Matching Engine (DME) APIs. 

We will be making use of a MobiledgeX library: The **MobiledgeX MatchingEngine** library exposes various services that MobiledgeX offers such as finding the nearest MobiledgeX Cloudlet Infrastructure for client-server communication or workload processing offload.

This library has been published to a Maven repository and we will be adding them to our workshop project. To interface with the library, you must update the workshop code to instantiate several classes and implement several interfaces. This document will walk you through how to do this.

## Workshop Goals
* Start with the Workshop Skeleton app, add Edge support to register the client, and find the closest Edge cloudlet.
* Gain understanding of Distributed Matching Engine APIs

## Prerequisites
* Experience with Android app development.
* A current install of Android Studio. This installation can be a lengthy process, therefore, we recommend that you ensure your development environment is stable before you begin the workshop.
* [Git](https://www.atlassian.com/git/tutorials/install-git) installed
* Access to the Workshop Skeleton app code.
* Access to the MobiledgeX maven SDK repository. Go here to create a user login, and validate the email address.
* Note: If you are a hackathon participant, please see the event organizer for your specific user name and password.

## Download the  Android Studio Workshop Project
Create a directory that you want your edge-cloud-sampleapps repo to be located in and check out the edge-cloud-sampleapps repository from the the MobiledgeX Github:
```
mkdir <name_of_your_directory>
cd <name_of_your_directory>
git clone https://github.com/mobiledgex/edge-cloud-sampleapps.git
```
## Open Project in Android Studio
* If this is your first time using Android Studio, select **Open an existing Android Studio project**. If you already have another project open, select **File>Open...**
* Navigate to the edge-cloud-sampleapps/android directory and select **WorkshopSkeleton**.
* Make sure to select the **Android** view of the project to follow this guide.

![alt text](https://drive.google.com/file/d/1RNvXupti6awGwgbFM6lUtFDe61WntN0z/preview)

## Add Maven Credentials
These credentials allow access to MobiledgeX's private Maven repository.
1. In the WorkshopSkeleton project, open Gradle Scripts/local.properties.
<another image>

2. Add the following at the end of the file:
```
artifactory_user=<user from console.mobiledgex.net user creation>
artifactory_password=<your password>
```
## Run the Skeleton App
Right out of the box, the Workshop app can be built and ran, though it won't do much. Let's run it now to verify our development environment.

1. Run the app by selecting Run/Run App in Android Studio. A Select Deployment Target dialog appears.
2. Select your device, and click OK. The app will be built and installed on your device.
3. Now on your device, click the red button in the lower right. A message will display Not successfully coded. We will remedy this shortly.
<another image>
4. The Skeleton app also has a placeholder for a Face Detection activity. To view this, on the main menu (upper left corner "hamburger" icon), click Face Detection, and the message Not implemented yet.
<another image>
  
## Update Dependencies
1. Open Gradle Scripts/build.gradle (Module: app).
<another image>
2. Locate the dependencies section of the file, and note that the Matching Engine SDK dependencies have already been added. This was to facilitate getting the skeleton app into a compilable state. If you are starting a new project from scratch, you will need to add these dependencies manually.
```
// Matching Engine SDK
implementation 'com.mobiledgex:matchingengine:1.4.14'
implementation "io.grpc:grpc-okhttp:${grpcVersion}"
implementation "io.grpc:grpc-stub:${grpcVersion}"
implementation "io.grpc:grpc-protobuf-lite:${grpcVersion}"
```
3. In order to have access to the MobiledgeX Computer Vision library, add the following to the dependencies section:
```
// MobiledgeX Computer Vision library
implementation 'com.mobiledgex:computervision:1.0.6'
implementation 'com.android.volley:volley:1.1.1'
```
4. The Gradle files have changed alert text box will appear. Click Sync Now to pull in the newly added libraries.
  <another image>
Now that you have loaded the MobiledgeX Computer Vision Library, we can start updating the workshop app code to use it.

## Device Permissions-Manifest
<another image>
1. Open manifests/AndroidManifest.xml and add the following permissions:
```
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-feature android:name="android.hardware.camera" />
<uses-feature android:name="android.hardware.camera.autofocus" />
```
Now, the manifest should look like this:    
<another image>
## Permissions Utility
The app must explicitly ask the users for any special permissions. The Matching Engine SDK provides a utility to simplify this: com.mobiledgex.matchingengine.util.RequestPermissions. To use it, you'll need to add some code to app/java/com.mobiledgex.workshopskeleton/MainActivity.java. 
<another image>
1. At the top of the file, add the following import statement:
```
import com.mobiledgex.matchingengine.util.RequestPermissions;
```
2. Now, define a new class variable right after the MatchingEngine variable. Search for private MatchingEngine matchingEngine and add the following right below it:
```
private RequestPermissions mRpUtil;
```
3. Add code to request the permissions ,if needed. At the end of the onCreate method, add the following:
```
/**
 * MatchEngine APIs require special user approved permissions to READ_PHONE_STATE and
 * one of the following:
 * ACCESS_FINE_LOCATION or ACCESS_COARSE_LOCATION. This creates a dialog, if needed.
 */
mRpUtil = new RequestPermissions();
if (mRpUtil.getNeededPermissions(this).size() > 0) {
    mRpUtil.requestMultiplePermissions(this);
    return;
}
```
4. Finally, copy this code in the onResume method in case permissions are changed during use of app:
```
/*
* Check permissions here, as the user has the ability to change them on the fly through
* system settings
*/
if (mRpUtil.getNeededPermissions(this).size() > 0) {
    // Opens a UI. When it returns, onResume() is called again.
    mRpUtil.requestMultiplePermissions(this);
    return;
}
```
Note: To make your app more robust, you would also add this same permissions check before any code that requires permissions.

## Your First MatchingEngine API Call
We will now write some code that will create a RegisterClientRequest and use it in a createRegisterClientRequest call to the SDK. We will include specific information about the application, including the name, version, and the carrier that the mobile device is running on. Using this information, we can use the registerClient API call to register our client with the Distributed Matching Engine (DME). 

These changes will also go in app/java/com.mobiledgex.workshopskeleton/MainActivity.java.

1. Search for the following comment:```// TODO: Copy/paste the code to register the client. Replace all "= null" lines here.
2. Directly after that (or in place of it), copy and paste the following code:
```
host = matchingEngine.generateDmeHostAddress();
if(host == null) {
    host = "sdkdemo.dme.mobiledgex.net";   //fallback host
    Log.e(TAG, "Could not generate host");
}
port = matchingEngine.getPort(); // Keep same port.
AppClient.RegisterClientRequest registerClientRequest = matchingEngine.createRegisterClientRequest(ctx,
        devName, appName, appVersion, carrierName, null);
AppClient.RegisterClientReply registerStatus = matchingEngine.registerClient (registerClientRequest, host,
        port, 10000);
```
3. Directly above that code, make sure you pass in the correct app/developer information to dme:
```
appName = "Face Detection Demo";
devName = "MobiledgeX"; // HackathonTeam-N, where N is your team number
carrierName = "TDG"; // Rogers for UBC Hackathon
appVersion = "1.0";
```
4. Add the following code right after the permissions code in the onResume method:
```

// Ensures that user can switch from wifi to cellular network data connection (required to verifyLocation)
matchingEngine = new MatchingEngine(ctx);
matchingEngine.setNetworkSwitchingEnabled(true);  // false -> wifi (Stick with wifi for workshop.)
```
5. Replace the statusText: ```registerClient call is not successfully coded. Search for TODO in code.``` 
with ```Failed to register client```

## Locate the Closest Cloudlet
Search for the TODO comment that refers to Change these coordinates, and update the latitude and longitude to match where you are currently located. Once the device has been successfully registered, the findCloudlet API call can be used to find the nearest available cloudlet that has your server application running. The findCloudlet API returns the URI of the application that can be used to form a client-server communication.
Now, we will add code to do this. Search for the TODO comment that refers to "find the cloudlet closest" and paste the following code:
```
AppClient.FindCloudletRequest findCloudletRequest= matchingEngine.createFindCloudletRequest (ctx,
        carrierName, location);
mClosestCloudlet = matchingEngine.findCloudlet(findCloudletRequest, host, port, 10000);
```
## Verify Location
The verifyLocation API is used to verify the location of the user. The user's carrier returns the distance between the location that the user claims and what the carrier knows. Public fields such as getTowerStatus(), getGpsLocationStatus(), and getGpsLocationAccuracyKm() allow the developer to ensure the user is located where they claim to be.

1. Search for the TODO comment in the verifyLocation method. Copy and paste the following code:
```
AppClient.VerifyLocationRequest verifyLocationRequest = matchingEngine.createVerifyLocationRequest(ctx, carrierName, loc);
if (verifyLocationRequest != null) {
    try {
        AppClient.VerifyLocationReply verifyLocationReply = matchingEngine.verifyLocation(verifyLocationRequest, host, port, 10000);
        Log.i(TAG, "[Location Verified: Tower: " + verifyLocationReply.getTowerStatus() +
                ", GPS LocationStatus: " + verifyLocationReply.getGpsLocationStatus() +
                ", Location Accuracy: " + verifyLocationReply.getGpsLocationAccuracyKm() + " ]\n");
        return true;
    } catch(NetworkOnMainThreadException ne) {
        Log.e(TAG, "Network thread exception");
        return false;
    }
} else {
    Log.e(TAG, "Cannot verify location");
    return false;
}
```
2. Replace the statusText: ```verifyLocation call is not successfully coded. Search for TODO in code.```
with ```Failed to verify location```

3. After you have completed the previous TODOs, run the app and press the red button. You should successfully populate the Client Registered, Cloudlet Found and Location Verified boxes. Also, in the Logcat you should see the Tower the user is connected to, their GPS LocationStatus, and their Location Accuracy.
```
/MainActivity: [Location Verified: Tower: TOWER_UNKNOWN, GPS LocationStatus: LOC_UNKNOWN, Location Accuracy: -1.0 ]
```
The LocationStatus possible values:
* LOC_UNKNOWN
* LOC_VERIFIED
* LOC_MISMATCH_SAME_COUNTRY
* LOC_MISMATCH_OTHER_COUNTRY
* LOC_ROAMING_COUNTRY_MATCH
* LOC_ROAMING_COUNTRY_MISMATCH
* LOC_ERROR_UNAUTHORIZED
* LOC_ERROR_OTHER

Location Accuracy possible values are:
* -1: unverified location
* 2: within 2km
* 10: within 10km
* 100: withing 100km

Roaming Cases are:
* 1: Location match, with sensitivity of <= 2km
* 2: Location mismatch, with deviation of > 2km and <= 10km
* 3: Location mismatch, with deviation of > 10km and <= 100km
* 4: Location mismatch: > 100km in same country
* 5: Location mismatch: other country (roaming)
* 6: Country match (roaming)
* 7: Country mismatch (roaming)

## Get QoS Position
The getQoSPositionKpi API is used to get quality of service statistics from developer specified GPS locations. Some of the statistics are latency and throughput (given that the app is connected to a cloudlet).

1. Search for the TODO comment in the getQoSPositionKpi method. Copy and paste the following code:
```
List<AppClient.QosPosition> requests = createPositionList(loc, 45, 200, 1);
if (requests.isEmpty()) {
    Log.e(TAG, "No items added to the position list");
    return false;
}
AppClient.QosPositionRequest qosPositionRequest = matchingEngine.createQoSPositionRequest(requests, 0, null);
if(qosPositionRequest != null) {
    try {
        ChannelIterator<AppClient.QosPositionKpiReply> qosPositionKpiReplies = matchingEngine.getQosPositionKpi(qosPositionRequest, host, port, 20000);
        if (!qosPositionKpiReplies.hasNext()) {
            Log.e(TAG, "Replies is empty");
            return false;
        }
        while (qosPositionKpiReplies.hasNext()) {
            Log.i(TAG, "Position results are " + qosPositionKpiReplies.next().getPositionResultsList());
        }
        return true;
    } catch (NetworkOnMainThreadException ne) {
        Log.e(TAG, "Network thread exception");
        return false;
    }
} else {
    Log.e(TAG, "QoS request is null");
    return false;
}
```
2. Then, replace the TODO comment about the list of GPS locations with the following function. This enables us to retrieve a list of GPS locations within totalDistanceKm from the user's location in the direction_degrees direction.
```
private List<AppClient.QosPosition> createPositionList(LocOuterClass.Loc loc, double direction_degrees, double totalDistanceKm, double increment) {
    List<AppClient.QosPosition> positions = new ArrayList<>();
    long positionId = 1;

    AppClient.QosPosition firstPosition = AppClient.QosPosition.newBuilder()
            .setPositionid(positionId)
            .setGpsLocation(loc)
            .build();
    positions.add(firstPosition);

    LocOuterClass.Loc lastLocation = LocOuterClass.Loc.newBuilder()
            .setLongitude(loc.getLongitude())
            .setLatitude(loc.getLatitude())
            .build();

    double kmPerDegreeLat = 110.57; //at Equator
    double kmPerDegreeLong = 111.32; //at Equator
    double addLatitude = (Math.sin(direction_degrees * (Math.PI/180)) * increment)/kmPerDegreeLat;
    double addLongitude = (Math.cos(direction_degrees * (Math.PI/180)) * increment)/kmPerDegreeLong;
    for (double traverse = 0; traverse + increment < totalDistanceKm; traverse += increment) {
        LocOuterClass.Loc next = LocOuterClass.Loc.newBuilder()
                .setLongitude(lastLocation.getLongitude() + addLongitude)
                .setLatitude(lastLocation.getLatitude() + addLatitude)
                .build();
        AppClient.QosPosition nextPosition = AppClient.QosPosition.newBuilder()
                .setPositionid(++positionId)
                .setGpsLocation(next)
                .build();

        positions.add(nextPosition);
        Log.i(TAG, "Latitude is " + nextPosition.getGpsLocation().getLatitude() + " and Longitude is " + nextPosition.getGpsLocation().getLongitude());
        lastLocation = next;
    }
    return positions;
}
```
Replace the statusText:`qosPositionKpi call is not successfully coded. Search for TODO in code.` with `Failed to get qosPositions`

 In the Logcat, if you search for Position results are, you should be able to see the following info for each GPS location in the list of generated locations:
```
dluserthroughput_avg: 57.85048
dluserthroughput_max: 100.27839
dluserthroughput_min: 2.0583756
gps_location {
latitude: 37.3382
longitude: -121.8863
}
latency_avg: 34.918453
latency_max: 65.803474
latency_min: 25.180304
positionid: 1
uluserthroughput_avg: 33.63164
uluserthroughput_max: 57.953064
uluserthroughput_min: 4.4424434, # distributed_match_engine.AppClient$QosPositionResult@a44e83f1
```
4. If you get a Failed to get qosPositions message for your current location, go back to the TODO: Change these coordinates to where you're actually located, and use Latitude: 52.52 and Longitude: 13.4040 (Berlin) to see an example of QoS data.

## Expected Output
After you have completed the previous TODOs, running the app and pressing the red button should successfully populate the cloudlet info.
<another image>

If you examine the Logcat output, you can find a log similar to this: `com.mobiledgex.workshopskeleton I/COPY_PASTE: ping -c 4 mpk-tip.tip.mobiledgex.net`

Copy the full ping command and paste it into a terminal to see the kind of latency you can expect. For example:
```
$ ping -c 4 mpk-tip.tip.mobiledgex.net
PING mpk-tip.tip.mobiledgex.net (209.249.227.19): 56 data bytes
64 bytes from 209.249.227.19: icmp_seq=0 ttl=47 time=61.891 ms
64 bytes from 209.249.227.19: icmp_seq=1 ttl=47 time=63.034 ms
64 bytes from 209.249.227.19: icmp_seq=2 ttl=47 time=67.771 ms
64 bytes from 209.249.227.19: icmp_seq=3 ttl=47 time=62.398 ms
```
Now that this section is complete, you have successfully found the Edge cloudlet closest to our location and can begin using it for the lowest latency possible for interacting with our backend software. Let's add some code to let us work with the FaceDetectionServer backend.

## What's Next?
Go to [How To: Android Workshop: Adding Face Detection to Workshop App]() to fill the rest of the skeleton app and learn the benefits of edge computing for latency sensitive applications.



































    
    
    
