## Introduction
This How-To article will provide steps to add Face Detection to the Android workshop app. Through this application, you will see the benefits of Edge computing for latency sensitive applications. We will also take a look at GPU-accelerated body pose detection as an example of offloading a heavy workload to an Edge cloudlet. Before starting this workshop, make sure you have worked through the How-To: Android Workshop: Add Edge Support to Workshop App which shows you how to call several of MobiledgeX's Distributed Matching Engine APIs.

We will be making use of a MobiledgeX library:

* The **MobiledgeX ComputerVision** library provides classes and interfaces to work with the Android device's camera, and to send images to a Face Detection server and receive and process the server's responses.

This library has been published to a Maven repository and we will be adding itfd to our workshop project. To interface with the library, we must update the workshop code to instantiate several classes and implement several interfaces. This document will walk you through how to do this.

## Workshop Goals
1. Add a face detection activity using the MobiledgeX Computer Vision library.
2. Add Edge support to the face detection activity, to get best possible latency for on-screen face tracking.
3. Experiment with face recognition and pose detection.
4. Hack on the app with your own ideas.

## Prerequisites
Android app development experience.
Android Studio installed and up to date. This installation can be a lengthy process, so please have your development environment in good shape before the start of the workshop.
Access to the **Workshop Skeleton** app code.
Worked through How-To: Android Workshop: Add Edge Support to Workshop App

## Instructions
Face Processor Fragment Layout
Add Face Box Renderer to Layout
<add image>

In the WorkshopSkeleton project, there is an **edgeFaceBoxRender** placeholder that is defined as a generic View. Let's update it to a FaceBoxRenderer. Open **app/res/layout/fragment_face_processor.xml**. If the editor comes up in "Design" mode, change to the "Text" view (see pink oval below).
<add image>

Now search for **edgeFaceBoxRender**. Change its definition from "View" to "com.mobiledgex.computervision.FaceBoxRenderer".

It will now look like this:
```
<com.mobiledgex.computervision.FaceBoxRenderer
    android:id="@+id/edgeFaceBoxRender"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:layout_gravity="center" />
```

## Modify FaceProcessorFragment
Now let's edit the main class we will be using for Face Detection. Open **app/java/com.mobiledgex.workshopskeleton/FaceProcessorFragment.java**.

<add image>


### Imports
In **FaceProcessorFragment.java**, add the following imports:
```
import com.mobiledgex.computervision.Camera2BasicFragment;
import com.mobiledgex.computervision.FaceBoxRenderer;
import com.mobiledgex.computervision.ImageProviderInterface;
import com.mobiledgex.computervision.ImageSender;
import com.mobiledgex.computervision.ImageServerInterface;
import com.mobiledgex.computervision.RollingAverage;
```

### Class Definition 
Update the definition of the FaceProcessorFragment class to extend ImageProcessorFragment, implement the library methods, and then add a FaceBoxRenderer variable. The class definition should now look like this:
```
public class FaceProcessorFragment extends com.mobiledgex.computervision.ImageProcessorFragment implements ImageServerInterface,
        ImageProviderInterface {
    private FaceBoxRenderer mFaceBoxRenderer;
```

## Define Variables
In this section we will instantiate our camera fragment, image sender, and **face box renderer**. 

### Preferences
Before we instantiate our classes, we'll need access to some preferences and default values. Those are defined in the library's base ImageProcessorFragment class, but we need to add some code to populate the values.

Search for the following comment:
```
// TODO: Copy/paste the code to access preferences.
```

Directly after that (or in place of it), copy and paste the following code:
```
SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(getContext());
prefs.registerOnSharedPreferenceChangeListener(this);
onSharedPreferenceChanged(prefs, "ALL");
```

### Camera Fragment
The Camera2BasicFragment class provides the image frames that we will be sending to the Face Detection server for processing. Let's instantiate ours now.

Search for the TODO that refers to "Camera2BasicFragment" and paste the following code:
```
mCamera2BasicFragment = new Camera2BasicFragment();
mCamera2BasicFragment.setImageProviderInterface(this);
String prefKeyFrontCamera = getResources().getString(R.string.preference_fd_front_camera);
mCamera2BasicFragment.setCameraLensFacingDirection(prefs.getInt(prefKeyFrontCamera, CameraCharacteristics.LENS_FACING_FRONT));
FragmentTransaction transaction = getChildFragmentManager().beginTransaction();
transaction.replace(com.mobiledgex.computervision.R.id.child_camera_fragment_container, mCamera2BasicFragment).commit();
```

### Image Sender
The ImageSender class sends images to Face Detection server and receives results specifying rectangular coordinates for any detected faces.  Let's instantiate our ImageSender now.

Search for the TODO that refers to "ImageSender" and paste the following code:
```
String host = mHostDetectionEdge; //The default from the base class.
mImageSenderEdge = new ImageSender(getActivity(), this, CloudletType.EDGE, host, FACE_DETECTION_HOST_PORT, PERSISTENT_TCP_PORT);
mImageSenderEdge.setCameraMode(ImageSender.CameraMode.FACE_DETECTION);
mCameraMode = ImageSender.CameraMode.FACE_DETECTION;
mCameraToolbar.setTitle("Face Detection");
```

### Face Box Renderer
The FaceBoxRenderer draws the coordinates received from the Face Detection server. It is defined in **fragment_face_processor.xml**, but we need to assign a variable to it. We can also customize the look by changing the shape type (RECT or OVAL) and the color.

Search for the TODO referring to "FaceBoxRenderer" and paste the following into the class:
```
mFaceBoxRenderer = view.findViewById(R.id.edgeFaceBoxRender);
mFaceBoxRenderer.setShapeType(FaceBoxRenderer.ShapeType.OVAL);
mFaceBoxRenderer.setColor(Color.CYAN);
```

## Implement MobiledgeX Computer Vision Library Interfaces
In this section, we will implement custom versions of the methods defined by the library's interfaces.

### ImageProcessorInterface
The ImageProcessorInterface defines methods that must be implemented by classes that process images from the camera. Because our FaceProcessorFragment extends com.mobiledgex.computervision.ImageProcessorFragment, these methods are already implemented. However, we want to perform custom processing every time an image is available, so we will be overriding onBitmapAvailable().

Search for the TODO referring to "ImageProcessorInterface" and paste the following into the class:
```
/**
 * Perform any processing of the given bitmap.
 *
 * @param bitmap  The bitmap from the camera or video.
 * @param imageRect  The coordinates of the image on the screen. Needed for scaling/offsetting
 *                   resulting pose skeleton coordinates.
 */
@Override
public void onBitmapAvailable(Bitmap bitmap, Rect imageRect) {
    if(bitmap == null) {
        return;
    }

    mImageRect = imageRect;
    mServerToDisplayRatioX = (float) mImageRect.width() / bitmap.getWidth();
    mServerToDisplayRatioY = (float) mImageRect.height() / bitmap.getHeight();

    Log.d(TAG, "mImageRect="+mImageRect.toShortString()+" mImageRect.height()="+mImageRect.height()+" bitmap.getWidth()="+bitmap.getWidth()+" bitmap.getHeight()="+bitmap.getHeight()+" mServerToDisplayRatioX=" + mServerToDisplayRatioX +" mServerToDisplayRatioY=" + mServerToDisplayRatioY);

    mImageSenderEdge.sendImage(bitmap);
}
```

### ImageServerInterface
The ImageServerInterface defines methods that must be implemented by classes that send images to the MobiledgeX Face Detection server and receive results back. These methods do things like draw rectangles around identified faces, or update the network latency stats. We will be implementing custom versions of many methods in the interface.

Search for the TODO referring to "ImageServerInterface" and paste the following into the class:
```
/**
 * Update the face rectangle coordinates.
 *
 * @param cloudletType  The cloudlet type determines which FaceBoxRender to use.
 * @param rectJsonArray  An array of rectangular coordinates for each face detected.
 * @param subject  The Recognized subject name. Null or empty for Face Detection.
 */
@Override
public void updateOverlay(final CloudletType cloudletType, final JSONArray rectJsonArray, final String subject) {
    Log.i(TAG, "updateOverlay Rectangles("+cloudletType+","+rectJsonArray.toString()+","+subject+")");
    new Handler(Looper.getMainLooper()).post(new Runnable() {
        @Override
        public void run() {
            if (getActivity() == null) {
                //Happens during screen rotation
                Log.e(TAG, "updateOverlay abort - null activity");
                return;
            }

            boolean mirrored = mCamera2BasicFragment.getCameraLensFacingDirection() ==
                    CameraCharacteristics.LENS_FACING_FRONT
                    && !mCamera2BasicFragment.isLegacyCamera()
                    && !mCamera2BasicFragment.isVideoMode();

            Log.d(TAG, "mirrored=" + mirrored + " mImageRect=" + mImageRect.toShortString() + " mServerToDisplayRatioX=" + mServerToDisplayRatioX +" mServerToDisplayRatioY=" + mServerToDisplayRatioY);

            mFaceBoxRenderer.setDisplayParms(mImageRect, mServerToDisplayRatioX, mServerToDisplayRatioY, mirrored, prefMultiFace);
            mFaceBoxRenderer.setRectangles(rectJsonArray, subject);
            mFaceBoxRenderer.invalidate();
            mFaceBoxRenderer.restartAnimation();
        }
    });
}

public void updateFullProcessStats(final CloudletType cloudletType, RollingAverage rollingAverage) {
    final long stdDev = rollingAverage.getStdDev();
    final long latency = rollingAverage.getAverage();
    if(getActivity() == null) {
        Log.w(TAG, "Activity has gone away. Abort UI update");
        return;
    }
    getActivity().runOnUiThread(new Runnable() {
        @Override
        public void run() {
            mLatencyFull.setText("Full Process Latency: " + String.valueOf(latency/1000000) + " ms");
            mStdFull.setText("Stddev: " + new DecimalFormat("#.##").format(stdDev/1000000) + " ms");
        }
    });
}

@Override
public void updateNetworkStats(final CloudletType cloudletType, RollingAverage rollingAverage) {
    final long stdDev = rollingAverage.getStdDev();
    final long latency;
    latency = rollingAverage.getAverage();

    if(getActivity() == null) {
        Log.w(TAG, "Activity has gone away. Abort UI update");
        return;
    }
    getActivity().runOnUiThread(new Runnable() {
        @Override
        public void run() {
           mLatencyNet.setText("Network Only Latency: " + String.valueOf(latency/1000000) + " ms");
           mStdNet.setText("Stddev: " + new DecimalFormat("#.##").format(stdDev/1000000) + " ms");            }
    });
}
```

## Almost There
<add image>


## Miscellaneous
### Remove "Not implemented" message
Search for the TODO referring to the "Not implemented" message. Paste the following:
```
mStatusText.setText("");
```

### Add getStatsText Method
Search for the TODO referring to "getStatsText". Update the method to look like this:
```
public String getStatsText() {
    return mImageSenderEdge.getStatsText();
}
```

## Define Menu Item Actions
Our Face Detector has an options menu with a few items available, but they currently do nothing. Let's define the actions for each menu item.

Search for the TODO referring to "onOptionsItemSelected". Paste the following:
```
@Override
public boolean onOptionsItemSelected(MenuItem item) {
    int id = item.getItemId();
    if (id == R.id.action_camera_swap) {
        mCamera2BasicFragment.switchCamera();
        SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(getContext());
        String prefKeyFrontCamera = getResources().getString(R.string.preference_fd_front_camera);
        prefs.edit().putInt(prefKeyFrontCamera, mCamera2BasicFragment.getCameraLensFacingDirection()).apply();
        return true;
    } else if (id == R.id.action_camera_video) {
        mCameraToolbar.setVisibility(View.GONE);
        mCamera2BasicFragment.startVideo();
        return true;
    } else if (id == R.id.action_camera_debug) {
        mCamera2BasicFragment.showDebugInfo();
        return true;
    }
    return false;
}
```

## Success!
If you start the Face Detection activity now and point the camera at your face, you should see a cyan oval drawn around it. Move the camera around a bit and notice how the shape tracks your face. Depending on your connection to the Face Detection Server, this tracking may be smooth or laggy.

### Things to Try
* Switch to the rear-facing camera.
* Multiple faces in the same image.
* If you turn on auto-rotation in your phone's settings, landscape mode is supported for face detection.
<add image>

## Going Farther
### Use Closest Cloudlet as Face Detection Host
In this section we will use the results of the findCloudlet SDK call to determine what Face Detection server to connect to.

In **MainActivity.java**, in onNavigationItemSelected(), right before "startActivity(intent);", add the following:
```
if(mClosestCloudlet != null) {
    intent.putExtra("EXTRA_EDGE_CLOUDLET_HOSTNAME", mClosestCloudletHostName);
}
```
This will pass the hostname found by the MatchingEngine's findCloudlet call. Now in **FaceProcessorFragment.java**, find this code:
```
String host = mHostDetectionEdge;
```
Right after that line, paste the following:
```
Intent intent = getActivity().getIntent();
String cloudletHostname = intent.getStringExtra("EXTRA_EDGE_CLOUDLET_HOSTNAME");
if(cloudletHostname != null) {
    host = cloudletHostname;
}
```
Now build and run the app again. After clicking the button to find the closest cloudlet, start the Face Detection activity. You are now connecting to the closest possible Edge host that is running the face detection server. At this point, you should now see much lower latency and smoother face tracking.

### Cloud/Edge Comparison App
If you would like to see a more fully featured face detection app that compares a cloud server to an edge server and demonstrates the latency differences, you can replace your FaceProcessorFragment with one provided by the com.mobiledgex.computervision library. Follow these steps:

1. Open **AndroidManifest.xml**.
2. Add the ImageProcessorActivity activity that is included in the com.mobiledgex.computervision library. We will also include the Settings activity so we can tweak the behavior.
```
<activity
    android:name="com.mobiledgex.computervision.ImageProcessorActivity"
    android:label="Face Detector"
    android:theme="@style/AppTheme.NoActionBar" />
<activity
    android:name="com.mobiledgex.computervision.SettingsActivity"
    android:label="Settings" />
```
3. Open **MainActivity.java**.
4. Add the following import:
```
import com.mobiledgex.computervision.ImageProcessorActivity;
```

5. In **MainActivity.java**, in onNavigationItemSelected(), change the intent to start with ImageProcessorActivity instead of FaceProcessorActivity. Your change should look like this:
```
Intent intent = new Intent(this, ImageProcessorActivity.class);
```

6. Now when you start Face Detection, you will see 2 sets of latency numbers: Cloud in red, and Edge in green. 
7. You can tap the Settings icon to access the preferences for this activity where you can change several things:
  1. Change host names for both Cloud and Edge.
  2. Toggle extra latency statistics fields.
  3. Toggle latency test method.

## Face Recognition and Pose Detection
The com.mobiledgex.computervision library is capable of more than just Face Detection. You can access additional features by starting different included activities.

### Face Recognition
1. Open **MainActivity.java**.
2. In the onNavigationItemSelected method, replace FaceProcessorActivity with ImageProcessorActivity and add a boolean extra named "extra_face_recognition". That section will now look like this:
```
Intent intent = new Intent(this, ImageProcessorActivity.class);
intent.putExtra("EXTRA_FACE_RECOGNITION", true);
```

3. Now the Face Detection activity will start in Recognition mode. If you are a new user who has never submitted face training data, the faces will most likely be identified as "Unknown".
4. If you would like to submit face training data, choose "Sign In with Google" from the main menu and enter your Gmail account credentials.
5. At that point, you can choose "Training Mode" from the options menu in the Face Detection activity.
6. After several images are accepted by the Face Training Server and processed, your face should start to be recognized correctly.

## Pose Detection
1. Open **AndroidManifest.xml**.
2. Add the PoseProcessorActivity that is included in the com.mobiledgex.computervision library.
```
<activity
    android:name="com.mobiledgex.computervision.PoseProcessorActivity"
    android:label="Pose Detector"
    android:theme="@style/AppTheme.NoActionBar" />
```
3. Open **MainActivity.java**.
4. Add the following import:
```
import com.mobiledgex.computervision.PoseProcessorActivity;
```

5. In the onNavigationItemSelected method, redefine the Intent to be launched with the PoseProcessorActivity class. Replace ImageProcessorActivity with PoseProcessorActivity:
```
Intent intent = new Intent(this, PoseProcessorActivity.class);
```

6. Now the Face Detection activity will start in Pose Detection mode.
7. Select the rear camera and point at someone's full body (or a group of people) to see their pose drawn as a colorful skeleton.
swdev > How-To: Android Workshop: Add Face Detection To Workshop App > image2019-4-12_16-1-58.png

TODO: Override hard-coded Bonn cloudlet if additional GPU-enabled cloudlets are available.

## Multi-Purpose App
You can add a new menu entry for each of these different detection modes, and demonstrate any of them without needing code changes in between. Hint: Additional menu items are added in **app/res/menu/activity_main_drawer.xml** and the different ways you launched the intent above, can be added as separate "if else" blocks in onNavigationItemSelected().

## Video Playback
In any of the modes, the camera class is capable of playing a canned video, one of which is included in the library. This is useful for running a demo where the Android phone may be unattended at times. Try it out by tapping the options menu, and selecting "Play Video".

## Another Speed-Up: Persistent TCP Connection
By default, the MobiledgeX Computer Vision library communicates to the server running on the Edge cloudlet via a REST interface. Both our server and our library support a persistent TCP connection mode, which can achieve significant speed improvements. To use the persistent TCP connection mode, we just need to turn on that option. See Computer Vision Persistent TCP Connection for more background and details.

In **FaceProcessorFragment.java**, right after "mImageSenderEdge = new ImageSender", add the following:
```
ImageSender.setPreferencesConnectionMode(ImageSender.ConnectionMode.PERSISTENT_TCP, mImageSenderEdge);
```

## Related articles
The content by label feature displays related articles automatically, based on labels you choose. To edit options for this feature, select the placeholder below and tap the pencil icon.

Related issues	






 
