# Websites archiver

As many other pentesters out there, I keep my notes in a bunch of text files on my phone.
Besides that, I also keep a list of all the interesting blog posts I read every day (which are _a lot_, there's plenty of clever people out there writing interesting content :D).
Normally, my text file contains the link, and a couple of sentences summarising what I though was interesting.
My intention doing so is to be able to later find that blog post when I need to revisit some concepts (i.e.: reproduce their findings, double-check that I'm quoting it properly because my memory sucks, ...).

However, I sometimes may want to do a deeper search on the actual contents (e.g.: get all posts that mention "SMB", even if that's not the main focus of the article), for which having the full text would be useful.

While browsing for options I could use, I came accross [markdownload](https://github.com/deathau/markdownload), which very much suited my needs of saving the text content.
Unfortunately, on Firefox for Android the pictures cannot be saved (the [downloads API](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/API/downloads#browser_compatibility) is not available anymore), and it doesn't look to be possible to save the output to arbitrary locations like `/sdcard/Documents/exported_files`.

Therefore, I decided to take the amazing work of the markdownload developers, port it into an app, and add on top the ability to screenshot the webpage.
I also wanted to give React Native a shot, which fits perfectly for this JS-based project.
In the end I simply injected JS into a WebView; but my original plan was to somehow do this from the React side.

# Usage

Easy peasy:

  1. Enter a URL on the text box.

  2. Press "Load website" and wait for it to finish (there should be a notification).

  3. Export to markdown or screenshot the site.

  4. Wait for the notifications to appear. In the case of screenshotting, it may take a good couple of minutes, depending on your hardware. You can leave the app in the background running while it generates the exports.

  5. Check the output directory (`/sdcard/Documents/archived_websites`) to see if the export went well. If it didn't, feel free to [create an issue](https://github.com/Foo-Manroot/websites-archiver/issues) on this repo.


# Limitations

- The screenshotting is slow as fuck.

- Sometimes the total height of the page is not properly calculated and you end up with a bunch of additional empty screenshots. The contents are properly saved, though, it's just additional garbage that will be generated. This is most likely because I'm forcing the load of the desktop webpage changing the `viewport`.


# Develop

For development, I used `npx react-native start`. Make sure your device is connected (via USB or with `adb connect <...>`).

The webview can be debugged with a chromium instance pointing to `chrome://inspect/#devices`.

## Code structure

Since I'm a shitty programmer, everything is inside `App.tsx`. The only changes I made to the other parts of the code are simply the resources (`assets/*.js`, pointed at by `android/app/src/main/assets/*.js`) added to the Android app.

## App.tsx

**To configure where the data is stored**, go to the beginning of the file and search for `BASE_PATH`. It currently points to `/sdcard//Documents/archived_websites`.

The file is quite self-explanatory; but here are a few pointers:
    - The conversion to markdown is handled by `injectMarkdownConverter`, which injects the dependencies ([Readability.js](https://github.com/mozilla/readability/blob/main/Readability.js) and [Turndown](https://github.com/mixmark-io/turndown)) into the webview, does a bunch of stuff, and returns a base64-encoded message to the app using `window.ReactNativeWebView.postMessage()`.
    - Similarly, screenshotting the webpage is handled by `injectHtml2canvas`, which relies on the [html2canvas](https://github.com/niklasvh/html2canvas) dependency.
    - Posted messages from the webview are handled by `handleOnMessage()`
    - File contents are saved by `saveFile()`
    - To attempt improving the screenhots and avoid saving the (better for readability, but worse for archiving) mobile version, the viewport is modified to the value `width=5000` right after loading the site


# Building for release

To create the APK, I used the following command: `npx react-native build-android --mode=release`

I guess for iOS is a similar process; but I haven't done so, so I don't really know how it's exactly done.
Feel free to contribute on that :)
