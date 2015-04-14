# web-midi-api logo

This is a working folder for development on the
web-midi-api logo.  For more information, see this thread:

- https://lists.w3.org/Archives/Public/public-audio/2015AprJun/0018.html

## Quick Thoughts

- Incorporate MIDI logo
  - http://www.midi.org/

- Use open/free fonts for anything else:
  - http://www.google.com/fonts

- Incorporate HTML5 branding / creative and possibly add to the "builder" here:
  - http://www.w3.org/html/logo/

- Reference MIDIappy mascot somewhere (in svg comment or readme)
  - http://www.g200kg.com/archives/2014/10/midiappy-images.html


## Initial Logo (Rough Draft)

### MIDI SVG Logo

1. google image search for "midi logo"
2. `wget http://www.grantmuller.com/wp-content/uploads/midi_logo.png -O midi_logo.png`
3. `convert midi_logo.png midi_logo.bmp`
4. `potrace -s midi_logo.bmp -o midi_logo.svg --flat --opttolerance 1 --invert`

### Web Text

1. http://www.google.com/fonts
2. WEB text: Source Sans Pro, 72px, Bold 700
3. Screenshot and convert to svg

### Create Rough Draft

1. Combine "WEB" and "MIDI" using the online tool
   [svg-edit](http://svg-edit.googlecode.com/), then hand editing.

### Credits

- SVG-edit - http://svg-edit.googlecode.com/
- potrace 1.12, written by Peter Selinger 2001-2015


## License

- w3c: http://www.w3.org/Consortium/Legal/2002/copyright-software-20021231
- midi logo: www.midi.org
- "web" text/font: http://www.google.com/fonts (Source Sans Pro)

