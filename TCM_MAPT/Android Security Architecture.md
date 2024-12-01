1. Android = Linux
2. Directory, file, and application permissions are based on the Linux permissions model `(_ xxx xxx xxx)` ![[Screenshot 2024-11-30 at 12.58.39 PM.png]]
3. All applications were run in a virtual machine known as the Dalvik runtime `(DEPRECATED, replaced by ART)`
4. Android Runtime (ART)
	 - The modern translation layer from the application's bytecode to device instructions
	 - Every app runs in its own sandboxed virtual machine
	 - Likewise, in the file system applications are isolated by creating a new user unique for that application

### Android Identity and Access Management
1. As mentioned above, each application gets its own user from the OS
2. This user is the owner of the application directory (UID between `10000` and `999999`, username similar: i.e.` u0_a188` for UID `10188`)
	- `data/app/com.example.app` - generic application data
	- `data/data/com.example.app` - runtime storage data
	- `mnt/sdcard/Android/data/com.example.app` - externally stored location for runtime
	- `data/data/com.example2.app` - a different app requiring a different user
3. This stops applications from interacting with each other unless explicitly granted permissions, or a `Content Provider` / `Broadcast Receiver` is exposed ![[Screenshot 2024-11-30 at 1.15.20 PM.png]]
4. Root user, system level accounts: **This is why using a rooted device for testing is nice**
	1. Emulators w/ non-Google Play APIs allow root
5. Profiles: Separated app data, useful for things like BYOD
	1. Work Profile, Personal Profile, Always-On VPN for certain apps
	2. Still has access to system level functions like wifi, bluetooth, 4G LTE, etc. But can have isolated aspects for data loss prevention purposes like clipboard, contacts, camera etc.
6. Primary User = this is the user created the first time the phone is started, always running, can only be removed by factory reset
7. Secondary User = Additional users you can add to the device, and can be created/deleted by the primary user
8. Guest User - can only be one guest user at a time
9. Kids Mode - Google Kids Spaces (tablets only), Profiles/Accounts for Kids, usually vendor specific (like Samsung Kid Mode).
### Android Architecture
![[Screenshot 2024-11-30 at 1.27.05 PM 1.png]]

##### Linux Kernel
- Android apps support multiple CPU architectures (ARM, SoC, 32bit, 64bit)
- Applications are explicitly told which version of the ART/API level to run on via `AndroidManifest.xml`
- Different API levels have different vulnerabilities and different characteristics
	- Test on multiple devices on multiple API levels
- Access to physical components of the Linux device is controlled by drivers (bluetooth, camera, etc)
##### Hardware Abstraction Layer (HAL)
- Allows applications to access hardware components irrespective of the device manufacturer or type
	- Allows applciations to simply access "the camera", as opposed to using device-specific drivers, for example
- New/Upcoming HAL types:
	- Automotive - Android Auto, Apple Car Play
	- IoT
		- Fitness watches
		- Fitness devices (Peloton, etc)
		- IoT Home devices (Alexa, Google Home, etc)
	- Gaming Peripherals
##### Native C vs ART
- C and C++ are the device's native languages
	- Do not require VMs
	- Webkit - built-in web browser app (iFrame)
	- OpenMAX AL, OpenGL EW - UI frameworks for 2D/3D models
- Java/Kotlin are easier to code in, meaning most developers prefer them instead
##### Java API Framework
- Allows the app to interact with other apps and the device itself
- Content Providers: a way of sharing data to other applications via a specific directory (if exported)
- View System: app UI
- Managers:
	- Notifications
	- Telephony - calls, contacts
	- Package - app package, updates, integrity
	- Location - (as in geolocation, i.e. location services)

##### System Applications Layer
- Pre-installed apps like Contacts, Phone, System Settings etc
- Google apps
- Vendor Specific apps
- The great thing about Android is that you can always set a new default app to replace the vendor-supplied or system app