---
title: "Set Up Firebase Emulator Suite For Your iOS App"
date: 2025-12-19T12:32:48-08:00
# last_modified_at: 2025-12-19T12:32:48-08:00
categories:
  - Blog
tags:
  - firebase
  - emulator suite
  - setup
# link: https://github.com
---

When you have an iOS app that uses Firebase, you want to connect your app to the Firebase Emulator Suite to prevent any data writes to production. This can be especially costly if you have your own Cloud Functions that will run, and there may be an error in the implementation, causing these functions to run in an infinite loop. If these functions are writing data to production non-stop, you will incur a large cost. Testing and debugging in development to the Emulator Suite can prevent this.

### Install the Firebase CLI (if you haven't already)

This is a one-time setup on your development machine.
1. Open your Terminal application.
2. Install Node.js and npm (Node Package Manager): If you don't have them, the easiest way is usually to download the installer from nodejs.org .
3. Install the Firebase CLI globally:
	```
	npm install -g firebase-tools
	```
	This command downloads and installs the firebase command-line tool on your computer. 

To update Firebase CLI instead:
```
  npm update -g firebase-tools
```
### Log in to Firebase through the CLI

1. In your Terminal, run:
	```
	firebase login
	```
2. This will open a browser window asking you to authenticate with your Google account. Make sure you log in with the same Google account that owns your Firebase project. 

### Create a new, separate directory for your Cloud Functions

This directory will not be inside your Xcode project. It should be a standalone folder somewhere on your computer, perhaps named [Project_Name]FirebaseBackend.

In your Terminal, navigate to where you want to create this new directory (e.g., cd ~/Developer/FirebaseProjects ).

Create the directory:
```
mkdir [Project_Name]FirebaseBackend # Or whatever you want to name your project folder
cd [Project_Name]FirebaseBackend
```

### Install Java for Firestore Emulator
In the Terminal, run:
```
brew install openjdk@[java version #] // Example: brew install openjdk@21
sudo ln -sfn /opt/homebrew/opt/openjdk@21/libexec/openjdk.jdk /Library/Java/JavaVirtualMachines/openjdk-21.jdk
echo 'export PATH="/opt/homebrew/opt/openjdk@21/bin:$PATH"' >> ~/.zshrc
```

### Configure Firebase Services to Run in Emulator
When you run:
`firebase emulators:start`

It will start the emulator, but by default, it doesn't automatically assume you want _every single_ available emulator running. Instead, it looks for two main things to decide which emulators to start:

1. **Your `firebase.json` file:** This file is the central configuration for your Firebase project locally. It lists which Firebase services you've configured for your project.
2. **What you initialized with `firebase init` :** When you first run `firebase init` and select services like "Firestore," "Functions," "Hosting," "Authentication," etc., these choices populate your `firebase.json` file.

If your `firebase.json` file currently only lists configurations for `functions` and `extensions` , then `firebase emulators:start` will naturally only start those emulators.

**How to Make It Run All Your Desired Services**

To ensure all the services you need (like Authentication, Firestore, Realtime Database, etc.) are started, you have two primary methods:

1. **Modify your `firebase.json` file:** The most robust way is to update your `firebase.json` file to explicitly include all the emulators you want to use. You'll typically find an `emulators` section in this file. Make sure it lists entries for each service:

```
{
  "firestore": {
    "rules": "firestore.rules",
    "indexes": "firestore.indexes.json"
  },
  "functions": {
    "source": "functions"
  },
  "hosting": {
    "public": "public",
    "ignore": [
      "firebase.json",
      "**/.*",
      "**/node_modules/**"
    ]
  },
  // ... potentially other service configurations ...

  "emulators": {
    "auth": {
      "port": 9099 // Default port, can be customized
    },
    "firestore": {
      "port": 8080 // Default port
    },
    "database": { // For Realtime Database
      "port": 9000 // Default port
    },
    "functions": {
      "port": 5001 // Default port
    },
    "hosting": {
      "port": 5000 // Default port
    },
    "storage": {
      "port": 9199 // Default port
    },
    "ui": {
      "port": 4000 // Emulator UI
    },
    "dataConnect": { // If you're using Firebase Data Connect
      "port": 9399
    }
  }
}
```

If you don't have an `emulators` section, you can add it, or you can run 
`firebase init emulators`

to set it up interactively. Once `firebase.json` lists these emulators, `firebase emulators:start` will automatically launch them.

**Use the `--only` flag with `firebase emulators:start` :** If you want to quickly spin up a specific set of emulators for a particular session, you can explicitly tell the CLI which ones to start using the `--only` flag. This overrides what's in `firebase.json` for that command.

For example, to start Authentication, Firestore, and Functions:
`firebase emulators:start --only auth,firestore,functions`

To start _all_ possible emulators that Firebase supports, you can use:
`firebase emulators:start --only auth,firestore,database,functions,hosting,storage,pubsub,extensions,eventarc,dataConnect`

1. (Note: `pubsub` and `eventarc` are relevant for Cloud Functions triggers, and `extensions` is for Firebase Extensions). You only need to include the ones you are actually using in your project.

**Recap:**
- `firebase emulators:start` starts emulators based on the `emulators` section of your `firebase.json` file.
- If you're only seeing Functions and Extensions, it means your `firebase.json` likely only specifies those, or you only initialized those services with `firebase init` .
- Update `firebase.json` to include all desired emulators in the `emulators` section.
- Alternatively, use `firebase emulators:start --only [list,of,emulators]` for a one-off run.

### Connect Firebase Services From Your App to the Emulator

[By default, the emulators only respond to requests from `localhost`. This means that you'll be able to access your hosted content from your computer's web browser but not from other devices on your network. If you'd like to test from other local devices, configure your `firebase.json` like so:](https://firebase.google.com/docs/hosting/test-preview-deploy#emulators-no-local-host)

```
  "emulators": {
    "auth": {
      "port": 9099 // Default port, can be customized
	  "host": "0.0.0.0"
    },
    "firestore": {
      "port": 8080 // Default port
      "host": "0.0.0.0"
    },
    // ...
  }
```

**Using `0.0.0.0` tells the emulator to listen on all network interfaces, making it accessible from other devices on your network (i.e. The "0.0.0.0" host tells your computer to use localhost and your local ip for your network at the same time)

```Swift
//import Firebase
import FirebaseAuth
import FirebaseCore
//import FirebaseDataConnect
//import FirebaseDatabase
//import FirebaseFunctions
import FirebaseFirestore
//import FirebaseStorage
import SwiftUI


@main
struct YourApp: App {
  @UIApplicationDelegateAdaptor(AppDelegate.self) var delegate
  var body: some Scene {
    WindowGroup {
	    ContentView()
    }
  }
}

class AppDelegate: NSObject, UIApplicationDelegate {
  func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey : Any]? = nil
  ) -> Bool {
    FirebaseApp.configure()
#if DEBUG
    connectFirebaseEmulatorSuite(to: .device)
#endif
    returntrue
  }

  private enum FirebaseEmulatorHardwareConnection {
    case device, simulator
  }

  private func connectFirebaseEmulatorSuite(to hardware: FirebaseEmulatorHardwareConnection = .simulator) {
    print("Connecting to Firebase Emulator Suite...")
    switch hardware {
    case .device:
      // If you're running on a physical device instead of the simulator, localhost won't work—you need your Mac's local IP address:
      print("Connection from Physical Device to Firebase Emulator Suite")
      // Authentication Emulator
      Auth.auth().useEmulator(withHost: "MAC_LOCAL_IP_ADDRESS", port: 9099)

      // Cloud Firestore Emulator
      Firestore.firestore().useEmulator(withHost: "MAC_LOCAL_IP_ADDRESS", port: 8080)

      // Firebase Realtime Database Emulator
      // Database.database().useEmulator(withHost: "MAC_LOCAL_IP_ADDRESS", port: 9000)

      // Cloud Storage for Firebase Emulator
      // Storage.storage().useEmulator(withHost: "MAC_LOCAL_IP_ADDRESS", port: 9199)

      // Cloud Functions for Firebase Emulator
      // Replace "us-central1" with your function's region if different
      // Functions.functions(region: "us-central1").useEmulator(withHost: "MAC_LOCAL_IP_ADDRESS", port: 5001)

      // Firebase Data Connect Emulator (if used)
      // Replace "your_connector_id" with your actual connector ID from firebase.json
      // DataConnect.your_connector_id.useEmulator(port: 9399)

	case .simulator:
	  print("Connection from Simulator to Firebase Emulator Suite")
      // Authentication Emulator
      Auth.auth().useEmulator(withHost: "localhost", port: 9099)
    
      // Cloud Firestore Emulator
      Firestore.firestore().useEmulator(withHost: "localhost", port: 8080)
    
      // Firebase Realtime Database Emulator
      // Database.database().useEmulator(withHost: "localhost", port: 9000)
    
      // Cloud Storage for Firebase Emulator
      // Storage.storage().useEmulator(withHost: "localhost", port: 9199)
    
      // Cloud Functions for Firebase Emulator
      // Replace "us-central1" with your function's region if different
      // Functions.functions(region: "us-central1").useEmulator(withHost: "localhost", port: 5001)
    
      // Firebase Data Connect Emulator (if used)
      // Replace "your_connector_id" with your actual connector ID from firebase.json
      // DataConnect.your_connector_id.useEmulator(port: 9399)    
#endif
    return true
  }
}
```

Now any service that makes a call will go to the emulator instead of production.