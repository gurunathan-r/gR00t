
## What is an APK?

![[Pasted image 20250903071547.png]]


![[Pasted image 20250903071647.png]]


# Getting APKs from a device

First, find the package name of the application you want to download. You can list all third-party packages using:

```bash
adb shell pm list packages -3
```

Next, we have to get the path to the APK. This can be done using:

```bash
adb shell pm path <package_name>
```

Now we can just pull the APK from the phone:

```bash
adb pull <path_to_apk
```

