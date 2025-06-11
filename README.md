# azure-VM

deployed in VM both successfully, should make it accessible with public ip
### Access with public IP
- Go to network security group
- Add inbound rule
```markdown
Protocol->Any
Source port ranges->*
Destination->Any
Service->Custom
Destination port ranges->8081
Protocol->Any
Action->Allow
Priority->310
Name->HTTP_port
```
```markdown
Protocol->Any
Source port ranges->*
Destination->Any
Service->Custom
Destination port ranges->8080
Protocol->Any
Action->Allow
Priority->100
Name->AllowAnyCustom8081Inbound
```
- Firstly, we have to change the appsettings.production.json [Backend] and appconfig.production [Frontend] to take the http://20.204.162.91:8080/ instead of localhost:8080 for upsi inside the VM created.
- Initially i kept destination port as 80 and i was able to access the myinsider application with http://20.204.162.91:8081/ in my outside the VM environment computer.[8081 is the port where i deployed myinsider in VM IIS ]
- But was unable to access the upsi application with http://20.204.162.91:8080/ in my outside the VM environment computer. [8080 is the port where i deployed upsi in VM IIS ]
  ```bash
  polyfills.03c2a556ba39e09e.js:1  Refused to connect to 'http://20.204.162.91:8080/api/AbpUserConfiguration/GetAll?d=1744178883471' because it violates the following Content Security Policy directive: "connect-src 'self'
  ```
- To resolve the error:


### What is CSP?

CSP (Content Security Policy) is a security feature implemented by browsers to prevent a range of attacks, such as cross-site scripting (XSS) and data injection attacks. It restricts the sources from which content (e.g., scripts, images, data) can be loaded.

In your case, the default `connect-src 'self'` policy allows connections to only the same domain and not to external services or APIs. Since you're trying to connect to an external IP (`20.204.162.91`), it violates the policy.

### How to Fix the Issue

There are a few ways to address this problem:

#### 1. Modify the Content Security Policy (CSP) in Your Web Server Configuration

You'll need to update your CSP to allow connections to the external IP or domain from which you're trying to load the API. Here's how you can do it:

- **For IIS** (If you're using IIS Web Server on the Azure VM):

  1. Open **IIS Manager**.
  2. In the left panel, select your site.
  3. In the middle panel, double-click on **HTTP Response Headers**.
  4. Click **Add...** in the right panel to add a new header.
  5. For **Name**, enter `Content-Security-Policy`.
  6. For **Value**, set the value to:
     ```
     connect-src 'self' http://20.204.162.91:8080;
     ```
     This will allow the page to make API requests to `http://20.204.162.91:8080` while still restricting other external connections.

  Alternatively, you can also modify the CSP in your site's HTML `<meta>` tag:
  ```html
  <meta http-equiv="Content-Security-Policy" content="connect-src 'self' http://20.204.162.91:8080;">
  ```
- Im VM machine-> Windows defender settings-> Domain:On, Private:On, Public:Off
- In local outside computer-> Firewall and Network portection-> Domain:On, Private:On, Public:On
 
Now getting:
```bash
now getting this after i changed the content security policy to connect-src 'self' http://20.204.162.91:8080;
:
polyfills.03c2a556ba39e09e.js:1 
 
 GET http://20.204.162.91:8080/api/AbpUserConfiguration/GetAll?d=1744179723290 net::ERR_CONNECTION_TIMED_OUT
it	@	polyfills.03c2a556ba39e09e.js:1
```

- To resolve this i changed the network azure VM -> security group -> inbound rule port from 80 -> 8080 [Name: HTTP_port]

wt next for this repo !?
