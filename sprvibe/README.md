These were samples created programmatically from the SuperDirt synth 'supervibe', spanning MIDI notes 12-96.

NB: I've included an extra _ in the filenames for notes 12-59, so that notes 60-96 will precede them in the file indexing. So,

`n "0 -1" # s "sprvibe" `

will produce MIDI notes 60 and 59 respectively, instead of MIDI notes 12 and 96 (i.e., it behaves the same as the actual supervibe synth).

<br>

Here is the SuperCollider code that I used to create the files (based mostly on this [SO post]( https://stackoverflow.com/a/22394238/2717159))

```
(
SynthDef(\recordTone, { |buffer|
    RecordBuf.ar(In.ar(0,2), buffer, loop: 0, doneAction: 2);
}).add;
)
```

```
(
Routine({
    var recordfn = { |synthName, noteName, duration|
        var server = Server.local;
        var buffer = Buffer.alloc(server, server.sampleRate * duration * 1.05, 2);
		var filename = synthName ++ "_" ++ noteName ++ ".wav";
		var freq = noteName.midicps;

        server.sync;

        server.makeBundle(func: {
            var player = Synth(synthName, [\freq, freq, \sustain, duration, \decay, 0.1]);
            var recorder = Synth.after(player, \recordTone, [\buffer, buffer]);
        });

        duration.wait;

        buffer.write(
			"/Users/your/file/path" ++ filename,
            "WAVE",
            "int16",
            completionMessage: ["/b_free", buffer]
        );
    };

		for (60, 96, { arg i; recordfn.value(\supervibe,i,1.0);});


}).next)
```

Usage: 

* execute the \recordTone SynthDef
* replace "/Users/your/file/path" with your filepath
* replace the line: `for (60, 96, { arg i; recordfn.value(\supervibe,i,1.0);});` with
     	`for (<minNote>, <maxNote>, { arg i; recordfn.value(<a SynthDef you loaded>,i,<duration in seconds>);});`
* execute the routine
