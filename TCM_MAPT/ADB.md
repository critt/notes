
`adb root`
`adb unroot`
`adb shell`

`adb start-server`
`adb -a nodaemon server`
`adb kill-server`
`top -l 1 | grep adb`
`kill <PID>`

`adb -H 192.168.1.81 -P 5037 shell`

Pull an APK from a device:
`adb shell`
`pm list packages | grep injured`
`pm path b3nac.injuredandroid`
`exit`
`adb pull /path/to/InjuredAndroid-1.0.12-release.apk InjuredAndroid-1.0.12-release.apk`

Launch an exported activity
`am start b3nac.injuredandroid/.b25lActivity` <-- Note the inserted `/`

