# TEC Redshift Disk Specification (Revision 002)
A specification of the TEC Redshift exported images format in the [EXAPUNKS](https://www.zachtronics.com/exapunks) game by [Zachtronics](https://www.zachtronics.com/).

[The original specification](https://github.com/Rami-Sabbagh/TEC-Redshift-Disk-Specification "The original specification") was written by Rami Sabbagh. I decided to fork this project and clean up the layout a little bit, so someone looking to implement this in software can go about it in a more linear fashion.

One discovery I made *very* recently is that the uncompressed solution data is literally **exactly the same** as the .SOLUTION format used by the game, which explains the redundant "PB039" in the header, and might help me figure out the purpose of some of those unknown bytes and ints.

# Reading the image file
The solution data is stored directly on the image using a method called *steganography*. To read out the image, implement this code in your preferred language:

```
For each pixel in the image (Left to right, then top to bottom):
	Add lowest bit of red channel to bit stream
	Add lowest bit of green channel to bit stream
	Add lowest bit of blue channel to bit stream
    
```

Please note that with very large solutions, the game may [store data in higher bit planes.](https://www.reddit.com/r/exapunks/comments/dumqv1/if_youre_curious_a_game_cartridge_filled_to_the/ "store data in higher bit planes.") It's recommended that your reader/writer program supports this mode.

You should now have a bit stream, which with normal disk images should contain 279,000 bits. Next, you need to convert this bit stream into a byte stream (*or add those bits to the byte stream immediately*), LSB first, like this:

```
| 1 2 4 8 16 32 64 128 | 1 2 4 8 16 32 64 128 | ...
  ^                    --->
First bit         Stream flow
```

# Data types
The .SOLUTION file format uses four different data types:
- (Byte) Pretty self-explanatory, just a single byte
- (Boolean) 1 byte, True = 0x01, False = 0x00
- (Int) 32-bit little-endian integer
- (String) An Int containing the length, followed by raw ASCII text

# Raw disk image data
The data on the disk image itself consists of the following:
- (Int) Length of compressed solution data
- (Int) 16-bit Fletcher's checksum of compressed solution data, stored in a 32-bit Int, for some reason
- (Byte[]) Compressed solution data
- Any data you might find after this point is irrelevant garbage data. Just don't use it - If you're reading the disk in real time, you should just stop at this point.

The solution data is compressed using Zlib / DEFLATE, to save disk space.

# Decompressed image data
As mentioned at the start, the decompressed image data is literally a .SOLUTION file. I'd imagine that, since Zachtronics already made a file format just to hold user code, they decided to not make **another** format just to load onto the disks, and instead just loaded a compressed copy of a .SOLUTION file onto the disk.

The .SOLUTION file format consists of a **header** and one or more **agents**, stuck together with no delimiters or padding.

## Header data
- (Int) Unknown purpose - I'll try to find out what this is for
- (String) Level ID - For Redshift disks, this is always "PB039"
- (String) Solution name - Usually written on the disk image
- (Int) Unknown purpose - I'll try to find out what this is for [1]
- (Int) Total line count for solution - Can be omitted for quick and dirty scripts [1]
- (Int) Unknown purpose - I'll try to find out what this is for [1]
- (Int) Agent count - How many EXAs you start the solution with [1]

[1] It's possible that this group of four Ints is used to store the statistics for a given solution. If so, they would be sequenced like: Cycles, Size, Activity, EXAs. Since the Redshift doesn't bother counting cycles or activity, since its code is made to run indefinitely, these values would be set to 0.

## Agent data
- (Byte) Unknown purpose, always 0x0A for some reason [1]
- (String) Agent / EXA name
- (String) Agent / EXA code (Same as editor, using Unix line endings; 0x0A)
- (Byte) Editor view mode
  - 0 = Default view, maximized
  - 1 = Minimized, don't show code or registers
  - 2 = Follow current instruction
- (Boolean) Is M-bus local by default?
- (Boolean[]) Default sprite [2]

[1] It's possible that this value is intended as a line ending, to make the file more human readable in medium-tech editors like Notepad++
[2] Array of 100 Booleans, read left to right, then top to bottom - True = White pixel, False = Black pixel

# Outro
That's what we've discoverd about the TEC disks format so far, use it for good please ;)

**Happy hacking !**
