---
title: Android App Quick Start
---

The following information will help you integrate the BeCo SDK with your app using Android Studio and Java.

### Requirements
 * Android 4.0.3 and above
 * `GPS`, `INTERNET` and `ACCESS_COARSE_LOCATION` permissions

### Create an account

You'll need to [create an account](https://staging.becomap.com) to complete the steps below. Already have an account? [Sign in](https://staging.becomap.com/accounts/login/).

### Steps

++toc ++remove-line-start-steps

## Step 1: Ask for Location Services permissions & check Bluetooth 

The BeCo SDK makes use of Location Services and Bluetooth 

Your **AndroidManifest.xml** should include the following permissions and features:

```groovy
...
</application>

<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.BLUETOOTH"/>
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN"/>
<uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
</manifest>
```

Beginning with Android 6.0, users have to explicitly grant location permission to an app when it is running. If your app has not asked the user for the required permission, you can use the following code snippet to do so:

```java
// Define PERMISSION_REQUEST_CODE field in your class
private static final int PERMISSION_REQUEST_CODE = 1;

// Request for location permissions
if (ActivityCompat.checkSelfPermission(this /*context*/, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED &&
        ActivityCompat.checkSelfPermission(this /*context*/, Manifest.permission.ACCESS_COARSE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
    String[] permissions = new String[]{Manifest.permission.ACCESS_COARSE_LOCATION, Manifest.permission.ACCESS_FINE_LOCATION};
    ActivityCompat.requestPermissions(this, permissions, PERMISSION_REQUEST_CODE /*int requestCode*/);
}

// Request permissions results
@Override
public void onRequestPermissionsResult(int requestCode, String permissions[], int[] grantResults) {
    switch (requestCode) {
        case PERMISSION_REQUEST_CODE:
            Arrays.sort(grantResults);
            if (Arrays.binarySearch(grantResults, PackageManager.PERMISSION_DENIED) >= 0) {
                //Location permission not given by the user. Can try showing rationale to the user.
            } else {
                //Location permissions given by the user.
            }
            break;
    }
}
```

before initialising the SDK you need to check whether bluetooth is enabled or not. 

## Step 2: Integrate the SDK

### Get a Usage Token

<aside class="notice">

[Get Usage Token](https://staging.becomap.com/){className="btn"}

</aside>


### Install the SDK From Maven
Add the following lines in the build.gradle file for your app:

```groovy
repositories {
    maven {url "https://raw.githubusercontent.com/iBeCo/beco-android-sdk-release/master"}
}

dependencies {
    compile 'com.beco:sdk:1.1'
}
```

### The XML layout file

Add the below xml code to your xml file 

```groovy
...
<com.beco.sdk.map.BEMapFragment
        android:id="@+id/beMap"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/theme_background"
        />
```
You can change the ``` android:background ``` to match your theme

### The maps activity Java file
Below is a Java file that defines the maps activity and is named MapsActivity.java. It should contain the following code after your imports. OnMapReadyCallback interface will give you onMapReady when the map is ready and onMapLoaded when the map is loaded

You should initialise your SDK only in onMapReady method as described in the below code

```groovy

import com.beco.sdk.map.BEMapFragment;

public class MapsActivity extends FragmentActivity implements OnMapReadyCallback{
        BEMapFragment beMapFragment;
        
        @Override
        protected void onCreate(Bundle savedInstanceState) {
                super.onCreate(savedInstanceState);
                setContentView(R.layout.activity_maps);
                beMapFragment = (BEMapFragment) findViewById(R.id.beMap);
                //Check your location and bluetooth permissions
                beMapFragment.getMapAsync(this,this); //Activity activity, OnMapReadyCallback mapReadyCallback
        }
        
        @Override
        public void onMapReady() {
                Log.d(TAG,"onMapReady");
                initSDK();
        }
        
        @Override
        public void onMapLoaded() {
                Log.d(TAG,"onMapLoaded");
        }
}
```

### Initialize the SDK

Initialize BeCo with the context and usage token.This will give you a call back with error or success.Use the following code snippet to do so:

```java
private BeCo sdk;
String USAGE_TOKEN = "PASTE_YOUR_USAGE_TOKEN_HERE";
sdk = BeCo.init();
sdk.configure(this, new BeCoCredentialProvider(usageToken), new BEValidateCallback(){
    @Override
    public void onError(BEErrorCode errorCode) {
        Log.d(TAG,"error- The application is not authorized to use the Beco SDK Platform");
    }

    @Override
    public void onLoadFinished() {
        Log.d(TAG,"success");
        getSite();
    }
});
```
Keep in mind that you should initialize the BeCo only *once* in your application. On the success call back you should get sites assosiated with the selected organisation


### Get the Sites

Use the below snippet to get sites.This will give you a call back with error or success(with BESite)

```java
String loc = "lat,lng"
sdk.getSites(loc, new BESiteCallback() {
    @Override
    public void onError(BEErrorCode status) {
        Log.d(TAG,"error");
    }

    @Override
    public void onLoadFinished(List<BESite> sites) {
        Log.d(TAG,"got sites");
        beSiteList = sites;
        Log.d(TAG, beSiteList.toString());
        initMap();
    }
});
```

### Set site to show the map

Select the sites from the getSites call back. Use the following code snippet to do so:

```java
   beMapFragment.setSite(beSiteList.get(0));
```

Once the map is loaded you will get a call back on onMapLoaded() 

## Step 3: Build UI for the point search and navigation

BeCO SDK only provides the UI for current location and floor switch as a widget. Other UI componetes (like search, destination seletion, navigation) as required can be added by the developer

#### Get all points

```java
List<BEPoint> points = beMapFragment.getPoints()
```

#### Select a point on the map

Select the point from the getPoints call back on the map. Use the following code snippet to do so:

```java
beMapFragment.selectPoint(point);
```

#### Clear selected point on the map

```java
beMapFragment.clearPoint()
```

#### Implementing Point click call back

OnPointClickListener defines signatures for methods that are called when a point is clicked or tapped on the map. To get a call back when a point is clicked on the map implement OnPointClickListener on your java class and add the following code snippet

```java
beMapFragment.setOnPointClickListener(this);

@Override
public void onPointClick(BEPoint point) {
        Log.d(TAG,"point selected on Map");
}
```

#### Implementing onDestroy method

Don't forget to call the onDestroy method on your activity Ondestroy. Use the following code snippet to do so:


```java
@Override
protected void onDestroy() {
     Log.d(TAG,"onDestroy in Main activity");
     if(beMapFragment != null){
        beMapFragment.onDestroy();
     }
     super.onDestroy();
}
```

## Step 4: Implementing Current Location Call back

BeCo SDK will give you a current location call back when a beacon is detected. OnCurrentLocationChangeListener defines signatures for methods that are called when a beacon is detected. To get a call back when a beacon is detected on the map implement OnCurrentLocationChangeListener on your java class and add the following code snippet

```java
beMapFragment.setOnCurrentLocationChangeListener(this);

@Override
public void onCurrentLocation(BEPoint point) {
      Log.d(TAG,point.getName());
}
```

## Step 5: Set up navigation

If you have a source and destination point use the below code snippet to getRoute

```java
routeFloors = beMapFragment.getRoute(source,destination); // This will give you the list of floors assosiated with the route
beMapFragment.showRouteOnFloor(routeFloors.get(currentFloor)); // To show the route for a floor in the map

try {
     beMapFragment.navigate(); //start the navigation
} catch (BeaconNotFoundException e) {
     Log.d(TAG,"error- No beacons on the current route");
}
```

OnNavigationListener defines signatures for methods that are called when a navigation in progress.

```java
beMapFragment.setOnNavigationListener(this);
@Override
public void onNavigationEnd() {
     //Called when a navigation ends or manually finished
}
```



## Step 6: Set up Re Routing

OnReRouteClickListener defines signatures for methods that are called when re routing is required.

```java
beMapFragment.setOnReRouteClickListener(this);
@Override
public void onReRoute(BEPoint point) {
        Log.d(TAG,"onReRoute in Main activity"+point.getName());
        final AlertDialog.Builder builder = new AlertDialog.Builder(this);
        builder.setTitle("Do you want to Re Route?");
        builder.setMessage("You have deviated from your route.");
        builder.setPositiveButton(android.R.string.ok, new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialogInterface, int i) {
                //create new route with new source
            }
        });
        builder.setOnDismissListener(dialog -> {
        });
        builder.show();
}
```


## Step 7: Test your app

The BeCo SDK should now be fully integrated in your app. Run the app and view your map. You’re done!


### Next Steps

