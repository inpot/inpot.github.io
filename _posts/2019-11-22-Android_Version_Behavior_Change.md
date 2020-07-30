Android Version Behavior Change

# Android 10.0

1. Dark Theme
2. Gesture navigation
3. ART optimizations
4. Foldable Device support
5. Restriction of Non SDK APIs
6. TLS 1.3 enabled by default
7. Removed execute permission for app home directory

# 9.0 Pie

1. Privacy. Limited access to sensors in background;Restricted access to call logs,phone numbers,
2. Restrictions on use of non-SDK interfaces
3. Security behavior changes. DOT or DOH

# 8.0 Oreo

1. Background execution limit.  Background Apps can not receive most of   implicit broadcasts .
2. Background location limit
3. App shotcuts
4. AlertWindow
5. Privacy. Use ```ANDROID_ID``` for device  identifier. ```READPHONE_STATE``` permisson required if you need device serial, and should use ``Build.getSerial()`` 

# 7.0 Nougat

1. Doze,  not necessarily stationary 
2. Background Optimizations. Apps can receive ```CONNECTIVITY_ACTION``` broadcast only which use ```Context.registReceiver()``` registed. ```ACTION_NEW_PICTURE``` and ```ACTION_NEW_VIDEO``` has been removed
3. Permissions Changes. Apps should use ```FileProvider``` instead ```"file://"``` to share files.

# 6.0 Marshmallow

1. Runtime Permission
2. Doze and App Stanby.  unplugs and  and leaves it stationary, with its screen off, for a period of time, the device goes into *Doze* mode, Doze not allow ```Jobschduler``` and ```AlarmManager``` to run.  If user is not actively using an app,  this app will be Stanby.
3. Apache Http Client Remove
4. use BoringSSL from OpenSSL
5. HW Identifier, ```WifiInfo.getMacAddress()``` and ```BluetoothAdapter.getAddress()``` will return ```" 02:00:00:00:00:00 " ```

# 5.0 Lollipop

1. Material Design
2. use ART Runtime instead Dalvik, which support AOT, JIT.
3. Job Scheduling API, allow you optimize battery life by deferring jobs for the system to run at a later time or under specified conditions, such as when the device is charging or connected to Wi-Fi  

