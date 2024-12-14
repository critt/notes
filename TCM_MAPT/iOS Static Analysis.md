### Manual analysis
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

### Automated analysis
##### MobSF
The same tool can be used to analyze APKs and IPAs. Follow the Android notes for MobSF setup:
[[Android Dynamic Analysis#MobSF magical GUI tool for dummies]]

