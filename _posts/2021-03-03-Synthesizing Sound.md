---
title: Synthesizing Sound
author: lijiabao
date: 2020-12-06 01:47:59.227335500 +0800
category: Sound
categories: Sound
tags: Sound
---

# Synthesizing Sound

<a name="121725" id="121725"></a> Most programs that avail themselves of the Java Sound API's MIDI package do so to synthesize sound. The entire apparatus of MIDI files, events, sequences, and sequencers, which was previously discussed, nearly always has the goal of eventually sending musical data to a synthesizer to convert into audio. (Possible exceptions include programs that convert MIDI into musical notation that can be read by a musician, and programs that send messages to external MIDI-controlled devices such as mixing consoles.)

<a name="121727" id="121727"></a> The `Synthesizer` interface is therefore fundamental to the MIDI package. This page shows how to manipulate a synthesizer to play sound. Many programs will simply use a sequencer to send MIDI file data to the synthesizer, and won't need to invoke many `Synthesizer` methods directly. However, it's possible to control a synthesizer directly, without using sequencers or even `MidiMessage` objects, as explained near the end of this page.

<a name="121729" id="121729"></a> The synthesis architecture might seem complex for readers who are unfamiliar with MIDI. Its API includes three interfaces:

<li><a name="121730" id="121730"></a> 
[`Synthesizer`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/Synthesizer.html)</li>
<li><a name="121731" id="121731"></a> 
[`MidiChannel`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/MidiChannel.html)</li>
<li><a name="121732" id="121732"></a> 
[`Soundbank`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/Soundbank.html)</li>

<a name="121733" id="121733"></a> and four classes:

<li><a name="121734" id="121734"></a> 
[`Instrument`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/Instrument.html)</li>
<li><a name="121735" id="121735"></a> 
[`Patch`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/Patch.html)</li>
<li><a name="121736" id="121736"></a> 
[`SoundbankResource`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/SoundbankResource.html)</li>
<li><a name="121737" id="121737"></a> 
[`VoiceStatus`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/VoiceStatus.html)</li>

<a name="121739" id="121739"></a>As orientation for all this API, the next section explains some of the basics of MIDI synthesis and how they're reflected in the Java Sound API. Subsequent sections give a more detailed look at the API.

<a name="121742" id="121742"></a>

## Understanding MIDI Synthesis

<a name="121745" id="121745"></a> How does a synthesizer generate sound? Depending on its implementation, it may use one or more sound-synthesis techniques. For example, many synthesizers use wavetable synthesis. A wavetable synthesizer reads stored snippets of audio from memory, playing them at different sample rates and looping them to create notes of different pitches and durations. For example, to synthesize the sound of a saxophone playing the note C#4 (MIDI note number 61), the synthesizer might access a very short snippet from a recording of a saxophone playing the note Middle C (MIDI note number 60), and then cycle repeatedly through this snippet at a slightly faster sample rate than it was recorded at, which creates a long note of a slightly higher pitch. Other synthesizers use techniques such as frequency modulation (FM), additive synthesis, or physical modeling, which don't make use of stored audio but instead generate audio from scratch using different algorithms.

<a name="121747" id="121747"></a>

### Instruments

<a name="123181" id="123181"></a> What all synthesis techniques have in common is the ability to create many sorts of sounds. Different algorithms, or different settings of parameters within the same algorithm, create different-sounding results. An **instrument** is a specification for synthesizing a certain type of sound. That sound may emulate a traditional musical instrument, such as a piano or violin; it may emulate some other kind of sound source, for instance, a telephone or helicopter; or it may emulate no "real-world" sound at all. A specification called General MIDI defines a standard list of 128 instruments, but most synthesizers allow other instruments as well. Many synthesizers provide a collection of built-in instruments that are always available for use; some synthesizers also support mechanisms for loading additional instruments.

<a name="123190" id="123190"></a> An instrument may be vendor-specific&#226;&#128;&#148;in other words, applicable to only one synthesizer or several models from the same vendor. This incompatibility results when two different synthesizers use different sound-synthesis techniques, or different internal algorithms and parameters even if the fundamental technique is the same. Because the details of the synthesis technique are often proprietary, incompatibility is common. The Java Sound API includes ways to detect whether a given synthesizer supports a given instrument.

<a name="121753" id="121753"></a> An instrument can usually be considered a preset; you don't have to know anything about the details of the synthesis technique that produces its sound. However, you can still vary aspects of its sound. Each Note On message specifies the pitch and volume of an individual note. You can also alter the sound through other MIDI commands such as controller messages or system-exclusive messages.

<a name="121755" id="121755"></a>

### Channels

<a name="121759" id="121759"></a> Many synthesizers are **multimbral** (sometimes called **polytimbral**), meaning that they can play the notes of different instruments simultaneously. (**Timbre** is the characteristic sound quality that enables a listener to distinguish one kind of musical instrument from other kinds.) Multimbral synthesizers can emulate an entire ensemble of real-world instruments, instead of only one instrument at a time. MIDI synthesizers normally implement this feature by taking advantage of the different MIDI channels on which the MIDI specification allows data to be transmitted. In this case, the synthesizer is actually a collection of sound-generating units, each emulating a different instrument and responding independently to messages that are received on a different MIDI channel. Since the MIDI specification provides only 16 channels, a typical MIDI synthesizer can play up to 16 different instruments at once. The synthesizer receives a stream of MIDI commands, many of which are channel commands. (Channel commands are targeted to a particular MIDI channel; for more information, see the MIDI specification.) If the synthesizer is multitimbral, it routes each channel command to the correct sound-generating unit, according to the channel number indicated in the command.

<a name="121761" id="121761"></a> In the Java Sound API, these sound-generating units are instances of classes that implement the `MidiChannel` interface. A `synthesizer` object has at least one `MidiChannel` object. If the synthesizer is multimbral, it has more than one, normally 16. Each `MidiChannel` represents an independent sound-generating unit.

<a name="121763" id="121763"></a> Because a synthesizer's `MidiChannel` objects are more or less independent, the assignment of instruments to channels doesn't have to be unique. For example, all 16 channels could be playing a piano timbre, as though there were an ensemble of 16 pianos. Any grouping is possible&#226;&#128;&#148;for instance, channels 1, 5, and 8 could be playing guitar sounds, while channels 2 and 3 play percussion and channel 12 has a bass timbre. The instrument being played on a given MIDI channel can be changed dynamically; this is known as a **program change**.

<a name="121765" id="121765"></a> Even though most synthesizers allow only 16 or fewer instruments to be active at a given time, these instruments can generally be chosen from a much larger selection and assigned to particular channels as required.

<a name="121767" id="121767"></a>

### Soundbanks and Patches

<a name="121769" id="121769"></a> Instruments are organized hierarchically in a synthesizer, by bank number and program number. Banks and programs can be thought of as rows and columns in a two-dimensional table of instruments. A bank is a collection of programs. The MIDI specification allows up to 128 programs in a bank, and up to 128 banks. However, a particular synthesizer might support only one bank, or a few banks, and might support fewer than 128 programs per bank.

<a name="121771" id="121771"></a> In the Java Sound API, there's a higher level to the hierarchy: a soundbank. Soundbanks can contain up to 128 banks, each containing up to 128 instruments. Some synthesizers can load an entire soundbank into memory.

<a name="121773" id="121773"></a> To select an instrument from the current soundbank, you specify a bank number and a program number. The MIDI specification accomplishes this with two MIDI commands: bank select and program change. In the Java Sound API, the combination of a bank number and program number is encapsulated in a `Patch` object. You change a MIDI channel's current instrument by specifying a new patch. The patch can be considered the two-dimensional index of the instruments in the current soundbank.

<a name="121775" id="121775"></a> You might be wondering if soundbanks, too, are indexed numerically. The answer is no; the MIDI specification does not provide for this. In the Java Sound API, a `Soundbank` object can be obtained by reading a soundbank file. If the soundbank is supported by the synthesizer, its instruments can be loaded into the synthesizer individually as desired, or all at once. Many synthesizers have a built-in or default soundbank; the instruments contained in this soundbank are always available to the synthesizer.

<a name="121777" id="121777"></a>

### Voices

<a name="121779" id="121779"></a> It's important to distinguish between the number of **timbres** a synthesizer can play simultaneously and the number of **notes** it can play simultaneously. The former was described above under "Channels." The ability to play multiple notes at once is referred to as **polyphony**. Even a synthesizer that isn't multitimbral can generally play more than one note at a time (all having the same timbre, but different pitches). For example, playing any chord, such as a G major triad or a B minor seventh chord, requires polyphony. Any synthesizer that generates sound in real time has a limitation on the number of notes it can synthesize at once. In the Java Sound API, the synthesizer reports this limitation through the `getMaxPolyphony` method.

<a name="121783" id="121783"></a> A **voice** is a succession of single notes, such as a melody that a person can sing. Polyphony consists of multiple voices, such as the parts sung by a choir. A 32-voice synthesizer, for example, can play 32 notes simultaneously. (However, some MIDI literature uses the word "voice" in a different sense, similar to the meaning of "instrument" or "timbre.")

<a name="121785" id="121785"></a> The process of assigning incoming MIDI notes to specific voices is known as **voice allocation**. A synthesizer maintains a list of voices, keeping track of which ones are active (meaning that they currently have notes sounding). When a note stops sounding, the voice becomes inactive, meaning that it's now free to accept the next note-on request that the synthesizer receives. An incoming stream of MIDI commands can easily request more simultaneous notes than the synthesizer is capable of generating. When all the synthesizer's voices are active, how should the next Note On request be handled? Synthesizers can implement different strategies: the most recently requested note can be ignored; or it can be played by discontinuing another note, such as the least recently started one.

<a name="121787" id="121787"></a> Although the MIDI specification does not require it, a synthesizer can make public the contents of each of its voices. The Java Sound API includes a `VoiceStatus` class for this purpose.

<a name="121788" id="121788"></a> A `VoiceStatus` reports on the voice's current active or inactive status, MIDI channel, bank and program number, MIDI note number, and MIDI volume.

<a name="121790" id="121790"></a> With this background, let's examine the specifics of the Java Sound API for synthesis.

<a name="121793" id="121793"></a>

## Managing Instruments and Soundbanks

<a name="121795" id="121795"></a> In many cases, a program can make use of a `Synthesizer` object without explicitly invoking almost any of the synthesis API. For example, suppose you're playing a standard MIDI file. You load it into a `Sequence` object, which you play by having a sequencer send the data to the default synthesizer. The data in the sequence controls the synthesizer as intended, playing all the right notes at the right times.

<a name="121797" id="121797"></a> However, there are cases when this simple scenario is inadequate. The sequence contains the right music, but the instruments sound all wrong! This unfortunate situation probably arose because the creator of the MIDI file had different instruments in mind than the ones that are currently loaded into the synthesizer.

<a name="123339" id="123339"></a> The MIDI 1.0 Specification provides for bank-select and program-change commands, which affect which instrument is currently playing on each MIDI channel. However, the specification does not define what instrument should reside in each patch location (bank and program number). The more recent General MIDI specification addresses this problem by defining a bank containing 128 programs that correspond to specific instrument sounds. A General MIDI synthesizer uses 128 instruments that match this specified set. Different General MIDI synthesizers can sound quite different, even when playing what's supposed to be the same instrument. However, a MIDI file should for the most part sound similar (even if not identical), no matter which General MIDI synthesizer is playing it.

<a name="123341" id="123341"></a> Nonetheless, not all creators of MIDI files want to be limited to the set of 128 timbres defined by General MIDI. This section shows how to change the instruments from the default set that the synthesizer comes with. (If there is no default, meaning that no instruments are loaded when you access the synthesizer, you'll have to use this API to start with in any case.)

<a name="121803" id="121803"></a>

### Learning What Instruments Are Loaded

<a name="121805" id="121805"></a> To learn whether the instruments currently loaded into the synthesizer are the ones you want, you can invoke this `Synthesizer` method:

```

Instrument[] getLoadedInstruments() 

```

and iterate over the returned array to see exactly which instruments are currently loaded. Most likely, you would display the instruments' names in the user interface (using the `getName` method of `Instrument`), and let the user decide whether to use those instruments or load others. The `Instrument` API includes a method that reports which soundbank the instrument belongs to. The soundbank's name might help your program or the user ascertain exactly what the instrument is.

<a name="123382" id="123382"></a> This `Synthesizer` method:

```

Soundbank getDefaultSoundbank() 

```

gives you the default soundbank. The `Soundbank` API includes methods to retrieve the soundbank's name, vendor, and version number, by which the program or the user can verify the bank's identity. However, you can't assume when you first get a synthesizer that the instruments from the default soundbank have been loaded into the synthesizer. For example, a synthesizer might have a large assortment of built-in instruments available for use, but because of its limited memory it might not load them automatically. <a name="121824" id="121824"></a>

### Loading Different Instruments

<a name="122802" id="122802"></a> The user might decide to load instruments that are different from the current ones (or you might make that decision programmatically). The following method tells you which instruments come with the synthesizer (versus having to be loaded from soundbank files):

```

Instrument[] getAvailableInstruments()

```

You can load any of these instruments by invoking:

```

boolean loadInstrument(Instrument instrument) 

```

The instrument gets loaded into the synthesizer in the location specified by the instrument's `Patch` object (which can be retrieved using the `getPatch` method of `Instrument`).

<a name="121836" id="121836"></a> To load instruments from other soundbanks, first invoke `Synthesizer's` `isSupportedSoundbank` method to make sure that the soundbank is compatible with this synthesizer (if it isn't, you can iterate over the system's synthesizers to try to find one that supports the soundbank). You can then invoke one of these methods to load instruments from the soundbank:

```

boolean loadAllInstruments(Soundbank soundbank) 
boolean loadInstruments(Soundbank soundbank, 
  Patch[] patchList) 

```

As the names suggest, the first of these loads the entire set of instruments from a given soundbank, and the second loads selected instruments from the soundbank. You could also use `Soundbank's` `getInstruments` method to access all the instruments, then iterate over them and load selected instruments one at a time using `loadInstrument`.

<a name="121843" id="121843"></a> It's not necessary for all the instruments you load to be from the same soundbank. You could use `loadInstrument` or `loadInstruments` to load certain instruments from one soundbank, another set from a different soundbank, and so on.

<a name="121846" id="121846"></a> Each instrument has its own `Patch` object that specifies the location on the synthesizer where the instrument should be loaded. The location is defined by a bank number and a program number. There's no API to change the location by changing the patch's bank or program number.

<a name="121852" id="121852"></a> However, it is possible to load an instrument into a location other than the one specified by its patch, using the following method of `Synthesizer`:

```

boolean remapInstrument(Instrument from, Instrument to) 

```

This method unloads its first argument from the synthesizer, and places its second argument in whatever synthesizer patch location had been occupied by the first argument. <a name="121858" id="121858"></a>

### Unloading Instruments

<a name="123434" id="123434"></a> Loading an instrument into a program location automatically unloads whatever instrument was already at that location, if any. You can also explicitly unload instruments without necessarily replacing them with new ones. `Synthesizer` includes three unloading methods that correspond to the three loading methods. If the synthesizer receives a program-change message that selects a program location where no instrument is currently loaded, there won't be any sound from the MIDI channel on which the program-change message was sent.

<a name="123435" id="123435"></a>

### Accessing Soundbank Resources

<a name="123439" id="123439"></a> Some synthesizers store other information besides instruments in their soundbanks. For example, a wavetable synthesizer stores audio samples that one or more instruments can access. Because the samples might be shared by multiple instruments, they're stored in the soundbank independently of any instrument. Both the `Soundbank` interface and the `Instrument` class provide a method call `getSoundbankResources`, which returns a list of `SoundbankResource` objects. The details of these objects are specific to the synthesizer for which the soundbank is designed. In the case of wavetable synthesis, a resource might be an object that encapsulates a series of audio samples, taken from one snippet of a sound recording. Synthesizers that use other synthesis techniques might store other kinds of objects in the synthesizer's `SoundbankResources` array.

<a name="123441" id="123441"></a>

## Querying the Synthesizer's Capabilities and Current State

<a name="121873" id="121873"></a> The `Synthesizer` interface includes methods that return information about the synthesizer's capabilities:

```

    public long getLatency()
    public int getMaxPolyphony()

```

The latency measures the worst-case delay between the time a MIDI message is delivered to the synthesizer and the time that the synthesizer actually produces the corresponding result. For example, it might take a synthesizer a few milliseconds to begin generating audio after receiving a note-on event.

[Voices](#121777). As mentioned in the same discussion, a synthesizer can provide information about its voices. This is accomplished through the following method:

```

public VoiceStatus[] getVoiceStatus()

```

Each `VoiceStatus` in the returned array reports the voice's current active or inactive status, MIDI channel, bank and program number, MIDI note number, and MIDI volume. The array's length should normally be the same number returned by `getMaxPolyphony`. If the synthesizer isn't playing, all its `VoiceStatus` objects have their active field set to `false`.

<a name="121886" id="121886"></a> You can learn additional information about the current status of a synthesizer by retrieving its `MidiChannel` objects and querying their state. This is discussed more in the next section.

<a name="121890" id="121890"></a>

## Using Channels

<a name="121892" id="121892"></a> Sometimes it's useful or necessary to access a synthesizer's `MidiChannel` objects directly. This section discusses such situations.

<a name="121894" id="121894"></a>

### Controlling the Synthesizer without Using a Sequencer

<a name="121896" id="121896"></a> When using a sequence, such as one read from a MIDI file, you don't need to send MIDI commands to the synthesizer yourself. Instead, you just load the sequence into a sequencer, connect the sequencer to the synthesizer, and let it run. The sequencer takes care of scheduling the events, and the result is a predictable musical performance. This scenario works fine when the desired music is known in advance, which is true when it's read from a file.

<a name="121898" id="121898"></a> In some situations, however, the music is generated on the fly as it's playing. For example, the user interface might display a musical keyboard or a guitar fretboard and let the user play notes at will by clicking with the mouse. As another example, an application might use a synthesizer not to play music per se, but to generate sound effects in response to the user's actions. This scenario is typical of games. As a final example, the application might indeed be playing music that's read from a file, but the user interface allows the user to interact with the music, altering it dynamically. In all these cases, the application sends commands directly to the synthesizer, since the MIDI messages need to be delivered immediately, instead of being scheduled for some determinate point in the future.

<a name="121900" id="121900"></a> There are at least two ways of sending a MIDI message to the synthesizer without using a sequencer. The first is to construct a `MidiMessage` and pass it to the synthesizer using the send method of `Receiver`. For example, to produce a Middle C (MIDI note number 60) on MIDI channel 5 (one-based) immediately, you could do the following:

```

    ShortMessage myMsg = new ShortMessage();
    // Play the note Middle C (60) moderately loud
    // (velocity = 93)on channel 4 (zero-based).
    myMsg.setMessage(ShortMessage.NOTE_ON, 4, 60, 93); 
    Synthesizer synth = MidiSystem.getSynthesizer();
    Receiver synthRcvr = synth.getReceiver();
    synthRcvr.send(myMsg, -1); // -1 means no time stamp

```

The second way is to bypass the message-passing layer (that is, the `MidiMessage` and `Receiver` API) altogether, and interact with the synthesizer's `MidiChannel` objects directly. You first need to retrieve the synthesizer's `MidiChannel` objects, using the following `Synthesizer` method:

```

public MidiChannel[] getChannels()

```

after which you can invoke the desired `MidiChannel` methods directly. This is a more immediate route than sending the corresponding `MidiMessages` to the synthesizer's `Receiver` and letting the synthesizer handle the communication with its own `MidiChannels`. For example, the code corresponding to the preceding example would be:

```

    Synthesizer synth = MidiSystem.getSynthesizer();
    MidiChannel chan[] = synth.getChannels(); 
    // Check for null; maybe not all 16 channels exist.
    if (chan[4] != null) {
         chan[4].noteOn(60, 93); 
    }

```

### Getting a Channel's Current State

<a name="121939" id="121939"></a> The `MidiChannel` interface provides methods that correspond one-to-one to each of the "channel voice" or "channel mode" messages defined by the MIDI specification. We saw one case with the use of the noteOn method in the previous example. However, in addition to these canonical methods, the Java Sound API's `MidiChannel` interface adds some "get" methods to retrieve the value most recently set by corresponding voice or mode "set" methods:

```

    int       getChannelPressure()
    int       getController(int controller)
    boolean   getMono()
    boolean   getOmni() 
    int       getPitchBend() 
    int       getPolyPressure(int noteNumber)
    int       getProgram()

```

These methods might be useful for displaying channel state to the user, or for deciding what values to send subsequently to the channel. <a name="121953" id="121953"></a>

### Muting and Soloing a Channel

<a name="123492" id="123492"></a> The Java Sound API adds the notions of per-channel solo and mute, which are not required by the MIDI specification. These are similar to the solo and mute on the tracks of a MIDI sequence.

<a name="123493" id="123493"></a> If mute is on, this channel will not sound, but other channels are unaffected. If solo is on, this channel, and any other soloed channel, will sound (if it isn't muted), but no other channels will sound. A channel that is both soloed and muted will not sound. The `MidiChannel` API includes four methods:

```

    boolean      getMute() 
    boolean      getSolo()
    void         setMute(boolean muteState) 
    void         setSolo(boolean soloState)

```

## Permission to Play Synthesized Sound

<a name="123505" id="123505"></a> The audio produced by any installed MIDI synthesizer is typically routed through the sampled-audio system. If your program doesn't have permission to play audio, the synthesizer's sound won't be heard, and a security exception will be thrown. For more information on audio permissions, see the previous discussion of Permission to Use Audio Resources 
[Permission to Use Audio Resources](accessing.html#113223).

<a name="121968" id="121968"></a>