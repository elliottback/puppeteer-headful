# NOTE!

This is no longer needed after Chrome v137 and [Chrome for Testing](https://developer.chrome.com/blog/chrome-for-testing).

You can achieve the desired headful test by using something like this in your ci:

```yaml
    - name: Start Xvfb and Test
      run: |
        Xvfb :99 -screen 0 1920x1080x24 &
        export DISPLAY=:99
        npm test
```

And something like this to start puppeteer:

```javascript
    browser = await puppeteer.launch({
        headless: false, 									// extension are allowed only in headful mode
        // devtools: true,                                  // Enable DevTools for debugging
        args: [
          `--no-sandbox`,									//Required for this to work in github CI environment
          `--start-maximized`,                              // this flag maximizes the browser window
          `--display=${process.env.DISPLAY ?? ':0'}`,       // fix for LXDE desktops
          `--disable-extensions-except=${extensionPath}`,
          `--load-extension=${extensionPath}`
        ]
    });
```

# Puppeteer Headful

[Github Action](https://github.com/features/actions) for [Puppeteer](https://github.com/GoogleChrome/puppeteer) that can be ran "headful" or not headless.

> Versioning of this container is based on the version of NodeJS in the container

## Fork

The original action is not working for me, I think because pupeteer has stopped working with "stable" tag.  This fork will use the action version.

## Purpose

This container is available to Github Action because there are some situations, mostly testing [Chrome Extensions](https://pptr.dev/#?product=Puppeteer&version=v1.18.1&show=api-working-with-chrome-extensions), where you can not run Puppeteer in headless mode.

## Usage

This action installs Puppeteer on top of a [NodeJS](https://nodejs.org) container, so you have access to run [npm](https://www.npmjs.com) scripts using args. For this hook, we hijack the entry point of the [Dockerfile](https://docs.docker.com/engine/reference/builder/), so we can start up [Xvfb](https://www.x.org/releases/X11R7.6/doc/man/man1/Xvfb.1.xhtml) before your testing starts.

```yaml
name: CI
on: push
jobs:
  installDependencies:
    name: Install Dependencies
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Dependencies
        uses: actions/setup-node@v2
        env:
          PUPPETEER_SKIP_CHROMIUM_DOWNLOAD: "true"
        with:
          args: install
      - name: Test Code
        uses: mujo-code/puppeteer-headful@16.6.0
        env:
          CI: "true"
        with:
          args: npm test
```

> Note: You will need to let Puppeteer know not to download Chromium. By setting the env of your install task to PUPPETEER_SKIP_CHROMIUM_DOWNLOAD = 'true' so it does not install conflicting versions of Chromium.

Then you will need to change the way you launch Puppeteer. We export out a nifty ENV variable `PUPPETEER_EXEC_PATH` that you set at your `executablePath`. This should be undefined locally so it should function perfectly fine locally and on the action.

```javascript
browser = await puppeteer.launch({
  args: ['--no-sandbox'],
  executablePath: process.env.PUPPETEER_EXEC_PATH, // set by docker container
  headless: false,
  ...
});
```
