# Web MIDI API Explainer

## Examples of Web MIDI API Usage in JavaScript
The following are some examples of common MIDI usage in JavaScript.

### Getting Access to the MIDI System
This example shows how to request access to the MIDI system.

```js
let midi = null;  // global MIDIAccess object

function onMIDISuccess( midiAccess ) {
  console.log( "MIDI ready!" );
  midi = midiAccess;  // store in the global (in real usage, would probably keep in an object instance)
}

function onMIDIFailure(msg) {
  console.log( "Failed to get MIDI access - " + msg );
}

navigator.requestMIDIAccess().then( onMIDISuccess, onMIDIFailure );
```

### Requesting Access to the MIDI System with System Exclusive Support
This example shows how to request access to the MIDI system, including the
ability to send and receive System Exclusive messages.

```js
let midi = null;  // global MIDIAccess object

function onMIDISuccess( midiAccess ) {
  console.log( "MIDI ready!" );
  midi = midiAccess;  // store in the global (in real usage, would probably keep in an object instance)
}

function onMIDIFailure(msg) {
  console.log( "Failed to get MIDI access - " + msg );
}

navigator.requestMIDIAccess( { sysex: true } ).then( onMIDISuccess, onMIDIFailure );
```

### Listing Inputs and Outputs
This example gets the list of the input and output ports and prints their
information to the console log, using ES6 for...of notation.

```js
function listInputsAndOutputs( midiAccess ) {
  for (let entry of midiAccess.inputs) {
    let input = entry[1];
    console.log( "Input port [type:'" + input.type + "'] id:'" + input.id +
      "' manufacturer:'" + input.manufacturer + "' name:'" + input.name +
      "' version:'" + input.version + "'" );
  }

  for (let entry of midiAccess.outputs) {
    let output = entry[1];
    console.log( "Output port [type:'" + output.type + "'] id:'" + output.id +
      "' manufacturer:'" + output.manufacturer + "' name:'" + output.name +
      "' version:'" + output.version + "'" );
  }
}
```

### Handling MIDI Input
This example prints incoming MIDI messages on a single arbitrary input port to
the console log.

```js
function onMIDIMessage( event ) {
  let str = "MIDI message received at timestamp " + event.timeStamp + "[" + event.data.length + " bytes]: ";
  for (let i=0; i&lt;event.data.length; i++) {
    str += "0x" + event.data[i].toString(16) + " ";
  }
  console.log( str );
}

function startLoggingMIDIInput( midiAccess, indexOfPort ) {
  midiAccess.inputs.forEach( function(entry) {entry.onmidimessage = onMIDIMessage;});
}
```

### Sending MIDI Messages to an Output Device
This example sends a middle C note on message immediately on MIDI channel 1
(MIDI channels are 0-indexed, but generally referred to as channels 1-16), and
queues a corresponding note off message for 1 second later.

```js
function sendMiddleC( midiAccess, portID ) {
  let noteOnMessage = [0x90, 60, 0x7f];    // note on, middle C, full velocity
  let output = midiAccess.outputs.get(portID);
  output.send( noteOnMessage );  //omitting the timestamp means send immediately.
  output.send( [0x80, 60, 0x40], window.performance.now() + 1000.0 ); // Inlined array creation- note off, middle C,
                                                                      // release velocity = 64, timestamp = now + 1000ms.
}
```

### A Simple Loopback
This example loops all input messages on the first input port to the first
output port - including System Exclusive messages.

```js
let midi = null;  // global MIDIAccess object
let output = null;

function echoMIDIMessage( event ) {
  if (output) {
    output.send( event.data, event.timeStamp );
  }
}

function onMIDISuccess( midiAccess ) {
  console.log( "MIDI ready!" );
  let input = midiAccess.inputs.entries().next();
  if (input)
    input.onmidimessage = echoMIDIMessage;
  output = midiAccess.outputs.values().next().value;
  if (!input || !output)
    console.log("Uh oh! Couldn't get i/o ports.");
}

function onMIDIFailure(msg) {
  console.log( "Failed to get MIDI access - " + msg );
}

navigator.requestMIDIAccess().then( onMIDISuccess, onMIDIFailure );
```

### A Simple Monophonic Sine Wave MIDI Synthesizer
This example listens to all input messages from all available input ports, and
uses note messages to drive the envelope and frequency on a monophonic sine wave
oscillator, creating a very simple synthesizer, using the Web Audio API. Note on
and note off messages are supported, but sustain pedal, velocity and pitch bend
are not. This sample is also hosted on <a href=
"http://webaudiodemos.appspot.com/monosynth/index.html">webaudiodemos.appspot.com</a>.

```js
let context=null;   // the Web Audio "context" object
let midiAccess=null;  // the MIDIAccess object.
let oscillator=null;  // the single oscillator
let envelope=null;    // the envelope for the single oscillator
let attack=0.05;      // attack speed
let release=0.05;   // release speed
let portamento=0.05;  // portamento/glide speed
let activeNotes = []; // the stack of actively-pressed keys

window.addEventListener('load', function() {
      // patch up prefixes
      window.AudioContext=window.AudioContext||window.webkitAudioContext;

      context = new AudioContext();
      if (navigator.requestMIDIAccess)
        navigator.requestMIDIAccess().then( onMIDIInit, onMIDIReject );
      else
        alert("No MIDI support present in your browser.  You're gonna have a bad time.")

      // set up the basic oscillator chain, muted to begin with.
      oscillator = context.createOscillator();
      oscillator.frequency.setValueAtTime(110, 0);
      envelope = context.createGain();
      oscillator.connect(envelope);
      envelope.connect(context.destination);
      envelope.gain.value = 0.0;  // Mute the sound
      oscillator.start(0);  // Go ahead and start up the oscillator
} );

function onMIDIInit(midi) {
      midiAccess = midi;

      let haveAtLeastOneDevice=false;
      let inputs=midiAccess.inputs.values();
      for ( let input = inputs.next(); input &amp;& !input.done; input = inputs.next()) {
        input.value.onmidimessage = MIDIMessageEventHandler;
        haveAtLeastOneDevice = true;
      }
      if (!haveAtLeastOneDevice)
        alert("No MIDI input devices present.  You're gonna have a bad time.");
}

function onMIDIReject(err) {
      alert("The MIDI system failed to start.  You're gonna have a bad time.");
}

function MIDIMessageEventHandler(event) {
      // Mask off the lower nibble (MIDI channel, which we don't care about)
      switch (event.data[0] & 0xf0) {
        case 0x90:
          if (event.data[2]!=0) {  // if velocity != 0, this is a note-on message
            noteOn(event.data[1]);
            return;
          }
          // if velocity == 0, fall thru: it's a note-off.  MIDI's weird, y'all.
        case 0x80:
          noteOff(event.data[1]);
          return;
      }
}

function frequencyFromNoteNumber( note ) {
      return 440 * Math.pow(2,(note-69)/12);
}

function noteOn(noteNumber) {
      activeNotes.push( noteNumber );
      oscillator.frequency.cancelScheduledValues(0);
      oscillator.frequency.setTargetAtTime( frequencyFromNoteNumber(noteNumber), 0, portamento );
      envelope.gain.cancelScheduledValues(0);
      envelope.gain.setTargetAtTime(1.0, 0, attack);
}

function noteOff(noteNumber) {
      let position = activeNotes.indexOf(noteNumber);
      if (position!=-1) {
        activeNotes.splice(position,1);
      }
      if (activeNotes.length==0) {  // shut off the envelope
        envelope.gain.cancelScheduledValues(0);
        envelope.gain.setTargetAtTime(0.0, 0, release );
      } else {
        oscillator.frequency.cancelScheduledValues(0);
        oscillator.frequency.setTargetAtTime( frequencyFromNoteNumber(activeNotes[activeNotes.length-1]), 0, portamento );
      }
}
```
