# elevenlabs-go v0.3.X

![Go version](https://img.shields.io/badge/go-1.24-blue)
![License](https://img.shields.io/github/license/WP-Nova-GmbH/elevenlabs-go)

This is a Go client library for the [ElevenLabs](https://elevenlabs.io/) voice cloning and speech synthesis platform. It provides a interface for Go programs to interact with the ElevenLabs [API](https://docs.elevenlabs.io/api-reference).

## About this Fork

This repo is a fork of the original [elevenlabs-go](github.com/haguro/elevenlabs-go) and integrates the updates from [Mlivui79](https://github.com/Mliviu79/elevenlabs-go)`s fork.

**Why this is needed?:**  
This evolution of elvenlabs-go by WP-Nova-GmbH is primarly a package support takeover because we need some of
the elevenlabs features that aren't supported by the versions before. There isn't a need to reinvent the weel
but a need for up-to-date packages, when integrating TTS/STT in the long run. So here we go!

To make a clear seperation between the package versions they are labeld like this:

- Original: v0.1.X
- Mlivui79`s fork: v0.2.X
- WP-Nova-GmbH: v0.3.X

And here is the feature overview so you can decide what version to use :).

| Feature                 | v0.1.X | v.0.2.X | v0.3.X |
| ----------------------- | :----: | :-----: | :----: |
| TextToSpeech            |  [ ]   |   [x]   |   [ ]  |
| TextToSpeechStream      |  [ ]   |   [x]   |   [ ]  |
| GetModels               |  [x]   |   [ ]   |   [ ]  |
| GetVoices               |  [x]   |   [ ]   |   [ ]  |
| GetDefaultVoiceSettings |  [x]   |   [ ]   |   [ ]  |
| GetVoiceSettings        |  [x]   |   [ ]   |   [ ]  |
| GetVoice                |  [x]   |   [ ]   |   [ ]  |
| DeleteVoice             |  [x]   |   [ ]   |   [ ]  |
| EditVoiceSettings       |  [x]   |   [ ]   |   [ ]  |
| AddVoice                |  [x]   |   [ ]   |   [ ]  |
| EditVoice               |  [x]   |   [ ]   |   [ ]  |
| DeleteSample            |  [x]   |   [ ]   |   [ ]  |
| GetSampleAudio          |  [x]   |   [ ]   |   [ ]  |
| GetHistory              |  [x]   |   [ ]   |   [ ]  |
| GetHistoryItem          |  [x]   |   [ ]   |   [ ]  |
| DeleteHistoryItem       |  [x]   |   [ ]   |   [ ]  |
| GetHistoryItemAudio     |  [x]   |   [ ]   |   [ ]  |
| DownloadHistoryAudio    |  [x]   |   [ ]   |   [ ]  |
| GetSubscription         |  [x]   |   [ ]   |   [ ]  |
| GetUser                 |  [x]   |   [ ]   |   [ ]  |
| SpeechToText            |  [ ]   |   [ ]   |   [x]  |

## Installation

```bash
go get github.com/WP-Nova-GmbH/elevenlabs-go@v0.3.0
```

## Example Usage

Make sure to replace `"your-api-key"` in all examples with your actual API key. Refer to the official Elevenlabs [API documentation](https://docs.elevenlabs.io/api-reference/quick-start/introduction) for further details.

### Using a New Client Instance

Using the `NewClient` function returns a new `Client` instance will allow to pass a parent context, your API key and a timeout duration.

```go
package main

import (
 "context"
 "log"
 "os"
 "time"

 "github.com/WP-Nova-GmbH/elevenlabs-go"
)

func main() {
 // Create a new client
 client := elevenlabs.NewClient(context.Background(), "your-api-key", 30*time.Second)

 // Create a TextToSpeechRequest
 ttsReq := elevenlabs.TextToSpeechRequest{
  Text:    "Hello, world! My name is Adam, nice to meet you!",
  ModelID: "eleven_monolingual_v1",
 }

 // Call the TextToSpeech method on the client, using the "Adam"'s voice ID.
 audio, err := client.TextToSpeech("pNInz6obpgDQGcFmaJgB", ttsReq)
 if err != nil {
  log.Fatal(err)
 }

 // Write the audio file bytes to disk
 if err := os.WriteFile("adam.mp3", audio, 0644); err != nil {
  log.Fatal(err)
 }

 log.Println("Successfully generated audio file")
}
```

### Using the Default Client and proxy functions

The library has a default client you can configure and use with proxy functions that wrap method calls to the default client. The default client has a default timeout set to 30 seconds and is configured with `context.Background()` as the the parent context. You will only need to set your API key at minimum when taking advantage of the default client. Here's the a version of the above example above using shorthand functions only.

```go
package main

import (
 "log"
 "os"
 "time"

 el "github.com/WP-Nova-GmbH/elevenlabs-go"
)

func main() {
 // Set your API key
 el.SetAPIKey("your-api-key")

 // Set a different timeout (optional)
 el.SetTimeout(15 * time.Second)

 // Call the TextToSpeech method on the client, using the "Adam"'s voice ID.
 audio, err := el.TextToSpeech("pNInz6obpgDQGcFmaJgB", el.TextToSpeechRequest{
   Text:    "Hello, world! My name is Adam, nice to meet you!",
   ModelID: "eleven_monolingual_v1",
  })
 if err != nil {
  log.Fatal(err)
 }

 // Write the audio file bytes to disk
 if err := os.WriteFile("adam.mp3", audio, 0644); err != nil {
  log.Fatal(err)
 }

 log.Println("Successfully generated audio file")
}
```

### Streaming

The Elevenlabs API allows streaming of audio "as it is being generated". In elevenlabs-go, you'll want to pass an `io.Writer` to the `TextToSpeechStream` method where the stream will be continuously copied to. _Note that you will need to set the client timeout to a high enough value to ensure that request does not time out mid-stream_.

```go
package main

import (
 "context"
 "log"
 "os/exec"
 "time"

 "github.com/WP-Nova-GmbH/elevenlabs-go"
)

func main() {
 message := `The concept of "flushing" typically applies to I/O buffers in many programming 
languages, which store data temporarily in memory before writing it to a more permanent location
like a file or a network connection. Flushing the buffer means writing all the buffered data
immediately, even if the buffer isn't full.`

 // Set your API key
 elevenlabs.SetAPIKey("your-api-key")

 // Set a large enough timeout to ensure the stream is not interrupted.
 elevenlabs.SetTimeout(1 * time.Minute)

 // We'll use mpv to play the audio from the stream piped to standard input
 cmd := exec.CommandContext(context.Background(), "mpv", "--no-cache", "--no-terminal", "--", "fd://0")

 // Get a pipe connected to the mpv's standard input
 pipe, err := cmd.StdinPipe()
 if err != nil {
  log.Fatal(err)
 }

 // Attempt to run the command in a separate process
 if err := cmd.Start(); err != nil {
  log.Fatal(err)
 }

 // Stream the audio to the pipe connected to mpv's standard input
 if err := elevenlabs.TextToSpeechStream(
  pipe,
  "pNInz6obpgDQGcFmaJgB",
  elevenlabs.TextToSpeechRequest{
   Text:    message,
   ModelID: "eleven_multilingual_v1",
  }); err != nil {
  log.Fatalf("Got %T error: %q\n", err, err)
 }

 // Close the pipe when all stream has been copied to the pipe
 if err := pipe.Close(); err != nil {
  log.Fatalf("Could not close pipe: %s", err)
 }
 log.Print("Streaming finished.")

 // Wait for mpv to exit. With the pipe closed, it will do that as
 // soon as it finishes playing
 if err := cmd.Wait(); err != nil {
  log.Fatal(err)
 }

 log.Print("All done.")
}
```

## TextToSpeechRequest Struct

The `TextToSpeechRequest` struct has been updated to match the latest ElevenLabs API:

```go
type TextToSpeechRequest struct {
	Text                            string                           `json:"text"`
	ModelID                         string                           `json:"model_id,omitempty"`
	LanguageCode                    string                           `json:"language_code,omitempty"`
	PronunciationDictionaryLocators []PronunciationDictionaryLocator `json:"pronunciation_dictionary_locators,omitempty"`
	Seed                            int                              `json:"seed,omitempty"`
	PreviousText                    string                           `json:"previous_text,omitempty"`
	NextText                        string                           `json:"next_text,omitempty"`
	PreviousRequestIds              []string                         `json:"previous_request_ids,omitempty"`
	NextRequestIds                  []string                         `json:"next_request_ids,omitempty"`
	ApplyTextNormalization          bool                             `json:"apply_text_normalization,omitempty"`
	ApplyLanguageTextNormalization  bool                             `json:"apply_language_text_normalization,omitempty"`
	VoiceSettings                   *VoiceSettings                   `json:"voice_settings,omitempty"`
}
```

## Text-to-Speech with Timestamps

This fork includes support for the new ElevenLabs API endpoint that provides character-level timestamps along with the generated audio. The response contains:

- `audio_base64`: Base64 encoded audio data
- `alignment`: Timestamp information for each character in the original text
- `normalized_alignment`: Timestamp information for each character in the normalized text

### Example Usage

```go
package main

import (
 "context"
 "log"
 "os"
 "time"

 "github.com/Mliviu79/elevenlabs-go"
)

func main() {
 // Create a new client
 client := elevenlabs.NewClient(context.Background(), "your-api-key", 30*time.Second)

 // Create a TextToSpeechRequest
 ttsReq := elevenlabs.TextToSpeechRequest{
  Text:    "Hello, world! This is a test with timestamps.",
  ModelID: "eleven_monolingual_v1",
 }

 // Create a file to write the JSON response containing audio and timestamps
 file, err := os.Create("response_with_timestamps.json")
 if err != nil {
  log.Fatal(err)
 }
 defer file.Close()

 // Call the TextToSpeechWithTimestamps method, streaming the JSON response to the file
 err = client.TextToSpeechWithTimestamps(file, "pNInz6obpgDQGcFmaJgB", ttsReq)
 if err != nil {
  log.Fatal(err)
 }

 log.Println("Successfully generated audio with timestamps")
}
```

The response structure includes:

```go
type TextToSpeechWithTimestampsResponse struct {
	AudioBase64         string        `json:"audio_base64"`
	Alignment           AlignmentInfo `json:"alignment"`
	NormalizedAlignment AlignmentInfo `json:"normalized_alignment"`
}

type AlignmentInfo struct {
	Characters                 []string  `json:"characters"`
	CharacterStartTimesSeconds []float64 `json:"character_start_times_seconds"`
	CharacterEndTimesSeconds   []float64 `json:"character_end_times_seconds"`
}
```

## Status and Future Plans

This library is an evolution of the original [elevenlabs-go](github.com/haguro/elevenlabs-go) library. This project is not designed to be a complete coverage of the elevenlabs API but more a project to cover the needs for
our (WP-Nova GmbH) AI platfrom: [chat.wp-nova.ai](chat.wp-nova.ai).

Four our AI speech features we use the elevenlabs services so I guess if you want to do the typical "text"/"speech" stuff and don't need special neachy elevenlabs features this lib is what you want.

## Contributing

Contributions are welcome! If you have any ideas, improvements, or bug fixes, please open an issue or submit a pull request.

## Looking for a Python library?

The Elevenlabs's official [Python library](https://github.com/elevenlabs/elevenlabs-python) is excellent and fellow Pythonistas are encouraged to use it (and also to give Go, a [go](https://gobyexample.com/) 😉🩵)!

## Disclaimer

This is an independent project and is not affiliated with or endorsed by Elevenlabs. Elevenlabs and its trademarks are the property of their respective owners. The purpose of this project is to provide a client library to facilitate access to the public API provided Elevenlabs within Go programs. Any use of Elevenlabs's trademarks within this project is for identification purposes only and does not imply endorsement, sponsorship, or affiliation.

## License

This project is licensed under the [MIT License](LICENSE).

## Test Data

The audio files in this directory are from the Mozilla Common Voice dataset.

- **Source**: [Mozilla Common Voice](httpss://commonvoice.mozilla.org/)
- **License**: [Creative Commons CC0](httpss://creativecommons.org/publicdomain/zero/1.0/) (Public Domain)

The files have been included here to be used for integration tests for this library.

## Warranty

This code library is provided "as is" and without any warranties whatsoever. Use at your own risk. More details in the [LICENSE](LICENSE) file.
