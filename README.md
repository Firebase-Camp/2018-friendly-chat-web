# 2018-friendly-chat-web
Code Walkthrough &amp; Notes for running a 2-hour "Firebase Codelab for Web" using the FriendlyChat app https://codelabs.developers.google.com/codelabs/firebase-web/index.html (version relevant as of Dec 12, 2017)

The goal of this README is to become a living document that I can update and use a self-guided tour of the codelab, along with some additional information or details on relevant features/APIs, for running a half-day or 1-day Firebase Camp for beginners.

The steps here correspond to the steps fo the codelab -- however, you may also see occasional **DETOUR** sections indicating that I added content or information that is meant to provide additional context or information for the programmer, that is potentially outside the scope of just this codelab.

<hr />

## FIREBASE PRODUCTS (Walkthrough)

### [1. Initialization](https://firebase.google.com/docs/web/setup)

To use Firebase in Web apps, you need to do the following:

 * Install the Firebase SDK (import appropriate dist in script)
 * Set Firebase project config (copy from project in Console)
 * Call ```firebase.initializeApp(config)``` in your code
 * Access firebase product APIs using relevant ```firebase.x``` 
     - firebase.auth()
     - firebase.storage()
     - firebase.database()
     - firebase.firestore()

To reduce codebloat, instead of full Firebase SDK, you can import the 
firebase-app (core) and then add only those (optional) pieces you use:

 * firebase-app - The core firebase client (required).
 * firebase-auth - Firebase Authentication (optional).
 * firebase-database - The Firebase Realtime Database (optional).
 * firebase-firestore - Cloud Firestore (optional).
 * firebase-storage - Cloud Storage (optional).
 * firebase-messaging - Firebase Cloud Messaging (optional).

You can initialize multiple apps within same codebase by simply having different config variables for each, then initializing them with those separately. (Allows one web app to link to multiple Firebase projects).

```
// Initialize the default app
firebase.initializeApp(defaultAppConfig);

// Initialize another app with a different config
var otherApp = firebase.initializeApp(otherAppConfig, "other");

console.log(firebase.app().name);  // "[DEFAULT]"
console.log(otherApp.name);        // "other"

// Use the shorthand notation to retrieve the default app's services
var defaultStorage = firebase.storage();
var defaultDatabase = firebase.database();

// Use the otherApp variable to retrieve the other app's services
var otherStorage = otherApp.storage();
var otherDatabase = otherApp.database();
```


### 

### [2. Authentication](https://firebase.google.com/docs/auth/web/start)

Allow sign-in with one or more sign-in methods including:
 * Email/Password 
     - You own the user credentials
     - Admin provides easy verification, reset services
 * Identity Service Providers
     - Google
     - Facebook
     - Twitter
     - GitHub
 * Anonymous

First, go to console and ENABLE the authentication methods 
you want to support. If you are hosting your app at a custom domain, you will also need to add that to the list of "authorized domains" for OAuth. The hosted app domain is added for you automatically.

Sign UP new users:

```
firebase.auth()
    .createUserWithEmailAndPassword(email, password)
    .catch(function(error) {
      // Handle Errors here.
      var errorCode = error.code;
      var errorMessage = error.message;
      // ...
    });
```

Sign IN current users:

```
firebase.auth()
    .signInWithEmailAndPassword(email, password)
    .catch(function(error) {
      // Handle Errors here.
      var errorCode = error.code;
      var errorMessage = error.message;
      // ...
    });
```

Sign OUT current users:

```
firebase.auth().signOut();
```

Register for auth-change events so you can update your application in real-time to reflect auth status of user:

```
firebase.auth().onAuthStateChanged(function(user) {
  if (user) {
    // User is signed in.
    var displayName = user.displayName;
    var email = user.email;
    var emailVerified = user.emailVerified;
    var photoURL = user.photoURL;
    var isAnonymous = user.isAnonymous;
    var uid = user.uid;
    var providerData = user.providerData;
    // ...
  } else {
    // User is signed out.
    // ...
  }
});
```

You can sign in the user ANONYMOUSLY to create a trackable user object whose actions can subsequently be converted into a new account (sign up) or transferred to an existing account (sign in)

```
firebase.auth()
    .signInAnonymously()
    .catch(function(error) {
      // Handle Errors here.
      var errorCode = error.code;
      var errorMessage = error.message;
      // ...
    });
```

If successful, this also triggers the ```onAuthStateChanged``` callback with a "signed in" event but now you can check within that callback to see if the sign is from an anonymous user, or from an identity provider.

```
firebase.auth().onAuthStateChanged(function(user) {
  if (user) {
    // User is signed in & is anonymous
    var isAnonymous = user.isAnonymous;
    var uid = user.uid;
    // ...
  } else {
    // User is signed out.
    // ...
  }

  //...

});
```

If the anonymous user later logs in via a provider (you or other), you can "limnk" that identified user to the anonymous user, allowing the former account to access Firebase data/traces created by the latter. See [this page](https://firebase.google.com/docs/auth/web/anonymous-auth) for more information

```
auth.currentUser.link(credential).then(function(user) {
  console.log("Anonymous account successfully upgraded", user);
}, function(error) {
  console.log("Error upgrading anonymous account", error);
});
```

You can also have a [custom auth system](https://firebase.google.com/docs/auth/web/custom-auth), sign in with a [phone number](https://firebase.google.com/docs/auth/web/phone-auth) (especially with Digits integration) and leverage the open-source [FirebaseUI](https://opensource.google.com/projects/firebaseui) components to get pre-made user interaction flows and UI.

Actions like 'Verify Email' or 'Reset Password' can be triggered directly from the Admin Console. But you can also trigger admin actions using the [Admin SDK](https://firebase.google.com/docs/reference/admin/)

For email-driven actions, you can pass state information (e.g., what application link the user should go to) using ```ActionCodeSettings```. However the domains for links used _must_ be whitelisted in the Authentication tab of the Admin Console. Learn more [here](https://firebase.google.com/docs/auth/web/passing-state-in-email-actions).


### [3. Real-Time Database](https://firebase.google.com/docs/database/)



### [4. Hosting](https://firebase.google.com/docs/hosting/)


### [5. Security](https://firebase.google.com/docs/database/security/)

Realtime Database rules determine 

 * who has read/write access to your data
 * what indexes exist
 * how data is structured

Rules live on the Firebase server, but can be maintained in a local file in your development environment and "deployed" using Firebase CLI. Every read/write will proceed only if the rules enforced at the relevant node (reference) allow it.

Rules have a JavaScript-like syntax and are of 4 types:

 * read - can this user read this data node
 * write - can this user write to this data node
 * validate - does this data have valid format (data type, children)
 * indexOn - what is the index used for child order/query

**Rules cascade!** 
 
 * A rule established at parent is immediately enforced at children
 * Shallow overrides deep - so a child's rule is overridden by parent!

So make sure that your shallowest rules are strong and don't weaken the child-set rules accidentally.

Example:
For data at the path "/foo" in my database, anyone can read it and no one can write to (update) it. Also, by cascade, any one at /foo/bar/1 and /foo/baz/1 will have the same read/write restrictions _even if_ they specified a new rule at that level that allowed writes.
```
{
  "rules": {
    "foo": {
      ".read": true,
      ".write": false
    }
  }
}
```

Firebase has [built-in variables and functions](https://firebase.google.com/docs/database/security/securing-data#predefined_variables) you can use as wildcards to create more contextually relevant access. In addition to these "$"
variables (wildcards for ids or child keys) you can also use "data" (before), "newData" (after), now (current time in ms), auth (token) and root (root reference before) as contexts

Example:
This rule indicates that write access at the path "/users/<uid>"  is allowed only if this 'wildcard' $uid matches that of the currently-logged in user.
```
{
  "rules": {
    "users": {
      "$uid": {
        ".write": "$uid === auth.uid"
      }
    }
  }
}
```


Rules can be used to enforce data validation but validation rules do NOT cascade. So ALL relevant validations must be true before the write can proceed.
Example:
This validates that new data written to "/foo" can only be of type string and mus have a length less than 100 characters.
```
{
  "rules": {
    "foo": {
      ".validate": "newData.isString() && newData.val().length < 100"
    }
  }
}
```


Use rules to create indexes that can support more efficient querying of data (and ordering of results). By default, ad hoc querying is supported but indexes are recommended if your data size grows. Use the _indexOn_ rule for establishing a new index on the data at that node.

Example:
This says that the data under "/dinosaurs" has properties "height" and "length" that should be used to create default indexes for result sets.
```
{
  "rules": {
    "dinosaurs": {
      ".indexOn": ["height", "length"]
    }
  }
}
```

Also see:

 1. [Database Rules](https://firebase.google.com/docs/reference/security/database/)
 2. [Database Regular Expressions](https://firebase.google.com/docs/reference/security/database/regex)
 3. [Storage Rules](https://firebase.google.com/docs/reference/security/storage/)
 4. [Security & Storage](https://firebase.google.com/docs/storage/security/)
 5. [Firestore Rules](https://firebase.google.com/docs/firestore/reference/security/)
 6. [Security & FireStore](https://firebase.google.com/docs/firestore/security/overview).





<hr />

## FIREBASE AS A BACKEND (FriendlyChat Codelab)

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

Now it's time to start modifying the code.

The structure of the user interface is defined in ```index.html``` and uses simple mdl [Material Design Lite](https://getmdl.io) components to scaffold a simple page that has a top-level layout-header, a central messages-card-container and a  bottom-level snackbar for alerts.

The styling for this simple page is in the ```styles/main.css``` file and is pretty basic. 

All the control code for this codelab can be found in a single file ```scripts/main.js```. There were no front-end frameworks used here, so all the code is simply vanilla JS. Here is what the basic code outline is:

 1. On window load, instantiate a FriendlyChat object to manage user interactions.

 2. The FriendlyChat prototype defines the following functions to help manage the user interactions
    * checkSetup = checks that the Firebase SDK is configured correctly
    * initFirebase = configure the relevant Firebase services used by app
    * signIn = handle sign-in button click
    * signOut = handle sign-out button click
    * onAuthStateChanged = handle auth-change events when user signs in/out
    * checkSignedInWithMessage = checks user is signed in before posting
    * saveMessage = handle submit buttom click (when user fills message form)
    * toggleButton = enables/disables submit button (on validating input)
    * loadMessages = loads chat messages history, listens for new additions
    * displayMessage = fills template & adds message to container for display
    * saveMessage = saves new message to Firebase backend (RT DB)
    * saveMessagingDeviceToken = saves messaging device token to datastore
    * requestNotificationsPermissions = ask permission to show notifications

 3. In its constructor, it verifies that the Firebase SDK is setup correctly and configured. It then creates shortcut variables for the DOM elements, adds event listeners (with callbacks) to relevant elements, then calls _initFirebase()_ to configure Firebase services that this app is using.

**First: initFirebase()**

Creates shortcuts to features used (auth, database, storage), and registers the callback that should be triggered when the Firebase Auth service determines that the auth state has changed.

```
FriendlyChat.prototype.initFirebase = function() {
  // Shortcuts to Firebase SDK features.
  this.auth = firebase.auth();
  this.database = firebase.database();
  this.storage = firebase.storage();
  // Initiates Firebase auth and listen to auth state changes.
  this.auth.onAuthStateChanged(this.onAuthStateChanged.bind(this));
};
```


**Next: signIn()**

Next implement the callback for the sign-in button handler. For now, this simply uses the easy "signInWithPopup" option that exists for the GoogleAuthProvider as a 1-line-code helper. 

There are [other provider options](https://firebase.google.com/docs/auth/web/start) for Facebook, GitHub, Twitter providers, or for using email/password or phone number (Digits) as auth options.

There are also [other UI options](https://firebase.google.com/docs/auth/web/google-signin) for use with Google Sign-on including redirecting to a separate page. The "provider" object can optionally be extended to specify OAuth scopes or custom parameters for use with the login.

```
FriendlyChat.prototype.signIn = function() {
  // Sign in Firebase using popup auth and Google as the identity provider.
  var provider = new firebase.auth.GoogleAuthProvider();
  this.auth.signInWithPopup(provider);
};
```

Sign out is easy. 

```
FriendlyChat.prototype.signOut = function() {
  // Sign out of Firebase.
  this.auth.signOut();
};
```

Note that both signout and signin, will in turn trigger Auth Change events - handle these in the onAuthStateChange callback that pre-existed.(Also note that it is within this callback that the signin/signout buttons are contextually shown/hidden based on auth state. So once you initialize this far, the sign-in button should appear by default)

When authenticated, the "this.auth.currentUser" property will exist (be true), so we can use this to trigger the Snack bar to remind users to sign in if they try to post something without doing so.

```
FriendlyChat.prototype.checkSignedInWithMessage = function() {
  // Return true if the user is signed in Firebase
  if (this.auth.currentUser) {
    return true;
  }

  ...
```

At ths point you should have a login button. And logging in should cause the onAuthChanged callback to automatically show the additional elements (user name and profile pic) associated with the login.

### 6. Read Messages (as data, from Real-Time Database)

Open console and check the Database tab.


### 7. Send Messages (as data, to Real-Time Database)

### 8. Send Media (as files, written to Cloud Storage)

### 9. Send Notifications (using Cloud Messaging & Service Workers)

### 10. Secure Database: Access Rules

### 11. Secure Storage: Access 

### 12. Deploy App (using Firebase Hosting)

The starter code that we have downloaded previously has a **firebase.json** file that effectively identifies default Hosting (deployment) configuration for the app. 
<hr />


## GOING SERVERLESS WITH FIREBASE (Cloud Functions Codelab)

### 1. Get the Sample Code

### 2. Install Dependencies (from functions/package.json)

### 3. Import required Modules (in functions/index.js)

### 4. Trigger Auth events function (deploy using Firebase CLI, test on Cloud)

### 5. Trigger Storage events function (and integrate Cloud ML API)

### 6. Trigger Database events function (and integrate Cloud Messaging)
