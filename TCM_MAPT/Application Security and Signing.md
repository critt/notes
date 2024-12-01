1. Every Android app can be reverse-engineered, rebuilt, re-signed, and re-run
2. This means attackers can modify app functionality
3. The source code is available by using a tool like JADX-GUI or Apktool
![[Screenshot 2024-11-30 at 1.46.27 PM.png]]

##### Compilation Process: Normal Java (left), Android (right)
![[Screenshot 2024-11-30 at 1.48.23 PM 1.png]]

##### Application Signing
![[Screenshot 2024-11-30 at 1.51.48 PM.png]]

- Since anyone could modify an application and publish it to the Play Store, how do we ensure it's integrity?
	- Public Key Cryptograph
- Today there are three methods of verifying signatures:
	- APK Signature scheme v1, v2, v3
	- In addition, Google implemented Google Play signing which adds unique signatures to the apps
	- keytool, jarsigner, zipalign
- Unsigned apps don't even run apparently