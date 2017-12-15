# 2018-friendly-chat-web
Code Walkthrough &amp; Notes for running a 2-hour "Firebase Codelab for Web" using the FriendlyChat app https://codelabs.developers.google.com/codelabs/firebase-web/index.html (version relevant as of Dec 12, 2017)

The goal of this README is to become a living document that I can update and use a self-guided tour of the codelab, along with some additional information or details on relevant features/APIs, for running a half-day or 1-day Firebase Camp for beginners.

The steps here correspond to the steps fo the codelab -- however, you may also see occasional **DETOUR** sections indicating that I added content or information that is meant to provide additional context or information for the programmer, that is potentially outside the scope of just this codelab.

<hr />

## FIREBASE AS A BACKEND (Core Features)

### 1. Get the Sample Code

Simply clone or download [this repo](git clone https://github.com/firebase/friendlychat-web) from GitHub. Unzip it and open it in the IDE of your choice. _I use Sublime Text 3_

You should see the following files and directories:

```
friendlychat-web-master/
    CONTRIBUTING.md
    LICENSE
    README.md
    cloud-functions/
    cloud-functions-angular/
    cloud-functions-angular-start/
    cloud-functions-start/
    initial_messages.json
    web/
    web-start/
```

**TODO: Update this section with detailed listing of all files in subdirs**

For now we will focus only on the ```web-start/``` directory which contains the basic starter code for the web app we are building. This is what you should see when you first start off. _Note: The ```web/``` directory by contrast has the completed codelab version for reference and debugging help._

```
web-start/ 
    README.md
    firebase.json
    functions/
    images/
    index.html
    manifest.json
    manifest.webapp
    scripts/
    styles/
```

Let's get a quick sense of what these contents are:

 * **README.md** - default app documentation placeholder. Not critical.
 * **firebase.json** - required for deployment with Firebase Hosting. [Learn more here.](https://firebase.google.com/docs/hosting/full-config)
 * **manifest.json** - web app manifest as required for progressive web apps. [Learn more here.](https://developers.google.com/web/fundamentals/web-app-manifest/)
 * **manifest.webapp** - supporting manifest in proprietary Firefox OS manifest format. [Learn more here](https://developer.mozilla.org/en-US/docs/Archive/B2G_OS/Firefox_OS_apps/Building_apps_for_Firefox_OS/Manifest).
 * **index.html** - entry point for web app. Pre-configured to use [Material Design Lite](https://getmdl.io/started/index.html), [Google Fonts](https://fonts.google.com/), [Progressive Web Apps](https://developers.google.com/web/progressive-web-apps/checklist) (manifest, homescreen) and complete [Firebase](https://firebase.google.com/docs/web/setup) support by default. We can tune these for specific subsets of usage if needed (later). _The default web app _


### 2. Create a Firebase Project

This step is primarily about two things:
 
 1. _Creating a Firebase Account as a developer._ Currently this requires you to use a free (Gmail) address or paid (Google apps) account. Once you have one, simply log in by going to [Firebase Console](https://console.firebase.google.com) 

 2. _Creating a Common Backend for your front-end apps._ This simply requires you to "Add Project" and pick a unique project name for your account. This name influences the choices for the hosted database endpoints and app. If a globally similar name pre-exists, Firebase adds random numbers to the end to make yours unique - your best bet is to pick a hyphenated project name that has multiple parts to make it default unique globally.

 At this point you should have access to the [Firebase Console](https://console.firebase.google.com) for your app.


### 3. Install the Firebase CLI

While the Firebase Console is required for administering your backend, the Firebase CLI is required for managing the build and deployment process on your development platform.

The Firebase CLI is available as an NPM (Node Package Manager) package. So first make sure you have Node/NPM installed, then simply install Firebase CLI as follows:

```
npm -g install firebase-tools
```

Validate your install & login to enable CLI access to your backend projects

```
firebase --version
firebase login
```

### 4. Run the Starter App

Simply change to the directory containing the code you want to run/deploy (e.g, web-start) then run

```
firebase serve
```

You should see
```
=== Serving from '~/friendlychat-web-master/web-start'...

 hosting: Serving hosting files from: ./
✔  hosting: Local server: http://localhost:5000
```

This will only happen because the project contains a **firebase.json** file that provides the deployment configuration for your app. The starter code has this as the default:

```
{
  "hosting": {
    "public": "./",
    "ignore": [
      "firebase.json",
      "database-rules.json",
      "storage.rules",
      "functions"
    ],
    "headers": [{
      "source" : "**/*.@(js|html)",
      "headers" : [ {
        "key" : "Cache-Control",
        "value" : "max-age=0"
      } ]
    }]
  }
}
```

The directory specified in "public" is the one that gets deployed to Firebase Hosting, and is the only **required** property. 

The "ignore" setting identifies files that should _not_ be added to the deployment and is optional.

The "headers" setting contains any custom header definitions you want enforced by the server when hosting and serving your pages e.g., this one sets default browser cache settings for all JS and HTML files to 0 (no caching)

If instead of using starter code, you create a Firebase project from scratch, you would use 

```
firebase init
```

to generate the default firebase.json file. Here is an example of [a full hosting configuration example from the docs](https://firebase.google.com/docs/hosting/full-config#section-full-firebasejson):

```
{
  "hosting": {

    "public": "app",

    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ],

    "redirects": [ {
      "source" : "/foo",
      "destination" : "/bar",
      "type" : 301
    }, {
      "source" : "/firebase/*",
      "destination" : "https://www.firebase.com",
      "type" : 302
    } ],

    "rewrites": [ {
      "source": "/app/**",
      "destination": "/app/index.html"
    } ],

    "headers": [ {
      "source" : "**/*.@(eot|otf|ttf|ttc|woff|font.css)",
      "headers" : [ {
        "key" : "Access-Control-Allow-Origin",
        "value" : "*"
      } ]
      }, {
      "source" : "**/*.@(jpg|jpeg|gif|png)",
      "headers" : [ {
        "key" : "Cache-Control",
        "value" : "max-age=7200"
      } ]
      }, {
      "source" : "404.html",
      "headers" : [ {
        "key" : "Cache-Control",
        "value" : "max-age=300"
      } ]
    } ],

    "cleanUrls": true,
    "trailingSlash": false
  }
}
```


You can learn more about the parameters and their meanings in the [Deployment Configuration documentation](https://firebase.google.com/docs/hosting/full-config).

_At this point you should see the basic web app UI show up on localhost:5000_

### 5. Handle User Sign-In


### 6. Read Messages (as data, from Real-Time Database)

### 7. Send Messages (as data, to Real-Time Database)

### 8. Send Media (as files, written to Cloud Storage)

### 9. Send Notifications (using Cloud Messaging & Service Workers)

### 10. Secure Database: Access Rules

### 11. Secure Storage: Access 

### 12. Deploy App (using Firebase Hosting)

The starter code that we have downloaded previously has a **firebase.json** file that effectively identifies default Hosting (deployment) configuration for the app. 
<hr />


## GOING SERVERLESS WITH FIREBASE (Cloud Functions)

### 1. Get the Sample Code

### 2. Install Dependencies (from functions/package.json)

### 3. Import required Modules (in functions/index.js)

### 4. Trigger Auth events function (deploy using Firebase CLI, test on Cloud)

### 5. Trigger Storage events function (and integrate Cloud ML API)

### 6. Trigger Database events function (and integrate Cloud Messaging)
