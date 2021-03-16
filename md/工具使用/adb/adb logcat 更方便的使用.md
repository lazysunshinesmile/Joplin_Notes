adb logcat 更方便的使用

adb logcat 更方便的使用

Package names are guaranteed to be unique so you can **use the `Log` function with the tag as your package name and then filter by package name**:

*NOTE: As of Build Tools 21.0.3 this will no longer work as TAGS are restricted to 23 characters or less.*

`Log.<log level>("<your package name>", "message");`
`adb -d logcat <your package name>:<log level> *:S`

`-d` denotes an actual device and `-e` denotes an emulator. If there's more than 1 emulator running you can use `-s emulator-<emulator number>` (eg, `-s emulator-5558`)

Example: `adb -d logcat com.example.example:I *:S`

Or if you are using `System.out.print` to send messages to the log you can use `adb -d logcat System.out:I *:S` to show only calls to System.out.

You can find all the log levels and more info here: https://developer.android.com/studio/command-line/logcat.html

http://developer.android.com/reference/android/util/Log.html

EDIT: Looks like I jumped the gun a little and just realized you were asking about logcat in Eclipse. What I posted above is for using logcat through adb from the command line. I'm not sure if the same filters transfer over into Eclipse.

加颜色
 adb logcat -v color | grep callhistory