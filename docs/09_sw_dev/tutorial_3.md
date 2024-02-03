# Building ML Web Apps: Part 2

## Table of Contents

1. Overview
2. Tools
4. ML Microservices
5. Front- and Back-End
6. Final Thoughts
7. Exercises

## 1. Overview

This section focuses on building multiple microservices and combining their usage, in anticipation 
of a larger application via inference graphs.

## 2. Tools

- Demucs
- Sentence Transformers
- transformers
- panns_inference
- nicegui

## 3. ML Microservices

### Music Splitter


```python
from IPython.display import Audio
from pedalboard.io import AudioFile
import demucs.api
import requests
import torch
```


```python
with AudioFile("05mUf9x3V3RIqafuY4H54E.mp3", "r") as f:
    first_song = f.read(f.frames)
    first_sample_rate = f.samplerate
```


```python
first_song.shape
```


```python
separator = demucs.api.Separator(device='cpu')
```


```python
tensong = torch.from_numpy(first_song)
```


```python
original, results = separator.separate_tensor(tensong)
```


```python
for k, v in results.items():
    print(k)
    display(Audio(v.numpy(), rate=first_sample_rate))
    print('-'*50)
```

Let's now create a server.


```python
%%writefile servers/splitter/split_model.py

from mlserver import MLModel
from mlserver.codecs import decode_args
import numpy as np
import demucs.api
import torch

class SongSplitter(MLModel):
    async def load(self):
        self.separator = demucs.api.Separator(device='cpu')

    @decode_args
    async def predict(self, song: np.ndarray) -> np.ndarray:
        tensong = torch.from_numpy(song)
        original, result = self.separator.separate_tensor(tensong)
        return self.post_processor(result)

    def post_processor(self, tensor_dict: dict[torch.Tensor]) -> np.ndarray:
        tensor_dict["instruments"] = tensor_dict["bass"] + tensor_dict["drums"] + tensor_dict["other"]
        del tensor_dict["bass"],  tensor_dict["drums"],  tensor_dict["other"]
        return torch.vstack([tensor_dict['vocals'], tensor_dict['instruments']]).numpy()
```


```python
%%writefile servers/splitter/model-settings.json
{
    "name": "music_splitter",
    "implementation": "split_model.SongSplitter"
}
```


```python
%%writefile servers/splitter/settings.json
{
    "http_port": 5010,
    "grpc_port": 5022,
    "metrics_port": 5035
}
```

In the command line, run the following command to start MLServer.

```bash
mlserver start servers/splitter
```

Now, let's test our model.


```python
endpoint = "http://localhost:5010/v2/models/music_splitter/infer"
```


```python
with AudioFile("6I9VzXrHxO9rA9A5euc8Ak.mp3", "r") as f:
    first_song = f.read(f.frames)
    first_sample_rate = f.samplerate
```


```python
input_request = {
    "inputs": [
        NumpyCodec.encode_input(name="song", payload=first_song).dict()
    ]
}
```


```python
res = requests.post(endpoint, json=input_request)
```


```python
res.json()
```

### Transcriber


```python
from mlserver.codecs import NumpyCodec
from transformers import pipeline
import numpy as np
import librosa
```


```python
pipe = pipeline("automatic-speech-recognition", model="openai/whisper-medium")
```


```python
def pre_process(song: np.ndarray, sample_rate) -> np.ndarray:
    return librosa.resample(
        song[0], orig_sr=sample_rate, target_sr=pipe.feature_extractor.sampling_rate
    )
```


```python
new_first = pre_process(first_song, first_sample_rate)
```


```python
new_first
```


```python
trans = pipe(new_first, max_new_tokens=2000)['text']
trans
```


```python
%%writefile servers/transcriptor/asr_model.py

from mlserver import MLModel
from mlserver.codecs import decode_args
from transformers import pipeline
from typing import List
import librosa
import numpy as np


class ASRServer(MLModel):
    async def load(self):
        self.pipe = pipeline("automatic-speech-recognition", model="openai/whisper-medium")

    @decode_args
    async def predict(self, song: np.ndarray, sample_rate: np.ndarray) -> List[str]:
        resampled_song = self.pre_process(song, sample_rate[0][0])
        return [self.pipe(resampled_song, max_new_tokens=2000)['text']]
    
    def pre_process(self, song: np.ndarray, sample_rate) -> np.ndarray:
        return librosa.resample(
            song[0], orig_sr=sample_rate, target_sr=self.pipe.feature_extractor.sampling_rate
        )
```


```python
endpoint = "http://localhost:5060/v2/models/music_transcriber/infer"
```


```python
input_request = {
    "inputs": [
        NumpyCodec.encode_input(name="song", payload=first_song).dict(),
        NumpyCodec.encode_input(name="sample_rate", payload=np.array([first_sample_rate])).dict(),
    ]
}
```


```python
res = requests.post(endpoint, json=input_request)
```


```python
res.json()
```

Time to create a server 😎

Note that before we wrote a `post_processor` but you can also create pre-processing steps in the same 
fashion.


```python
%%writefile servers/transcriptor/model-settings.json
{
    "name": "music_transcriber",
    "implementation": "asr_model.ASRServer"
}
```


```python
%%writefile servers/transcriptor/settings.json
{
    "http_port": 5060,
    "grpc_port": 5040,
    "metrics_port": 5048
}
```

### Text Embeddings

Text embeddings are numerical representations of words or phrases used in natural language processing tasks. These representations capture semantic relationships between words, enabling algorithms to understand their meanings. By converting words into dense vectors, text embeddings allow machines to process and analyze textual data more efficiently, making them crucial for tasks like language translation, sentiment analysis, and text similarity comparisons.


```python
from sentence_transformers import SentenceTransformer
```


```python
model = SentenceTransformer('all-MiniLM-L6-v2')
```


```python
trans
```


```python
model.encode(trans)
```

## Exercise

1. Create a Python file called `text_embs.py` with a model definition for the sentence transformers model.
2. Create a `settings.json` file with the server configuration.
3. Create a `model-settings.json` file with the details of your model.

Add all three files to the `./servers/test_embeddings/` directory and start your server once 
you finish writing the files.


```python

```


```python

```


```python

```


```python

```


```python
from mlserver.codecs import StringCodec
```


```python
endpoint = "http://localhost:4080/v2/models/text_embedding/infer"
```


```python
input_request = {
    "inputs": [
        StringCodec.encode_input(name="lyrics", payload=[trans], use_bytes=False).dict()
    ]
}
input_request
```


```python
embs = requests.post(endpoint, json=input_request)
```


```python
embs.json()
```

Batch Inference

One nice feature of MLServer that can come in handy at any time is its ability to run a batch inference
jobs with a few parameters. Say we have the following group of results:


```python
%%writefile sample-text.txt
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum is simply dummy text of the printing and typesetting industry."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum has been the industrys standard dummy text ever since the 1500s when an unknown printer took a galley of type and scrambled it to make a type specimen book."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum is simply dummy text of the printing and typesetting industry."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum has been the industrys standard dummy text ever since the 1500s when an unknown printer took a galley of type and scrambled it to make a type specimen book."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum is simply dummy text of the printing and typesetting industry."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum has been the industrys standard dummy text ever since the 1500s when an unknown printer took a galley of type and scrambled it to make a type specimen book."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum is simply dummy text of the printing and typesetting industry."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum has been the industrys standard dummy text ever since the 1500s when an unknown printer took a galley of type and scrambled it to make a type specimen book."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum is simply dummy text of the printing and typesetting industry."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum has been the industrys standard dummy text ever since the 1500s when an unknown printer took a galley of type and scrambled it to make a type specimen book."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum is simply dummy text of the printing and typesetting industry."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum has been the industrys standard dummy text ever since the 1500s when an unknown printer took a galley of type and scrambled it to make a type specimen book."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum is simply dummy text of the printing and typesetting industry."]}]}
{"inputs": [{"name": "lyrics", "shape": [1, 1], "datatype": "BYTES", "parameters": {"content_type": "str"}, "data": ["Lorem Ipsum has been the industrys standard dummy text ever since the 1500s when an unknown printer took a galley of type and scrambled it to make a type specimen book."]}]}
```

If we have the details of our server, we can send requests in parallel using the following command.


```bash
%%bash

mlserver infer -u localhost:4080 -m text_embedding -i sample-text.txt -o text-output.txt --workers 5
```


```python

```

The command above
- sends our file full of requests to the (-u) url `localhost:4080`
- to a (-m) model named text_embedding
- using the `sample-text.txt` file as (-i) input
- and it saves the (-o) result in a file called `text-output.txt`
- all of this happens in parallel with 5 (--)workers

### Audio Embeddings

Audio embeddings are numerical representations of audio signals, often used in machine learning tasks related to sound and music. These embeddings capture the essential characteristics of audio, such as timbre and rhythm, in a condensed form. By converting audio data into embeddings, it becomes easier for algorithms to process, analyze, and compare different sounds, enabling applications like music recommendation, sound recognition, and audio-based machine learning models.

I had fine-tuned a Wave-2-Vec model on the for music genre classification on the [Ludwig Music Dataset (Moods and Subgenres)](https://www.kaggle.com/datasets/jorgeruizdev/ludwig-music-dataset-moods-and-subgenres)


```python
import pandas as pd
import io
```


```python
model = pipeline("audio-classification", model="ramonpzg/wav2musicgenre")
```


```python
df = pd.read_csv('payload.csv')
df.head()
```


```python
random_sample = df.sample(1)
print(random_sample['artist_song'].iloc[0])
Audio(url=random_sample['urls'].iloc[0])
```


```python
phil_collins = df.loc[df['artist'] == 'Phil Collins', 'artist_song']
phil_collins.iloc[0]
```


```python
def download_song(metadata, song_to_get):
    song_url = metadata.loc[metadata['artist_song'] == song_to_get, 'urls'].iloc[0]
    with requests.get(song_url, stream=True) as music:
        fil = io.BytesIO(music.content)
        with AudioFile(fil, "r") as f:
            song = f.read(f.frames)
            sample_rate = f.samplerate
    return song, sample_rate
```


```python
phil_song, phil_sr = download_song(df, phil_collins.iloc[0])
```


```python
Audio(phil_song, rate=phil_sr)
```


```python
model(phil_song[0])
```

You can try it our with different songs that you might like best. It is not a perfect model but it 
works relatively well for this example.

Time to build our server! 😎


```python
%%writefile servers/music_cls/music_cls.py

from mlserver import MLModel
from mlserver.codecs import decode_args
from transformers import pipeline
import numpy as np
import pandas as pd

class MusicClassifier(MLModel):
    async def load(self):
        self.model = pipeline("audio-classification", model="ramonpzg/wav2musicgenre")

    @decode_args
    async def predict(self, song: np.ndarray) -> pd.DataFrame:
        result = self.model(song[0])
        return pd.DataFrame(result)
```




```python
%%writefile servers/music_cls/settings.json
{
    "http_port": 5080,
    "grpc_port": 5060,
    "metrics_port": 5070
}
```


```python
%%writefile servers/music_cls/model-settings.json
{
    "name": "music_classifier",
    "implementation": "music_cls.MusicClassifier"
}
```


```python
endpoint = "http://localhost:5080/v2/models/music_classifier/infer"
```

We want to send a single channel to our song so we'll reshape our array into (sample, song_array) 
in our request.


```python
phil_song.shape, phil_song[0][None].shape
```


```python
input_request = {
    "inputs": [
        NumpyCodec.encode_input(name="song", payload=phil_song[0][None]).dict(),
    ]
}
```


```python
audio_embs = requests.post(endpoint, json=input_request)
```


```python
audio_embs.json()
```

### Sentiment Classification

Lastly, we'll create a server to classify the emotions of the transcribed songs.

We will be use the [Roberta Base Go Emotions](https://huggingface.co/SamLowe/roberta-base-go_emotions).


```python
sentiment_pipeline = pipeline(task="text-classification", model="SamLowe/roberta-base-go_emotions", top_k=None)
```


```python
pd.DataFrame(sentiment_pipeline(trans)[0])
```


```python
%%writefile servers/sentiment/emotions.py

from mlserver import MLModel
from mlserver.codecs import decode_args
from transformers import pipeline
from typing import List
import pandas as pd

class EmotionClassifier(MLModel):
    async def load(self):
        self.model = pipeline(task="text-classification", model="SamLowe/roberta-base-go_emotions", top_k=None)

    @decode_args
    async def predict(self, lyrics: List[str]) -> pd.DataFrame:
        result = self.model(lyrics)
        return pd.DataFrame(result[0])
```


```python
%%writefile servers/sentiment/model-settings.json
{
    "name": "sentiformer",
    "implementation": "emotions.EmotionClassifier"
}
```


```python
%%writefile servers/sentiment/settings.json
{
    "http_port": 5010,
    "grpc_port": 5020,
    "metrics_port": 5018
}
```


```python
endpoint = "http://localhost:5010/v2/models/sentiformer/infer"
```


```python
input_request = {
    "inputs": [
        StringCodec.encode_input(name="lyrics", payload=[trans], use_bytes=False).dict()
    ]
}
input_request
```


```python
sentiment = requests.post(endpoint, json=input_request)
sentiment.json()
```

## 5. Front-End

We will walk through the file called `main.py` in the current directory. It contains a `nicegui` 
application with a few moving parts. 

## 7. Exercises

Create a function to showcase any of the audio transcription of a song.
You will need, a widget and a callback to send requests to the server.
