![[Screenshot 2024-12-13 at 3.08.33 PM.png]]
### High-level themes
- Application install is much more restricted on iOS. There are three ways to install an application:
	- from the App Store
	- by jailbreaking the device
	- by side loading the application
- Unlike Android, iOS employs hardware security features
	- Stuff like signing code with physical component IDs specific to a given device
	- I.e. replacing components on many iOS devices will brick them
- Apps are run in sandboxes
- All apps are signed by Apple
- You need a paid developer profile to release Apps on the App Store
- **Free developer account allows sideloading**
- The filesystem is **partitioned** into User and OS partitions
- Application code is delivered in IPA files, which are similar to APKs.
	- Signed bundle of directories and files
	- You can unarchive the file by giving it a `.zip` extension
	- `/Payload` is a particularly interesting directory
		- `/Payload/Application.app` - the app itself
			- You can browse this in Finder by right clicking and selecting "Show Package Contents"
		- `/Payload/iTunesMetadata.plist` - developer metadata
		- `/Payload/Application.app/Info.plist` - app manifest
		- `Payload/Application.app/GoogleService-Info.plist`
		- and more!

### Getting IPAs from the App Store
- [AnyTrans](https://www.imobie.com/anytrans/), [ipatool](https://github.com/majd/ipatool)
	- Requires **GIVING YOUR APPLE CREDENTIALS TO A RANDOM APPLICATION**
	- The service obtains and provides the IPA using the provided account
	- This feels sketch (even with a burner, all Apple accounts require a valid email and phone number)
- Just Google "IPA download"
	- IDK, this seems better if possible lol

### Emulation
- Xcode simulator
- [Corellium](https://www.corellium.com/)
- [Appetize](https://appetize.io/)

### Additional reading
[Apple Architecture and Additional Reading (Mobile Pentest Gitbook)]([https://mobile-security.gitbook.io/mobile-security-testing-guide/ios-testing-guide/0x06a-platform-overview](https://mobile-security.gitbook.io/mobile-security-testing-guide/ios-testing-guide/0x06a-platform-overview))
[Apple Security Documentation]([https://support.apple.com/guide/security/welcome/web](https://support.apple.com/guide/security/welcome/web))