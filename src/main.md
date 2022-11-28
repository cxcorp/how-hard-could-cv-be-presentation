<!-- .slide: data-background-color="#fff" -->

## Automating the boring parts of Cyberpunk 2077

*How hard could computer vision possibly be?*

Notes:
- questions: after presentation

---

![](img/cyberpunk.jpg)

Notes:
- so this game called Cyberpunk 2077 was released
- I really liked playing it
- and...

----
<!-- .slide: data-transition="fade" data-background-image="img/cyberpunk_puzzle.jpg" data-background-size="contain" -->

Notes:
- it had this cool *hacking minigame*, where you had to...

----
<!-- .slide: data-transition="fade" data-background-image="img/cyberpunk_puzzle_1.jpg" data-background-size="contain" -->

Notes:
- find all of these sequences...

----
<!-- .slide: data-transition="fade" data-background-image="img/cyberpunk_puzzle_2.jpeg" data-background-size="contain" -->

Notes:
- from this "code matrix", and ...

----
<!-- .slide: data-transition="fade" data-background-image="img/cyberpunk_puzzle_3.jpeg" data-background-size="contain" -->

Notes:
- you have to do it with 4 to 8 clicks, depending on your upgrades.
- And

----
<!-- .slide: data-transition="fade" data-background-image="img/cyberpunk_puzzle_4.jpeg" data-background-size="contain" -->

Notes:
- when you pick a digit, your next guess is locked to the same row or column.
- Oh, and...

----
<!-- .slide: data-transition="fade" data-background-image="img/cyberpunk_puzzle_5.jpeg" data-background-size="contain" -->

Notes:
- you have like 30 seconds, **good luck**
- at first *it was fun*, but it got old very quickly and started to get boring.
- So like every programmer, I decided I would just...

---

![](img/noik.png)

## Just brute force it. ™

----

<iframe src="https://cyberpunk-hacker.com" width="960px" height="700px"></iframe>

Notes:

- So I made this website, you could write the matrix there and it would solve it for you

```
7A 55 E9 E9 1C 55
55 7A 1C 7A E9 55
55 1C 1C 55 E9 BD
BD 1C 7A 1C 55 BD
BD 55 BD 7A 1C 1C
1C 55 55 7A 55 7A
```

```
BD E9 1C
BD 7A BD
BD 1C BD 55
```

----

![](img/solved.png) <!-- .element: class="r-stretch" -->

Notes:
- and this was cool for a while
- but then I got tired of writing out the matrix

---

## There's gotta be a better way?

- I don't want to alt-tab out of the game <!-- .element: class="fragment" -->
- Point phone at screen and get solution? <!-- .element: class="fragment" -->

Notes:
- There's gotta be a better way
- I don't want to alt-tab, it's distracting
- I just wanna point my phone and get a solution
- **then I realized**, we already do something like this with out phones

----

![](img/qr_rick_roll.png)

1. point phone at screen <!-- .element: class="fragment" -->
2. ??? <!-- .element: class="fragment" -->
3. it knows what's on the screen <!-- .element: class="fragment" -->

Notes:
**QR codes, right?!**
1. point phone at screen
2. something magic computer vision happens
3. profit


---

## How hard could computer vision possibly be?

----

![](img/opencv_wikipedia.png)

Notes:
- Turns out, there's this de-facto library for computer vision called **OpenCV**
- You can use C++, Python
- Even has a JavaScript library which uses WebAssembly
- **A-ha!**

----

### Great documentation!

<iframe src="https://docs.opencv.org/4.6.0/d5/d10/tutorial_js_root.html" width="960px" height="700px"></iframe>

Notes:
- The documentation is surprisingly good
- Teaches you basic....*primitives* of sorts
- Sort of "basic building blocks" for computer vision
  - blurring
  - thresholding
  - *morphological transforms*: spread/pull together pixels
  - contours

---

### Building the thing

1. Find if there's a code matrix in the image <!-- .element: class="fragment" -->
2. Extract that image <!-- .element: class="fragment" -->
3. Run OCR <!-- .element: class="fragment" -->
4. ??? <!-- .element: class="fragment" -->
5. Profit? <!-- .element: class="fragment" -->

---

### Find and extract the matrix

- It already looks like a grid
- How do we find the coordinates of the grid?

----

#### 1. Crop

<div style="display: flex; align-items: center; justify-content: center;">
<img src="img/img-original.jpg" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/img-cropped.png" width="300px"/>
</div>

Notes:
- After we write some code to get a video stream from the user's camera
- we crop it to try to get rid of lens distortion and get a nice square image
- Lets us draw a nice red square for the user to focus on

----

#### 2. Grayscale & blur it

<div style="display: flex; align-items: center; justify-content: center;">
<img src="img/img-cropped.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/img-grayscale.png" width="300px"/>
</div>
<div style="display: flex; align-items: center; justify-content: center;">
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/img-blurred.png" width="300px"/>
</div>

Notes:
- Many computer vision algorithms first blur the image to get rid of noise:
  - JPG artifacts
  - dust on the screen
- We don't want the colors because it makes things slow
- We don't **need the colors**

----

#### 3. Threshold & dilate

<div style="display: flex; align-items: center; justify-content: center;">
<img src="img/img-blurred.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/img-thresholded.png" width="300px"/>
</div>
<div style="display: flex; align-items: center; justify-content: center;">
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img class="fragment" src="img/img-dilated.png" width="300px"/>
</div>

Notes:
- Since our image is now black&white, it's basically just a grid of pixels
- Basically numbers from 0 to 255 from dark to bright
- Set some limit or *threshold* like 185
  - Pixel dimmer than 185 -> black
  - Brighter -> white
- Then we *dilate* the pixels, sort of expand white pixels
- Now it's already looking like a grid of points!

----

#### 4. Contours

<div style="display: flex; align-items: center; justify-content: center;">
<img src="img/img-dilated.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/img-contours.png" width="300px"/>
</div>
<div style="display: flex; align-items: center; justify-content: center;">
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img class="fragment" src="img/img-rects.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img class="fragment" src="img/img-rect_filtered.png" width="300px"/>
</div>

Notes:

- So how do we turn this grid into actual points?
- We grab so called **contours**, basically we find the outlines of shapes where the color changes
- The contours are a huge list of pixels or lines so...
-...we fit a rectangle around them, and throw away any rectangles that look too weird
- Looking good already!

----

#### 5. Corners of the grid

<div style="display: flex; align-items: center; justify-content: center;">
<img src="img/img-rect_filtered.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/img-midpoints.png" width="300px"/>
</div>
<div style="display: flex; align-items: center; justify-content: center;">
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img class="fragment" src="img/img-cornerlines.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img class="fragment" src="img/img-corners.png" width="300px"/>
</div>

Notes:
- Now we can just grab the midpoints of the rectangles
- and find which are closest to the corners of the image

----

#### 6. We did it!

<div style="display: flex; align-items: center; justify-content: center;">
<img src="img/img-corners.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/img-found_grid.png" width="300px"/>
</div>
<div style="display: flex; align-items: center; justify-content: center;">
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img class="fragment" src="img/img-output.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img class="fragment" src="img/img-tiles.png" width="300px"/>
</div>

Notes:
- And there it is!
- Now we just follow a tutorial to fix the perspective so that instead of looking at it from the side, it looks like we scanned it.
- We can also then guess the grid size from the grid points

----

#### 7. Show the user

<div style="display: flex; align-items: center; justify-content: center;">
<img src="img/img-found_grid.png" width="300px"/>
&nbsp;&nbsp;➝&nbsp;&nbsp;
<img src="img/code_matrix_detect.gif" height="500px"/>
</div>

Notes:
- Now we can show the user that we found the matrix
- Makes it easier to position the camera

---

### Building the thing

1. Find if there's a code matrix in the image <!-- .element: style="opacity: 0.2" -->
2. Extract that image <!-- .element: style="opacity: 0.2" -->
3. Run OCR
4. ???
5. Profit?


---


### OCR

<!-- .slide: data-background-color="#fff" -->

<iframe src="https://en.wikipedia.org/wiki/Optical_character_recognition" width="960px" height="700px"></iframe>

Notes:
OCR: optical character recognition
- in goes image, out comes text

----

<!-- .slide: data-background-color="#fff" -->

<iframe src="http://localhost:3005/" width="960px" height="700px"></iframe>

Notes:
- one of the many libraries is Tesseract
  - open source, **free**
  - someone compiled it to WebAssembly
  - can run directly in the browser

----

<!-- .slide: data-auto-animate -->

### Helping Tesseract

- Train the language model with the game's font <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->
- Restrict possible characters <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->
- Change "page modes" <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->

<pre class="javascript"><code data-line-numbers="8">await tesseract.load();
await tesseract.loadLanguage("eng_custom_font");
await tesseract.initialize("eng_custom_font");
await tesseract.setParameters({
  tessedit_char_whitelist: " 1579ABDCEF",
  tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
});

</code></pre>

Notes:
- out of the box, tesseract is optimized for scanning book pages in a specific language

----

<!-- .slide: data-auto-animate -->

### Helping Tesseract

- Train the language model with the game's font
- Restrict possible characters <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->
- Change "page modes" <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->

<pre class="javascript"><code data-line-numbers="2-3">await tesseract.load();
await tesseract.loadLanguage("eng_custom_font");
await tesseract.initialize("eng_custom_font");
await tesseract.setParameters({
  tessedit_char_whitelist: " 1579ABDCEF",
  tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
});

</code></pre>

Notes:
- fine tune, we know the font you can train tesseract

----

<!-- .slide: data-auto-animate -->

### Helping Tesseract

- Train the language model with the game's font <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->
- Restrict possible characters
- Change "page modes" <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->

<pre class="javascript"><code data-line-numbers="5">await tesseract.load();
await tesseract.loadLanguage("eng_custom_font");
await tesseract.initialize("eng_custom_font");
await tesseract.setParameters({
  tessedit_char_whitelist: " 1579ABDCEF",
  tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
});

</code></pre>

Notes:
- **restrict:** we know euro symbols and Is aren't there
- text modes: blocks, rows, columns, pages of text book

----

<!-- .slide: data-auto-animate -->

### Helping Tesseract

- Train the language model with the game's font <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->
- Restrict possible characters <!-- .element: style="opacity: 0.2; transition-duration: 250ms !important" -->
- Change "page modes"

<pre class="javascript"><code data-line-numbers="6">await tesseract.load();
await tesseract.loadLanguage("eng_custom_font");
await tesseract.initialize("eng_custom_font");
await tesseract.setParameters({
  tessedit_char_whitelist: " 1579ABDCEF",
  tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
});

</code></pre>

Notes:
- text modes: blocks, rows, columns, pages of text book