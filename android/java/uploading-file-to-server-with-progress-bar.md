# Android: Uploading File to Server with Progress Bar

In this tutorial, I will explain how to upload any kind of file from your Android app to PHP server showing the percentage of progress. To begin with, let’s code the server side.

## Creating Android Project

### 1. Create a project named Upload File To Server.
### 2. Add the following dependencies in the build.gradle file:

```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 26
    defaultConfig {
        applicationId "com.androiddeft.uploadfiletoserver"
        minSdkVersion 16
        targetSdkVersion 26
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }

    //For including Apache http libraries
    useLibrary 'org.apache.http.legacy'

}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation 'com.android.support:appcompat-v7:26.1.0'
    implementation 'com.android.support.constraint:constraint-layout:1.0.2'

    //Easy permissions for handling Run time permissions
    implementation 'pub.devrel:easypermissions:0.2.0'

    //Apache httpclient-android library
    implementation group: 'org.apache.httpcomponents', name: 'httpclient-android', version: '4.3.5.1'

    //Excluding httpclient since it is already part of httpclient-android
    implementation('org.apache.httpcomponents:httpmime:4.3') {
        exclude module: "httpclient"
    }
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'com.android.support.test:runner:1.0.1'
    androidTestImplementation 'com.android.support.test.espresso:espresso-core:3.0.1'

}
```

### 3. Add required permissions to AndroidManifest.xml:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.androiddeft.uploadfiletoserver">
 
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
 
    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
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

### 4. Create a new XML file inside the drawable folder with the name ic_file.xml and update it with the following SVG code. This is a vector graphics icon used to display file icon as a preview when a non-image file is chosen to upload.

```xml
<vector android:height="24dp" android:viewportHeight="24.0"
    android:viewportWidth="24.0" android:width="24dp" xmlns:android="http://schemas.android.com/apk/res/android">
    <path android:fillColor="#000000" android:pathData="M6,2c-1.1,0 -1.99,0.9 -1.99,2L4,20c0,1.1 0.89,2 1.99,2L18,22c1.1,0 2,-0.9 2,-2L20,8l-6,-6L6,2zM13,9L13,3.5L18.5,9L13,9z"/>
</vector>
```

### 5. Add the string resources used in the project to strings.xml

```xml
<resources>
    <string name="app_name">Upload File To Server</string>
    <string name="read_file">This app needs access to your file storage so that it can read files.</string>
    <string name="choose_file">Choose File</string>
    <string name="upload">Upload</string>
    <string name="file_name">File Name</string>
</resources>
```

### 6. In activity_main.xml, add 2 buttons, one for choosing file and another for uploading file. 1 ImageView for showing file preview and 1 TextView to display file name.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center_vertical"
    android:orientation="vertical"
    tools:context="com.androiddeft.uploadfiletoserver.MainActivity">


    <Button
        android:id="@+id/btn_choose_file"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="@string/choose_file" />

    <ImageView
        android:id="@+id/iv_preview"
        android:layout_width="100dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:src="@drawable/ic_file"
        android:visibility="gone" />

    <TextView
        android:id="@+id/tv_file_name"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:gravity="center"
        android:text="@string/file_name"
        android:textSize="20sp"
        android:visibility="gone" />

    <Button
        android:id="@+id/btn_upload"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:text="@string/upload"
        android:visibility="gone" />


</LinearLayout>
```

### 7. Create a custom class named MyHttpEntity, which extends HttpEntityWrapper. This helps in calculating the progress percentage while uploading the file.

```java
package com.androiddeft.uploadfiletoserver;

import org.apache.http.HttpEntity;
import org.apache.http.entity.HttpEntityWrapper;

import java.io.FilterOutputStream;
import java.io.IOException;
import java.io.OutputStream;

/**
 * Created by Abhi on 11 Feb 2018 011.
 */

public class MyHttpEntity extends HttpEntityWrapper {

    private ProgressListener progressListener;

    public MyHttpEntity(final HttpEntity entity, final ProgressListener progressListener) {
        super(entity);
        this.progressListener = progressListener;
    }

    @Override
    public void writeTo(OutputStream outStream) throws IOException {
        this.wrappedEntity.writeTo(outStream instanceof ProgressOutputStream ? outStream :
                new ProgressOutputStream(outStream, this.progressListener,
                        this.getContentLength()));
    }

    public interface ProgressListener {
        void transferred(float progress);
    }

    public static class ProgressOutputStream extends FilterOutputStream {

        private final ProgressListener progressListener;

        private long transferred;

        private long total;

        public ProgressOutputStream(final OutputStream outputStream,
                                    final ProgressListener progressListener,
                                    long total) {

            super(outputStream);
            this.progressListener = progressListener;
            this.transferred = 0;
            this.total = total;
        }

        @Override
        public void write(byte[] buffer, int offset, int length) throws IOException {

            out.write(buffer, offset, length);
            this.transferred += length;
            this.progressListener.transferred(this.getCurrentProgress());
        }

        @Override
        public void write(byte[] buffer) throws IOException {

            out.write(buffer);
            this.transferred++;
            this.progressListener.transferred(this.getCurrentProgress());
        }

        private float getCurrentProgress() {
            return ((float) this.transferred / this.total) * 100;
        }
    }
}
```

### 8. Finally, update the MainActivity with the following code. Here when the user clicks on Choose File button, the app asks permission for reading files from the external directory. When the user grants the permission, it prompts with the options like Gallery, File Manager etc so that user can choose the file. Once the user chooses the file, File name and Preview will be displayed. Choose file button will be hidden and Upload file button will get displayed. When the user clicks on the Upload button, the file starts getting uploaded to the server showing the progress percentage.

```java
package com.androiddeft.uploadfiletoserver;

import android.Manifest;
import android.app.Activity;
import android.app.ProgressDialog;
import android.content.ContentResolver;
import android.content.Context;
import android.content.Intent;
import android.database.Cursor;
import android.graphics.Bitmap;
import android.graphics.BitmapFactory;
import android.net.Uri;
import android.os.AsyncTask;
import android.os.Bundle;
import android.provider.MediaStore;
import android.support.annotation.NonNull;
import android.support.v7.app.AppCompatActivity;
import android.util.Log;
import android.view.View;
import android.widget.Button;
import android.widget.ImageView;
import android.widget.TextView;
import android.widget.Toast;

import org.apache.http.HttpEntity;
import org.apache.http.HttpResponse;
import org.apache.http.client.ClientProtocolException;
import org.apache.http.client.HttpClient;
import org.apache.http.client.methods.HttpPost;
import org.apache.http.entity.mime.MultipartEntityBuilder;
import org.apache.http.entity.mime.content.FileBody;
import org.apache.http.impl.client.DefaultHttpClient;
import org.apache.http.util.EntityUtils;

import java.io.File;
import java.io.IOException;
import java.io.UnsupportedEncodingException;
import java.util.List;

import pub.devrel.easypermissions.EasyPermissions;
import pub.devrel.easypermissions.EasyPermissions.PermissionCallbacks;

public class MainActivity extends AppCompatActivity implements PermissionCallbacks {
    private static final String TAG = MainActivity.class.getSimpleName();
    private static final int REQUEST_FILE_CODE = 200;
    private static final int READ_REQUEST_CODE = 300;
    private static final String SERVER_PATH = "http://192.168.43.72:8888/api/upload_files/index.php";
    Button fileBrowseBtn;
    Button uploadBtn;
    ImageView previewImage;
    TextView fileName;
    Uri fileUri;
    private File file;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        fileBrowseBtn = findViewById(R.id.btn_choose_file);
        uploadBtn = findViewById(R.id.btn_upload);
        previewImage = findViewById(R.id.iv_preview);
        fileName = findViewById(R.id.tv_file_name);

        fileBrowseBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                //check if app has permission to access the external storage.
                if (EasyPermissions.hasPermissions(MainActivity.this, Manifest.permission.READ_EXTERNAL_STORAGE)) {
                    showFileChooserIntent();

                } else {
                    //If permission is not present request for the same.
                    EasyPermissions.requestPermissions(MainActivity.this, getString(R.string.read_file), READ_REQUEST_CODE, Manifest.permission.READ_EXTERNAL_STORAGE);
                }

            }
        });

        uploadBtn.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (file != null) {
                    UploadAsyncTask uploadAsyncTask = new UploadAsyncTask(MainActivity.this);
                    uploadAsyncTask.execute();

                } else {
                    Toast.makeText(getApplicationContext(),
                            "Please select a file first", Toast.LENGTH_LONG).show();

                }
            }
        });
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == REQUEST_FILE_CODE && resultCode == Activity.RESULT_OK) {
            fileUri = data.getData();
            previewFile(fileUri);

        }
    }

    /**
     * Show the file name and preview once the file is chosen
     * @param uri
     */
    private void previewFile(Uri uri) {
        String filePath = getRealPathFromURIPath(uri, MainActivity.this);
        file = new File(filePath);
        Log.d(TAG, "Filename " + file.getName());
        fileName.setText(file.getName());

        ContentResolver cR = this.getContentResolver();
        String mime = cR.getType(uri);

        //Show preview if the uploaded file is an image.
        if (mime != null && mime.contains("image")) {
            BitmapFactory.Options options = new BitmapFactory.Options();

            // down sizing image as it throws OutOfMemory Exception for larger
            // images
            options.inSampleSize = 8;

            final Bitmap bitmap = BitmapFactory.decodeFile(filePath, options);

            previewImage.setImageBitmap(bitmap);
        } else {
            previewImage.setImageResource(R.drawable.ic_file);
        }

        hideFileChooser();
    }

    /**
     * Shows an intent which has options from which user can choose the file like File manager, Gallery etc
     */
    private void showFileChooserIntent() {
        Intent fileManagerIntent = new Intent(Intent.ACTION_GET_CONTENT);
        //Choose any file
        fileManagerIntent.setType("*/*");
        startActivityForResult(fileManagerIntent, REQUEST_FILE_CODE);

    }

    /**
     * Returns the actual path of the file in the file system
     *
     * @param contentURI
     * @param activity
     * @return
     */
    private String getRealPathFromURIPath(Uri contentURI, Activity activity) {
        Cursor cursor = activity.getContentResolver().query(contentURI, null, null, null, null);
        String realPath = "";
        if (cursor == null) {
            realPath = contentURI.getPath();
        } else {
            cursor.moveToFirst();
            int idx = cursor.getColumnIndex(MediaStore.Images.ImageColumns.DATA);
            realPath = cursor.getString(idx);
        }
        if (cursor != null) {
            cursor.close();
        }

        return realPath;
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, @NonNull String[] permissions, @NonNull int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
        EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, MainActivity.this);
    }

    @Override
    public void onPermissionsGranted(int requestCode, List<String> perms) {
        showFileChooserIntent();
    }

    @Override
    public void onPermissionsDenied(int requestCode, List<String> perms) {
        Log.d(TAG, "Permission has been denied");
    }

    /**
     * Hides the Choose file button and displays the file preview, file name and upload button
     */
    private void hideFileChooser() {
        fileBrowseBtn.setVisibility(View.GONE);
        uploadBtn.setVisibility(View.VISIBLE);
        fileName.setVisibility(View.VISIBLE);
        previewImage.setVisibility(View.VISIBLE);
    }

    /**
     *  Displays Choose file button and Hides the file preview, file name and upload button
     */
    private void showFileChooser() {
        fileBrowseBtn.setVisibility(View.VISIBLE);
        uploadBtn.setVisibility(View.GONE);
        fileName.setVisibility(View.GONE);
        previewImage.setVisibility(View.GONE);

    }

    /**
     * Background network task to handle file upload.
     */
    private class UploadAsyncTask extends AsyncTask<Void, Integer, String> {

        HttpClient httpClient = new DefaultHttpClient();
        private Context context;
        private Exception exception;
        private ProgressDialog progressDialog;

        private UploadAsyncTask(Context context) {
            this.context = context;
        }

        @Override
        protected String doInBackground(Void... params) {

            HttpResponse httpResponse = null;
            HttpEntity httpEntity = null;
            String responseString = null;

            try {
                HttpPost httpPost = new HttpPost(SERVER_PATH);
                MultipartEntityBuilder multipartEntityBuilder = MultipartEntityBuilder.create();

                // Add the file to be uploaded
                multipartEntityBuilder.addPart("file", new FileBody(file));

                // Progress listener - updates task's progress
                MyHttpEntity.ProgressListener progressListener =
                        new MyHttpEntity.ProgressListener() {
                            @Override
                            public void transferred(float progress) {
                                publishProgress((int) progress);
                            }
                        };

                // POST
                httpPost.setEntity(new MyHttpEntity(multipartEntityBuilder.build(),
                        progressListener));


                httpResponse = httpClient.execute(httpPost);
                httpEntity = httpResponse.getEntity();

                int statusCode = httpResponse.getStatusLine().getStatusCode();
                if (statusCode == 200) {
                    // Server response
                    responseString = EntityUtils.toString(httpEntity);
                } else {
                    responseString = "Error occurred! Http Status Code: "
                            + statusCode;
                }
            } catch (UnsupportedEncodingException | ClientProtocolException e) {
                e.printStackTrace();
                Log.e("UPLOAD", e.getMessage());
                this.exception = e;
            } catch (IOException e) {
                e.printStackTrace();
            }

            return responseString;
        }

        @Override
        protected void onPreExecute() {

            // Init and show dialog
            this.progressDialog = new ProgressDialog(this.context);
            this.progressDialog.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
            this.progressDialog.setCancelable(false);
            this.progressDialog.show();
        }

        @Override
        protected void onPostExecute(String result) {

            // Close dialog
            this.progressDialog.dismiss();
            Toast.makeText(getApplicationContext(),
                    result, Toast.LENGTH_LONG).show();
            showFileChooser();
        }

        @Override
        protected void onProgressUpdate(Integer... progress) {
            // Update process
            this.progressDialog.setProgress((int) progress[0]);
        }
    }

}
```

Here the IP address 192.168.43.72 and port 8888 are my machine IP and ports. You may update it with your machine IP or server domain URL.

Now run the application on your Android device or emulator.

## Demonstration

Below is the demonstration of the app:
![img-01]

-------

[source]: http://www.androiddeft.com/android-uploading-file-to-server-with-progress-bar/
[img-01]: img/Upload-file-to-server.gif

