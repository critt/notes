##### APK Unpacking
`jadx`, `jadx-gui`, `apktool`
##### Strings
Example strings of interest
- Login bypass (username/password or client credentials)
- URLs
- API keys
- Firebase paths/IDs/credentials

Low hanging locations
- AndroidManifest.xml
- strings.xml
- any XML resource, really

`jadx` has really good search features for strings in source
![[Screenshot 2024-12-02 at 9.27.36 AM.png]]

##### Cloud resource enumeration
- Strings like app or org names, bucket names, AWS IDs, and AWS secrets can lead to other interesting stuff in the cloud
- `cloud_enum` can help by quickly seeing if anything can be learned from a given piece of information
	- `cloud_enum` supports AWS, Azure, and GCP
	- example (keyword):![[Screenshot 2024-12-02 at 12.01.53 PM.png]]
	- Example workflow (AWS):
		1. Find AWS secrets in APK during static analysis
		2. Find S3 path using `cloud_enum` (or during static analysis)
		3. Use them with the AWS CLI to access the bucket: ![[Screenshot 2024-12-02 at 12.15.27 PM 1.png]]
- `firebaseEnum` is a similar tool specifically for Firebase enumeration. It has similar features, including keyword search
	- Example workflow:
		1. Find firebase DB URL using `firebaseEnum` (or during static analysis) (`https://injuredandroid.firebase.io`)
		2. Find relative DB path during static analysis (`flags/`)
		3. Use `.json` trick:![[Screenshot 2024-12-02 at 12.32.24 PM 1.png]]