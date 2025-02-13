---
title: Pixel Data
layout: compform_chapter.pug

image: /pixels/images/og_image.png
hero_title: Pixel Data
old_description: Internally, computers often represent images as a array of pixel values. Accessing, processing, and generating pixel data directly allows you to explore a variety of low-level techniques.

description: Access pixel values directly to process and generate images.
software: p5.js
---

<script src="https://cdn.jsdelivr.net/npm/p5@1.4.0/lib/p5.js"></script>
<script src="/mess.js"></script>
<script src="./pixel_mess.js"></script>

## Pixels

Today, most computers use color displays that produce images using a grid of pixels. Conceptually, each color pixel is made up of three sub-pixels—red, green, and blue—though the actual hardware may use a different pattern. Because these pixels are small and close together, the image appears continuous. Because our eyes can only directly distinguish red, green, and blue light, the screen can reproduce most of the colors we are capable of seeing.
Today, most computers use color displays that produce images using a grid of pixels. Conceptually, each color pixel is made up of three sub-pixels—red, green, and blue—though the actual hardware may use a different pattern. Because these pixels are small and close together, the image appears continuous. Because our eyes can only directly distinguish red, green, and blue light, the screen can reproduce most of the colors we are capable of seeing.

<div class="one-up">

![Pixels](https://upload.wikimedia.org/wikipedia/commons/c/cb/XO_screen_01_Pengo.jpg) Photo by [Peter Halasz](http://commons.wikimedia.org/wiki/File:XO_screen_01_Pengo.jpg)

</div>

<div class="sidebar link-box">

[**Basic 2D Rasterization** Technical Article](https://magcius.github.io/xplain/article/rast1.html)

</div>

When you work with a graphics library like p5.js or the Javascript [Canvas API](https://developer.mozilla.org/en-US/docs/Web/API/Canvas_API), you don't have to think about individual pixels. You can use high-level methods like `ellipse()` and `image()`, and the library will do the low-level work of setting individual pixel values for you. This process of drawing shapes with pixels is called _rasterization_. Because the p5.js drawing calls handle the details of rasterization for you, you give up some control. When you draw a circle with `ellipse()` every pixel in the circle will be changed to the current `fill()` color. The p5.js library provides no way to fill the ellipse with random colors, or a gradient, or colors generated by a function. To get this level of control, you need to take control of the rasterization yourself. This is the classic high-level/low-level trade-off: coding at a low level takes more work but offers more control.

<div class="callout">

### Vector Monitors

Computer displays don't have to use pixels at all. One of the earliest video games, [Tennis for Two](https://en.wikipedia.org/wiki/Tennis_for_Two), used a cathode ray oscilloscope as its display. In a cathode ray oscilloscope, signals are drawn by magnetically deflecting a beam of electrons as they race from an emitter—the cathode—towards a phosphor-coated glass screen. The phosphor glows where the electrons strike it, creating an image. This type of display can show smooth lines without any of the aliasing artifacts of a pixel display.

[Vector Monitors](https://en.wikipedia.org/wiki/Vector_monitor) were used in later video games as well, including the [Tempest](<https://en.wikipedia.org/wiki/Tempest_(video_game)>) arcade game and several games for the [Vectrex](https://en.wikipedia.org/wiki/Vectrex) home console.

![Tennis for Two](https://upload.wikimedia.org/wikipedia/commons/5/50/Tennis_For_Two_on_a_DuMont_Lab_Oscilloscope_Type_304-A.jpg)

</div>

<!-- ## Gallery -->

This high/low level tradeoff has some important implications for the creative coder. When you work at a high level you are not responsible for the details. Because you are not responsible for the details you tend to think about them less. Working directly with the pixels leads to thinking about drawing with code differently.

{% slides %}
{% include slides.yaml %}
{% endslides %}

### Video Memory

Modern video pipelines are complicated, but at a basic level they work something like this: The red, green, and blue brightness values of every pixel on a display are stored in the computers RAM. In comptuers with a dedicated video card, this data is usually stored on the video card's VRAM. Once per display refresh, the video hardware reads this data from memory, pixel by pixel, and sends it to the display over a display interface like DVI or HDMI. Hardware in the display receives this data and updates the brightness of each pixel as needed. If you change the values in the RAM, you will see the changes reflected on the screen.

The memory used to store the screen's image is called the video buffer or framebuffer. Direct access to the screen's framebuffer is pretty unusual on modern computers, and high level libraries like p5.js don't (and can't) provide it. But P5.js does give you access to a pixel buffer storing the image shown on your sketch's canvas. When you call drawing functions like `rect()` and `ellipse()`, p5.js updates the appropriate values in this buffer. The buffer is then composited into the rendered webpage by the browser. The browser window is composited onto the display's framebuffer by the operating system and video hardware. The p5.js library provides two ways to directly read and set the color of a single pixel: `get()/set()` and the `pixels[]` array. Using `get()` and `set()` is easier, but using the `pixels[]` array is faster. This chapter has examples for using both.

<div class="callout">

A high-definition display is 1920 pixels wide and 1080 tall: 345,600 total pixels. Each pixel needs three bytes to represent its color value: one byte each for the red, green, and blue channels. In total that is 6,220,800 bytes—about 6 megabytes—of memory to keep track of the full HD image.

Today, 6 megabytes isn't much, but many older computers didn't even have enough RAM to keep an entire full-color image of the screen in memory at all. These computers used lower resolutions, limited palettes, and a [variety of tricks](https://www.youtube.com/watch?v=Tfh0ytz8S0k) to output to screen.

</div>

<!-- <div class="callout">

### Changes in Perspective

> Paradigm: a framework containing the basic assumptions, ways of thinking, and methodology that are commonly accepted by members of a scientific community.

Dictionary.com{attrib}

Beginning in this chapter we will look at different approaches to making form that encourage you to think about form making in fundamentally different ways.

- High Level Drawing APIs vs Direct Pixel Manipulation
- Absolute vs Relative Coordinates
- Vector Drawing vs Raster Drawing
- Immediate Mode Drawing vs Retained Mode Drawing
- Serial Drawing on a CPU vs Parallel Drawing on a GPU

</div> -->

## Writing Pixel Data

### Random Pixels Example

This example uses `set()` to set each pixel in a 10x10 image to a random color. This example doesn't write to the canvas pixels directly. Instead, it creates an empty image, writes to its pixels, and then draws the image to the canvas. This approach is more flexible and avoids the complexities of [pixelDensity](https://p5js.org/reference/#/p5/pixelDensity). This also lets us draw the image scaled-up so you can see the pixels easier.

{% js-lab "sketches/basic_pixels.js" %}

Let's look at the code in depth.

**Line 10:** Use `createImage()` to create a new, empty 10x10 image in memory. We can draw this image just like an image loaded from a `.jpg` or `.gif`.

**Line 11:** Use `loadPixels()` to tell p5 that we want to access the pixels of the image for reading or writing. You must call `loadPixels()` before using `set()`, `get()`, or the `pixels[]` array.

**Line 13:** Set up a nested loop. The inner content of the loop will be run once for every pixel.

**Line 15:** Use the `color()` function to create a color value, which is assigned to `c`. Color values hold the R, G, B, and A values of a color. The color function takes into account the current `colorMode()`.

**Line 16:** Use `set()` to set the color of the pixel at `x, y`.

**Line 20:** Use `updatePixels()` to tell p5 we are done accessing the pixels of the image.

**Line 22:** Use `noSmooth()` to tell p5 not to smooth the image when we scale it: we want it pixelated. This resembles Photoshop's 'nearest neighbor' scaling method.

**Line 23:** Draw the image, scaling up so we can clearly see each pixel.

### Gradient Example

This example has the same structure as the first one, but draws a gradient pixel-by-pixel.

{% js-lab "sketches/basic_pixels_2.js" %}

**Line 15:** Instead of choosing a color at random, this example calculates a color based on the current `x` and `y` position of the pixel being set.

### Random Access Example

The first two examples use a nested loop to set a value for every pixel in the image. The loop visits every pixel in a sequential order. That tactic is commonly used in pixel generating and processing scripts, but not always. This example places red pixels at random places on the image.

{% js-lab "sketches/basic_pixels_3.js" %}

<div class="activity challenges">

## Coding Challenges One

Explore this chapter's example code by completing the following challenges.{intro}

### Modify the Basic Example

1. Change the image resolution to `20x20`. `•`
1. Change the image resolution to `500x500`. `•`
1. Change the image resolution back to `10x10`. `•`
1. Make each pixel a random shade of blue. `••`
1. Make each pixel a random shade of gray. `••`
1. Color each pixel with `noise()` to visualize its values. `•••`

### Modify the Gradient Example

1. Make a horizontal black-to-blue gradient. `•`
1. Make a vertical green-to-black gradient. `•`
1. Make a horizontal white-to-blue gradient. `•`
1. Make a vertical rainbow gradient. Tip: `colorMode()` `••`
1. Create an inset square with a gradient, surrounded by randomly-colored pixels. `•••`
1. Make a radial gradient from black to red. Tip: `dist()` `•••`
1. Create a diagonal gradient. `•••`
   {continue}

### Modify the Random Access Example

1. Change the image resolution to `50x50`, adjust the code to fill the image. `•`
1. Instead of drawing single pixels, draw little plus marks (`+`) at random locations. `••`
1. Make each `+` a random color. `••`
   {continue}

### Start from Scratch

1. Use `sin()` to create a repeating black-to-red-to-black color wave. `•••`
1. Create a `128x128` image and set the blue value of each pixel to `(y&x) * 16`. `•••`
   {continue}

</div>

## Reading + Processing Pixel Data

The p5.js library also allows you to read pixel data, so you can process images or use images as inputs. These examples use this low-res black-and-white image of Earth.

![world.png](sketches/world.png){scale pixel}

### Read Pixels Example 1

This example loads the image of Earth, loops over its pixels, and white pixels to red and black pixels to blue.

{% js-lab "sketches/read_pixels_alt.js" %}

#### First we need to load an image to read pixel data from.

**Line 3:** Declare a variable to hold our image.

**Line 5:** The `preload()` function. Use this function to load assets. p5.js will wait until all assets are loaded before calling `setup()` and `draw()`

**Line 6:** Load the image.

#### With our image loaded we can process the pixels.

**Line 18:** Set up a nested loop to cover every pixel.

**Line 20:** Use `get()` to load the color data of the current pixel. `get()` returns an array like `[255, 0, 0, 255]` with components for red, green, blue, and alpha.

**Lines 22, 23, 24:** Read the red, blue, and green parts of the color.

**Line 27:** Check if the red value is 255 to see if it is black or white. Since we know the image is only black and white this is enough to check.

**Line 28 and Line 30:** Set `out_color` to red or blue.

**Line 33:** Set the pixel's color to `out_color`.

**Line 34:** Use `updatePixels()` to tell the image there has been an update. We didn't need to do this in the loop when we were just setting pixels, but here we mix `set()` and `get()`. p5.js requires calling `updatePixels()` anytime we switch from setting to getting or drawing.

<div class="callout">

Every time we switch from writing/setting to reading/getting, we have to call `updatePixels()`. This is because internally, p5.js will call `loadPixels()` when we call `get()` which will overwrite our changes in the pixel array. This extra updating and loading is why `get()` and `set()` are slower than accessing the `pixels[]` array directly.

</div>

### Read Pixels Example 2

This example compares each pixel to the one below it. If the upper pixel is darker, it is changed to magenta.

{% js-lab "sketches/read_pixels_2.js" %}

### Image as Input Example

This example doesn't draw the image at all. Instead, the image is used as an input that controls where the red ellipses are drawn. Using images as inputs is a powerful technique that allows you to mix manual art and procedurally-generated content.

{% js-lab "sketches/read_pixels_3.js" %}

<div class="activity challenges">

## Coding Challenges Two

Explore this chapter's example code by completing the following challenges.{intro}

### Modify Read Pixels Example 1

1. Make the program turn white pixels green. `•`
1. Turn the black pixels to a random shade of red. `•`
1. Turn the black pixels into a vertical, black-to-red gradient. `••`
1. Comment out line 34 which calls `updatePixels()`. What happens? `••`

<!-- <img src="sketches/world_100.png" style="image-rendering: pixelated;"> -->

### Modify Read Pixels Example 2

1. Change the `lightness()` comparison to `>`. `•`
1. Change the `lightness()` comparison to `!=`. `•`
1. Add an `else` block that changes the pixels to black. `••`
1. Starting with the original code without any changes, set the `out_color` to the average of `this_color` and `below_color`. Here is an example you could follow:`•••`
   {continue}

```javascript
var color_a = color(worldImage.get(0, 1));
var color_b = color(worldImage.get(0, 2));
var blended_color = lerpColor(color_a, color_b, 0.5);
```

1. Change `worldImage.set(x, y, out_color);` to `worldImage.set(x, y+1, out_color);`. `•••`
1. Remove the `if` statement (but not its contents) so that its content always runs. `•••`
   {continue}

### Modify Read Pixels Example 3

1. Tell the program to use the image below by switching which `loadImage()` call is commented out in `preload()`. `•`
1. Adjust the expression that determines `dot_size` to make the result prettier. `••`
   {continue}

<img src="sketches/world_100.png" style="image-rendering: pixelated;">

</div>

## Working Directly with the `pixels[]` Array

You can read and write individual pixel values with the `get()` and `set()` methods. These methods are easy to use, but they are really, really slow. A faster approach is to use `loadPixels()` and `updatePixels()` to copy the buffer to and from the p5 [pixels[]](https://p5js.org/reference/#/p5/pixels) array. Then, with a little bit of math, you can work directly with the `pixels[]` array data. This is a little more work but can easily be **thousands of times faster**.

### Performance

The built-in p5 `get()` function gets the RGBA values of a pixel in an image. Internally `get()` calls `loadPixels()` to make sure it is working with up-to-date information. This means that even when getting the values for a single pixel, _every_ pixel is read _every_ time you call `get()`. As noted in the [reference](https://p5js.org/reference/#/p5/get), this makes `get()` slower than accessing the values in the `pixels[]` array directly. In fact, `get()` can easily be 1000s of times slower.

<div class="callout">

The `get()` call is slow, and gets slower the larger your image is.

A `10 x 10` image has `100` pixels. Reading each pixel reloads all `100` pixel values. That means `10,000` pixel values are copied from the image into pixels[] array.

A `50 x 50` image has `2,500` pixels. Reading each pixel reloads all `2,500` pixels. That is `6,250,000` pixels copied.

A `1,920 x 1,080` image has `2,073,600` pixels. Reading all of those pixels with `get()` would require copying `4,299,816,960,000` pixels, but your browser will hang or crash first.

</div>

We can get much faster results by loading all of the pixel values **once** with `loadPixels()`, and then reading and writing the `pixels[]` array directly. Since we are reading from `pixels[]` ourselves, we can make sure we haven't changed the values we are trying to read and bypass the safety measures that slow down `get()`. We have to be a little more careful about what we are doing, though, or we might create bugs.

The `getQuick()` function below reads a pixel's color value from an image's `pixels[]` array. You must call `loadPixels()` before calling this function. When you are done working with the `pixels[]` array, you should call `updatePixels()` to update the image with your changes.

```javascript
// returns the RGBA values of the pixel at x, y in the img's pixels[] array
// returns values as an array [r, g, b, a]
// use instead of p5s built in .get(x,y), for much better performance (more than 1000x better in many cases)
// see: http://p5js.org/reference/#/p5/pixels[]
// we don't need to worry about screen pixel density here, because we are not reading from the canvas

function getQuick(img, x, y) {
  var i = (y * img.width + x) * 4;
  return [
    img.pixels[i],
    img.pixels[i + 1],
    img.pixels[i + 2],
    img.pixels[i + 3],
  ];
}
```

Copy the `getQuick()` function above into your sketch. You can then replace a built-in p5 `get` call with a call to `getQuick`:

#### Using `get()`

```javascript
// in loop
c = img.get(x, y);
```

#### Using `getQuick()`

```javascript
// before loop
img.loadPixels();

// in loop
c = getQuick(img, x, y);

// after loop
img.updatePixels();
```

### `get()` vs `getQuick()`

The following example compares the performance of using `get()` and `getQuick()` to read and invert the color value of a small image.

{% js-lab "sketches/performance.js" %}

### The Canvas + Pixel Density

You can work with the pixels in an image using `image.pixels[]` or the pixels of the canvas with just `pixels[]`.
When accessing the pixel data of the canvas itself, you need to consider the pixel density p5 is using. By default, p5 will create a high-dpi canvas when running on a high-dpi (retina) display. You can call `pixelDensity(1)` to disable this feature. Otherwise, you'll need to take into account the density when calculating a position in the `pixels[]` array.

The examples on this page work with the pixels of images instead of the canvas to avoid this issue altogether. If you need to work with the canvas, the [pixels](https://p5js.org/reference/#/p5/pixels) documentation has info on working with higher pixel densities.

<!--
### Using the `pixels` array

The p5.js `pixels` array holds four color values—red, green, blue, and alpha—for every pixel on the canvas. Images are two dimensional, but the `pixels` array is one dimensional. In order to read or write values in the `pixels` array, you need to find the index—the position in the array—of your target pixel. To do that, you need to know how the data is laid out. In p5.js the upper left pixel is the first pixel in the array and the other pixels follow left to right, top to bottom. Each pixel is represented by four values: red, green, blue, and alpha.

```
[r, g, b, a, r, g, b, a, r, g, b, a, r, g, b, a, r, g, b, a, r, g, b, a, ...]
 --- p1 ---  --- p2 ---  --- p3 ---  --- p4 ---  --- p5 ---  --- p6 ---
```

With data laid out like this we can find the index values as follows.

```
pixelNumber = y * imageWidth + x;
redIndex = pixelNumber * 4;
greenIndex = pixelNumber * 4 + 1;
blueIndex = pixelNumber * 4 + 2;
alphaIndex = pixelNumber * 4 + 3;
```

[[ diagram ]]

<div class="callout">  .warn
On a high-dpi or retina display, the canvas array may have more values than you expect. When you call `createCanvas(512,512)` on a high pixel density display, p5.js actually creates a canvas that is 1024x1024, or potentially even larger if the display pixel density is very large.
</div>




[get and set are slow, really slow, and why] -->

## Growing Grass

This example uses an image as an input to control the density and placement of drawn grass.

#### Input Image

![cf.png](./sketches/cf.png){scale}

{% js-lab "sketches/grass.js" %}

<div class="assignment">

## Keep Sketching!

### Sketch

Explore working with image pixel data directly. Most of your sketches should be still images.

<!-- This week, most of your posts should be still images. -->

Create **at least one** sketch **for each** of the following:

1. Generate an image from scratch: pixel by pixel. Don't call any high-level drawing function like `ellipse()` or `rect()`.
2. Load an image and process its pixels.
3. Use an image as an input source to control a drawing. Don't show the original image, just the output.

### Challenge: Pixel Ouroboros

Create code that processes an image. Feed the result back into your code and process it again. What happens after several generations?

<!-- Post your source image, the result after one generation, and the result after several generations. Alternately, capture 90 generations as frames and post as a video. -->

### Pair Challenge: Generate / Process

Work with a partner.

1. Make a sketch that generates an image pixel by pixel.
2. Give your image to your partner.
3. Create a sketch that pixel processess that image.

</div>

## Explore

<div class="link-box">

[**Reaction Diffusion in Photoshop** Tutorial](https://vimeo.com/61154654)
Create a pattern in Photoshop by repeatedly applying filters.

[**Factorio** Game Page](http://store.steampowered.com/app/427520/)
A game in which players gather resources to create increasingly complex technology and factories—sometimes building these structures into pixel art.

[**Icon Machine** Generator](http://brianmacintosh.com/iconmachine/)
A pixel art web app that randomly generates potion bottle icons.

[**Memory-Mapped Video: The Scanning Game** Technical Article](https://www.atariarchives.org/cgp/Ch02_Sec03.php)
An article on ASCII encoding and storage, part of a 1979 primer on computer graphics.

</div>
