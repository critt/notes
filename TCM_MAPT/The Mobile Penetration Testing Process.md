1. Reconnaissance
	1. Earnings reports and press releases often contain info about mobile apps
	2. App store information:
		- Reviews
		- Who created the app
		- Patch notes
		- Related apps
1. Static Analysis
	1. Reading source code via manual or automated tools 
		1. Looking for strings, security misconfigurations, and additional targets.
	2. A lot of companies use a special API gateway for their mobile applications. Sometimes they will have configuration files for the mobile app stored their site under an obscure directory, and sometimes the app and web frontend talk with different backends. Other examples:
		1. Find email/username: recon using phonebook, dumps etc
		2. Find storage bucket: recon enumerate with cloud_enum
		3. Find URL: recon, enumerate, exploit etc
		4. Swagger docs: API enumeration
	3. ![[Screenshot 2024-11-30 at 12.44.11 PM.png]]
		
2. Dynamic Analysis: Running the application and manipulating it
	- Intercepting proxies (Burp Suite, Proxyman)
	- Dumping memory and looking for stuff
	- Checking local storage at/after runtime
	- Breaking SSL Pinning at runtime
	1. Dynamic Analysis can often result in attacks related to the OWASP Top 10: SQLi, XSS, IDOR, XXE, etc.
		- Note on XSS: it will often not be XSS in the app itself, but there might be some interplay available between the app and a web frontend. For example: the two frontends could sanitize inputs differently, leading to a stored XSS vuln whereby the payload must be stored by the mobile app but then executes in the browser.
3. Reporting
	- Often contains executive summary as well as specific vulnerabilities discovered
	- Write report with both OWASP Top 10 Web and Mobile in mind
	- Provide subject with criticality as well as steps to reproduce
	- Highlight positive security implementations