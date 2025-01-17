# QuickFont User Guide

QuickFont is a user font editor in BASIC for the Timex Sinclair 2068 and 
Sinclair ZX Spectrum.

Originally written on a TS 2068 back in 1985. More recently typed it into an emulator
from a listing. A few bugs fixed and
improvements made years later.

Forked for conversion into Boriel ZX Basic

## User Fonts

The ZX Spectrum and compatibles had the ability for a user-defined font by
changing the system variable `CHARS` to point to the bytes containing the 
character bit patterns. Normally, this variable points to the character bytes
in the ROM. The characters from space (32) to the copyright symbol (127) are
definable in this group. The block graphics characters are not included (their
codes are above 127). The user-defined graphics (UDG) characters are separately
definable using the methods described in the user manual. 

There are 96 definable characters, each defined by 64 bits contained in 8 
sequential bytes for a total of 768 bytes. The patterns are defined the same as
with UDG characters. There is no reserved memory for a user font as there is 
for the UDGs, so you have to reserve the memory yourself. This program does this
for you by moving the top of the BASIC area down by 768 bytes to place the font 
just below the UDGs in memory. When you are done, you could clear the program 
with NEW and the font will still be there at address 64600.

Once a user font is loaded it is "activated" by pointing `CHARS` to its 
location. You actually point `CHARS` to the location minus 256 bytes, which is
because the ROM saves instructions and time by not subtracting 32 (the code of 
the first definable character) from a character code when computing the byte 
offset in the font for the character. It just multiplies the code by 8 (bytes), 
and since 8*32 = 256, this means the first byte of the first character is at 
offset 256, so they made the font address pointer store the actual address 
less 256 bytes so that adding the offset gets you to the right place. This 
seems odd, but since this opertion is done so frequently in printing, not 
having to subtract 32 each time helped speed things up on old, slow hardware.

Once loaded and activated, your program listing, editing, and printing will
use the font. This includes printing to the thermal printer. Printing to 
regular printers through an interface will not use the font as those work by
sending character codes rather than bit patterns. 

## QuickFont Font Storage

QuickFont reserves some memory immediately below the UDGs by using the `CLEAR` 
statement which moves the top of the BASIC area down as defined in the system 
variable `RAMTOP`. It then lets you edit the characters, storing them in this 
area. This address is normally 64600. It lets you save and load the font to and
from tape, so they will have this as a default address stored in the saved file.
However, you can always load them into whatever location you need to by giving
the address when loading the bytes, e.g.: `LOAD ""CODE address`

To load a font for use, you need to reserve memory for it as well as point the
system variable `CHARS` to it. There is a sample font loader program in 
`FontLoader.tap` to show you how to do it. You can load the font into a 
different location if you wish by giving the `LOAD ""CODE` command the desired
address. Since the font is above `RAMTOP`, it will persist even after a `NEW` 
command until you reset the machine or move `RAMTOP` above it. An exception is
when loading the font into an reserved area such as with OS-64 storing the font
in RAM in the unused screen attribute area.

There are some sample fonts included stored in `.tap` files. You can use the
`FontLoader` program to load them, or load them yourself.

## Files

The files in the distribution (`QuickFont.zip`) are:

* `QuickFont.tap`

    This is the main program. It only contains the main program. It will autorun
    when you load the program.

* `QuickFont_OS64.tap`

    This is the version made for running with Zebra System's OS-64 cartridge. It
    is useful if you are going to edit a font or UDGs to be used under OS-64.
    This is still an older version (2.2f), and needs updating. OS-64 copies the
    ROM font to RAM (in the display file 1 attributes area which is unused),
    which makes loading user fonts easier since you don't have to `POKE` the
    font address, but you do have to load it into that address.

* Font files (`*_font.tap`)

    I've included several font files to get you started. Many of them were  ones
    I made back in the 80s. They are a `CODE` bytes segment, each in separate
    .tap files. You can mount the .tap file and then load the font into the
    editor.

* Font Loader Example (`FontLoader.tap`)

    This is a small program that demonstrates how you might load a font in your
    own programs. The .tap file contains the sample program and a font.

* Tilde and Vertical Bar Characters (`tildebar.tap`)

    The Timex Sinclair 2068 issn't able to display these two characters from the
    Sinclair font, although the bit patters are in the ROM. Timex replaced these
    with the new tokens for the `FREE` and `STICK` functions. You can still edit
    these in the font but on the 2068, you can't use them directly. This utility
    copies the bytes for them into the first two UDG characters. 

* `QuickFont_zmb.bas`

    This is the main source file in a [Zmakebas][] format from which
    `QuickFont.tap` is made. 
  
* `QuickFont.bas`

    This is made from `QuickFont.tap` to not have the comments from
    `QuickFont_zmb.bas`. 

[Zmakebas]: https://github.com/chris-y/zmakebas

## Machine Code

To do some of its operations quickly, QuickFont uses a little bit of machine
code to help out. This should be already stored in the line 1 REM statement and
not have to be loaded on startup. If is does need to be loaded, then the program
will do that from some DATA statements when it is first run. 

The cartridge (.dck file) version runs with the BASIC code in ROM, so it has to
load the machine code into RAM. It puts it just below the user font in memory
and adjusts RAMTOP down to just below that. A future version may just always
store the machine code this way, loading it from a separate CODE segment in the
.tap file.

## View Mode

![View mode display](ViewMode.png)

The main screen dispays the current user defined font in 4 rows with the ROM 
font character underneath each for reference. There is the help for the view
mode to the left, a preview of the selected character to the right, and a
magnified view of the selected character in the middle. You can press a key to 
view that character of the font. You then press `Enter` to go into *Edit Mode*
to edit the selected character. 

You can change the background color for the character preview on the right side
by pressing shift 5 and 8 to cycle the background color. This also works when in 
*Edit Mode*, except you don't use shift.

From *View Mode*, you can press `Symbol Shift` and `A` (STOP) to quit the 
program. This will display the `POKE` commands for activating the font. 

### Extended Mode Characters

There are a few characters that are normally entered in *Symbol Shift* mode 
(`E` cursor). You can select them similarly by pressing `Caps shift` and
`Symbol Shift`, after which you hear a beep and can then press the key of the
extended mode letter, except you don't have to hold down `Symbol Shift`. This 
applies to the symbols `[`, `]', `\`, `{`, `}`, and the copyright symbol, each 
under the keys `y`, `u`, `d`, `f`, `g`, and `p`, respectively. This entry system 
also applies to typing in *Test Mode*. Pressing any other key cancels the 
temporary mode. In view mode only, you can actually just press `symbol shift` 
and the letter, except for `p`, to get the character since these keys have 
word tokens for their regular *symbol shift*.

Note that the tilde `~` and vertical bar `|` don't appear on the TS 2068 
keyboard because these were replaced with the keywords `FREE` and `STICK`, and
unfortunately, these characters can't be normally printed even though they
are there in the font. You can select them and edit them on a 2068, but you 
will see `FREE` and `STICK` in displayed except for in the magnified bits
view. See the bonus program in `tildebar.tap` which gives you access to these 
two characters by copying their bytes from ROM to the UDGs A and B.

Also, on the TS 2068, the `{` and `}` characters don't appear on the keyboard, 
but as the manual describes, they do exist in extended mode on the `f` and `g` 
keys. This seems to be because `ON ERR` and `SOUND` are keyword mode commands 
and `FREE` and `STICK` are leter mode functions, so the extended shift of `f` 
and `g` in letter mode doesn't make sense to insert a command keyword, so it can
insert the brace characters instead. The letter mode functions are entered in
extended letter mode, so it's not possible to also get the tilde and vertical 
bar characters in the same mode.

The keys in view mode are:

    Key            Function

    Enter          Go into edit mode for the current character
    Shift+1        Show the character byte codes
    Shift+2        Cycle the application background color
    Shift+3        Save the font or UDGs to tape
    Shift+4        Load a font or UDGs from tape
    Shift+5 or 8   Change the preview box background color
    Shift+6 or 7   Change the preview box foreground color
    Shift+9        Switch between editing the font and editing the UDG
                   characters in the A-U characters
    Shift+0        Enter the test mode to type with the font
    SymbolShift+q  Quit the program
    Shift+Symbol   Enter a one-keypress extended mode to get the characters
          Shift      [ ] ~| \ { } and copyright by then pressing y u a s d f g or p
    SymbolShift+   Shortcuts to enter the characters: [ ] ~ | \ { }
     y u a s d f g
    SymbolShift+e  Copy the ROM font over the user font (with confirmation)
    Other keys     Views that character

## UDG Editing

In the view menu, you can press shift+9 to toggle editing the UDG characters in
character positions A though U. The character labels below them with change to 
inverse video to indicate this mode. During this mode, the save and load 
functions will apply to the UDG chaaracters only. It will keep separate file
names for the font and UDGs which is shown on the top line.

Also, during this mode, the user font characters A-U are temporarily swapped with 
the UDGs A-U, so stopping the program here abnormally, will leave them swapped. 
You can either just use `CONT` to continue the program, or try `GO SUB 7300` to
swap them back if variables haven't been cleared. Otherwise, you could `RUN` the
program again --- it will not think they are swapped --- and press shift+0 to 
swap them, press `BREAK` (shift+space), and then re-`RUN` the program.

## Edit Mode

![Edit mode display](EditMode.png)

This lets you edit the bits of the current character by pressing the space bar 
and using the keys `a`, `z`, `k`, and `l` to move the cursor left, right, up, 
and down, respectively. When done editing, you press `enter` again to save the
changes. While you are editing, an actual size preview is shown in the box on 
the right of the screen.

The keys in edit mode are:

    Key         Function

    a and z     Move edit cursor up and down
    k and l     Move edit cursor left and right
    Arrow keys  Also for edit cursor movement
    space or b  Toggle bit on/off
    5 and 8     Change preview box background color
    6 and 7     Change preview box foreground color
    0           Clear all bits (with confirmation)
    e or >=     Copy bits from the ROM character with confirmation
    E	          Copy bits from the ROM character without confirmation
    Shift+1     Cancel edits and go to view mode
    Enter       Save edits and go to view mode
    ?           Show other key help:
    A,Z,K,L     Shifted a/z/k/l shifts bits up/down/left/right
    c and p     Copy and paste bits between chars
    h and v     Flip bits horizontally or vertically
    r           Rotate bits left (less than key)
    t           Rotate bits right (greater than key)
    
When you save changes to a character, the updated character will then appear in  
the user font shown in the top half of the screen.

## Test Mode

![Test mode display](TestMode.png)

Pressing `shift`+`9` in view mode takes you to test mode which lets you type 
using the user font. It uses the paper and ink colors from the character preview
box. You can type for multiple lines as well as use the delete key to backspace. 
Then press `STOP` (`symbol shift` + `a`) to quit and return to the view mode.
You can access the extended mode characters here with the one-key extended mode
as well. 

The keys in test mode are:

    Key            Function

    Shift+2        Toggle caps lock
    Shift+0        Backspace delete
    Shift+9        Enter a one-keypress block graphics character mode
    Shift+Symbol   Enter a one-keypress extended mode to get the characters:
          Shift      [ ] ~| \ { } and copyright by then pressing y u a s d f g or p
    SymbolShift+a  Exit test mode and return to view mode


## Loading

Pressing `shift`+`4` in view mode lets you load a font from tape previously 
saved as a code block by this program or manually. It should be a code block of 
768 bytes, but it doesn't have to be from the same address as it will be loaded 
into the address the program is currently using for the user font.

It prompts you for the name of the file on tape, but you can just leave it blank 
and just press enter to load the next code block on the tape. For running in an
emulator, you may need to mount a different tape file. To not load a file, enter
the keyword `STOP` as the name by pressing `Symbol Shift` + `a` then `enter`, 
and you will return to the view mode.

If you are currently editing the UDG characters, the load applies to those 
instead, and it keeps a separate file name for them. In this case, 168 bytes 
will be loaded to address 65368 (USR "a"). 

## Saving

Pressing `shift`+`3` in view mode lets you save the current user font to tape. 
It will prompt you for a file name to give it. You can enter nothing to cancel 
saving and return to the view mode. The font will be saved to tape as a code 
block of 768 bytes from the address 64600. This can be loaded later, and the
address it loads to can be changed as with loading any code block by giving the
address with the load command as `LOAD "name"CODE address`. See the sample font
loader program.

If you are currently editing the UDG characters, the load applies to those 
instead, and it keeps a separate file name for them. In this case, 168 bytes 
will saved from address 65368 (USR "a"). 

## Using the Font

See the included sample font loader program for an example of how to reserve 
memory for the font, load the font, and activate it. You might change the 
location it is stored, in which case, you need to load the code to that location 
as well as set the `CHARS` system variable to point to that location (less 256 
bytes).

The example uses the typical location of 64600:

1. Reserve memory by moving `RAMTOP` to at least one less than the font location:

        CLEAR 64600-1 
    
2. Load the font from tape. Its best to specify the location in case it is not
   what you expect. Specify the length for the same reason.
   
        LOAD ""CODE 64600,768
    
3. Activate the font by setting the `CHARS` system variable. This is a two-byte
   variable since it specifies a memory address, and thus needs two `POKE` 
   commands to set. If done manually, you should do this with a multiple 
   statement line of two `POKE` commands, otherwise, you set one of the bytes, 
   and the font pointer immediately changes, making the next command very hard 
   to enter since you won't be able to read what you are typing.
   
        POKE 23606,88: POKE 23607,251
    
   This causes the `CHARS` system variable to point to store the address 
   `88+256*251`, which is 64344. This is the actual address of the font, 64600, 
   minus 256 bytes as previously explained.
   
After creating a font (and saving it), when you quit the program, you can do 
these POKEs to activate the font still in memory. You can then give the NEW 
command and start programming using your new font. 

You could also do these operations in your own program to have it use your font.
Just be aware that the `CLEAR` command with a memory address also does clear the 
variables like the regular `CLEAR` command, so you will need to do that at the 
beginning of your program before setting variables.

## Revision History

### Version 2.0 - 4 September 1985
This was pretty much as it was in the printed listing with a few flaws and 
probably not the last version I made, but it will be a while before I get to
resurrecting files from my old tapes.

### Version 2.0a - 6 June 2022
I started to make improvements and fix a few issues.
It adds foreground color changing for the preview box and shortcuts for 
selecting most of the extended mode characters. It also adds using the arrow 
keys to move the edit cursor around, mostly since emulators map the arrow keys 
to the Sinclair cursor keys (shifted 5 to 8).

### Version 2.1 - 11 June 2022
I started to change the look a bit more. I don't remember why I used uppercase 
so much; it gave it a ZX81 look. I think I didn't like the lowercase in the 
Sinclair font very much, so was avoiding it whereas everyone else was mad about 
it since it was new and not too many micros at the time had lowercase.

Several small changes, including improvements to the exit screen saying how to 
use the font. I better optimized the edit key loop to make it more responsive.
An improvement in test mode when typing near the end of a line or bottom of the 
screen.

### Version 2.1a - 26 June 2022
Further optimization of the edit key loop to make it much better than the last 
improvement.

### Version 2.2 - 13 August 2022
First introduction of machine code to help speed some things up. This included
copying the ROM font and decoding the character bytes into bits to make the
magnified array of pixels. The latter took some playing around given how the 
system variable `DEST` behaved to be able to capture the address of the a$ 
character array. The machine code is loaded from a hex string into a padded line 
1 REM statement rather than into high memory which allows a NEW to get rid of 
the BASIC program and leave just the font. The machine code was generated with
the *z80asm* assembler from `mcode.asm`, then I entered the hex codes into my 
loader string.

Switched to using key codes for checking in key loops since quoted tokens get
converted to text by *listbasic* and don't convert back with *ZMakeBAS*. Added 
initial specialization for the ZX Spectrum to show the tilde and vertical bar
characters in the font display and keyboard selection of them. Refined the view 
and edit menus.

### Version 2.2b - 14 August 2022
This changed the MC to not have to poke the number of bytes and addresses for 
the mcopy routine in favor of just copying the 8 bytes for the selected 
character using PEEKs and using a short MC routine that copies the whole ROM 
font. Now the character offset and a$ location are just POKEd into the MC where
the addresses are used since they are now only referenced once. Changed the MC
loader to print all the hex digits then invert each pair as it loads the bytes
to give a progress bar effect. 

### Version 2.2c - 16 August 2022
Added changing the app background color with `shift+1` and `shift+2`.
Added caps lock toggling in test mode.
Fixed not zeroing the d() array when using the clear all bits function, which
was scrambling the subsequent editing.

### Version 2.2d - 17 August 2022
Changed the quit app to `symbol shift+Q` (`~<=`) so I could make 
`symbol shift+A` (`STOP`) be the shortcut for the tilde character. Also made 
`symbol shift+S` (`NOT`) the vertical bar shortcut. The copy ROM font function 
was moved to `symbol shift+E` (`>=`).

### Version 2.2e - 17 September 2022
Notionally improved support for the ZX Spectrum by making the memory locations 
less hard-coded and use system variables to get addresses more. Rather than 
using 64600 for the font location, use the UDG address minus 768 bytes. This 
lets it work on a 16K Spectrum, however, that is too little RAM to load this
program. Also check if RAMTOP needs moving or is it already below the font
address. Changed the test mode to not ask for paper and ink color and it just
uses the current character preview colors. Fixed a few cosmetic bugs with the 
menus. Also improved the sample font loader program.

### Version 2.2f - 30 September 2022
Fixed wrongly labeled forground/background preview color change keys. I'd had 
the FG and BG labels reversed this whole time. Switched the MC loader to use 
`DATA` statements. This is simpler, less code, and about twice as fast. Had to
code a variable that says how many bytes there are since we can't do it by 
checking the length of the hex string anymore. The loading progress is indicated
by an "x/y" bytes counter.

This version got a variant, 2.2f_64, that will run under the Zebra Systems' 
OS-64 cartridge ROM that provides the high-res 64 column mode. The screen layout
was tweaked to better use the larger screen area. This will probably be a 
challenge to keep up with the changes to the main program, but being able to 
output the .tap files as text and diff them should make this easier. I'll 
probably also modify the code to perhaps minimize the differences with 
parametric layout values. The memory layout is slightly different when running
under OS-64, one of which was the ROM font is at a different address.

### Version 2.3 - 31 October 2022

- Allow it to run as an AROS program from a .dck cartridge image since we have
  utilities that can convert a basic program to do so. It just loads the MC 
  below the font. I may just always do that and avoid the REM line storage.
- Detect loaded bytes name and show in title bar
- Add ability to edit UDGs, toggle with shift+0. Separate save/load names.
- Add MC routine to copy UDGs to/from user font area, swapping bytes.
- Add ability to type UDGs in test mode (shift+9)
- Add shift bits up/down/left/right (shift+A/Z/K/L) (BASIC code)
- Add copy/paste bits between characters (symbShift+C/V)
- New MC loader progress bar that doesn't slow the load down much
- Had intended to add MC code for flips and rotates but it is still buggy.
- Added a few REM labels for menu key codes and gosub calls

### Version 2.4 - 10 February 2024

- Make the UI BG color only cycle around after white back to black rather than
  shift+1 and shift+2 for back and forth
- Make shift+1 reveal the character bytes (not documented on screen)
- Captures the loaded file name to show at the top and set it when you save
- Allow editing of the UDG characters with shift+9 to switch back and forth.
  They replace letters A through U, which are indicated with inverse letters,
  and you just select and edit those. While in UDG mode, the save and load
  apply to the UDGs only. The UDG filename is kept separately.
- Changed copy ROM chars to e/E/>= in edit mode
- Add extra keys help on ? key in edit mode
- Add h/v flip (not MC) in edit mode: keys h and v
- Add l/r rotate (not MC) in edit mode: keys r and t
