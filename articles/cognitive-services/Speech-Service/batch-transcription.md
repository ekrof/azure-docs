---
title: How to use batch transcription - Speech service
titleSuffix: Azure Cognitive Services
description: Batch transcription is ideal if you want to transcribe a large quantity of audio in storage, such as Azure blobs. By using the dedicated REST API, you can point to audio files with a shared access signature (SAS) URI, and asynchronously receive transcriptions.
services: cognitive-services
manager: nitinme
author: eric-urban
ms.author: eur
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: how-to
ms.date: 01/23/2022
ms.devlang: csharp
ms.custom: devx-track-csharp
---

# How to use batch transcription

Batch transcription is a set of REST API operations that enables you to transcribe a large amount of audio in storage. You can point to audio files by using a typical URI or a [shared access signature (SAS)](../../storage/common/storage-sas-overview.md) URI, and asynchronously receive transcription results. With the v3.0 API, you can transcribe one or more audio files, or process a whole storage container.

You can use batch transcription REST APIs to call the following methods:

|    Batch transcription operation                                             |    Method    |    REST API call                                   |
|------------------------------------------------------------------------------|--------------|----------------------------------------------------|
|    Creates a new transcription.                                              |    POST      |    speechtotext/v3.0/transcriptions            |
|    Retrieves a list of transcriptions for the authenticated subscription.    |    GET       |    speechtotext/v3.0/transcriptions            |
|    Gets a list of supported locales for offline transcriptions.              |    GET       |    speechtotext/v3.0/transcriptions/locales    |
|    Updates the mutable details of the transcription identified by its ID.    |    PATCH     |    speechtotext/v3.0/transcriptions/{id}       |
|    Deletes the specified transcription task.                                 |    DELETE    |    speechtotext/v3.0/transcriptions/{id}       |
|    Gets the transcription identified by the specified ID.                        |    GET       |    speechtotext/v3.0/transcriptions/{id}       |
|    Gets the result files of the transcription identified by the specified ID.    |    GET       |    speechtotext/v3.0/transcriptions/{id}/files |

For more information, see the [Speech-to-text REST API v3.0](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) reference documentation.

Batch transcription jobs are scheduled on a best-effort basis. You can't estimate when a job will change into the running state, but it should happen within minutes under normal system load. When the job is in the running state, the transcription occurs faster than the audio runtime playback speed.

## Prerequisites

As with all features of the Speech service, you create a Speech resource from the [Azure portal](https://portal.azure.com).

>[!NOTE]
> To use batch transcription, you need a standard Speech resource (S0) in your subscription. Free resources (F0) aren't supported. For more information, see [pricing and limits](https://azure.microsoft.com/pricing/details/cognitive-services/speech-services/).

If you plan to customize models, follow the steps in [Acoustic customization](./how-to-custom-speech-train-model.md) and [Language customization](./how-to-custom-speech-train-model.md). To use the created models in batch transcription, you need their model location. You can retrieve the model location when you inspect the details of the model (the `self` property). A deployed custom endpoint isn't needed for the batch transcription service.

>[!NOTE]
> As a part of the REST API, batch transcription has a set of [quotas and limits](speech-services-quotas-and-limits.md#batch-transcription). It's a good idea to review these. To take full advantage of the ability to efficiently transcribe a large number of audio files, send multiple files per request or point to an Azure Blob Storage container with the audio files to transcribe. The service transcribes the files concurrently, which reduces the turnaround time. For more information, see the [Configuration](#configuration) section of this article.

## Batch transcription API

The batch transcription API supports the following formats:

| Format | Codec | Bits per sample | Sample rate             |
|--------|-------|---------|---------------------------------|
| WAV    | PCM   | 16-bit  | 8 kHz or 16 kHz, mono or stereo |
| MP3    | PCM   | 16-bit  | 8 kHz or 16 kHz, mono or stereo |
| OGG    | OPUS  | 16-bit  | 8 kHz or 16 kHz, mono or stereo |

For stereo audio streams, the left and right channels are split during the transcription. A JSON result file is created for each channel. To create an ordered final transcript, use the timestamps that are generated per utterance.

### Configuration

Configuration parameters are provided as JSON. You can transcribe one or more individual files, process a whole storage container, and use a custom trained model in a batch transcription.

If you have more than one file to transcribe, it's a good idea to send multiple files in one request. The following example uses three files:

```json
{
  "contentUrls": [
    "<URL to an audio file 1 to transcribe>",
    "<URL to an audio file 2 to transcribe>",
    "<URL to an audio file 3 to transcribe>"
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

To process a whole storage container, you can make the following configurations. Container [SAS](../../storage/common/storage-sas-overview.md) should contain `r` (read) and `l` (list) permissions:

```json
{
  "contentContainerUrl": "<SAS URL to the Azure blob container to transcribe>",
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "displayName": "Transcription of container using default model for en-US"
}
```

Here's an example of using a custom trained model in a batch transcription. This example uses three files:

```json
{
  "contentUrls": [
    "<URL to an audio file 1 to transcribe>",
    "<URL to an audio file 2 to transcribe>",
    "<URL to an audio file 3 to transcribe>"
  ],
  "properties": {
    "wordLevelTimestampsEnabled": true
  },
  "locale": "en-US",
  "model": {
    "self": "https://westus.api.cognitive.microsoft.com/speechtotext/v3.0/models/{id}"
  },
  "displayName": "Transcription of file using default model for en-US"
}
```

### Configuration properties

Use these optional properties to configure transcription:

:::row:::
   :::column span="1":::
      **Parameter**
   :::column-end:::
   :::column span="2":::
      **Description**
:::row-end:::
:::row:::
   :::column span="1":::
      `profanityFilterMode`
   :::column-end:::
   :::column span="2":::
      Optional, defaults to `Masked`. Specifies how to handle profanity in recognition results. Accepted values are `None` to disable profanity filtering, `Masked` to replace profanity with asterisks, `Removed` to remove all profanity from the result, or `Tags` to add profanity tags.
:::row-end:::
:::row:::
   :::column span="1":::
      `punctuationMode`
   :::column-end:::
   :::column span="2":::
      Optional, defaults to `DictatedAndAutomatic`. Specifies how to handle punctuation in recognition results. Accepted values are `None` to disable punctuation, `Dictated` to imply explicit (spoken) punctuation, `Automatic` to let the decoder deal with punctuation, or `DictatedAndAutomatic` to use dictated and automatic punctuation.
:::row-end:::
:::row:::
   :::column span="1":::
      `wordLevelTimestampsEnabled`
   :::column-end:::
   :::column span="2":::
      Optional, `false` by default. Specifies if word level timestamps should be added to the output.
:::row-end:::
:::row:::
   :::column span="1":::
      `diarizationEnabled`
   :::column-end:::
   :::column span="2":::
      Optional, `false` by default. Specifies that diarization analysis should be carried out on the input, which is expected to be a mono channel that contains two voices. Requires `wordLevelTimestampsEnabled` to be set to `true`.
:::row-end:::
:::row:::
   :::column span="1":::
      `channels`
   :::column-end:::
   :::column span="2":::
      Optional, `0` and `1` transcribed by default. An array of channel numbers to process. Here, a subset of the available channels in the audio file can be specified to be processed (for example `0` only).
:::row-end:::
:::row:::
   :::column span="1":::
      `timeToLive`
   :::column-end:::
   :::column span="2":::
      Optional, no deletion by default. A duration to automatically delete transcriptions after completing the transcription. The `timeToLive` is useful in mass processing transcriptions to ensure they will be eventually deleted (for example, `PT12H` for 12 hours).
:::row-end:::
:::row:::
   :::column span="1":::
      `destinationContainerUrl`
   :::column-end:::
   :::column span="2":::
      Optional URL with [ad hoc SAS](../../storage/common/storage-sas-overview.md) to a writeable container in Azure. The result is stored in this container. SAS with stored access policies isn't supported. If you don't specify a container, Microsoft stores the results in a storage container managed by Microsoft. When the transcription is deleted by calling [Delete transcription](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription), the result data is also deleted.
:::row-end:::

### Storage

Batch transcription can read audio from a public-visible internet URI,
and can read audio or write transcriptions by using a SAS URI with [Blob Storage](../../storage/blobs/storage-blobs-overview.md).

## Batch transcription result

For each audio input, one transcription result file is created. The [Get transcriptions files](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptionFiles) operation returns a list of result files for this transcription. The only way to confirm the audio input for a transcription, is to check the `source` field in the transcription result file. 

Each transcription result file has this format:

```json
{
  "source": "...",                      // sas url of a given contentUrl or the path relative to the root of a given container
  "timestamp": "2020-06-16T09:30:21Z",  // creation time of the transcription, ISO 8601 encoded timestamp, combined date and time
  "durationInTicks": 41200000,          // total audio duration in ticks (1 tick is 100 nanoseconds)
  "duration": "PT4.12S",                // total audio duration, ISO 8601 encoded duration
  "combinedRecognizedPhrases": [        // concatenated results for simple access in single string for each channel
    {
      "channel": 0,                     // channel number of the concatenated results
      "lexical": "hello world",
      "itn": "hello world",
      "maskedITN": "hello world",
      "display": "Hello world."
    }
  ],
  "recognizedPhrases": [                // results for each phrase and each channel individually
    {
      "recognitionStatus": "Success",   // recognition state, e.g. "Success", "Failure"
      "speaker": 1,                     // if `diarizationEnabled` is `true`, this is the identified speaker (1 or 2), otherwise this property is not present
      "channel": 0,                     // channel number of the result
      "offset": "PT0.07S",              // offset in audio of this phrase, ISO 8601 encoded duration
      "duration": "PT1.59S",            // audio duration of this phrase, ISO 8601 encoded duration
      "offsetInTicks": 700000.0,        // offset in audio of this phrase in ticks (1 tick is 100 nanoseconds)
      "durationInTicks": 15900000.0,    // audio duration of this phrase in ticks (1 tick is 100 nanoseconds)

      // possible transcriptions of the current phrase with confidences
      "nBest": [
        {
          "confidence": 0.898652852,    // confidence value for the recognition of the whole phrase
          "lexical": "hello world",
          "itn": "hello world",
          "maskedITN": "hello world",
          "display": "Hello world.",

          // if wordLevelTimestampsEnabled is `true`, there will be a result for each word of the phrase, otherwise this property is not present
          "words": [
            {
              "word": "hello",
              "offset": "PT0.09S",
              "duration": "PT0.48S",
              "offsetInTicks": 900000.0,
              "durationInTicks": 4800000.0,
              "confidence": 0.987572
            },
            {
              "word": "world",
              "offset": "PT0.59S",
              "duration": "PT0.16S",
              "offsetInTicks": 5900000.0,
              "durationInTicks": 1600000.0,
              "confidence": 0.906032
            }
          ]
        }
      ]
    }
  ]
}
```

The result contains the following fields:

:::row:::
   :::column span="1":::
      **Field**
   :::column-end:::
   :::column span="2":::
      **Content**
:::row-end:::
:::row:::
   :::column span="1":::
      `lexical`
   :::column-end:::
   :::column span="2":::
      The actual words recognized.
:::row-end:::
:::row:::
   :::column span="1":::
      `itn`
   :::column-end:::
   :::column span="2":::
      The inverse-text-normalized (ITN) form of the recognized text. Abbreviations (for example, "doctor smith" to "dr smith"), phone numbers, and other transformations are applied.
:::row-end:::
:::row:::
   :::column span="1":::
      `maskedITN`
   :::column-end:::
   :::column span="2":::
      The ITN form with profanity masking applied.
:::row-end:::
:::row:::
   :::column span="1":::
      `display`
   :::column-end:::
   :::column span="2":::
      The display form of the recognized text. Added punctuation and capitalization are included.
:::row-end:::

## Speaker separation (diarization)

*Diarization* is the process of separating speakers in a piece of audio. The batch pipeline supports diarization and is capable of recognizing two speakers on mono channel recordings. The feature isn't available on stereo recordings.

The output of transcription with diarization enabled contains a `Speaker` entry for each transcribed phrase. If diarization isn't used, the `Speaker` property isn't present in the JSON output. For diarization, the speakers are identified as `1` or `2`.

To request diarization, set the `diarizationEnabled` property to `true`. Here's an example:

```json
{
  "contentUrls": [
    "<URL to an audio file to transcribe>",
  ],
  "properties": {
    "diarizationEnabled": true,
    "wordLevelTimestampsEnabled": true,
    "punctuationMode": "DictatedAndAutomatic",
    "profanityFilterMode": "Masked"
  },
  "locale": "en-US",
  "displayName": "Transcription of file using default model for en-US"
}
```

Word-level timestamps must be enabled, as the parameters in this request indicate.

## Best practices

The batch transcription service can handle a large number of submitted transcriptions. You can query the status of your transcriptions with [Get transcriptions](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/GetTranscriptions). Call [Delete transcription](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/DeleteTranscription)
regularly from the service, after you retrieve the results. Alternatively, set the `timeToLive` property to ensure the eventual deletion of the results.

> [!TIP]
> You can use the [Ingestion Client](ingestion-client.md) tool and resulting solution to process a high volume of audio.

## Sample code

Complete samples are available in the [GitHub sample repository](https://aka.ms/csspeech/samples), inside the `samples/batch` subdirectory.

Update the sample code with your subscription information, service region, URI pointing to the audio file to transcribe, and model location if you're using a custom model.

[!code-csharp[Configuration variables for batch transcription](~/samples-cognitive-services-speech-sdk/samples/batch/csharp/batchclient/program.cs#transcriptiondefinition)]

The sample code sets up the client and submits the transcription request. It then polls for the status information and prints details about the transcription progress.

```csharp
// get the status of our transcriptions periodically and log results
int completed = 0, running = 0, notStarted = 0;
while (completed < 1)
{
    completed = 0; running = 0; notStarted = 0;

    // get all transcriptions for the user
    paginatedTranscriptions = null;
    do
    {
        // <transcriptionstatus>
        if (paginatedTranscriptions == null)
        {
            paginatedTranscriptions = await client.GetTranscriptionsAsync().ConfigureAwait(false);
        }
        else
        {
            paginatedTranscriptions = await client.GetTranscriptionsAsync(paginatedTranscriptions.NextLink).ConfigureAwait(false);
        }

        // delete all pre-existing completed transcriptions. If transcriptions are still running or not started, they will not be deleted
        foreach (var transcription in paginatedTranscriptions.Values)
        {
            switch (transcription.Status)
            {
                case "Failed":
                case "Succeeded":
                    // we check to see if it was one of the transcriptions we created from this client.
                    if (!createdTranscriptions.Contains(transcription.Self))
                    {
                        // not created form here, continue
                        continue;
                    }

                    completed++;

                    // if the transcription was successful, check the results
                    if (transcription.Status == "Succeeded")
                    {
                        var paginatedfiles = await client.GetTranscriptionFilesAsync(transcription.Links.Files).ConfigureAwait(false);

                        var resultFile = paginatedfiles.Values.FirstOrDefault(f => f.Kind == ArtifactKind.Transcription);
                        var result = await client.GetTranscriptionResultAsync(new Uri(resultFile.Links.ContentUrl)).ConfigureAwait(false);
                        Console.WriteLine("Transcription succeeded. Results: ");
                        Console.WriteLine(JsonConvert.SerializeObject(result, SpeechJsonContractResolver.WriterSettings));
                    }
                    else
                    {
                        Console.WriteLine("Transcription failed. Status: {0}", transcription.Properties.Error.Message);
                    }

                    break;

                case "Running":
                    running++;
                    break;

                case "NotStarted":
                    notStarted++;
                    break;
            }
        }

        // for each transcription in the list we check the status
        Console.WriteLine(string.Format("Transcriptions status: {0} completed, {1} running, {2} not started yet", completed, running, notStarted));
    }
    while (paginatedTranscriptions.NextLink != null);

    // </transcriptionstatus>
    // check again after 1 minute
    await Task.Delay(TimeSpan.FromMinutes(1)).ConfigureAwait(false);
}
```

For full details about the preceding calls, see the [Speech-to-text REST API v3.0](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0) reference documentation. For the full sample shown here, go to [GitHub](https://aka.ms/csspeech/samples) in the `samples/batch` subdirectory.

This sample uses an asynchronous setup to post audio and receive transcription status. The `PostTranscriptions` method sends the audio file details, and the `GetTranscriptions` method receives the states. `PostTranscriptions` returns a handle, and `GetTranscriptions` uses it to create a handle to get the transcription status.

This sample code doesn't specify a custom model. The service uses the base model for transcribing the file or files. To specify the model, you can pass on the same method the model reference for the custom model.

> [!NOTE]
> For baseline transcriptions, you don't need to declare the ID for the base model.

## Next steps

> [!div class="nextstepaction"]
> [Speech to text v3.0 API reference](https://centralus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0/operations/CopyModelToSubscription)
