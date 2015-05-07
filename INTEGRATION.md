![Gilt Tech logo](https://raw.githubusercontent.com/gilt/Cleanroom/master/Assets/gilt-tech-logo.png)

# CleanroomASL Integration Notes

This document describes how to integrate CleanroomASL into your application.

*Integration* is the act of embedding the `CleanroomASL.framework` binary (and its required dependencies) into your project and exposing the API it provides to your code.

Note that CleanroomASL is built as a *Swift framework*. This has several implications, not the least of which is that it will only work on iOS 8 or above. It is also only supported for use by other Swift code. Some, all or none of it may work from Objective-C; we haven't tried it, we wouldn't recommend it, and we don't support it.

### Requirements

CleanroomASL requires a **mimimum Xcode version of 6.3** to be built, and the resulting binary can be used on **iOS 8.1 and higher**.

We'll also be using the `git` command in the Terminal for installing the CleanroomASL repo in your codebase. These steps have been tested with **git 2.3.2 (Apple Git-55)**, although they should be compatible with a wide range of git versions.

Lastly, some familiarity with the Terminal application and the bash command line is assumed.

### About Frameworks

Official support for third-party iOS frameworks was introduced with Xcode 6 and iOS 8. Prior to that, developers had been using frameworks in an unsupported fashion by placing shared libraries and resources inside a filesystem structure that mimicked that of the frameworks published by Apple.

Given that *real* third-party iOS framework support is still in its infancy, it should be no surprise that there are still kinks in the development process when using them:

- The iOS Simulator uses a different processor architecture than real devices. As a result, **frameworks compiled for device won't work in the simulator, and vice-versa**. Sure, you *could* use `lipo` to stitch together a universal binary and use that instead, but...

- **Apple will not accept App Store binaries containing iOS Simulator code.** That means if you go the universal binary route, now you need a custom build step to make the universal binary, and you need another step to un-make the universal binary when you're building for submission. Okay, well, why not just build *two* frameworks: one for the iOS Simulator and one for devices?

- **Xcode won't be happy if you import two separate frameworks with the same symbols.** Xcode won't notice that there really is no conflict because the frameworks are compiled for different processor architectures. All Xcode will care about is that you're importing two separate frameworks that both claim to have the same module name.

For these reasons, we do not release Cleanroom projects as framework binaries. Instead, we provide the source, the Xcode project file, and options for integrating so that you can use it from within your code and submit an app that Apple will accept. (Or, at the very least, if Apple *doesn't* accept your app, we don't want it to be the fault of *this* project!)

### Options for integration

There are two supported options for integration:

- **Manual integration** — The `CleanroomASL.xcodeproj` Xcode project file is embedded directly within your project. You then add `CleanroomASL.framework` and its dependencies to the *Embedded Binaries* and *Linked Frameworks and Libraries* sections under the *General* tab for your application target.

- **Carthage integration** — [Carthage](https://github.com/Carthage/Carthage) is a dependency package manager designed to build frameworks. To add CleanroomASL using Carthage, you would add `github "emaloney/CleanroomASL"` to your `Cartfile` and then issue the command `carthage update`.

Whether you choose one over the other largely depends on your preferences and—in the case of Carthage—whether you're already using that solution for other dependencies.

## Manual Integration

Manual integration involves embedding `CleanroomASL.xcodeproj` directly in your Xcode project.

This will ensure that CleanroomASL is built with the exact same settings you’re using for your app. You won’t have to fiddle with different settings for different architectures — when you're running in the simulator, it will work; when you then switch to building for device, it'll work, too.

You’ll also be able to step into CleanroomASL code directly in the debugger without worrying about `.dSYM` resolution, which is very helpful.

### The strategy

Manual integration is a bit involved, but there are three high-level tasks that you'll need to perform:

1. Download the CleanroomASL source into your project structure

2. Embed `CleanroomASL.xcodeproj` in your Xcode project

3. Build `CleanroomASL.framework`; this will also cause any required dependencies to be built

4. Add `CleanroomASL.framework` and the required dependencies to your application target

5. Fix the way Xcode references the frameworks you added in Step 4

#### Getting Started

Launch Terminal on your Mac, and `cd` to the directory that contains your application.

For our integration examples, we're going to be showing the top-level `CleanroomASL` directory inside a `Libraries` directory at the root level of of your application's source.

> You do not *need* to use this structure, although we'd recommend it, if only to make the following examples work for you without translation.

If you do not already have a `Libraries` directory, create one:

```bash
mkdir libraries
```

Next, `cd` into `Libraries` and follow the instructions below.

### 1. Download the CleanroomASL source

If you're already using git for version control, we recommend adding CleanroomASL to your project as a submodule. This will allow you to "lock" your codebase to specific versions of CleanroomASL, making it easier to incorporate new versions on whatever schedule works best for you.

If you're using some other form of version control of if you're not using version control at all—*shame on you!*—then you'll want to *clone* the CleanroomASL repository. We suggest putting the CleanroomASL clone somewhere within your application's directory structure, so that it is included in whatever version control regimen you're using.

#### Downloading CleanroomASL as a submodule

From within the `Libraries` directory, issue the following commands to download CleanroomASL and its dependencies:

```bash
git submodule add https://github.com/emaloney/CleanroomASL.git
git submodule update --init --recursive
```

#### Downloading CleanroomASL as a cloned repo

From within the `Libraries` directory, issue the following command to clone the CleanroomASL repository:

```bash
git clone --recursive https://github.com/emaloney/CleanroomASL.git
```

### 2. Embed CleanroomASL in your project

Enter the command `open CleanroomASL` to open the folder containing the CleanroomASL source in the Finder. This will reveal the `CleanroomASL.xcodeproj` Xcode project and all files needed to build `CleanroomASL.framework` and its dependencies.

Then, open your application in Xcode, and drag `CleanroomASL.xcodeproj` into the Xcode project browser. This will embed CleanroomASL in your project and allow you to add the targets built by CleanroomASL to your project.

### 3. Build CleanroomASL

Before we can add `CleanroomASL.framework` to your app, we have to build it, so Xcode has more information about the framework.

**Important:** The next step will only work when the framework is built for a **device-based run destination**. That means that you must either select the generic "iOS Device" run destination before building, or you must select an actual device (an option that's only available when a device is connected).

Once a device-based run destination has been selected, select the "CleanroomASL" build scheme. Then, select *Build* (⌘B) from the *Product* menu.

Once the build is complete, open `CleanroomASL.xcodeproj` in the project navigator and find the "Products" group. Open that, and right-click on `CleanroomASL.framework`. Select *Show in Finder*. This will open the folder containing the framework binary you just built.

If all went well, you should see several files in this folder; the ones we're concerned with are:

- `CleanroomASL.framework`
- `CleanroomBase.framework`

If those files aren't present, something went wrong with the build.

### 4. Add the necessary frameworks to your app target

In Xcode, select the *General* tab in the build settings for your application target. Scroll to the bottom of the screen to reveal the section entitled *Embedded Binaries* (the second-to-last section).

Go back to Finder, and option-click `CleanroomASL.framework` and `CleanroomBase.framework` to select them both, and then drag them into the list area directly below  *Embedded Binaries*`.

If successful, you should see `CleanroomASL.framework` and `CleanroomBase.framework` listed under both the *Embedded Binaries* and *Linked Frameworks and Libraries* sections.

### 5. Fix how Xcode references the frameworks

Unfortunately, Xcode will reference the frameworks you just added in a way that will eventually cause you pain, particularly if multiple developers are sharing the same project file (in which case the pain will be felt almost immediately).

So, to make things sane again, you'll need to make sure Xcode references `CleanroomASL.framework` and `CleanroomBase.framework` using a "Relative to Build Products" location.

To do this, repeat the following steps for each framework:

1. Locate the framework in the Xcode project browser
2. Select the framework
3. Ensure the Xcode project window's *Utilities* pane is open
4. Show the *File Inspector* in the *Utilities* pane
5. Under the *Identity and Type* section, find the dropdown for the *Location* setting
6. If the *Location* dropdown does not show "Relative to Build Products" as the setting, select "Relative to Build Products"

Once you've done this for each framework, **_you're all done integrating CleanroomASL!_**

Skip to the [Adding the Swift import](#adding-the-swift-import) section to see how you can import CleanroomASL for use in your Swift code.

## Carthage Integration

*Coming soon!*

## Adding the Swift import

Once CleanroomASL has been successfully integrated, all you will need to do is add the following `import` statement to any Swift source file where you want to use CleanroomASL:

```swift
import CleanroomASL
```

Want to read more about CleanroomASL? Check out [the README](https://github.com/emaloney/CleanroomASL/blob/master/README.md).

**_Happy coding!_**