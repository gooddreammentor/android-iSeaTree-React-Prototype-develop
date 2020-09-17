# iSeaTree-React-Prototype

Prototype for the iSeaTree (a mobile app which creates compatible data collection for the iTree project).

## Development

Project was initialized using [Expo](https://expo.io). Here is a guide how to [get-started with expo](https://docs.expo.io/versions/latest/get-started/installation).

### Prerequisites

- node v12.16.1
- [yarn v1.19.1](https://yarnpkg.com)
- [expo-cli v3.16](https://www.npmjs.com/package/expo-cli)

### Installation

From the project's root directory run:

```bash
yarn install
```

### Running

```bash
yarn start
```

### Secrets

`envVariables.ts` file holds all secret keys (api keys etc). It should never be commited to the repo. Before starting the app, get correct values from someone.

```bash
cp envVariables.example.ts envVariables.ts
```

Now you can follow displayed instructioned. For example, you can open Android emulator pressing `a` in the current tab.

## Linting

This project has configured [Eslint](https://eslint.org/) with recommended typescript and react rules.

## Firebase - Setup

1. Go to the https://console.firebase.google.com and create new project
2. Click `Database` in the left side panel and create `Firestore`(!) Database
3. Edit Firestore Database security rules
   1. Database needs certain rules to works correctly. Visit https://firebase.google.com/docs/firestore/security/get-started to learn more about security rules.
   2. Copy and paste below rules

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId} {
    	allow read, update, delete: if request.auth.uid == userId;
      allow create: if request.auth.uid != null;
    }

    match /trees/{userId} {
    	allow read, update, delete: if request.auth.uid == resource.data.userId;
      allow create: if request.auth.uid != null;
    }
  }
}
```
4. Click `Storage` in the left side panel and create storage
5. Click `Authentication` in the left side panel and click `Set up sign-in method`
   1. Enable `Email/Password` method (first from top)
6. Go to the project settings (Cog icon)
   1. Find `Your apps` section and register new app. You have to select web platform (</> icon)
   2. Copy keys from `firebaseConfig` object and use them to populate envVariables.ts inside this project


## Releasing app to the Play Store (Android)

First you have to obtain google map key:
- google map config - https://docs.expo.io/versions/latest/sdk/map-view/#deploying-to-a-standalone-app-on-android

Then you can follow these instructions: https://docs.expo.io/distribution/app-stores/

## Releasing app to the App Store (iOS)

Expo documentation continually refers to building for Testflight. That's incorrect. You build for the App Store 
(specifically appstoreconnect.apple.com). Maybe you release a build on Testflight, maybe you don't. Maybe you release
it straight to the App Store. Maybe you don't release it anywhere because Apple review finds problems, or because you
want to include a new feature. But every time, you're building for upload to App Store Connect.

There are three parts to putting an app into the App Store: generating a signing profile, generating an App Store
entry, and putting a particular build into the store. Expo will automate all of these phases. Their toolset and
documentation handles the first two as one step. Expo will also attempt to pull down defining information
from App Store Connect (specifically, the distribution certificate and the signing profile) on each build,
to make sure that Expo is using current information.

Builds happen on the Expo server, using Fastlane. Your Info.plist is created by Expo, based on the values in 
`app.json`.

### One-time setup on your Apple developer account

Create an app-specific password in developer account, if you want. Hal tried but the Expo build tools weren't 
able to authenticate using the app-specific password.

If you have not generated the deployable iOS version of your app with Expo before, then the first time your 
run `expo build:ios` you'll see a series of questions of 
questions about App Store Connect credentials. Expo can handle creation of provisioning profile. This worked smoothly 
and was easier than populating fields by hand.

In App Store Connect, you will see a newly-created entry for iSeaTree. Upload the icon (`icon.png` in 
the `assets` folder). Populate the remaining fields.

### Builds
Start the build using `expo build:ios`. Choose "archive" (for App
Store builds) or "simulator" (for Simulator testing).

After you start a build , you'll see a URL starting with `https://expo.io/dashboard/` that 
you can use to monitor the build.

At the end of the build, you'll see a URL starting with `https://expo.io/artifacts`
which is the location of your build product. For "archive" builds it's
an  IPA (the archive containing your iPhone application). For
"simulator" builds it's a .tar file.

If you built for App Store submission, download the IPA and upload it to the
Apple App Store using either the [Transporter](https://apps.apple.com/us/app/transporter/id1450874784?mt=12) GUI tool,
or using `xcrun altool` from the command line:

```
$ xcrun altool --upload-app --type ios --file <IPA_FILE_THAT_YOU_HAVE_UPLOAD_FROM_EXPO_BUILD> --username "YOUR_APPLE_ID_USER" --password "YOUR_ITMC_PASSWORD"
```

`xcrun altool` provides no feedback while running.

According to [Expo documentation on App Store uploads](https://docs.expo.io/distribution/uploading-apps/#22-if-you-choose-to-upload-your), 
Expo can handle the upload automatically. This would be a nice option for minor updates from a machine that can't 
run Xcode.

### Posting a build to App Store Connect

You must increment `buildNumber` in the `"ios"`dictionary for each new upload to the App Store. If `"version"` 
is "1.0.0"` and `"buildNumber"` is `"3"`, App Store Connect will display "1.0.0 (3)" for version information.
If you've already uploaded "1.0.0 (3)", you can't upload it again. Increment the build number, and upload "1.0.0 (4)".

A build must pass App Store review before being released to the App Store or to public Testflight. Team members
(who are listed in the organization's developer page, 
see [https://developer.apple.com/support/roles/](https://developer.apple.com/support/roles/)) have immediate
Testflight access to all builds. Public Testflight reviewers are added
manually by email invitation, or automatically (and anonymously) if a
public link is enabled.

To submit an uploaded build for Testflight review, use the Testflight menu,
and choose a build instance. New testflight builds are not
automatically released to public testers; you'll have to do this
manually after approval. Provide a support email address (someone who
will receive emails specifically sent from the Testflight testers), and a
privacy policy URL.

To submit an uploaded build for App Store review, verify the app
description, keywords, categories, support URL, marketing URL, and screenshots. 
Optionally, supply app preview videos (you can capture these
on device). You can't change
any of these after public release (but you _can_ update them during the TestFlight beta cycle). 
Updating them after public release requires a new
version. The "promotional text" field can be modified without a new
release. Click "Submit for review" and follow the prompts. When asked
if the app uses the IDFA advertising identifier, the answer is Yes,
because Expo uses the Segment Analytics, which uses it.

Screenshot devices required, as of May 2020, are iPhone
6.5" display (11 Pro Max, 11, Xs Max), iPhone 5.5" display (6s Plus, 7
Plus, 8 Plus), and iPad Pro 12.9" (3rd Gen). Don't use
the iPhone XR because the screenshots are half the pixel density of
the other 6.5" devices, and won't pass review.

### Interacting with the iOS Simulator

When Expo commands touch the Simulator, they will work with the most
recently launched instance, even if that's not the instance you last
used yourself.

`expo-cli client:install:ios` installs the Expo client to the
Simulator. If another app is running, that app won't be disturbed, but
you'll see the Expo icon on your home screen if you look for it. Log
in to your Expo account and you'll see any Expo projects you've built,
hopefully including iSeaTree. iPad
simulators show the app in full-screen mode. This is misleading,
because the native build comes up in 1x letterboxed mode.

To get a native build for the Simulator, use `expo build:ios`. At the
prompt, select "simulator". This will launch a build in the Expo
build queue, and eventually return the URL of a downloadtable .tar
file. Expand the .tar file and you'll see the iSeaTree app, marked as
unlaunchable. Drag that app onto your simulator's screen (it will
appear on the second page of the Home screen, which you'll have to
page to see). Building the
native app takes a while, but launching iSeaTree in multiple
simulators (which is necessary for making screenshots) is faster this
way.


