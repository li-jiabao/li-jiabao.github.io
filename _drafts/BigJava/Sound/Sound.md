
# Trail: Sound

The Java Sound API is a low-level API for effecting and controlling the input and output of sound media, including both audio and Musical Instrument Digital Interface (MIDI) data. The Java Sound API provides explicit control over the capabilities normally required for sound input and output, in a framework that promotes extensibility and flexibility. <a name="111814" id="111814"></a>

<a name="111816" id="111816"></a> The Java Sound API fulfills the needs of a wide range of application developers. Potential application areas include:

- <a name="111817" id="111817"></a>Communication frameworks, such as conferencing and telephony
- <a name="111818" id="111818"></a>End-user content delivery systems, such as media players and music using streamed content
- <a name="111819" id="111819"></a>Interactive application programs, such as games and Web sites that use dynamic content
- <a name="111820" id="111820"></a>Content creation and editing
- <a name="111821" id="111821"></a>Tools, toolkits, and utilities

<a name="111823" id="111823"></a> <!--
<h2> How Does the Java Sound API Relate to Other Interfaces? </h2>
-->

<a name="111825" id="111825"></a> The Java Sound API provides the lowest level of sound support on the Java platform. It provides application programs with a great amount of control over sound operations, and it is extensible. For example, the Java Sound API supplies mechanisms for installing, accessing, and manipulating system resources such as audio mixers, MIDI synthesizers, other audio or MIDI devices, file readers and writers, and sound format converters. The Java Sound API does not include sophisticated sound editors or graphical tools, but it provides capabilities upon which such programs can be built. It emphasizes low-level control beyond that commonly expected by the end user.

<a name="111827" id="111827"></a>

The Java Sound API includes support for both digital audio and MIDI data. These two major modules of functionality are provided in separate packages:

<li><a name="111834" id="111834"></a> 
[`javax.sound.sampled`](https://docs.oracle.com/javase/8/docs/api/javax/sound/sampled/package-summary.html) &#8211; This package specifies interfaces for capture, mixing, and playback of digital (sampled) audio.</li>
<li><a name="111837" id="111837"></a> 
[`javax.sound.midi`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/package-summary.html) &#8211; This package provides interfaces for MIDI synthesis, sequencing, and event transport.</li>

Two other packages permit service providers (as opposed to application developers) to create custom software components that extend the capabilities of an implementation of the Java Sound API: <a name="111842" id="111842"></a>

<li>
[`javax.sound.sampled.spi`](https://docs.oracle.com/javase/8/docs/api/javax/sound/sampled/spi/package-summary.html)</li>
<li><a name="111843" id="111843"></a> 
[`javax.sound.midi.spi`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/spi/package-summary.html)<a name="111845" id="111845"></a>
</li>

This page introduces the sampled-audio system, the MIDI system, and the SPI packages. Each package is then discussed in greater detail later in the tutorial. <a name="111847" id="111847"></a>

There are other Java platform APIs that also have sound-related elements. The 
[Java Media Framework API (JMF)](http://www.oracle.com/technetwork/java/javase/tech/index-jsp-140239.html) is a higher-level API that is currently available as a Standard Extension to the Java platform. JMF specifies a unified architecture, messaging protocol, and programming interface for capturing and playing back time-based media. JMF provides a simpler solution for basic media-player application programs, and it enables synchronization between different media types, such as audio and video. On the other hand, programs that focus on sound can benefit from the Java Sound API, especially if they require more advanced features, such as the ability to carefully control buffered audio playback or directly manipulate a MIDI synthesizer. Other Java APIs with sound aspects include Java 3D and APIs for telephony and speech. An implementation of any of these APIs might use an implementation of the Java Sound API internally, but is not required to do so.

## What is Sampled Audio?

<a name="112308" id="112308"></a> The 
[`javax.sound.sampled`](https://docs.oracle.com/javase/8/docs/api/javax/sound/sampled/package-summary.html) package handles digital audio data, which the Java Sound API refers to as sampled audio. **Samples** are successive snapshots of a signal. In the case of audio, the signal is a sound wave. A microphone converts the acoustic signal into a corresponding analog electrical signal, and an analog-to-digital converter transforms that analog signal into a sampled digital form. The following figure shows a brief moment in a sound recording.

A Sampled Sound Wave

This graph plots sound pressure (amplitude) on the vertical axis, and time on the horizontal axis. The amplitude of the analog sound wave is measured periodically at a certain rate, resulting in the discrete samples (the red data points in the figure) that comprise the digital audio signal. The center horizontal line indicates zero amplitude; points above the line are positive-valued samples, and points below are negative. The accuracy of the digital approximation of the analog signal depends on its resolution in time (the **sampling rate**) and its **quantization**, or resolution in amplitude (the number of bits used to represent each sample). As a point of reference, the audio recorded for storage on compact discs is sampled 44,100 times per second and represented with 16 bits per sample.

<a name="115938" id="115938"></a> The term "sampled audio" is used here slightly loosely. A sound wave could be sampled at discrete intervals while being left in an analog form. For purposes of the Java Sound API, however, "sampled audio" is equivalent to "digital audio."

<a name="115936" id="115936"></a> Typically, sampled audio on a computer comes from a sound recording, but the sound could instead be synthetically generated (for example, to create the sounds of a touch-tone telephone). The term "sampled audio" refers to the type of data, not its origin.

<a name="111869" id="111869"></a> The Java Sound API does not assume a specific audio hardware configuration; it is designed to allow different sorts of audio components to be installed on a system and accessed by the API. The Java Sound API supports common functionality such as input and output from a sound card (for example, for recording and playback of sound files) as well as mixing of multiple streams of audio. Here is one example of a typical audio architecture:

A Typical Audio Architecture

In this example, a device such as a sound card has various input and output ports, and mixing is provided in the software. The mixer might receive data that has been read from a file, streamed from a network, generated on the fly by an application program, or produced by a MIDI synthesizer. The mixer combines all its audio inputs into a single stream, which can be sent to an output device for rendering.

## What is MIDI?

<a name="111878" id="111878"></a> The 
[javax.sound.midi](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/package-summary.html) package contains APIs for transporting and sequencing MIDI events, and for synthesizing sound from those events.

<a name="111880" id="111880"></a> Whereas sampled audio is a direct representation of a sound itself, *MIDI data* can be thought of as a recipe for creating a sound, especially a musical sound. MIDI data, unlike audio data, does not describe sound directly. Instead, it describes events that affect the sounds (or actions) performed by a MIDI-enabled device or instrument, such as a synthesizer. MIDI data is analogous to a graphical user interface's keyboard and mouse events. In the case of MIDI, the events can be thought of as actions upon a musical keyboard, along with actions on various pedals, sliders, switches, and knobs on that musical instrument. These events need not actually originate with a hardware musical instrument; they can be simulated in software, and they can be stored in MIDI files. A program that can create, edit, and perform these files is called a sequencer. Many computer sound cards include MIDI-controllable music synthesizer chips to which sequencers can send their MIDI events. Synthesizers can also be implemented entirely in software. The synthesizers interpret the MIDI events that they receive and produce audio output. Usually the sound synthesized from MIDI data is musical sound (as opposed to speech, for example). MIDI synthesizers are also capable of generating various kinds of sound effects.

<a name="111884" id="111884"></a> Some sound cards include MIDI input and output ports to which external MIDI hardware devices (such as keyboard synthesizers or other instruments) can be connected. From a MIDI input port, an application program can receive events generated by an external MIDI-equipped musical instrument. The program might play the musical performance using the computer's internal synthesizer, save it to disk as a MIDI file, or render it into musical notation. A program might use a MIDI output port to play an external instrument, or to control other external devices such as recording equipment.

The following diagram illustrates the functional relationships between the major components in a possible MIDI configuration based on the Java Sound API. (As with audio, the Java Sound API permits a variety of MIDI software devices to be installed and interconnected. The system shown here is just one potential scenario.) The flow of data between components is indicated by arrows. The data can be in a standard file format, or (as indicated by the key in the lower right corner of the diagram), it can be audio, raw MIDI bytes, or time-tagged MIDI messages.

A Possible MIDI Configuration

In this example, the application program prepares a musical performance by loading a musical score that's stored as a standard MIDI file on a disk (left side of the diagram). Standard MIDI files contain tracks, each of which is a list of time-tagged MIDI events. Most of the events represent musical notes (pitches and rhythms). This MIDI file is read and then "performed" by a software sequencer. A sequencer performs its music by sending MIDI messages to some other device, such as an internal or external synthesizer. The synthesizer itself may read a soundbank file containing instructions for emulating the sounds of certain musical instruments. If not, the synthesizer will play the notes stored in the MIDI file using whatever instrument sounds are already loaded into it.

<a name="111898" id="111898"></a> As illustrated, the MIDI events must be translated into raw (non-time-tagged) MIDI before being sent through a MIDI output port to an external MIDI instrument. Similarly, raw MIDI data coming into the computer from an external MIDI source (a keyboard instrument, in the diagram) is translated into time-tagged MIDI messages that can control a synthesizer, or that a sequencer can store for later use.

## Service Provider Interfaces

<a name="111902" id="111902"></a> The 
[`javax.sound.sampled.spi`](https://docs.oracle.com/javase/8/docs/api/javax/sound/sampled/spi/package-summary.html) and 
[`javax.sound.midi.spi`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/spi/package-summary.html) packages contain APIs that let software developers create new audio or MIDI resources that can be provided separately to the user and "plugged in" to an existing implementation of the Java Sound API. Here are some examples of services (resources) that can be added in this way:

- <a name="111903" id="111903"></a>An audio mixer
- <a name="112110" id="112110"></a>A MIDI synthesizer
- <a name="111906" id="111906"></a>A file parser that can read or write a new type of audio or MIDI file
- <a name="111907" id="111907"></a>A converter that translates between different sound data formats

<a name="111910" id="111910"></a> In some cases, services are software interfaces to the capabilities of hardware devices, such as sound cards, and the service provider might be the same as the vendor of the hardware. In other cases, the services exist purely in software. For example, a synthesizer or a mixer could be an interface to a chip on a sound card, or it could be implemented without any hardware support at all.

<a name="111912" id="111912"></a> An implementation of the Java Sound API contains a basic set of services, but the service provider interface (SPI) packages allow third parties to create new services. These third-party services are integrated into the system in the same way as the built-in services. The 
[`AudioSystem`](https://docs.oracle.com/javase/8/docs/api/javax/sound/sampled/AudioSystem.html) class and the 
[`MidiSystem`](https://docs.oracle.com/javase/8/docs/api/javax/sound/midi/MidiSystem.html) class act as coordinators that let application programs access the services explicitly or implicitly. Often the existence of a service is completely transparent to an application program that uses it. The service-provider mechanism benefits users of application programs based on the Java Sound API, because new sound features can be added to a program without requiring a new release of the JDK or runtime environment, and, in many cases, without even requiring a new release of the application program itself.
