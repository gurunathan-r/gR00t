

**Hardware**
- Android uses ARM with x86-64 architecture

**Kernel**
- Android uses Linux kernel Long Term Support (LTS) branch

**FIle System**
- Until android 2.0 Android uses YAFFS2 after 2.3 they started using EXT4
- Important file directories and the uses
     - Android - default for app cache and saved data
     - DCIM - stores photos taken by Camera App
     - Downloads - Stores downloaded files
     - Cache - Storage of frequently used data and app components
     - Misc - Contains other important system setting information


**Android Architecture**

![[Android Architecture.png]]

- **Kernel** 
	  Provides an interface between user and the hardware . It has essential driver to communicate between hardware and the user
- **Hardware Abstraction Layer (HAL)** 
	   Logical division of code that serves as an Abstraction layer between physical hardware and its software
- **LIbraries** 
	   provide developer support to develop applications

- **Android Runtime** 
     - A State in which program can send instructions to the computers processor and access the computers RAM
     - Uses Virtual Machine to isolate the applications so that it wont affect the hostOS
     - DVM(Dalvik Virtual Machine) which has beem replaced by android Runtime (ART)
     - **DVM** - uses JIT(Just in time) which works by analysing and actively translating apps during runtime only whereas 
     - **ART** - uses AOT(Ahead of time) compilation that compiles .dex files during instalation and stored in the phone

- **Application Framework** 
     The entire feature-set of the Android OS is available to the developer through APIs
     the components of the Application framework
     - View System - Used to build an app's UI
     - Resource manager - Provides access to non-code resources
     - Activity manager- manages lifecycle of apps
     - Content Provides - Enables apps to access data from other apps like whatsapp access from contacts app



***APK COMPILATION AND DECOMPILATION***


**Compilation** 
  
   - Java code compiled by javac(java compiler)
   - Javac compiles main.java ---> java byte code(main.class)
   - Java Virtual MAchine (JVM) converts Java byte code ---> machine code using(JIT)
   - machine code is run by CPU

![[Java Compilation.png]]



- Android doesnt have JVM as it has only limited processor and RAM so it uses Dalvik VIrtual Machine(DVM)



- ![[Java compilation android.png]]


---


**Decompiling APK**

	**It is to be noted that an APK file is just a ZIP archive that contains XML files, dex code, resource files and other files**.


UNZIP:
```
`d2j-dex2jar classes.dex`
```
```
unzip name.apk
```


APK TOOL:

```
apktool d -rs sample.apk
```
 
USAGE: apktool

usage: apktool if|install-framework [options] <framework.apk>

 -p,--frame-path <dir>   Store framework files into <dir>.
 -t,--tag <tag>          Tag frameworks using <tag>.
usage: apktool d[ecode] [options] <file_apk>
 -f,--force              Force delete destination directory.
 -o,--output <dir>       The name of folder that gets written. (default: apk.out)
 -p,--frame-path <dir>   Use framework files located in <dir>.
 -t,--frame-tag <tag>    Use framework files tagged by <tag>.
usage: apktool b[uild] [options] <app_path>
 -f,--force-all          Skip changes detection and build all files.
 -o,--output <file>      The name of apk that gets written. (default: dist/name.apk)
 -p,--frame-path <dir>   Use framework files located in <dir>.
 
 Dex2Jar

The dex file contains the dalvik byte code to dissabassemble this file into standard class files we can use dex2jar tool
