
## The Saucer
Mount a trackpad to your ZSA Voyager!

With a few parts and a bit of soldering, you can use your ZSA Voyager as a pointing device as well as a keyboard. All your input needs in one small package.

![The Saucer](/readme_assets/enclosure-attached.JPG)

Well, maybe not all your needs. Here are a few caveats to this project to keep in mind before embarking:

- This trackpad is great for writers, office workers, and coders, but it may not ideal for creative work or gaming. Anyone can get use out of this trackpad, but it is not a complete replacement for all things you might need a mouse for.
- QMK's mouse support is good, but not perfect. There is limited gesture support, for example. The hardware is also not as polished as something like an Apple trackpad. Attaching this to a keyboard is a fundamental part of the design in more ways than one. The keyboard and the trackpad work together. You may need to get used to keyboard actions for some things you used a mouse for before.
- This is a DIY guide. I try to document the process here thoroughly, but this is not covered by any sort of warranty or support if things go wrong. This design does not involve modifying the ZSA Voyager, but if something does get damaged somehow as the result of following this guide, [ZSA cannot promise warranty support](https://www.zsa.io/warranty) either. Please take your time and be careful.

Finally, please note that although I work at ZSA, this is personal project of mine, not an official ZSA accessory. Please do not email ZSA with questions about this. If you do have questions, you can open an issue here and I will try to get back to you as soon as I can.

## Tools and parts

Here is what you will need to make a Saucer.

### Parts
- The 3D printed trackpad mount (left or right)
- A 40mm flat Cirque trackpad
- Four lengths of small gage wire about 16cm long (ideally four different colors)
- A TRRS splitter cable (I used [this one](https://www.amazon.com/MillSO-Headset-Splitter-TRRS-Headphones/dp/B07569QKQQ?th=1), but I'm sure there are others that would work. This one is a bit long.)
- Some small heat shrink tubing

### Tools
- A soldering iron and solder
- A wire stripper
- A multimeter
- Helping hands to hold electrical parts are nice, but not required

## Assembly
### Making the enclosure
You can either print the enclosure yourself (if you have access to a 3D printer) or order the part from a 3D printing service. 

If you have a 3D printer, here are the relevant settings I use. Honestly, I'm still pretty new to 3D printing, so if different settings yield a much better print for you, feel free to share. 
- PLA material
- 0.2mm layer height
- 2 walls
- 15% infill
- Tree supports with a 0.22mm top and bottom Z distance gap for slightly easier removal

If you don't have a 3D printer, I've used Shapeways in the past and like them, but there are other places that I'm sure could do a good job as well. 

### Firmware
Before you start assembling the hardware, it's a good idea to prepare your firmware. You'll need to compile locally for trackpad support. Luckily, this pretty straightforward if you use ZSA's QMK fork (you can still use mainline QMK, but it's a little trickier). Go to your layout in Oryx and choose "Download Source" at the bottom for your Oryx layout files. Then, follow the readme to get set up.

First, try compiling your layout without any changes to make sure that works. If not, work on fixing that first.

Once your existing layout compiles, you'll need to make the following changes:

In `rules.mk` add:

```

POINTING_DEVICE_ENABLE = yes

POINTING_DEVICE_DRIVER = cirque_pinnacle_i2c

```

In `config.h` add:

```

#define CIRQUE_PINNACLE_TAP_ENABLE

#define POINTING_DEVICE_ROTATION_90

```

Enabling the tap is technically optional, but I would for now at least for testing purposes. The 90 degree rotation assumes you're installing the trackpad how it's pictured in this guide.

There is a lot more mouse configuration you can dive into [in the QMK docs](https://docs.qmk.fm/#/feature_pointing_device). These are just some basics to make sure the trackpad is working once you have it assembled.

Try compiling your layout. If that works, then you're all set for future testing.

### Preparing the TRRS splitter

We'll connect the trackpad to the Voyager using I2C. Simply splitting the TRRS connection to the right half is all we have to do electrically. However, we have to confirm what connects to what.

![TRRS Splitter](/readme_assets/trrs_splitter.jpeg)

First, snip off one of the split connections. We don't need a ton of cable length left over: just a few centimeters. It's easier to route the internal wires ourselves and connect them later.

![Cutting TRRS splitter](/readme_assets/trrs_cut.jpeg)

At this point, determine whether your splitter has four independent wires (ideal) or three wires and loose wiring like mine. We'll need to figure out which of these wires connects to each part of the TRRS cable.

![Unraveling TRRS splitter](/readme_assets/trrs_unravel.jpeg)

A TRRS cable has four parts: tip, ring, ring, sleeve. Thus, it also has four internal wires. We'll use these for voltage, ground, SDA, and SCL. These are the four connections any device using I2C to communicate needs.

To determine which wire is which, we'll need to open up the back of the Voyager for a minute. Just remove the sticker on the back to access all the screws.

![Voyager sticker removed](/readme_assets/sticker_removed.jpeg)

Once unscrewed, the back and switch support structure will pop off.

![Voyager opened up](/readme_assets/board_open.jpeg)

Now, we want to look at the TRRS port. There are four small metal contacts on this side of it that we can use to test continuity with the internal wires of our TRRS cable. I will label them here:

![Voyager TRRS port](/readme_assets/trrs_port.jpeg)

- Top left is SCL
- Middle left is SDA
- Bottom left is VCC (5V power)
- Top right is GND (ground)

Plug the board in (be careful not to push on it and send a bunch of keystrokes) with the TRRS splitter attached. Then, using a multimeter, test for continuity between each of these small contacts and the wires of your splitter. Mine worked out like this:

- Bare metal wire is GND
- Red wire is VCC
- White wire is SDA
- Yellow wire is SCL

![Testing Voyager TRRS port](/readme_assets/multimeter_testing.jpeg)

Keep in mind your TRRS cable may be different, which is why we do this step. There isn't one exact standard for wire colors. Note your setup and leave the board open for now.

### Preparing the trackpad

![Cirque trackpad](/readme_assets/trackpad.jpeg)

To use the Cirque trackpad over I2C, we'll need to modify it slightly. Three resistors need to be removed: R1, R7, and R8. Removing R1 will let the trackpad use I2C, and removing R7 and R8 let the trackpad run at 5V, which we need here as well.

![Preparing to remove resistors](/readme_assets/trackpad_remove_r.jpeg)

The best method I have found for this is to use a standard flathead soldering iron and heat along the long edge of the resistor, then kind of sweep it off the pads. These resistors are extremely tiny, so be careful while you do this. You shouldn't need much heating time or force to remove them if you do this right. You don't have to worry about saving these or anything.

![Trackpad with resistors removed](/readme_assets/trackpad_rs_removed.jpeg)

Once this is done, tin the VCC, GND, SDA, and SCL test pads with little mounds of solder.

![Tinning test pads](/readme_assets/trackpad_tinned.jpeg)

Then take your lengths of wire and solder them to the test pads. I'm using wire colors the match up with the TRRS cable, but this isn't required. Just be sure you have some way to tell the wires apart.

![Wiring trackpad](/readme_assets/wires_trackpad.jpeg)

### Preparing the splitter

With the wires attached to the trackpad, the the last big thing to do is attach these to the TRRS splitter wires. 

Before you solder the wires together, it's a good idea to cut some short lengths of heat shrink tubing and slide them on to each wire before connecting them. That way, once you're sure things are working, you can just slide the tubing up to wrap the connections. 

Once you have the tubing on, tin each side of the wires, then heat them to connect. It should look like this. 

![TRRS splitter connected to trackpad](/readme_assets/trrs-wires-connected.jpeg)
*This image shows the a prior version of the design — the concept is still the same.*

You can do this before installing the trackpad in the enclosure. The design is meant to keep the whole trackpad-TRRS-wire-harness assembly removable, so you don't have to worry about not being able to access it if something goes wrong. 

The above image isn't the greatest angle because it looks like two wires are soldered together, but they aren't. You're looking for four independent wires, all connected to the corresponding wire on either side. This is where using different wire colors comes in handy since you can just match the wires by color. 

### Putting everything together

First, push fit the trackpad into the holder. It should be snug, but you can remove it if you mess up the orientation. Try to get it as straight up and down as possible.

![Clipping trackpad into enclosure](/readme_assets/inserting-trackpad.JPG)

Next, take your lengths of wires and push them into the channel that goes to the top of the board. If you didn't clean off the supports from the 3D print, these will be in the way, so break them off first.

![Routing wires](/readme_assets/trackpad-inserted.JPG)

Then just pop the legs on to secure the trackpad.

![Securing enclosure with feet](/readme_assets/legs-installed.JPG)

### Testing and finishing

If you've gotten to this point, you're probably excited — I get it! Before you try it out though, it's really important to do one more test. Without this test, if something goes wrong, you're not going to have any idea where to start.

Grab your open board again and plug in the splitter with the trackpad attached. We're going to repeat the same continuity test as before, but this time, we're going to measure continuity from the TRRS port to the trackpad test pads.

![Final TRRS test](/readme_assets/final_test.jpeg)
*This image shows the a prior version of the design — the concept is still the same.*

If these all show continuity like you would expect, then great; you probably soldered correctly, and if something isn't working, it's probably a firmware issue. If these don't work how you expect, fix it now before buttoning everything up.

As long as the electrical seems good, unplug the board and use some heat shrink to protect the soldered wire connections. You can also add another layer of larger heat shrink around all the wires for a cleaner look. Then, close up your board, make sure your modified layout is flashed to it (a quick way to test this is to try to connect through ZSA's Keymapp — it shouldn't connect), and plug everything in.

![Board lit up and working](/readme_assets/final-product.JPG)

Liftoff!

## Further firmware configuration
Your firmware configuration is far more personal, but here are a few things I've found that may help you tweak your own settings to your liking. 

In theory, tapping the trackpad to click and dragging around the outer edges to scroll are nice ideas, but in practice, I didn't really like using them. The surface of the trackpad was just too small. I started to unintentionally trigger these gestures while mousing normally. Someone else might be able to get used to them, but I ended up using a different QMK feature: [an automatic mouse layer](https://docs.qmk.fm/features/pointing_device#pointing-device-auto-mouse). 

An automatic mouse layer turns on when you move the mouse, and it turns off a short while after you stop moving it (the exact time is configurable). Since I have to shift my right hand over to use the trackpad, I keep my mouse functionality on my left hand. In effect, I have kind of a split mouse to go with my split keyboard. The movement is done with the right hand and all the buttons are on the left. Here's what my mouse layer looks like:

![Mouse layer layout](/readme_assets/layout-screenshot.png)

And here's how it looks in code. In config.h:
```
#define POINTING_DEVICE_AUTO_MOUSE_ENABLE

#define AUTO_MOUSE_TIME 280
```

And in keymap.c:
```
[7] = LAYOUT_voyager(
KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT,

KC_TRANSPARENT, KC_TRANSPARENT, KC_MS_WH_UP, LGUI(KC_LEFT),LGUI(KC_RIGHT), LGUI(KC_UP), KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT,

KC_TRANSPARENT, KC_TRANSPARENT, KC_MS_WH_DOWN, KC_MS_BTN2, KC_MS_BTN1, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT,

KC_TRANSPARENT, KC_MS_WH_LEFT, KC_MS_WH_RIGHT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT,

KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT, KC_TRANSPARENT
),
```

```
void pointing_device_init_user(void) {

set_auto_mouse_layer(7); // only required if AUTO_MOUSE_DEFAULT_LAYER is not set to index of <mouse_layer>

set_auto_mouse_enable(true); // always required before the auto mouse feature will work
}
```

Using a key to scroll rather than a wheel is a little different, though. I had to make some changes to get key scrolling to feel somewhat natural (this is an ongoing process). I'll share my settings here, but keep in mind that this may be different for you. It depends on how your OS scroll behavior is set up and how you like to scrolling to feel. Here's mine:
```
#undef MOUSEKEY_WHEEL_DELAY
#define MOUSEKEY_WHEEL_DELAY 100

#undef MOUSEKEY_WHEEL_INTERVAL
#define MOUSEKEY_WHEEL_INTERVAL 110

#undef MOUSEKEY_WHEEL_TIME_TO_MAX
#define MOUSEKEY_WHEEL_TIME_TO_MAX 50
```

Those are the main things I have set up that let me use the trackpad comfortably, but again, I feel like there is more to discover. If you have ideas or find a configuration you like, feel free to share it. 

---
## Acknowledgements
Thanks to all of ZSA for encouragement when I shared updates of my progress. Special thanks to Drashna, who talked over some technical aspects with me and who created a lot of the groundwork for this in QMK in the first place, and to Erez, who provided helpful feedback about the design and continues to be extremely supportive of this project.

And thank *you* for reading. If you give this project a try, please share your results. :)
