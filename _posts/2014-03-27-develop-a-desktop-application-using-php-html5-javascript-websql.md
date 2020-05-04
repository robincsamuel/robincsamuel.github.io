---
layout: page
title: Develop a desktop application using PHP, HTML5, JavaScript & WebSql.
description: I recently had to develop a desktop application real quick and `php-desktop` worked out. Php-desktop helped me to develop a clean native desktop application using technologies such as PHP, HTML5, JavaScript & Web Sql.
permalink: /develop-a-desktop-application-using-php-html5-javascript-websql/
tags: ['javascript', 'html5', 'desktop']
comments: true
---

**IMPORTANT: This is something I did in 2014, and there are much better and easier options to develop a desktop application using HTML5, CSS and Javascript. Checkout [Electron.js](https://www.electronjs.org/){:target="\_blank"} instead.**

I recently had to develop a desktop application real quick and `phpdesktop` worked out. Php-desktop helped me to develop a clean native desktop application using technologies such as PHP, HTML5, JavaScript & Web Sql. I'll introduce some tools for the development before I move on to the configuration part. In fact, we are just using some tools to wrap up the web application with a portable browser & server.

## Get the Tools

- PhpDesktop: PHP Desktop is an open source project founded by Czarek Tomczak in 2012. It is a combination of portable php server and a portable browser, and it uses chromium embedded framework to run the application. Php desktop can be downloaded from [here](https://github.com/cztomczak/phpdesktop){:target="\_blank"}.
- Resource Hacker: It's an application to modify the attributes of a .exe file, so that we can change the icon. Download resource hacker from [here](http://www.angusj.com/resourcehacker/){:target="\_blank"}.
- Inno Setup: Inno Setup is a free installer for Windows programs. This helps to compile our application to a easy installable setup file. And inno setup can be downloaded from [here](https://jrsoftware.org/isinfo.php){:target="\_blank"}.

Let's move on to the configuration, assuming you got all these tools.

## Setting up

Extract phpdesktop, and you may rename it to whatever you want. I'm sticking with `phpdesktop` and I'll be referring it like that through out the article. Locate the `www` folder at the root, and you may delete all the files inside. The `www` folder is actually the application root and we keep the web application here. It'd be better to develop the application as a web app, and then copy it to the `www` folder.

Since phpdesktop is using chromium engine, we don't have to worry much about the browser compatibility and only need to consider `webkit`. And therefore you can stick to chromium based browsers (google chrome, brave, etc) for development to avoid compatibility issues. `localStorage` or `websql` can be used for persistant storage. If you are new to websql, [here](http://html5doctor.com/introducing-web-sql-databases/){:target="\_blank"} is an introduction.

Once we have the web application ready, we can copy it to the `www` folder. And then, we need to change some configurations in the application settings.

locate the `settings.json` file in the `phpdesktop` folder.

I'm covering the major options below, but I would recommend going through the documentation of `setting.json` at the phpdesktop [wiki](https://github.com/cztomczak/phpdesktop/wiki/Settings){:target="\_blank"}.

Update the following option in the `settings.json` file.

```
show_console = false
listen_on = ["127.0.0.1", 0]
```

Replace `0` with any port number of your choice between, `49152–65535`. If you don't change this, OS will assign random ports on every run and this may affect the `localStorage` since the application url will be different everytime.

### Chrome settings

Phpdesktop allows to change the Chrome settings through the same `settings.json` file. I'm showing a few options below, all the possible options can be found [here](https://github.com/cztomczak/phpdesktop/wiki/Chrome-settings){:target="\_blank"}.

Set the `cache_path` from `webcache` to `C:/ProgramData/YouApplicationName/`, to avoid write permission issues.

And if we need access to HTML5 apis like `getUserMedia`, we have to add a flag to `command_line_switches`. A list of flags supported by chromium can be found [here](https://peter.sh/experiments/chromium-command-line-switches/){:target="\_blank"}.

```
"command_line_switches": {"enable-media-stream":""}
```

**Please remember to remove prefixed – or —- while adding the flag to the `command_line_switches` object, and assign an empty string, `""`, as value to make it a valid `JSON`.**

For example, adding `–enable-accelerated-2d-canvas` flag would look like this,

```
"command_line_switches": {"enable-media-stream":"", "enable-accelerated-2d-canvas":"", }
```

### Application icon

The application icon can be changed using the `resource hacker` tool I mentioned in the tools section. Once you have the resource hacker installed, open your executable (`phpdesktop-chrome.exe` is the default one), with the `resource hacker`. Expand the tree on the left side `/Version Info/1/1033;` and change the `FileDescription`,`FileVersion` and other related values.

The original `phpdesktop-chrome` will be renamed to `phpdesktop-chrome_original` and a new one will be created with the new icon. Rename it to your app name, am naming it `myapp.exe` and will refer it using that name going forward.

Execute the `myapp.exe` and your application should open in a new window.

## Develop an Installer

Now that our app is running, we need to configure an installer. I'm using Inno setup to develop an installer for the application.

Start Inno setup and click `new` from the file menu and continue with the wizard. The wizard is straight forward and one can follow the instrutions to finish it. I'm just explaining the main step, `Application Files`. Here we can define the applications' executable, locate the project directory and select `myapp.exe`.

The next step is to add folders & files. Select `add folder`, browse to phpdesktop folder and click `add`. We have the options to configure `Icons`, `start menu shortcuts`, `languages` in the next steps. Once you complete the wizard a script will be generated, and we have to tweak some of the configurations there, for example, creating the data directory.

To create a data directory, you can add,

```
[Dirs]
 Name: "{commonappdata}\{#MyAppName}\data"; Permissions: everyone-full;
```

Please note, this should constitute the same path as the `cache_path` we’ve defined in the setting.json.

We can now add the app to the start up programs, by adding the following under the `[icons]` section,

```
Name: "{commonstartup}\{#MyAppName}"; Filename: "{app}\{#MyAppExeName}"
```

The `Flags: unchecked;` property can be removed to make any of the checkboxes checked by default.

**We dont have to replace the above variables `MyAppExeName`, `commonstartup` etc, since they are variables.**

That's the ma about the configurations, we can now build the installer from `build > compile`. The Installer file will be generated at the destination you selected in the wizard.

Done!
