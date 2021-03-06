# Material Camera

Android's video recording APIs are very difficult to figure out, especially since a lot of manufacturers
like to mount their camera sensors upside down or sideways. This library is a result of lots of research
and experimentation to get video recording to work universally.

![Art](https://raw.githubusercontent.com/afollestad/material-camera/master/art/screen.png)

---

# Basics

```java
private final static int CAMERA_RQ = 6969; 

File saveFolder = new File(Environment.getExternalStorageDirectory(), "MaterialCamera Sample");
if (!saveFolder.mkdirs())
    throw new RuntimeException("Unable to create save directory, make sure WRITE_EXTERNAL_STORAGE permission is granted.");

new MaterialCamera(this)                       // Constructor takes an Activity
    .allowRetry(true)                          // Whether or not 'Retry' is visible during playback
    .autoSubmit(false)                         // Whether or not user is allowed to playback videos after recording. This can affect other things, discussed in the next section.
    .saveDir(saveFolder)                       // The folder recorded videos are saved to
    .primaryColorAttr(R.attr.colorPrimary)     // The theme color used for the camera, defaults to colorPrimary of Activity in the constructor
    .showPortraitWarning(true)                 // Whether or not a warning is displayed if the user presses record in portrait orientation
    .defaultToFrontFacing(false)               // Whether or not the camera will initially show the front facing camera
    .start(CAMERA_RQ);                         // Starts the camera activity, the result will be sent back to the current Activity
```

---

# Length Limiting

You can specify a time limit for recording. `lengthLimitMillis(long)`, `lengthLimitSeconds(float)`, 
and `lengthLimitMinutes(float)` are all methods for length limiting.

```java
new MaterialCamera(this)
    .lengthLimitMinutes(2.5f)
    .start(CAMERA_RQ);
```

When the time limit reaches 0, recording stops. There are different behaviors that can occur after this based on
`autoSubmit` and `autoRetry`:

1. `autoSubmit(false)`, `allowRetry(true)`
    * The user will be able to playback the recording, and the 'Retry' button will be visible. This is default behavior.
2. `autoSubmit(false)`, `allowRetry(false)`
    * The user will be able to playback the recording, but the 'Retry' button will be hidden.
3. `autoSubmit(true)`, `allowRetry(false)`
    * The user won't be able to playback the recording, the result will immediately be returned to the starting Activity.
4. `autoSubmit(true)`, `allowRetry(true)`
    * If you don't specify a length limit, the behavior will be the same as number 1. If you do specify a length limit, the user is allowed to retry, but the countdown timer will continue until it reaches 0. When the countdown is complete, the result will be returned to the starting Activity automatically.

---

# Receiving Results

```java
public class MainActivity extends AppCompatActivity {

    private final static int CAMERA_RQ = 6969;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        new MaterialCamera(this)
            .start(CAMERA_RQ);
    }

    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        // Received recording or error from MaterialCamera
        if (requestCode == CAMERA_RQ) {
        
            if (resultCode == RESULT_OK) {
                Toast.makeText(this, "Saved to: " + data.getDataString(), Toast.LENGTH_LONG).show();
            } else if(data != null) {
                Exception e = (Exception) data.getSerializableExtra(MaterialCamera.ERROR_EXTRA);
                e.printStackTrace();
                Toast.makeText(this, e.getMessage(), Toast.LENGTH_LONG).show();
            }
        }
    }
}
```