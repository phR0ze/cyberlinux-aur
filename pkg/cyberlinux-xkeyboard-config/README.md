# cyberlinux-xkeyboard-config
Custom keyboard layouts for machines supported by cyberlinux

***X Keyboard Extension (XKB)*** is how Linux configures keyboard layouts and key sequences.
During the creation of a layout you can use `xev` to determine key codes and symbol names.
The ***xkeyboard-config*** package provides the description files for the XKB system, settable with
***setxkbmap***.

## Resources
* https://wiki.archlinux.org/index.php/X_KeyBoard_extension
* https://github.com/dnschneid/crouton/blob/master/targets/keyboard
* https://www.x.org/releases/X11R7.7/doc/kbproto/xkbproto.html#The_Overlay1_and_Overlay2_Controls
* https://superuser.com/questions/801610/how-to-make-all-applications-respect-my-modified-xkb-layout
* https://unix.stackexchange.com/questions/144373/declare-a-new-modifier-key-with-xkb

## XKB Overview

### XKB Configuration
The XKB configuration has been decomposed into a number of components. The basic idea is that you
can adopt a mix-and-match approach to build a keyboard configuration.  Some pretty sophisticated
inclusion and augmentation rules allow a component to be built from a basic set-up and a number of
modifications for odd keyboard layouts and national percurliarities. The basic components are:

* ***key codes*** - A translation of the scan codes from the keyboard into a suitable symbolic form.
* ***key symbols*** - A tranlation of symbolic key codes into actual symbols, such as an ***A***
* ***compatibility map*** - A specification of what actions various special-purpose keys produce
* ***type*** - A specification of what various shift combinations produce.
* ***geometry*** - A description of the phyiscal layout of the keyboard
* ***rules*** and ***keymaps*** - Ways of packaging the main components into more usable
collections.

### Modifier Keys
The modifier keys are those keys, like ***Shift***, ***Control***, or ***Alt*** that are used to
change the meaning of the other keys.  Modifier keys can be combined, to give combinations like
***Control+Shift+Alt***. ***Virtual modifier*** keys allow a modifier or combination to be mapped
to a named virtual modifier. The ***types*** and ***compatibiltiy*** components are used to handle
this type of functionality.

### Levels and Groups
Once the modifier keys have been setup, it now becomes possible to have different combinations of
keys produce different characters from the keyboard. XKB uses two organization techniques
***levels*** and ***groups***.  A ***level*** represents the sort of thing that a shift key is
expected to do. For example the ***a*** character and the ***A*** characters are related by levels.
Non-shift level ***a*** and shift level ***A*** (i.e. think shifting a single key at a time).
Groups are a more complicated concept. The basic idea behind groups is that, sometimes, you want to
shift the entire keyboard over to some other character set (i.e. useful in overlays). The keys needed
to shift groups are less obvious than those needed to shift levels. Within each group there can be
multiple levels

### Key Codes
Raw key codes are the numeric codes generated by a particular keyboard to indicate that a key has
been pressed or released. The X system generates two events for each key press; one indicating the
key has been pressed, one to indicate it has been released. In both cases, the key code indicates
which key has been pressed or released. Hardware designers are free to choose whatever numbers they
like for keycodes, so despite typically having the same physical layout key code to symbolic
mappings need to be configured to get them the same.

### Key Symbols
Key symbols are the actual characters or glyphs that pressing a key produce. A symbol map maps
symbolic key codes onto the appropriate symbol. Symbol maps also handle specifying which keys are
modifier keys. Because groups and levels shift the meaning of the keys on the keyboard the symbol
map then becomes a matrix.  Symbol maps usually list multiple symbols for each key with the group
and level being used to look up the appropriate symbol.

### Rules and Keymaps
Rules and keymaps are ways to specify which set of components you'd like to use.

### XKB Config Files
https://www.charvolant.org/doug/xkb/html/node5.html

All XKB configuration files are structured in a similar way. A single configuration file contains
information on a set of similar themed items.  Each config file can contain multiple variants of
similar themed things. One of the variants will be marked as the ***default***, which is what you
get if you don't specify an explicit.

There are two ways of including information in the file: you can ***include*** or ***augment***. If
you include, using the syntax `include "file(variant)"`, note the lack of a semi-colon, then the
information from the included file overrides any configuration that already exists. if you augment
using the `augment "file(variant)"` then the information is only added if it doesn't override
something that already exists. In both cases the ***file*** param is referring to a file relative to
the same type your working with e.g. **symbols** then it would be the ***symbols*** dir.

#### Variant Options
| Option            | Description                                                               |
| ----------------- | ------------------------------------------------------------------------- |
| default           | Marks this as the default |
| partial           | |
| hidden            | |
| alphanumeric_keys | |
| modifier_keys     | |
| keypad_keys       | |
| function_keys     | |
| alternate_group   | |

### Putting it Together
The most likely approach to configuring XKB is to use the already existing configuration files and
glue them together such that you have to do little work but enjoy you're own overrides and
customizations. Each config file can contain multiple variants of similar themed things. One of the
variants will be marked as the ***default***, which is what you get if you don't specify an explicit
variant otherwise you would include the variant name in parenthesis. 

For example when you run ***setxkbmap -print -verbose 10*** to see the current settings you'll see:
```bash
xkb_keymap {
	xkb_keycodes  { include "evdev+aliases(qwerty)"	};
	xkb_types     { include "complete"	};
	xkb_compat    { include "complete"	};
	xkb_symbols   { include "pc+us+inet(evdev)"	};
	xkb_geometry  { include "pc(pc105)"	};
};
```

The geometry line is calling out ***pc(pc105)*** which means use the ***/usr/share/X11/xkb/geometry/pc***
file and explicitly the ***pc105*** variant within that file. This is required to use the
***pc105*** variant as when opening the file you'll notice that its actually the ***pc101*** variant
which has been marked as the default e.g.`default xkb_geometry "pc101"` 

This notion of defaults and variants per file holds true for all XKB components.

Once you have chosen a basic component, you can then extend that component by adding additional
components to it. These additional components modify or augment the meaning of the basic component.
The notion here is that you use ***+*** to ***override*** and existing definitions and/or augment.
An overridden definition always replaces an existing one, an augmented definition is only used if a
definition doesn't already exist. For example if you were using a US keyboard with 101 keys, but
wanted to swap the caps-lock and left-control keys, you could say `us(pc101)+ctrl(swapcaps)`

### Overlays
A keyboard overlay allows some subset of the keyboard to report alternate keycodes when the overlay
is enabled.  For example a keyboard overlay can be used to simulate a numeric or editing keypad on a
keyboard that does not actually have one by generating alternate keycodes for some keys when the
overlay is enabled. This technique is very common with ultra portable computers that have reduced
sized keyboards. XKB includes direct support for two keyboard overlays, using the ***Overlay1*** and
***Overlay2*** controls.  When ***Overlay1*** is enabled, all of the keys that are members of the
first keyboard overlay generate alternate keycodes. When ***Overlay2*** is enabled, all of the keys
that are members of the second keyboard overlay genearte alternate keycodes.

To specify the overlay to which a key belongs and the alternate keycode it should generate when that
overlay is enabled, assign it either the ***KB_Overlay1*** or ***KB_Overlay2*** key behaviors.

To specify the overlay to which a key belongs and the alternate keycode it should generate when that
overlay is enabled, assign the keys to a group where the overlay key is configured.

Keymap group or `*.xkb` file:

```bash
xkb_keymap {
    xkb_keycodes { include "evdev+aliases(qwerty)" };
    xkb_types { include "complete" };
    xkb_compatibility {
        include "complete"

        interpret Overlay0_Enable {
            action = SetControls(controls=overlay0);
        };
    };
    xkb_symbols {
        include "pc+us(dvorak)+inet(evdev)"

        key <RALT> {
            type[Group0] = "ONE_LEVEL",
            symbols[Group0] = [ Overlay1_Enable ]
        };
        key <AC06> { overlay1 = <LEFT> };
        key <AC7> { overlay1 = <DOWN> };
        key <AC8> { overlay1 = <RGHT> };
        key <AD7> { overlay1 = <UP> };
    };
    xkb_geometry { include "pc(pc103)" };
};
```

## Samsung Chromebook 3 (CELES)

### 1. Add Compatibility Map
https://wiki.archlinux.org/index.php/X_KeyBoard_extension#xkb_compatibility

Compatibility maps are specifications of what action should each key produce in order to preserve
compatibility with XKB-unaware clients.

1. Edit: ***compat/Makefile.am***
2. Add: ***celes*** to ***compat_Data*** list
3. Edit: ***compat/celes***
4. Add compatibility overlay
```bash
default partial xkb_compatibility "overlay"  {
    interpret Overlay1_Enable {
        action = SetControls(controls=Overlay1);
    };
};
```

***SetControls*** means change the control bit while the key is pressed and restore it on the key
release.

### 2. Add Key Codes

### 3. Add Key Symbols
```bash
// This mapping assumes that inet(evdev) will also be sourced
partial
xkb_symbols "overlay" {
    key <LWIN> { [ Overlay1_Enable ], overlay1=<LWIN> };
    key <AB09> { overlay1=<INS> };
    key <LEFT> { overlay1=<HOME> };
    key <RGHT> { overlay1=<END> };
    key <UP>   { overlay1=<PGUP> };
    key <DOWN> { overlay1=<PGDN> };
    key <FK01> { overlay1=<I247> };
    key <I247> { [ XF86Back ] };
    key <FK02> { overlay1=<I248> };
    key <I248> { [ XF86Forward ] };
    key <FK03> { overlay1=<I249> };
    key <I249> { [ XF86Reload ] };
    key <FK04> { overlay1=<I235> }; // XF86Display
    key <FK05> { overlay1=<I250> };
    key <I250> { [ XF86ApplicationRight ] };
    key <FK06> { overlay1=<I232> }; // XF86MonBrightnessDown
    key <FK07> { overlay1=<I233> }; // XF86MonBrightnessUp
    key <FK08> { overlay1=<MUTE> };
    key <FK09> { overlay1=<VOL-> };
    key <FK10> { overlay1=<VOL+> };
    key <AE01> { overlay1=<FK01> };
    key <AE02> { overlay1=<FK02> };
    key <AE03> { overlay1=<FK03> };
    key <AE04> { overlay1=<FK04> };
    key <AE05> { overlay1=<FK05> };
    key <AE06> { overlay1=<FK06> };
    key <AE07> { overlay1=<FK07> };
    key <AE08> { overlay1=<FK08> };
    key <AE09> { overlay1=<FK09> };
    key <AE10> { overlay1=<FK10> };
    key <AE11> { overlay1=<FK11> };
    key <AE12> { overlay1=<FK12> };
    key <BKSP> { overlay1=<DELE> };
    key <LALT> { overlay1=<CAPS> };
    key <RALT> { overlay1=<CAPS> };
    // For some strange reason, some Super_R events are triggered when
    // the Search key is released (i.e. with overlay on).
    // This maps RWIN to a dummy key (<I253>), to make sure we catch it.
    key <RWIN> { [ NoSymbol ], overlay1=<I253> };
    // Map dummy key to no symbol
    key <I253> { [ NoSymbol ] };
};
```
