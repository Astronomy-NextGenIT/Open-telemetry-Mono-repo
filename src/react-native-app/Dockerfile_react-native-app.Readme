# Dockerfile Explanation – React Native Android Application

This Dockerfile is used to build a **React Native Android application** and generate a release APK file.

The Dockerfile follows a **multi-stage build approach**:

* Stage 1: Build the Android application
* Stage 2: Store only the generated APK file

The final output of this Docker build is an Android APK file that can be installed on Android devices.

---

# What Does This Dockerfile Do?

This Dockerfile:

1. Creates a React Native build environment.
2. Installs project dependencies.
3. Generates Android native project files using Expo.
4. Builds the Android application.
5. Creates a release APK.
6. Copies only the APK into the final image.

---

# Complete Dockerfile

```dockerfile
FROM reactnativecommunity/react-native-android:v20.1 AS builder

WORKDIR /reactnativesrc/
COPY . .

RUN npm install

RUN npx expo prebuild --platform android --no-install

WORKDIR android/

RUN chmod +x gradlew

RUN ./gradlew assembleRelease

FROM scratch

COPY --from=builder /reactnativesrc/android/app/build/outputs/apk/release/app-release.apk /reactnativeapp.apk

ENTRYPOINT ["/reactnativeapp.apk"]
```

---

# Understanding Multi-Stage Builds

This Dockerfile has **two stages**.

## Stage 1 – Build Stage

Purpose:

```text
Build the Android APK
```

Contains:

* Node.js
* React Native tools
* Expo
* Android SDK
* Gradle

---

## Stage 2 – Final Stage

Purpose:

```text
Store only the generated APK
```

Contains:

* app-release.apk

Nothing else.

This makes the final image extremely small.

---

# Stage 1 – Build Stage

---

## Step 1: Use React Native Android Image

```dockerfile
FROM reactnativecommunity/react-native-android:v20.1 AS builder
```

### Purpose

Uses a pre-configured image for React Native Android development.

### What Is Included?

This image already contains:

* Node.js
* npm
* React Native CLI
* Android SDK
* Java JDK
* Gradle

### Why Use This Image?

Without this image, you would have to manually install:

```text
Node.js
Java
Android SDK
Gradle
React Native Tools
```

which can be very time-consuming.

---

## Step 2: Set Working Directory

```dockerfile
WORKDIR /reactnativesrc/
```

### Purpose

Creates and switches to:

```text
/reactnativesrc/
```

All future commands run from this folder.

---

## Step 3: Copy Project Files

```dockerfile
COPY . .
```

### Purpose

Copies the entire React Native project into the container.

### Example

If your project contains:

```text
my-app/
│
├── package.json
├── App.js
├── app.json
├── assets/
├── src/
└── components/
```

everything gets copied into:

```text
/reactnativesrc/
```

inside the container.

---

## Step 4: Install NPM Dependencies

```dockerfile
RUN npm install
```

### Purpose

Installs all Node.js packages required by the project.

### Reads

```text
package.json
```

### Example Dependencies

```json
{
  "dependencies": {
    "react": "...",
    "react-native": "...",
    "expo": "..."
  }
}
```

### What Gets Created?

```text
node_modules/
```

This folder contains all downloaded packages.

---

## Step 5: Generate Native Android Project

```dockerfile
RUN npx expo prebuild --platform android --no-install
```

### Purpose

Converts the Expo project into a native Android project.

### What Is Expo?

Expo makes React Native development easier.

Normally:

```text
React Native Code
        ↓
Expo
        ↓
Android Project
```

---

### What Does prebuild Do?

Generates:

```text
android/
```

folder automatically.

### Example

Before:

```text
App.js
package.json
app.json
```

After:

```text
App.js
package.json
android/
```

The Android folder contains:

```text
Gradle files
AndroidManifest.xml
Build configuration
Native Android code
```

---

### What Does --platform android Mean?

```dockerfile
--platform android
```

Only generates Android files.

No iOS project is created.

---

### What Does --no-install Mean?

```dockerfile
--no-install
```

Prevents Expo from running npm install again.

Since dependencies are already installed, this saves build time.

---

## Step 6: Move to Android Directory

```dockerfile
WORKDIR android/
```

### Purpose

Changes current directory to:

```text
/reactnativesrc/android/
```

This folder contains:

```text
build.gradle
gradlew
settings.gradle
app/
```

---

## Step 7: Make Gradle Executable

```dockerfile
RUN chmod +x gradlew
```

### Purpose

Gives execute permission to:

```text
gradlew
```

### Why?

Linux cannot run a file unless it has execute permission.

Without this command:

```text
Permission denied
```

error may occur.

---

## Step 8: Build Release APK

```dockerfile
RUN ./gradlew assembleRelease
```

### Purpose

Builds the Android application.

### What Is Gradle?

Gradle is Android's build tool.

Similar to:

| Technology | Build Tool |
| ---------- | ---------- |
| Java       | Maven      |
| Node.js    | npm        |
| Python     | pip        |
| Android    | Gradle     |

---

### What Happens During Build?

Gradle:

1. Compiles Java/Kotlin code
2. Compiles React Native code
3. Packages resources
4. Creates APK file

---

### Output Location

After successful build:

```text
android/
└── app/
    └── build/
        └── outputs/
            └── apk/
                └── release/
                    └── app-release.apk
```

---

### What Is app-release.apk?

This is the Android installation package.

Similar to:

```text
Windows → .exe
Android → .apk
```

Users install this APK on Android devices.

---

# Stage 2 – Final Stage

---

## Step 9: Use Scratch Image

```dockerfile
FROM scratch
```

### Purpose

Creates an empty image.

### What Is Scratch?

Scratch means:

```text
No OS
No Shell
No Libraries
Nothing
```

It is completely empty.

---

### Why Use Scratch?

The goal is only to store:

```text
app-release.apk
```

No runtime environment is needed.

---

## Step 10: Copy APK File

```dockerfile
COPY --from=builder /reactnativesrc/android/app/build/outputs/apk/release/app-release.apk /reactnativeapp.apk
```

### Purpose

Copies the generated APK from Stage 1.

### Source

```text
/reactnativesrc/android/app/build/outputs/apk/release/app-release.apk
```

### Destination

```text
/reactnativeapp.apk
```

inside the final image.

---

## Step 11: Entry Point

```dockerfile
ENTRYPOINT ["/reactnativeapp.apk"]
```

### Purpose

Specifies the default file in the image.

### Important Note

This APK cannot actually run inside a Docker container.

APK files are designed for:

```text
Android Devices
Android Emulators
```

not Linux containers.

In practice, this Docker image is mainly used as an artifact container to store and extract the APK.

---

# Build Process Flow

```text
React Native Source Code
            │
            ▼
Copy Project Files
            │
            ▼
Install Dependencies
            │
            ▼
Expo Prebuild
            │
            ▼
Generate Android Project
            │
            ▼
Gradle Build
            │
            ▼
Create app-release.apk
            │
            ▼
Copy APK to Final Image
            │
            ▼
APK Ready for Distribution
```

---

# Folder Structure During Build

```text
/reactnativesrc
│
├── package.json
├── App.js
├── node_modules/
│
├── android/
│   ├── gradlew
│   ├── app/
│   └── build/
│
└── app.json
```

After build:

```text
android/app/build/outputs/apk/release/

└── app-release.apk
```

---

# Summary

## Stage 1 – Builder

1. Uses React Native Android build image.
2. Copies React Native source code.
3. Installs npm dependencies.
4. Generates Android native project using Expo.
5. Makes Gradle executable.
6. Builds Android release APK.

## Stage 2 – Final Image

7. Creates an empty scratch image.
8. Copies only the APK file.
9. Produces a very small image containing the release APK.

### Final Result

The Docker build generates:

```text
app-release.apk
```

which can be:

* Installed on Android phones
* Installed on Android tablets
* Uploaded to testing platforms
* Distributed to users
* Published to app stores after signing

This Dockerfile is essentially an automated Android APK build pipeline inside Docker.
