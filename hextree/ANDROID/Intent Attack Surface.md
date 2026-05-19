
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
adb shell am start -
```