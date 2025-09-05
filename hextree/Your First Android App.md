
**Text**

Text change in ==MainActivity.java== which is already written in ==R.id.home_text==

```
public class MainActivity extends AppCompatActivity {  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        EdgeToEdge.enable(this);  
        setContentView(R.layout.activity_main);  
  
        TextView hometext = findViewById(R.id.home_text);  
        hometext.setText("Hi Guys...");  
    }  
}
```


**Button**


```
Button homebutton = findViewById(R.id.button);  
homebutton.setOnClickListener(new View.OnClickListener() {  
    @Override  
    public void onClick(View v) {  
        Log.i("Groot","Wellcome from groot")  
    }  
});
```


Intent

```
public class MainActivity extends AppCompatActivity {  
  
    private int counter;  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        EdgeToEdge.enable(this);  
        setContentView(R.layout.activity_main);  
  
        TextView hometext = findViewById(R.id.home_text);  
        hometext.setText("Hi Guys...");  
  
        Button homebutton = findViewById(R.id.button);  
        homebutton.setOnClickListener(new View.OnClickListener() {  
            @Override  
            public void onClick(View v) {  
                counter++;  
                hometext.setText(String.format("Clicked %d",counter));  
                if(counter==10){  
                    Intent BrowserIntent = new Intent(Intent.ACTION_VIEW, Uri.parse("http://hextree.com"));  
                    startActivity(BrowserIntent);  
                }  
  
            }  
        });  
    }
```


Android Chrome manifest :

![[Pasted image 20250902064526.png]]


See also `AndroidManifest.xml` of Chrome: [https://chromium.googlesource.com/chromium/src/+/b71e98cdf14f18cb967a73857826f6e8c568cea0/chrome/android/java/AndroidManifest.xml#156](https://chromium.googlesource.com/chromium/src/+/b71e98cdf14f18cb967a73857826f6e8c568cea0/chrome/android/java/AndroidManifest.xml#156)



RECIEVE INTENT

Manifest we see there is exported as true which can also be 
EXPOSED ACTIVITIES ATTACK SURFACE
```
<!-- Secret Activity -->  
<activity  
    android:name=".secret"  
    android:exported="true"  
    tools:ignore="ExtraText">  
    <intent-filter>        <action android:name="android.intent.action.SEND" />  
        <data android:mimeType="text/plain" />  
        <category android:name="android.intent.category.DEFAULT" />  
    </intent-filter></activity>
```


Secret.java

```
package com.example.groot;  
  
import android.content.Intent;  
import android.os.Bundle;  
import android.widget.TextView;  
  
import androidx.activity.EdgeToEdge;  
import androidx.appcompat.app.AppCompatActivity;  
import androidx.core.graphics.Insets;  
import androidx.core.view.ViewCompat;  
import androidx.core.view.WindowInsetsCompat;  
  
public class secret extends AppCompatActivity {  
  
    @Override  
    protected void onCreate(Bundle savedInstanceState) {  
        super.onCreate(savedInstanceState);  
        setContentView(R.layout.activity_secret);  
        Intent recievedIntent = getIntent();  
        String sharedText = recievedIntent.getStringExtra(Intent.EXTRA_TEXT);  
        if(sharedText!=null){  
            TextView debugtext =findViewById(R.id.secret);  
            debugtext.setText(new StringBuilder().append("TEXT:").append(sharedText).toString());  
  
  
        }  
    }  
}
```

First of all we have to `export` an activity that they can be started by other applications. By adding `intent-filter` we can also declare that our activity can handle certain intents:

```xml
// AndroidManifest.xml
<intent-filter>
    <action android:name="android.intent.action.SEND" />
    <data android:mimeType="text/plain" />
    <category android:name="android.intent.category.DEFAULT" />
</intent-filter>
```
	
If another application sent us an intent, we can handle this in our activity code by calling `getIntent()`.



## Using Android Studio's Debugger

allows us to set a breakoint in any line of code and run the application till that line..


# Android Challenge 1

link: https://github.com/hextreeio/android-challenge1

Where we were given a apk with 3 java files.. 
Main Activity , CHallenge Activity , Flag Activity..

So There are many ways to solve it but the way i used is to modify the conditions of the test cases to call the other Activities for example 

in Main Activity..

```
int counter = 0;  
@Override  
protected void onCreate(Bundle savedInstanceState) {  
    super.onCreate(savedInstanceState);  
    setContentView(R.layout.activity_main);  
  
    // this is the main activity... how to get to the challenge?  
    TextView text = findViewById(R.id.main_text);  
    text.setOnClickListener(new View.OnClickListener() {  
        @Override  
        public void onClick(View v) {  
            counter++;  
            text.setText("Counter: "+counter);  
            if(counter>9999) {  
                startActivity(new Intent(MainActivity.this, ChallengeActivity.class));  
            }  
        }
```


I Just changed that ***counter*** value to 0 
```
if(counter>0) {  
                startActivity(new Intent(MainActivity.this, ChallengeActivity.class));  
            }  
```


so that when the activity is first started the click on the text increases the counter from 0 to 1

```
text.setOnClickListener(new View.OnClickListener() {  
        @Override  
        public void onClick(View v) {  
            counter++;  
```


so that when the test case is tested it becomes true and the ***ChallengeActivity.class*** is called with Intent..



In The ChallengeActivity..


There were 10 buttons in the class but only the ***9 th*** Button triggers the ***FlagActivity.class***

```
findViewById(R.id.button9).setOnClickListener(new View.OnClickListener() {  
    @Override  
    public void onClick(View v) {  
        startActivity(new Intent(ChallengeActivity.this, FlagActivity.class));  
    }  
});
```

so we can call it by a click ...


In the FlagActivity...

We can see there is a ***decryptflag()*** function present which is called when the progress on the seekbar is 42

```
  
        @Override  
        public void onStopTrackingTouch(SeekBar seekBar) {  
            // Success!!! Show the flag now!  
            if(progress>42) {  
                text.setText(decryptFlag());  
            }  
        }  
    });  
}
```

so here we can change that condition completely to 
```
		if(1==1) 
		{   text.setText(decryptFlag());   }  
```

Now when the seekbar is touched and moved a little bit we can see the flag printed on the screen..

![[Pasted image 20250903070621.png]]

And that's it we have completed the Android Challenge 1 :) 