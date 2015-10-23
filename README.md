#Facebook's AutoPkg Recipes
This repository is a collection of recipes for use with [AutoPkg](https://github.com/autopkg/autopkg). This collection represents AutoPkg recipes that the Client Platform Engineering (CPE) team has authored.

You can see other projects from the CPE team at the [IT-CPE GitHub repo](https://github.com/facebook/IT-CPE).

## Usage

Clone the repo using `autopkg repo-add` or `git clone`:
`autopkg repo-add https://github.com/facebook/Recipes-for-AutoPkg.git`  
or  
`git clone https://github.com/facebook/Recipes-for-AutoPkg.git ~/Library/AutoPkg/RecipeRepos/com.github.autopkg.facebook-recipes`

Then run any of the recipes with `autopkg run`.

## Recipe List

### Android NDK

The [Android Native Development Kit](https://developer.android.com/ndk/guides/index.html) recipes use two custom processors, "AndroidNDKVersioner" and "LinuxBinExtractor".  

The Android NDK downloads as a Linux self-extracting binary (see [here](http://stackoverflow.com/questions/955460/how-do-linux-binary-installers-bin-sh-work) and [here](http://www.linuxjournal.com/node/1005818) for more details), which contains no version information and only a collection of files.  The "LinuxBinExtractor" processor changes directory to the download location and locally executes the self-extracting binary.

Versioning is determined naively by parsing the download URL itself.  If a local file was provided instead of a download, the self-extracting binary itself is queried for its table of contents, and the topmost directory is parsed for a version string.

Because of the way the self-extracting binary process executes, the DMG creation that takes place will generate a **new** DMG each time, which Munki would try to reimport each time because there's nothing on the disk image that Munki can use to determine a matching version. 

The only way to avoid Munki reimporting the same DMG each time is to abort the recipe  if the download hasn't changed.  Thus, by definition, this recipe is technically **not idempotent**. 

### Android SDK

The [Android Software Development Kit](https://developer.android.com/sdk/index.html) recipe uses a number of custom processors to determine versioning and URLs.  

This recipe parses two XML files that Google uses to inform the URLs, versions, and names of all packages related to the SDK. The Android SDK installer, which is a Java app, can download all of these manually (and uses those two XML URLs to figure out what to download).

This is a *minimal* set of packages necessary to start developing with the Android SDK. It is not the *complete* SDK. 

The Android SDK requires Java 7 in order to function.

### BlueJeans

[BlueJeans](http://bluejeans.com/) is a video conferencing tool. Previously, BlueJeans was only available as browser plugins, but they now offer desktop applications for Windows and OS X.

This recipe will download the BlueJeans installer package and create a straightforward app-dmg that copies the BlueJeans.app into `/Applications`.

### DbVisualizer

[DbVisualizer](https://www.dbvis.com/) is a database visualizer tool. The software has a Free version, and can be upgraded into Pro version with proper licensing.

### Duo

[Duo](https://www.duosecurity.com/) is a 2 factor authentication system. This recipe downloads the latest version of [Duo Unix](https://www.duosecurity.com/docs/duounix) and compiles it to be placed into /usr/local/bin/ rather than /usr/bin/, for compatibility with 10.11 El Capitan.

Since this is a standard configure / make / make install Linux installer, the custom processors provided here will create a tiny sandbox to do the custom install into the AutoPkg cache, so it won't affect your current system.

### ExpandDrive

[ExpanDrive](http://www.expandrive.com/) can mount remote network storage (SFTP, WebDav, Amazon Cloud Drive, S3, etc.) as local file system storage.

### Intellij IDEA

[IntelliJ IDEA](http://www.jetbrains.com/js2) is a lightweight developer IDE for Java, Groovy, and Scala.

This recipe contains a custom URL provider, "IntellijURLProvider" that searches their versioning API for the latest download.  

### LastPass

[LastPass](https://lastpass.com) is a set of password manager browser plugins for Safari, Chrome, and Firefox.  

LastPass's installer comes in a .zip file that unzips to an .app installer that the users must run, which installs browser plugins to a user-selected list of installed browsers.  This is not ideal for mass deployment, so instead, package recipes are provided for the big three major browsers.

The Safari installer **must be run as an optional install / while a user is logged in**.  The only way to install a browser plugin in Safari is interactively, and this recipe does not have any mechanism for deferred installs after a user has logged in.  If Munki (or any other deployment system) tries to install this package at the login window, it will **fail**, since it tries to launch Safari in the process.

The Firefox installer makes use of Mike Kaply's [Client Customization Kit](https://mike.kaply.com/cck2/) to install a configuration in Firefox that force the loading of this extension.  The appropriate configuration is provided by this recipe, but note that the CCK2 configuration is all generated by external code, and may break in future iterations of Firefox.  Please see the CCK2 website for details on licensing, usage, and support.

The Chrome installer uses a side-loading technique that is likely to stop working by the end of the year as Google continues to [tighten its requirements for installing browser extensions](https://productforums.google.com/forum/#!topic/chrome/d35tIyH8dVM%5B1-25%5D).  If this method stops working, [administrator policy](https://www.chromium.org/administrators) will likely be the only method to enforce installation of Chrome extensions.

### Lobby Video

This recipe is more of a proof of concept than a usable tool for others.  This recipe serves as an example for a way to take an unversionable binary file, such as a large movie file, and create a versionable item that Munki can handle.

This particular example uses a custom processor "DateVersioner" that simply uses the current date and time to name the item, and set a version for Munki importing.

Since the version is not deterministic and not repeatable unless you change the system clock to go back in time, this recipe is **not idempotent**. It should be run **manually** whenever the source media file is updated.

### Mosh

The [Mobile Shell](https://mosh.mit.edu/), known as "Mosh", is a remote terminal application ideal for certain network conditions and environments.  

Mosh can be obtained via [GitHub](https://github.com/mobile-shell/mosh) and built from source, installed via [Homebrew](http://brew.sh/), or downloaded as a prebuilt package.  This recipe set provides downloads for GitHub releases as well as the prebuilt package hosted on the website.

Two custom processors are included, "FBGitHubReleasesInfoProvider" and "Mosh Versioner".  

Versioning is accomplished with the custom processor by naively looking at the package name, which is fragile and dependent on the developers maintaining consistency with filenames.

The "FBGitHubReleasesInfoProvider" is nearly identical to the [AutoPkg GitHubReleasesInfoProvider](https://github.com/autopkg/autopkg/blob/master/Code/autopkglib/GitHubReleasesInfoProvider.py) processor, but contains some additional logic to parse the correct version, since the developers have been listing the release version as "mosh-1.2.x".

### SQLDeveloper

[SQLDeveloper](http://www.oracle.com/technetwork/developer-tools/sql-developer/overview/index-097090.html) is an integrated development environment from Oracle that makes it easier to work with Oracle Databases and database applications.

Unfortunately, the software is a native .app wrapped around a Java app, and the native .app version is static and does not change with updates.  A custom versioning processor looks for a file inside the Java core that thankfully is updated with new builds when new versions are released.

### VMware Fusion (Mass Deploy)

[VMWare Fusion](http://www.vmware.com/products/fusion-pro/) needs no introduction.  This recipe follows the instructions on creating a [mass-deployment package](http://kb.vmware.com/selfservice/microsites/search.do?language=en_US&cmd=displayKC&externalId=2058680) that is pre-serialized.  

The parent recipe for this is in [Justin Rummel's VMware Fusion](https://github.com/autopkg/justinrummel-recipes/tree/master/VMware%20Fusion) recipe set. **You must add Justin Rummel's recipes to your search directories before you can use this recipe:**  
`autopkg repo-add justinrummel-recipes`

To use this recipe, you must, at a bare minimum, create an override and replace the license key with your own.  The `deploy.ini` file contents are an input variable, so you can place in your own settings as you see fit.

Though this recipe does include both .pkg and .munki variants, the .pkg recipe only creates a bundle package, not a flat package, and therefore will need to be put into a DMG / archive of some sort to be used with other deployment mechanisms (JSS, Absolute, etc.).

## Shared Processors

Facebook also provides a number of [Shared Processors](https://github.com/autopkg/autopkg/wiki/Processor-Locations#shared-recipe-processors) that can be used in multiple processors.

Facebook's common convention for custom processors is to bundle together recipe-specific tools with the recipe itself - such as unique Versioners, URL providers, etc.  Some processors serve useful purposes for many recipes (such as rsync), and therefore make more sense to be shared in a common location.

These shared processors can be referred to by using the stub recipe identifier `com.facebook.autopkg.shared`:

```
    <dict>
      <key>Processor</key>
      <string>com.facebook.autopkg.shared/SubDirectoryList</string>
      <key>Comment</key>
      <string>Get list of folder contents</string>
      <key>Arguments</key>
      <dict>
        <key>root_path</key>
        <string>%RECIPE_CACHE_DIR%/unpack</string>
      </dict>
    </dict>
```

See the README inside the Shared_Processors directory for more details about the provided processors.