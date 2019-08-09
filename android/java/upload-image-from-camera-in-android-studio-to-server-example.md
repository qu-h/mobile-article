# Upload Image From Camera In Android Studio To PHP Server Example

Hello, Geeks. You are at upload image from camera in Android Studio example.

In upload image from camera in Android tutorial, learn how to choose image from camera and then send or upload it to PHP-MySQL server.

First, we will upload image to server then we will fetch or load uploaded image into ImageView using AQuery Library.

You will get the professional format to call remote Web Services in proper and easiest way at the end of upload image from camera in Android example.

First, check output of upload image from camera in Android example, then we will develop it.

If you want to select image from gallery then refer this: upload image from gallery in android.

## Creating Upload Image From Camera In Android Studio Tutorial step by step

### Developing PHP Script

Make a new PHP file named “config.php” and copy below

```php
<?php
$host="localhost";
$user="your username";
$password="your password";
$db = "your db name";
 
$con = mysqli_connect($host,$user,$password,$db);
 
// Check connection
if (mysqli_connect_errno())
  {
  echo "Failed to connect to MySQL: " . mysqli_connect_error();
  }else{  //echo "Connect"; 
  
   
   }
 
?>
```

Create a new PHP file and name it “uploadfile.php” and add below source code

```php
<?php
 if($_SERVER['REQUEST_METHOD']=='POST'){
  	// echo $_SERVER["DOCUMENT_ROOT"];  // /home1/demonuts/public_html
	//including the database connection file
  	include_once("config.php");
  	  	
  	//$_FILES['image']['name']   give original name from parameter where 'image' == parametername eg. city.jpg
  	//$_FILES['image']['tmp_name']  temporary system generated name
  
        $originalImgName= $_FILES['filename']['name'];
        $tempName= $_FILES['filename']['tmp_name'];
        $folder="uploadedFiles/";
         $url = "https://www.demonuts.com/Demonuts/JsonTest/Tennis/uploadedFiles/".$originalImgName;
        
        if(move_uploaded_file($tempName,$folder.$originalImgName)){
                $query = "INSERT INTO upload_image_video (pathToFile) VALUES ('$url')";
                if(mysqli_query($con,$query)){
                
                	 $query= "SELECT * FROM upload_image_video WHERE pathToFile='$url'";
	                 $result= mysqli_query($con, $query);
	                 $emparray = array();
	                     if(mysqli_num_rows($result) > 0){  
	                     while ($row = mysqli_fetch_assoc($result)) {
                                     $emparray[] = $row;
                                   }
                                   echo json_encode(array( "status" => "true","message" => "Successfully file added!" , "data" => $emparray) );
                                   
	                     }else{
	                     		echo json_encode(array( "status" => "false","message" => "Failed!") );
	                     }
			   
                }else{
                	echo json_encode(array( "status" => "false","message" => "Failed!") );
                }
        	//echo "moved to ".$url;
        }else{
        	echo json_encode(array( "status" => "false","message" => "Failed!") );
        }
  }
?>
```

I have a folder “uploadedFiles,” in which all uploaded images will be saved.

Change this line as per your folder directory

```php
 $url = "https://www.demonuts.com/Demonuts/JsonTest/Tennis/uploadedFiles/".$originalImgName;

```

### Step 1: Create a new project in Android Studio.
### Step 2: Updating AndroidManifest.xml file
Add required permissions between `<manifest>….</manifest>` tag.
```xml
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
```
Note: If you are targeting SDK version above 22 (Above Lollipop)  then you need to ask a user for granting runtime permissions. Check marshmallow runtime permission for more information.
Final code for AndroidManifest.xml file

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.exampledemo.parsaniahardik.uploadimagecamera">
 
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
 
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">
        <activity android:name=".MainActivity">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
 
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
    </application>
 
</manifest>
```

## Step 3: Updating build.gradle(Project: project_name) file:

Add the following

```gradle
maven { url 'https://jitpack.io' }
```

in the below structure.


```gradle
allprojects {
    repositories {
        jcenter()
        
    }
}
```

So final code for build.gradle(Project: project_name)  will look like this:

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.
 
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.5.0'
 
        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}
 
allprojects {
    repositories {
        jcenter()
        maven { url 'https://jitpack.io' }
    }
}
 
task clean(type: Delete) {
    delete rootProject.buildDir
}
```

## Step 4: Updating build.gradle(Module:app) file

Add following code into dependencies{} 

```gradle
 compile group: 'org.apache.httpcomponents' , name: 'httpclient-android' , version: '4.3.5.1'
    compile('org.apache.httpcomponents:httpmime:4.3') {
        exclude module: "httpclient"
    }
 compile 'com.github.androidquery:androidquery:0.26.9'
```

Add below code
```gradle
android{
        useLibrary  'org.apache.http.legacy'
    }
    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/NOTICE'
    }
```

in the parent, android{} as below
```gradle
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"
    ...
}
```

Final source code for build.gradle(Module:app) will be:
```gradle
apply plugin: 'com.android.application'
 
android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"
 
    defaultConfig {
        applicationId "com.exampledemo.parsaniahardik.uploadimagecamera"
        minSdkVersion 16
        targetSdkVersion 22
        versionCode 1
        versionName "1.0"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    android{
        useLibrary  'org.apache.http.legacy'
    }
    packagingOptions {
        exclude 'META-INF/LICENSE'
        exclude 'META-INF/NOTICE'
    }
}
 
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:appcompat-v7:23.4.0'
 
    compile group: 'org.apache.httpcomponents' , name: 'httpclient-android' , version: '4.3.5.1'
    compile('org.apache.httpcomponents:httpmime:4.3') {
        exclude module: "httpclient"
    }
    compile 'com.github.androidquery:androidquery:0.26.9'
}
```

## Step 5: Adding common classes
We will add some common classes which contain constant variables and public methods.

We can use these variables and methods anywhere in the whole project so that it will reduce data redundancy.

Means that we will write them only once and then we can use them anytime and anywhere when needed.

### Names of the classes are:
1. AndyUtils
2. AsyncTaskCompleteListener (Interface)
3. MultiPartRequester
4. ParseContent

## Step 6: Creating AndyUtils
Create a Java class named AndyUtils and add below source code

```java
import android.app.ProgressDialog;
import android.content.Context;
import android.net.ConnectivityManager;
import android.net.NetworkInfo;
 
public class AndyUtils {
 
    private static ProgressDialog mProgressDialog;
 
    public static void showSimpleProgressDialog(Context context, String title,
                                                String msg, boolean isCancelable) {
        try {
            if (mProgressDialog == null) {
                mProgressDialog = ProgressDialog.show(context, title, msg);
                mProgressDialog.setCancelable(isCancelable);
            }
 
            if (!mProgressDialog.isShowing()) {
                mProgressDialog.show();
            }
 
        } catch (IllegalArgumentException ie) {
            ie.printStackTrace();
        } catch (RuntimeException re) {
            re.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
    public static void showSimpleProgressDialog(Context context) {
        showSimpleProgressDialog(context, null, "Loading...", false);
    }
    public static void removeSimpleProgressDialog() {
        try {
            if (mProgressDialog != null) {
                if (mProgressDialog.isShowing()) {
                    mProgressDialog.dismiss();
                    mProgressDialog = null;
                }
            }
        } catch (IllegalArgumentException ie) {
            ie.printStackTrace();
 
        } catch (RuntimeException re) {
            re.printStackTrace();
        } catch (Exception e) {
            e.printStackTrace();
        }
 
    }
 
    public static boolean isNetworkAvailable(Context context) {
        ConnectivityManager connectivity = (ConnectivityManager) context
                .getSystemService(Context.CONNECTIVITY_SERVICE);
        if (connectivity == null) {
            return false;
        } else {
            NetworkInfo[] info = connectivity.getAllNetworkInfo();
            if (info != null) {
                for (int i = 0; i < info.length; i++) {
                    if (info[i].getState() == NetworkInfo.State.CONNECTED) {
                        return true;
                    }
                }
            }
        }
        return false;
    }
 
}
```

This class contains a methods to show(showSimplrProgressDialog()) and remove(removeSimpleProgressDialog()) progress dialog when app is fetching JSON data from server.

AndyUtils also includes a method (isNetworkAvailable()) to check whether the Internet of Android device is on or off

## Step 7: Creating AsyncTaskCompleteListener Interface

Prepare new interface named AsyncTaskCompleteListener and add following source code

```java
public interface AsyncTaskCompleteListener {
    void onTaskCompleted(String response, int serviceCode);
}
```

## Step 8: Creating MultiPartRequester
Create a new Java class named “MultiPartRequester” and add below

```java
import android.app.Activity;
import android.app.ActivityManager;
import android.content.Context;
import android.os.AsyncTask;
import android.util.Log;
import android.widget.Toast;
import org.apache.http.HttpResponse;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.ContentType;
import org.apache.http.entity.mime.MIME;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.params.HttpConnectionParams;
import org.apache.http.util.EntityUtils;
import java.io.File;
import java.util.Map;
 
public class MultiPartRequester {
 
    private Map<String, String> map;
    private AsyncTaskCompleteListener mAsynclistener;
    private int serviceCode;
    private HttpClient httpclient;
    private Activity activity;
    private AsyncHttpRequest request;
 
    public MultiPartRequester(Activity activity, Map<String, String> map,
                              int serviceCode, AsyncTaskCompleteListener asyncTaskCompleteListener) {
        this.map = map;
        this.serviceCode = serviceCode;
        this.activity = activity;
 
        // is Internet Connection Available...
 
        if (AndyUtils.isNetworkAvailable(activity)) {
            mAsynclistener = (AsyncTaskCompleteListener) asyncTaskCompleteListener;
            request = (AsyncHttpRequest) new AsyncHttpRequest().execute(map
                    .get("url"));
        } else {
            showToast("Network is not available!!!");
        }
 
    }
 
    class AsyncHttpRequest extends AsyncTask<String, Void, String> {
 
        @Override
        protected String doInBackground(String... urls) {
            map.remove("url");
            try {
 
                HttpPost httppost = new HttpPost(urls[0]);
                httpclient = new DefaultHttpClient();
 
                HttpConnectionParams.setConnectionTimeout(
                        httpclient.getParams(), 600000);
 
                MultipartEntityBuilder builder = MultipartEntityBuilder
                        .create();
 
                for (String key : map.keySet()) {
 
                    if (key.equalsIgnoreCase("filename")) {
                        File f = new File(map.get(key));
 
                        builder.addBinaryBody(key, f,
                                ContentType.MULTIPART_FORM_DATA, f.getName());
                    } else {
                        builder.addTextBody(key, map.get(key), ContentType
                                .create("text/plain", MIME.DEFAULT_CHARSET));
                    }
                    Log.d("TAG", key + "---->" + map.get(key));
                    // System.out.println(key + "---->" + map.get(key));
                }
 
                httppost.setEntity(builder.build());
 
                ActivityManager manager = (ActivityManager) activity
                        .getSystemService(Context.ACTIVITY_SERVICE);
 
                if (manager.getMemoryClass() < 25) {
                    System.gc();
                }
                HttpResponse response = httpclient.execute(httppost);
 
                String responseBody = EntityUtils.toString(
                        response.getEntity(), "UTF-8");
 
                return responseBody;
 
            } catch (Exception e) {
                e.printStackTrace();
            } catch (OutOfMemoryError oume) {
                System.gc();
 
                Toast.makeText(
                        activity.getParent().getParent(),
                        "Run out of memory please colse the other background apps and try again!",
                        Toast.LENGTH_LONG).show();
            } finally {
                if (httpclient != null)
                    httpclient.getConnectionManager().shutdown();
 
            }
            return null;
        }
 
        @Override
        protected void onPostExecute(String response) {
 
            if (mAsynclistener != null) {
                mAsynclistener.onTaskCompleted(response, serviceCode);
            }
        }
    }
    private void showToast(String msg) {
        Toast.makeText(activity, msg, Toast.LENGTH_SHORT).show();
    }
 
}
```

We will use methods of this class to establish a connection between an Android device and web server.

## Step 9: Creating  ParseContent
Open new Java class and give it a name ParseContent, then Add below source code

```java
import android.app.Activity;
import org.json.JSONArray;
import org.json.JSONException;
import org.json.JSONObject;
import java.util.ArrayList;
import java.util.HashMap;
 
public class ParseContent {
 
    private final String KEY_SUCCESS = "status";
    private final String KEY_MSG = "message";
    private Activity activity;
 
    ArrayList<HashMap<String, String>> arraylist;
 
    public ParseContent(Activity activity) {
        this.activity = activity;
    }
 
   public boolean isSuccess(String response) {
 
        try {
            JSONObject jsonObject = new JSONObject(response);
            if (jsonObject.optString(KEY_SUCCESS).equals("true")) {
                return true;
            } else {
 
                return false;
            }
 
        } catch (JSONException e) {
            e.printStackTrace();
        }
        return false;
    }
 
   public String getErrorCode(String response) {
 
        try {
            JSONObject jsonObject = new JSONObject(response);
            return jsonObject.getString(KEY_MSG);
 
        } catch (JSONException e) {
            e.printStackTrace();
        }
        return "No data";
    }
 
   public String getURL(String response) {
      String url="";
        try {
            JSONObject jsonObject = new JSONObject(response);
            jsonObject.toString().replace("\\\\","");
            if (jsonObject.getString(KEY_SUCCESS).equals("true")) {
 
                arraylist = new ArrayList<HashMap<String, String>>();
                JSONArray dataArray = jsonObject.getJSONArray("data");
 
                for (int i = 0; i < dataArray.length(); i++) {
                    JSONObject dataobj = dataArray.getJSONObject(i);
                    url = dataobj.optString("pathToFile");
                }
            }
        } catch (JSONException e) {
            e.printStackTrace();
        }
        return url;
    }
 
}
```

In above source code, isSuccess(String response)  method is used to check whether a status of response is true or false.

getErrorCode(String response) method is used to get the message of JSON data.

getURL(String response) method will parse JSON data.

If you want to learn how to parse JSON, then refer this example: JSON Parsing In Android

## Step 10: Updating activity_main.xml file
Copy and paste below source code in activity_main.xml file

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    tools:context="com.exampledemo.parsaniahardik.uploadgalleryimage.MainActivity">
 
    <Button
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:id="@+id/btn"
        android:layout_gravity="center_horizontal"
        android:layout_marginTop="20dp"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:text="Capture Image and upload to server" />
 
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Below image is fetched from server"
        android:layout_marginTop="5dp"
        android:textSize="23sp"
        android:gravity="center"
        android:textColor="#000"/>
 
    <ImageView
        android:layout_width="300dp"
        android:layout_height="300dp"
        android:layout_gravity="center"
        android:layout_marginTop="10dp"
        android:scaleType="fitXY"
        android:src="@mipmap/ic_launcher"
        android:id="@+id/iv"/>
 
</LinearLayout>
```

## Step 11: Preparing MainActivity.java class
Add following source code in MainActivity.java class

```java
import android.content.Intent;
import android.graphics.Bitmap;
import android.media.MediaScannerConnection;
import android.os.Environment;
import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.Toast;
import com.androidquery.AQuery;
import org.json.JSONException;
import java.io.ByteArrayOutputStream;
import java.io.File;
import java.io.FileOutputStream;
import java.io.IOException;
import java.util.Calendar;
import java.util.HashMap;
 
public class MainActivity extends AppCompatActivity implements AsyncTaskCompleteListener{
 
    private ParseContent parseContent;
    private Button btn;
    private ImageView imageview;
    private static final String IMAGE_DIRECTORY = "/demonuts_upload_camera";
    private final int CAMERA = 1;
    private AQuery aQuery;
 
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
 
        parseContent = new ParseContent(this);
        aQuery = new AQuery(this);
 
        btn = (Button) findViewById(R.id.btn);
        imageview = (ImageView) findViewById(R.id.iv);
 
        btn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Intent intent = new Intent(android.provider.MediaStore.ACTION_IMAGE_CAPTURE);
                startActivityForResult(intent, CAMERA);
            }
        });
 
    }
 
    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
 
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == this.RESULT_CANCELED) {
            return;
        }
        if (requestCode == CAMERA) {
            Bitmap thumbnail = (Bitmap) data.getExtras().get("data");
            String path = saveImage(thumbnail);
            try {
                uploadImageToServer(path);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    }
 
    private void uploadImageToServer(final String path) throws IOException, JSONException {
 
        if (!AndyUtils.isNetworkAvailable(MainActivity.this)) {
            Toast.makeText(MainActivity.this, "Internet is required!", Toast.LENGTH_SHORT).show();
            return;
        }
 
        HashMap<String, String> map = new HashMap<String, String>();
        map.put("url", "https://demonuts.com/Demonuts/JsonTest/Tennis/uploadfile.php");
        map.put("filename", path);
        new MultiPartRequester(this, map, CAMERA, this);
        AndyUtils.showSimpleProgressDialog(this);
    }
 
    @Override
    public void onTaskCompleted(String response, int serviceCode) {
        AndyUtils.removeSimpleProgressDialog();
        Log.d("res", response.toString());
        switch (serviceCode) {
 
            case CAMERA:
                if (parseContent.isSuccess(response)) {
                    String url = parseContent.getURL(response);
                    aQuery.id(imageview).image(url);
                }
        }
    }
 
    public String saveImage(Bitmap myBitmap) {
        ByteArrayOutputStream bytes = new ByteArrayOutputStream();
        myBitmap.compress(Bitmap.CompressFormat.JPEG, 90, bytes);
        File wallpaperDirectory = new File(
                Environment.getExternalStorageDirectory() + IMAGE_DIRECTORY);
        // have the object build the directory structure, if needed.
        if (!wallpaperDirectory.exists()) {
            wallpaperDirectory.mkdirs();
        }
 
        try {
            File f = new File(wallpaperDirectory, Calendar.getInstance()
                    .getTimeInMillis() + ".jpg");
            f.createNewFile();
            FileOutputStream fo = new FileOutputStream(f);
            fo.write(bytes.toByteArray());
            MediaScannerConnection.scanFile(this,
                    new String[]{f.getPath()},
                    new String[]{"image/jpeg"}, null);
            fo.close();
            Log.d("TAG", "File Saved::--->" + f.getAbsolutePath());
 
            return f.getAbsolutePath();
        } catch (IOException e1) {
            e1.printStackTrace();
        }
        return "";
    }
}
```

## Step 12: Description of MainActivity.java
MainActivity implements AsynTaskCompleteListener interface. So we will have to override onTaskCompleted() method.

```java
public class MainActivity extends AppCompatActivity implements AsyncTaskCompleteListener
```

When the button is clicked, Camera intent will be opened as following.

```java
Intent intent = new Intent(android.provider.MediaStore.ACTION_IMAGE_CAPTURE);
startActivityForResult(intent, CAMERA);
```

When user captures the image, compiler comes to onActivityResult() method as below.

```java
 @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
 
        super.onActivityResult(requestCode, resultCode, data);
        if (resultCode == this.RESULT_CANCELED) {
            return;
        }
        if (requestCode == CAMERA) {
            Bitmap thumbnail = (Bitmap) data.getExtras().get("data");
            String path = saveImage(thumbnail);
            try {
                uploadImageToServer(path);
            } catch (IOException e) {
                e.printStackTrace();
            } catch (JSONException e) {
                e.printStackTrace();
            }
        }
    }
```

Here, image is saved and uploaded to server.

saveImage(bitmap)  method will save the image as below.

```java
public String saveImage(Bitmap myBitmap) {
        ByteArrayOutputStream bytes = new ByteArrayOutputStream();
        myBitmap.compress(Bitmap.CompressFormat.JPEG, 90, bytes);
        File wallpaperDirectory = new File(
                Environment.getExternalStorageDirectory() + IMAGE_DIRECTORY);
        // have the object build the directory structure, if needed.
        if (!wallpaperDirectory.exists()) {
            wallpaperDirectory.mkdirs();
        }
 
        try {
            File f = new File(wallpaperDirectory, Calendar.getInstance()
                    .getTimeInMillis() + ".jpg");
            f.createNewFile();
            FileOutputStream fo = new FileOutputStream(f);
            fo.write(bytes.toByteArray());
            MediaScannerConnection.scanFile(this,
                    new String[]{f.getPath()},
                    new String[]{"image/jpeg"}, null);
            fo.close();
            Log.d("TAG", "File Saved::--->" + f.getAbsolutePath());
 
            return f.getAbsolutePath();
        } catch (IOException e1) {
            e1.printStackTrace();
        }
        return "";
    }
```

In above code, IMAGE_DIRECTORY is the folder name, in which images will be saved.

uploadImageToServer() method will call Web Service.

```java
private void uploadImageToServer(final String path) throws IOException, JSONException {
 
        if (!AndyUtils.isNetworkAvailable(MainActivity.this)) {
            Toast.makeText(MainActivity.this, "Internet is required!", Toast.LENGTH_SHORT).show();
            return;
        }
 
        HashMap<String, String> map = new HashMap<String, String>();
        map.put("url", "https://demonuts.com/Demonuts/JsonTest/Tennis/uploadfile.php");
        map.put("filename", path);
        new MultiPartRequester(this, map, CAMERA, this);
        AndyUtils.showSimpleProgressDialog(this);
    }
```

From here, compiler will go to onTaskCompleted() method.

```java
 @Override
    public void onTaskCompleted(String response, int serviceCode) {
        AndyUtils.removeSimpleProgressDialog();
        Log.d("res", response.toString());
        switch (serviceCode) {
 
            case CAMERA:
                if (parseContent.isSuccess(response)) {
                    String url = parseContent.getURL(response);
                    aQuery.id(imageview).image(url);
                }
        }
    }
```

The following line will fetch image from URL.

```java
aQuery.id(imageview).image(url);
```
So enough for upload image from camera in android example.

Feel free to comment your queries and reviews in below comment section. Thank you.


### Download [Source Code][code]
--------------------
[source]: https://demonuts.com/upload-image-camera/
[code]: https://demonuts.com/wp-content/uploads/2017/04/UploadImageCamera.zip