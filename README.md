Ispikit Unity Plugin
====================

### This is version 1.0 of the Ispikit Unity plugin

This is the documentation of the Ispikit Unity plugin, it brings speech recognition and pronunciation assessment to Unity applications.

This version is free to use and includes two limitations compared to the full version:

* Number of sentences for recognition is limited to 3
* Number of words per sentence is limited to 4

contact us at info@ispikit.com for the full version.

## 1. Features

* Audio recording: records user's voice through internal or external microphone.
* Speech recognition: recognizes what user said among several possible inputs. Recognized words are available in real time.
* Pronunciation assessment: returns overall pronunciation score.
* Playback of user's input: recorded voice can be played back.
* Mispronounced words detection: detects and flags words that have been mispronounced.
* Audio volume: During recording, audio volume callbacks can be used to display the audio input level.
* Waveform: during recording, waveform callbacks can be used to draw and display the recorded audio.
* Pitch tracking: during recording, pitch callbacks can be used to plot user's pitch contour (intonation).
* Local-only: everything happens locally, no network call made.

Note: this build includes binaries for all iOS device architectures. It will not work in the simulator or other platforms than iOS.

## 2. Content

This package includes:

* `README.md`: this file.
* `LICENSE`
* `upalbundle.bundle/`: contains resources necessary for the plugin to work.
* `iOS/`: contains the binaries for iOS (all ARM architectures).
* `Ispikit.cs`: Small boilerplate file to load and showcase the plugin's features.

## 3. Starting with the Ispikit Unity plugin

You can use the provided `Ispikit.cs` file to start building on the plugin:

* Add the content of the archive into the `Assets/Plugins` folder of a Unity application (`Ispikit.cs`, the `iOS/` and `upalbundle.bundle/` folders. If the `iOS`/` folder already exists, add the files into the existing folder).
* In the editor, add `Ispikit.cs` into a Game Object named "GameObject".
* Using the plugin inspector, make sure it only builds for iOS.
* In the editor, add the Game Object to a scene.
* Compile and launch the application on an iOS device.
* The application will:
  * Start initialization of the plugin as soon as the Game Object is "awake".
  * Once initialization is done, recording will start and last 3 seconds.
  * During recording, speak one of the expected inputs ("first", "second", "third").
  * After 3 seconds recording stops and analysis starts.
  * During analysis, a completion callback is called that shows what percentage of analysis has been completed.
  * At the end of analysis, the result string is given.
  * After result is displayed, the recorded voice is played back again.
  * When playback is done, it starts another recognition, this time with three other sentences : "I am learning English", "hello goodbye", "one two three four".
  * Then it continues looping through these steps indefinitely.

You can reuse this boilerplate code to build your application using the more detailed docs below. Note that you should use the correct Game Object names when registering callbacks.

We cannot directly access the microphone permissions from unity. As a work around, we recommend adding the following code to your awake function to call the microphone permissions when the app opens the first time.

void awake () {
  AudioSource audPermissions = GetComponent<AudioSource>();
  audPermissions.clip = Microphone.Start("Built-in Microphone", true, 1, 1);
}

## 4. Usage

The available API calls are listed in `iOS/UAnalyzer.h`. They include calls to send instructions to the plugins and calls to register callback functions. Callback are functions that take one argument, a string. They are identified by their method name and the name of the Game Object they are in. The object they belong to should not matter.

In this section, we also document how the strings in callback should be parsed to extract usable information.

### 4.a `int startInitialization(string callbackGameObjectName, string callbackMethodName);`

It starts initialization of the plugin. Typically it takes a dozen of seconds to initialize and once initialization is complete, it will call back `callbackMethodName` in Game Object `callbackGameObjectName`. This function should return 0 if successful, and the callback argument should also be "0", also meaning initialization is successful.

### 4.b `int setPlaybackDoneCallback(string callbackGameObjectName, string callbackMethodName);`

It specifies to the plugin the callback to be called once playback is complete. Its argument is always an empty string.

### 4.c `int setResultCallback(string callbackGameObjectName, string callbackMethodName);`

It specifies to the plugin the callback to be called once the result of analysis is ready. Typically is comes a few seconds after recording is stopped. The argument gives the pronunciation score, the speed measure as well as the list of recognized words and whether they are detected as mispronounced.

These data are encoded in the string, separated by commas:

* Pronunciation score is a number between 0 (worse) and 100 (perfect pronunciation).
* Speed is a measure of how many phonemes have been uttered in 10 seconds.
* Words are given with a bunch of indexes separated with dashes ("-"). The first index is the index of the sentence that was recognized (starting with 0, the first sentence given to the call to `startRecording`). The second index is the word index within the sentence, also starting with 0, and the third number is a flag: 0 if the word was correctly said, 1 if it was mispronounced. Word level mispronunciation detection is not 100% accurate, especially for short words.

For instance, if `startRecording` was called with "one two three,four five,six seven eight" and result is "55,82,2-0-0 2-2-1", it means that the overall pronunciation score is 55 (over a scale of 0 to 100), speed was 82 (phonemes per 10 seconds), and user said "six eight", "eight" being mispronounced.

### 4.d `int setCompletionCallback(string callbackGameObjectName, string callbackMethodName);`

It specifies to the plugin the callback to be called during analysis to give the percentage of completion. The string is a number between "0" and "100", representing the percentage of completion. It can be used to show a progress bar, especially for long inputs, where analysis can take a few seconds.

### 4.e `int setNewWordsCallback(string callbackGameObjectName, string callbackMethodName);`

It specifies to the plugin the callback to be called during recording to show which words have been recognized up to that point. It is a string where each word is given encoded with 4 indexes separated with dashes:

* First index is the index of the sentence.
* Second index is the word index within the sentence.
* Third and fourth indexes are not really meaningful from a user's point of view.

For instance, if `startRecording` was called with "one two three four five,six seven eight nine ten" and callback parameter is "1-0-0-0 1-1-0-0 1-3-0-0", it means that at this moment, user said "six seven nine".

### 4.f `int setNewAudioCallback(string callbackGameObjectName, string callbackMethodName);`

It specifies to the plugin the callback to be called during recording to show various data about the audio recording that can be used to give a nice UI. It contains 3 categories of data, separated by commas, respectively:

* volume: A number between 0 (minimum) and 100 (maximum) that gives the audio volume in the input.
* pitch: A few samples of pitch contour data, separated with spaces. It shows the user's intonation.
* waveformLow and waveformHigh: samples to draw the waveform. The samples are encoded with low points (typically negative values) then high points (typically positive) of the waveform contour.

### 4.g `int startRecording(string sentences);`

This starts audio recording, assuming that plugin was properly initialized before. It return 0 if successful, 1 otherwise (plugin not initialized). As argument, a string with one or several sentences to be recognized. Sentences are comma separated, with no punctuation and just one space between words.

Recording and recognition will start immediately, and last until `stopRecording` is called. During recording, audio and word callbacks will be called regularly.

### 4.h `int stopRecording();`

This stops recording. Returns 0 if successful, 1 otherwise. It also immediately starts analysis during which completion callbacks will be regularly called. Analysis is done when result callback comes.

### 4.i `int startPlayback();`

This starts replaying back the previously recorded audio. It calls back with the `playbackDone` callback that comes when audio completely played back. Audio playback can also be stopped with `stopPlayback`.

### 4.j `int stopPlayback();`

Stops playback immediately

### 4.k `int setStrictness(int strictness);`

If scores are considered too high for the application, strictness can be adjusted. 1 is the default, higher values means stricter scores (i.e. more difficult to reach high scores).

### 4.l `int addWord(string word, string pronunciation);`

All words in sentences to be recognized should be known by the Ispikit plugin. The plugin comes with a large dictionary that contains most English words. It can be viewed in `upalbundle.bundle/libdictionary.so` which is an ASCII text file. However, new words can be added with `addWord`, using the [CMU sphinx pronunciation dictionary](http://www.speech.cs.cmu.edu/cgi-bin/cmudict) syntax.
