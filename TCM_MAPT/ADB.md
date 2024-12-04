##### Server
`adb start-server`
`adb -a nodaemon server`
`adb kill-server`
`top -l 1 | grep adb`
`kill <PID>`
##### Shells and privs
`adb root`
`adb unroot`

`adb shell`
`adb devices`
`adb -s emulator-5554 shell`
`adb -H 192.168.1.81 -P 5037 shell`
##### Install an APK
`adb install InjuredAndroid-1.0.12-release.apk`
##### Install a package split into multiple APKs
`adb install-multiple base.apk split_config.en.apk split_config.es.apk split_config.xhdpi.apk`
##### Pull an APK from a device:
`adb shell`
`pm list packages | grep injured`
`pm path b3nac.injuredandroid`
`exit`
`adb pull /path/to/InjuredAndroid-1.0.12-release.apk InjuredAndroid-1.0.12-release.apk`
##### Launch an exported activity
`adb shell`
`am start b3nac.injuredandroid/.b25lActivity` <-- Note the inserted `/`