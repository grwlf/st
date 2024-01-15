# st-graphics

This is a fork of [st](https://st.suckless.org/) that implements a subset of
[kitty graphics protocol](https://sw.kovidgoyal.net/kitty/graphics-protocol/).

![Viewing images with icat-mini.sh in tmux in st](https://github.com/sergei-grechanik/st-graphics/assets/1084979/54a639ec-afea-45d8-ac18-4f26844e6678)

This repository also includes a simple script to display images `icat-mini.sh`.
Note: to make it work in tmux you need to enable pass-through sequences, i.e.
add something like this to your `.tmux.conf`:

    set -gq allow-passthrough all

## Installation

As usual, copy `config.def.h` to `config.h`, modify it according to your needs,
run `make install`.

In addition to the standard st dependencies (X11, fontconfig, freetype2),
you will need imlib2 and zlib for the graphics module.

## Configuration

You may want to change the graphics-related shortcuts and image size limits (see
`config.def.h`).

Default shortcuts:
- `Ctrl+Shift+RightClick` to preview the clicked image in feh.
- `Ctrl+Shift+MiddleClick` to see debug info (image id, placement id, etc).
- `Ctrl+Shift+F1` to toggle graphics debug mode. It has three states: 1) no
  debugging; 2) show general info and print logs to stderr; 3) print logs and
  show bounding boxes.
- `Ctrl+Shift+F6` to dump the state of all images to stderr.
- `Ctrl+Shift+F7` to unload all images from ram (but the cache in `/tmp` will be
  preserved).
- `Ctrl+Shift+F8` to toggle image display.

## Features

Originally I implemented it to prototype the Unicode placeholder feature (which
is now included in the kitty graphics protocol). Classic placements were
retrofitted later, and under the hood they are implemented via the same
placeholder mechanism. This means that a cell may be occupied only by one
placement.  It is the main reason why some things don't work or work a bit
differently.

Here is the list of supported (✅ ), unsupported (❌), and non-standard (⚡)
features.

- Uploading:
    - Formats:
        - ✅ PNG (`f=100`)
        - ✅ RGB, RGBA (`f=24`, `f=32`)
        - ✅ Compression with zlib (`o=z`)
        - ⚡ jpeg. Actually any format supported by imlib2 should work. The key
          value is the same as for png (`f=100`).
    - Transmission mediums:
        - ✅ Direct (`t=d`)
        - ✅ File (`t=f`)
        - ✅ Temporary file (`t=t`)
        - ❌ Shared memory object (`t=s`)
    - ❌ Size and offset specification (`S` and `O` keys)
    - ✅ Image numbers
    - ✅ Responses
    - ✅ Transmit and display (`a=T`)
- Placement:
    - ✅ Classic placements (but see the note above)
    - ✅ Unicode placeholders
    - ✅ Placement IDs
    - ❌ NOTE: Placement IDs must be 24-bit (between 1 and 16777215).
    - ✅ Cursor movement policies `C=1` and `C=0`
    - ✅ Source rectangle (`x, y, w, h`)
    - ✅ The number of rows/columns (`r, c`)
    - ❌ Cell offsets (`X, Y`)
    - ❌ z-index. Classic placements will erase old placements and the text on
      overlap.
    - ❌ Relative placements (`P, Q, H, V`)
- Deletion:
    - ✅ Deletion of image data when the specifier is uppercase
    - ✅ All visible classic placements (`d=a`)
    - ✅ By image id/number and placement id (`d=i`, `d=n`)
    - ❌ By position (specifiers `c, p, q, x, y, z`)
    - ❌ Animation frames (`d=f`)
- ❌ Animation - completely unsupported

## Things I have tested

### Apps that seem to work
- Kitty's icat (including from within tmux).
- [termpdf](https://github.com/dsanson/termpdf.py)
- [ranger](https://github.com/ranger/ranger) - I had to explicitly set
  `TERM=kitty`. This should be fixed in ranger of course.
- [tpix](https://github.com/jesvedberg/tpix)
- [pixcat](https://github.com/mirukana/pixcat) (modulo some unsupported keys
  that are currently ignored).
- [viu](https://github.com/atanunq/viu) - I had to explicitly set
  `TERM=kitty`.
- [timg](https://github.com/hzeller/timg) - I had to explicitly pass `-pk`
  (i.e. `timg -pk <image>`). If your timg is fresh enough, it even works in
  tmux!

### Apps that sort of work
- [hologram.nvim](https://github.com/edluffy/hologram.nvim) - There are some
  glitches, like erasure of parts of the status line.

### Apps that don't work
- [termvisage](https://github.com/AnonymouX47/termvisage) - seems to erase
  cells containing the image after image placement. In kitty this has no effect
  on classic placements because they aren't attached to cells, but in
  st-graphics classic placements are implemented on top of Unicode placements,
  so they get erased.

## Patches and additional features

This fork includes some patches and features that are not graphics-related
per se, but are hard to disentangle from the graphics implementation:
- [Anysize](https://st.suckless.org/patches/anysize/) - this patch is applied
  and on by default. If you want the expected anysize behavior (no centering),
  set `anysize_halign` and `anysize_valign` to zero in `config.h`.
