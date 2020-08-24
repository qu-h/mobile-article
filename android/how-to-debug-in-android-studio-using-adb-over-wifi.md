# How to debug in Android Studio using adb over WiFi

You can find the adb tool in /platform-tools/
```
cd Library/Android/sdk/platform-tools/
```
You can check your devices using:
```
./adb devices
```
My result:
```
List of devices attached
XXXXXXXXX   device
```
Set a TCP port:
```
./adb shell setprop service.adb.tcp.port 4444

./adb tcpip 4444
```
Result message:
```
restarting in TCP mode port: 4444
```
To init a wifi connection you have to check your device IP and execute:
```
./adb connect 192.168.101.123:4444
```
Good luck!