TetriPiSense - A Tetris-y game for the Sense Hat / Astro Pi / Unicorn Hat
=========================================================================

Andrew Richards, September 2015.

# Introduction
8 rows of 8 LEDs, teensy joystick... I found Snake as mentioned in the blurb
for the AstroPi, but no Tetris. So here's TetriPiSense, my motivation for
getting to grips with my Sense Hat... and indeed GitHub too.

# Description
The program uses the [sense_hat](https://pythonhosted.org/sense-hat/api/)
and [pygame](http://www.pygame.org/docs/) libraries to provide much of the
underlying functionality. The Sense Hat joystick is mapped to keyboard keys,
so for the Unicorn Hat (with no joystick), this program should still work -
just use your keyboard's arrow keys (also Esc key to quit).

# Installation
You'll need the `pygame` and `sense_hat` libraries installed on your Pi.
`pygame` is probably already installed - if not you'd look for the package
name `pygame` or `python-pygame` - for example on Debian-flavoured Linux
one of (for Python 2, Python 3 resp.),

    sudo apt-get install python-pygame
    sudo apt-get install python3-pygame

and if you've not already installed the `sense_hat` library, one of,

    sudo apt-get install python-sense-hat
    sudo apt-get install python3-sense-hat
    
Lots of other ways of installing Python packages instead, such as `pip`,

    pip install pygame
    pip install sense-hat

Now `tetripisense.py` just needs to be placed somewhere you / your system
can find it, see below for usage.
    
# Usage
If you're directly connected (keyboard, mouse, monitor) to the Raspberry Pi
with Sense Hat, you should be able to launch the game like this (although
writing with Python 3 in mind, Python 2.7 seems to work fine too, I don't
know about earlier versions of Python),

    python tetripisense.py

The joystick controls the block: Move left/right, rotate (clockwise), drop
the current block; all of these are also available via the keyboard
(USB-connected to Pi) using the equivalent arrow keys, as well as the Esc
key to exit the game.

If you're accessing the Raspberry Pi remotely (e.g. via ssh),

    export DISPLAY=0:0; sudo python tetripisense.py

Note that this runs the program with full administrator privileges, and makes
your Raspberry Pi system much more exposed and vulnerable to anything dodgy
that the game might try to do to your Pi. Note that the [ssh] keyboard doesn't
control the game when you're connected like this (although Ctrl-C will quit).

See below if you'd like the program to startup when your Pi starts.

# Authors
Original PiLite implementation: Stephen Blythe  
This Sense Hat implementation: Andrew Richards

# Website

    http://free.acrconsulting.co.uk/other/tetripisense.html

# License
Gnu General Public License, version 3 and above: Please see the included
`LICENSE` file.

# Known issues
Rotation: Please see `TODO.md`: Rotation uses pygame and rotates around an
axis of the top-left of the shape. This is mostly fine, but for a long
narrow block, rotating around the centre of the block might look a bit more
polished.

# Origins of the program
Possible starting points were Tetromino written by Al Sweigart included
within Raspbian, and Stephen Blythe's implementation for Ciseco's PiLite.
I chose to adapt the latter since it was aimed at a similar grid-of-LEDs
display so should map well to the Hat - indeed, my initial hacks to get
it working on the Sense Hat didn't require much work, although I did then
spend a fair amount of time polishing the code and ironing out a few bugs.

# Some notes on the code
Having only 8 vertical LEDs makes it harder to achieve a reasonably playable
game, so I added extra 'virtual' rows of pixels for the behind-the-scenes
pygame.Surface, which gives the sense of the new block emerging at the top
of the display, rather than the whole block suddenly appearing at the top of
the display - this gives the player a bit more time to manoevre the new block
and results in the game feeling a bit more polished.

I've tried to move some chunks of code into functions where possible to make
the logic clearer; unfortunately the main game loop still contains quite a lot
of code but is hopefully reasonably straightforward. Most of the manipulation
of the falling block and the background is done with pygame functions; actually
displaying the game uses the `sensehat_display()` function which converts from
the pygame representation of the display to the representation used by
sense_hat's `set_pixels()` method.

Unit or other tests are conspicuously absent here but would be very
worthwhile, even if only part of the code lends itself to this.

# Getting your Pi to run this game at startup
It's fun to have the Pi run this game on its own without a mouse, keyboard,
screen - a sort of minimal handheld games console, powered by an external
battery, such as the now fairly widely available external battery packs for
phones that provide power via USB cables.

To do this, the Pi will need to respond to the Hat's joystick when it starts
up. Given this is mapped to the keyboard, just add `run_game.py` to the Pi's
customisable startup script to wait for the joystick button (mapped to the
Return key) to be pressed (in fact any keypress, so moving the joystick
works too): Add the following line before the final `exit 0` line in
`/etc/rc.local`,

    python /home/pi/tetripisense/run_game.py

assuming `run_game.py` and `tetripisense.py` are in the directory above. When
you finish playing you can shutdown the Pi cleanly by turning it upside down
briefly which will initiate a `shutdown`: The SenseHat's accelerometer is
checked to see if the Hat is upside-down and calls `shutdown` if so - but
only during the `Press joystick` (`run_game.py`) loop, not during games.

Note that this approach will run `run_game.py` and `tetripisense.py` with
administrator privileges which is poor practice.
