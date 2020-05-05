---
layout: page
title: How to programatically convert an HTML5 slideshow to a video using Node.js, Puppeteer & FFMPEG.
description: When it comes to making a video programatically, the first choice is mostly FFMPEG. But that wasn't enough for me to convert an HTML5 slideshow to video, then I met Puppeteer.
permalink: /convert-an-html5-slideshow-to-a-video/
tags: ['html4', 'video', 'node.js']
comments: true
---

I wanted to build a web based videomaker that allows users to make their own videos by simply drag and dropping images to a template, similar to what [Animoto](https://animoto.com){:target="\_blank"} does. I tried using `FFMPEG` to stitch images together and position text elements with some translations, but it wasn't so smooth like the modern HTML5 slideshows you can create using simething like [Tweenmax](https://greensock.com/tweenmax/){:target="\_blank"}.

## Solution

This is going to be very high level explanation of the solution. And to make it simple, I'm not going to explain about how to build a video-like slide show with `tweenmax` in this post. I'll be happy to update this article to a low-level one with full code samples if anyone is interested to know.

This solution involves mainly three components,

1. A web application that renders the html slideshow.
2. A Node.js script that uses `puppeteer` to render the web application and capture frames. I'll refer this script as `puppeteer-runner`.
3. Another node.js script that uses `FFMPEG` to stitch the frames together. **Please note, you also need `ffmpeg` installed on the system.**

I'm going to assume that your slideshow is built using `tweenmax` for the sake of simplicity.

### Puppeteer

[Puppeteer](https://developers.google.com/web/tools/puppeteer) is a Node library, built by Google, which provides a high-level API to control headless Chrome or Chromium over the DevTools Protocol. It can also be configured to use full (non-headless) Chrome or Chromium.

### FFMPEG

[FFmpeg](https://github.com/FFmpeg/FFmpeg) is a collection of libraries and tools to process multimedia content such as audio, video, subtitles and related metadata.

### Approach

#### Capturing frames.

The web application should expose a url that runs the slideshow in fullscreen. However, it should not autoplay the slideshow. Instead, it should invoke the puppeteer script to start recording. Any of the `waitFor` methods of puppeteer can be used for this. For example, we can use `page.waitForSelector('.page-ready')` and create an element with class `page-ready` and puppeteer starts the process.

```javascript
// puppeteer-runner.js
const puppeteer = require('puppeteer');

const width = 1280;
const height = 720;
const url = '<your webpage url>';
const tempPath = '/tmp/';

puppeteer.launch({ headless: true }).then(async browser => {
  const page = await browser.newPage();
  page.setViewport({ width, height });
  await page.goto(url, { waitUntil: 'networkidle' });
  await page.waitFor('.page-ready');

  const captureFrame = async () => {}; // we'll define this below.

  await captureFrame();
});
```

The above code snippet initializes puppeteer and navigate to the webpage url and then executes the `captureFrame` method once the page is ready.
This method can be used to seek through the slideshow, remember, the slideshow is not autoplaying.

The web page should also expose a function, which can be called from the `puppeteer-runner` script. I'll call it `seekSlideShow` and this method should seek through the slide show frame by frame. For example the slideshow will be at position `0` when it starts, and calling this method should move that to `1` and then on next call, to `2` and continue like that.

With tweenmax, it'd look something like this,

```javascript
// clientside script

const animation = new TimelineMax();
// all yout slide show config goes here..

const seekSlideShow = () => {
  // animation
  animation.progress(this.current++ / this.frames);
};
```

We'll call this method using the `page.evaluate` method of puppeteer, from the `captureFrame` function, like this,

```javascript
// puppeteer-runner.js

let frame = 0
const captureFrame = async () => {

  // seekSlideShow this method should be defined by the webpage.
  await page.evaluate(() => seekSlideShow());
  const framePath = path.join(tempPath, frame,'.png' })
  await page.screenshot({ format: 'png', path: framePath)

  frame++;
  captureFrame();
};
```

As you can see in the above code,

1.  It evaluates `seekSlideShow` method, and the webpage changes to next slide.

2.  then it captures the frame using `page.screenshot` method and store it in the temporary directory.

3.  and finally it increments the frame number and calls the `captureFrame` method again in a recursive fashion.

This continues until all the frames are captured and we'll have the frames stored in the temporary directory.

#### Stitching the frames together.

Now that we have all the frames for the video, we are half done. The next task is to use `FFMPEG` to stitch all these together, and maybe also include an audio.

One thing to note here is, the frames should be sorted in sequence and the framerate you use for `FFMPEG` should be in sync with what you have for the slideshow.

There are several methods to combine images, but I used `image2pipe` method of ffmpeg and passed the images as `stdin`. The below code shows how I executed it, you might have to tweak the args as you want.

```javascript
// ffmpeg-executor.js

const ffargs = [
  '-i',
  audio,
  '-y',
  '-c:v',
  'png',
  '-f',
  'image2pipe',
  '-r',
  fps,
  '-i',
  '-',
  '-c:v',
  'libx264',
  '-pix_fmt',
  'yuv420p',
  '-af',
  'afade=t=out:st=' + afadest + ':d=3',
  '-crf',
  '27',
  '-preset',
  'veryfast',
  '-movflags',
  '+faststart',
  '-shortest',
  output,
];

const ffmpeg = spawn('ffmpeg', ffargs);
const async = require('async');

fs.readdir(tempPath, (err, files) => {
  files.sort(function(a, b) {
    return parseInt(a.split('.')[0]) - parseInt(b.split('.')[0]);
  });
  async.eachSeries(
    files,
    function(file, done) {
      const stream = fs.readFileSync(tempPath + '/' + file);
      ffmpeg.stdin.write(stream);
      done();
    },
    function() {
      ffmpeg.stdin.end();
      fsxt.rmrf(tempPath);
    }
  );
});
```

This will output the final video to the directory we specified as `output` in the `ffargs`.

#### Combining the above two steps

It is important that the above two processes should work in sync. I made these two as separate scripts and spawned it as `child_process` and then piped the output of `puppeteer-runner` to `ffmpeg-executer.js`

```javascript
// main.js

const puppeteerRunner = spawn('node', ['puppeteer-runner.js']);
const ffmpegExecuter = spawn('ffmpeg', ffargs);

puppeteerRunner.stdout.pipe(ffmpegExecuter.stdin);
ffmpegExecuter.stdout.pipe(process.stdin);

puppeteerRunner.on('close', function() {
  ffmpeg.stdin.end();
  console.log('pup closed');
});

ffmpegExecuter.on('close', function() {
  process.stdin.end();
  console.log('ffmpeg process completed');
  process.exit();
  //save video & info here and exit.
});
```

That's it. Hope that helps :)
