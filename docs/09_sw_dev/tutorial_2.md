# Building ML Web Apps: Part 1

## Table of Contents

1. Overview
2. Tools
3. ML Model
5. ML Microservices
    - Music Generator
    - Visualizations
    - Audio Effects
    - Midi Representations
6. Front- and Back-End
7. Exercise

## 1. Overview

Microservices architectures are like the building blocks of many modern applications, including 
those in the music and audio industry. Imagine a big, complex music platform - instead of one 
giant system handling everything, you break it down into smaller, manageable services. Each of these services (or microservices) has a specific job, like managing user profiles, handling payments, or serving song recommendations. This modular approach makes it easier to develop, deploy, and scale applications. In the music world, this translates to more streamlined music streaming services, efficient royalty payment systems, and personalized music recommendation engines.

Now, let's talk about machine learning (ML). ML is like the magic wand in the music industry. It's used to analyze massive amounts of data - like listening patterns, user preferences, and even the acoustic properties of songs. By going through large amounts of numbers, ML algorithms can create nice representations or features that help personalize your playlists and song recommendations.

Programming is the backbone of all these innovations. In the music industry, programming is like the language of creativity. Whether you're building a music app, designing audio effects, or creating virtual instruments, programming skills are crucial. Plus, with the rise of electronic music, coding is becoming an art form itself. Just think about artists who write their own algorithms to generate unique soundscapes or visuals for their live shows. It's mind-blowing!

In the next few sections, we'll dive into different practical examples to bring all of these together.

## 2. Tools

- MLServer
- Gradio
- Transformers
- matplotlib
- pedalboard
- librosa

## 3. ML Model


```python
from transformers import AutoProcessor, MusicgenForConditionalGeneration
from IPython.display import Audio
```


```python
processor = AutoProcessor.from_pretrained("facebook/musicgen-small")
model = MusicgenForConditionalGeneration.from_pretrained("facebook/musicgen-small")
```


```python
inputs = processor(
    text=["classic music with fast tempo, violin sounds, and guitar solo ending"],
    padding=True,
    return_tensors="pt",
)
```


```python
inputs
```


```python
sampling_rate = model.config.audio_encoder.sampling_rate
sampling_rate
```


```python
audio_values = model.generate(**inputs, do_sample=True, guidance_scale=3, max_new_tokens=350)
```


```python
audio_values.shape
```

📈👀 👌


```python
161920 / 32000
```


```python
audio_values[0, 0].numpy().shape
```


```python
audio_values[0].numpy().shape
```


```python
Audio(audio_values[0].numpy(), rate=sampling_rate)
```

We can add even more effects with other libraries like librosa.


```python
import librosa
```


```python
librosa.effects.hpss?
```


```python
y_har, y_per = librosa.effects.hpss(audio_values[0].numpy())
```


```python
Audio(y_har, rate=sampling_rate)
```

There are other libraries that give us further control over the knobs we can tune in our songs.


```python
from pedalboard import Pedalboard, Distortion, Delay, Reverb
from pedalboard.io import AudioFile
```


```python
with AudioFile("celia_juancito.mp3", "r") as f:
    song = f.read(f.frames)
    sr = f.samplerate
song
```


```python
song.shape
```


```python
Audio(song, rate=sr)
```


```python
board = Pedalboard([
    Distortion(drive_db=10),
    Delay(delay_seconds=1, feedback=0.1, mix=0.1),
    Reverb(room_size=0.10)
])
```


```python
sr
```


```python
new_song = board(song, sample_rate=44100)
```


```python
Audio(new_song, rate=sr)
```

To save our creations.

```sh
mkdir music
```


```python
new_song.shape
```


```python
with AudioFile("new_notes.mp3", "w", samplerate=sr, num_channels=1) as f:
    f.write(new_song[0])
```

If we did like that last song a lot, we can also create a new one with the tunes we likes as the base for the new song.


```python
new_song
```


```python
audio_values[0].numpy().shape
```


```python
inputs = processor(
    audio=audio_values[0].numpy()[0],
    sampling_rate=sampling_rate,
    text=["80s blues track with violin notes"],
    padding=True,
    return_tensors="pt",
)
audio_values = model.generate(**inputs, do_sample=True, guidance_scale=3, max_new_tokens=256)
```


```python
Audio(audio_values[0].numpy(), rate=sampling_rate)
```

## 5. ML Microservices

Time to get our service up and running.

### 5.1 MusicGen Server

We'll start with the imports.


```python
%%writefile servers/ml_model/ml_service.py

from mlserver import MLServer, Settings, ModelSettings, MLModel
from mlserver.codecs import decode_args

from transformers import AutoProcessor, MusicgenForConditionalGeneration
import numpy as np

from typing import List
import asyncio
```

Next, we'll append our main class to our server file.


```python
%%writefile -a servers/ml_model/ml_service.py

MUSICGEN = "facebook/musicgen-small"

class MusicGenServer(MLModel):
    async def load(self):
        self.processor = AutoProcessor.from_pretrained(MUSICGEN)
        self.model     = MusicgenForConditionalGeneration.from_pretrained(MUSICGEN)

    @decode_args
    async def predict(self, text: List[str], guidance_scale: np.ndarray, max_new_tokens: np.ndarray) -> np.ndarray:
        inputs       = self.processor(text=text, padding=True, return_tensors="pt")
        audio_values = self.model.generate(
            **inputs, do_sample=True, guidance_scale=guidance_scale[0][0], max_new_tokens=max_new_tokens[0][0]
        ).numpy()
        return audio_values[0]
```

Lastly, we'll need the settings and asyncio to run our app.


```python
%%writefile -a servers/ml_model/ml_service.py

async def main():
    settings = Settings(debug=True)
    my_server = MLServer(settings=settings)
    musicgen_generator = ModelSettings(name='musicgen_model', implementation=MusicGenServer)
    await my_server.start(models_settings=[musicgen_generator])

if __name__ == '__main__':
    loop = asyncio.get_event_loop()
    loop.run_until_complete(main())
```

We can now run our model with:

```sh
python servers/ml_model/ml_service.py
```

By default, MLServer will provide you with the following ports:
- https
- grpc
- metrics

We can test that our server is working as intended with the following requests.


```python
from mlserver.codecs import StringCodec, NumpyCodec
import numpy as np
import requests
```


```python
from rich import print
```


```python
song_text   = StringCodec.encode_input(name='text', payload=['slow bachata with violin sounds'], use_bytes=False)
parameter_1 = NumpyCodec.encode_input(name='guidance_scale', payload=np.array([5]))
parameter_2 = NumpyCodec.encode_input(name='max_new_tokens', payload=np.array([280]))
print(song_text)
```


```python
print(parameter_1.dict())
```

We can combine the as:


```python
input_request = {
    'inputs': [
        StringCodec.encode_input(name='text', payload=['slow bachata with violin sounds'], use_bytes=False).dict(),
        NumpyCodec.encode_input(name='guidance_scale', payload=np.array([5])).dict(),
        NumpyCodec.encode_input(name='max_new_tokens', payload=np.array([200])).dict()
    ]
}
```


```python
print(input_request)
```

Or as:


```python
input_request = {
    'inputs': [song_text.dict(), parameter_1.dict(), parameter_2.dict()]
}
```

and the resul will be the same.

Here is the endpoint we'll use for our request.


```python
endpoint = "http://localhost:8080/v2/models/musicgen_model/infer"
```

If the above seems a bit different than the endpoints of previous sections, that's because 
MLServer uses the Open Inference Protocol (OIP) or V2 Inference Protocol to communicate.

The OIP is a standard created in a effort to standardize the way in which machine learning 
microservices communicate with each other. It has been adopted by companies such as NVIDIA, 
Google, and others, and it is the standard used by tools such as Seldon Core and KServe. To 
learn more about the OIP, see the [OIP website](https://github.com/SeldonIO/seldon-core/blob/master/docs/protocol.md).

Now, back to making predictions.

**Note:** This could take a few seconds yo run.


```python
result = requests.post(endpoint, json=input_request)
result
```

Let's have a look at what the model and server will return to us.


```python
result.json()
```

Time to evaluate the quality of the song.


```python
Audio(result.json()['outputs'][0]['data'], rate=sampling_rate)
```

### Exercise

1. Test the endpoint by sending a few requests to it with your best description of a song.
2. Look at the part of our model that's making the inference and pick a parameter that has 
not been added to the model yet.
3. Stop the server, add the parameter to the model, and start the server again.
4. Try out the same songs you tested before against the new parameters of your model.


```python

```


```python

```

## 5.2 Visualizations

There are many ways to visualize sound, and, for our purposes, we will create a waveform and 
a spectogram for each of our newest songs.

**Waveform**
> A waveform plot is a visual representation of sound waves over time. Imagine you're looking at a graph where the horizontal axis shows time, and the vertical axis represents the intensity or amplitude of the sound. It looks like a bunch of wavy lines. Now, why is this useful? Well, it helps us "see" sound. When you talk, sing, or play music, these waves represent the vibrations in the air. By studying these waveforms, scientists, musicians, and engineers can understand the characteristics of sound, like pitch and volume. It's like looking at a musical recipe; you can see how loud or soft the different parts of a song are, just like you can see the ingredients and quantities in a recipe. So, waveform plots are like musical blueprints, helping us visualize and understand the language of sound!

**Spectogram**
> A spectogram is a visual representation of sound waveforms. Imagine you're looking at a graph where the horizontal axis shows frequency, and the vertical axis represents the amplitude or intensity of the sound. It looks like a bunch of wavy lines. Now, why is this useful? Well, it helps us "see" sound. When you talk, sing, or play music, these waveforms represent the vibrations in the air.

To create our visualizations, we will use `matplotlib`.


```python
import matplotlib.pyplot as plt
from matplotlib.figure import Figure
```


```python
def plot_waveform(waveform, sample_rate):
    # waveform = waveform.numpy()

    num_channels, num_frames = waveform.shape
    time_axis = np.arange(0, num_frames) / sample_rate

    figure = Figure() 
    axes = figure.subplots(num_channels, 1)
    if num_channels == 1:
        axes = [axes]
    for c in range(num_channels):
        axes[c].plot(time_axis, waveform[c], linewidth=1)
        axes[c].grid(True)
        if num_channels > 1:
            axes[c].set_ylabel(f"Channel {c+1}")
    figure.suptitle("Waveform")
    plt.show()
    return figure
```


```python
def plot_specgram(waveform, sample_rate, title="Spectrogram"):
    # waveform = waveform.numpy()

    num_channels, num_frames = waveform.shape
    figure = Figure() 
    axes = figure.subplots(num_channels, 1)
    # figure, axes = plt.subplots(num_channels, 1)
    if num_channels == 1:
        axes = [axes]
    for c in range(num_channels):
        axes[c].specgram(waveform[c], Fs=sample_rate)
        if num_channels > 1:
            axes[c].set_ylabel(f"Channel {c+1}")
    figure.suptitle(title)
    plt.show()
    return figure
```


```python
np.array([result.json()['outputs'][0]['data']]).shape
```


```python
sampling_rate
```


```python
plot_specgram(np.array([result.json()['outputs'][0]['data']]), sampling_rate)
```


```python
plot_waveform(np.array([result.json()['outputs'][0]['data']]), sampling_rate)
```

We can now add these to a plotting file.


```python
%%writefile src/plotting.py

from matplotlib.figure import Figure
import matplotlib.pyplot as plt
from pedalboard.io import AudioFile
import numpy as np

from glob import glob
import os
```


```python
%%writefile -a src/plotting.py

def get_latest_file():
    files =  glob("./music/*.mp3")
    latest_file = max(files, key=os.path.getctime)
    with AudioFile(latest_file, "r") as f:
        waveform = f.read(f.frames)
        sample_rate = f.samplerate
    return waveform, sample_rate
```


```python
%%writefile -a src/plotting.py

def make_spectogram():
    waveform, sample_rate = get_latest_file()

    num_channels, num_frames = waveform.shape
    with plt.xkcd():
        figure = Figure() 
        axes = figure.subplots(num_channels, 1)
        if num_channels == 1:
            axes = [axes]
        for c in range(num_channels):
            axes[c].specgram(waveform[c], Fs=sample_rate)
            if num_channels > 1:
                axes[c].set_ylabel(f"Channel {c+1}")
        figure.suptitle("Spectrogram")
    return figure
```


```python
result.json()
```


```python
type(result.json()['outputs'][0]['data'])
```

## 5.3 Audio Effects as a Service

From the perspective of an audio programmer, audio effects are software tools designed to modify and enhance sound in various ways. Think of them as virtual gadgets that manipulate audio signals. These effects can range from simple ones, like adjusting volume or adding echo, to complex ones, like simulating guitar pedals or creating futuristic sci-fi sounds. For programmers, creating these effects involves using algorithms and coding skills to transform raw audio data into something sonically appealing.

From the perspective of a musician, audio effects are like a palette of colors for painting music. They add depth, texture, and emotion to the sound. Musicians use effects to shape their tones, creating everything from the distorted crunch of an electric guitar to the dreamy reverb in a singer's voice. Effects are essential because they allow artists to express their creativity, enhancing the overall musical experience for the listeners.

Now, imagine audio effects software and products as a magical toolbox. Musicians are like artists, and these tools are their brushes and paints. With the right combination of effects, they can craft a masterpiece, just like an artist creates a beautiful painting using various colors and brushes. It's all about transforming ordinary sounds into something extraordinary, adding layers of richness to the musical canvas.


```python
from pedalboard import Pedalboard, Distortion, Delay, Reverb, Chorus, Gain, PitchShift, Compressor, Mix
```


```python
delay_and_pitch_shift = Pedalboard([
    Delay(delay_seconds=0.35, mix=0.8), PitchShift(semitones=9), Gain(gain_db=-4),
])
```


```python
sampling_rate
```


```python
Audio(audio_values[0].numpy(), rate=sampling_rate)
```


```python
new_audio = delay_and_pitch_shift(audio_values[0].numpy(), sample_rate=sampling_rate)
Audio(new_audio, rate=sampling_rate)
```


```python
%%writefile servers/pedal_board/audio_mixer.py

from pedalboard import Pedalboard, Distortion, Delay, Reverb, Chorus, Gain, PitchShift, Compressor, Mix
from pedalboard.io import AudioFile

from mlserver.codecs import decode_args
from mlserver import MLModel
import numpy as np
```


```python
%%writefile -a servers/pedal_board/audio_mixer.py

class AudioMixer(MLModel):
    async def load(self):
        self.delay_and_pitch_shift = Pedalboard([
            Delay(delay_seconds=0.25, mix=1.0), PitchShift(semitones=7), Gain(gain_db=-3),
        ])

    @decode_args
    async def predict(self, song: np.ndarray, sample_rate: np.ndarray) -> np.ndarray:

        self.passthrough = Gain(gain_db=1)
        self.board = Pedalboard([
            Compressor(),
            Mix([self.passthrough, self.delay_and_pitch_shift]),
            Reverb()
        ])
        self.new_audio = self.board(song, sample_rate=sample_rate[0][0])
        return self.new_audio
```


```python
%%writefile servers/pedal_board/model-settings.json
{
    "name": "novice_dj",
    "implementation": "audio_mixer.AudioMixer"
}
```


```python
%%writefile servers/pedal_board/settings.json
{
    "http_port": 7050,
    "grpc_port": 7060,
    "metrics_port": 6050
}
```


```python
audio_values[0].numpy().shape, sampling_rate
```


```python
new_music = {
    "inputs": [
        NumpyCodec.encode_input(name='song', payload=audio_values[0].numpy()).dict(),
        NumpyCodec.encode_input(name='sample_rate', payload=np.array([sampling_rate])).dict()
    ]
}
new_music
```


```python
response = requests.post("http://localhost:7050/v2/models/novice_dj/infer", json=new_music)
print(response)
```


```python
Audio(audio_values[0].numpy(), rate=sampling_rate)
```


```python
Audio(response.json()['outputs'][0]['data'], rate=sampling_rate)
```

## 5.4 Midi Representations

![midi](../images/midi_repr.png)

From an audio programmer's perspective, MIDI (Musical Instrument Digital Interface) representations are a digital language that computers and musical instruments use to communicate. MIDI data contains information about musical notes, like which note was played, how long it was held, and how loud it should be. Programmers use MIDI to control virtual instruments and audio software, allowing musicians to create music on computers. It's like a set of instructions that tell the computer exactly what notes to play and how to play them.

From a musician's perspective, MIDI representations serve as a bridge between creativity and technology. Musicians can compose intricate pieces without needing to play every instrument physically. MIDI allows them to input musical ideas into computers or synthesizers, which then play back the notes with different sounds. It's incredibly useful for composing, arranging, and experimenting with various musical ideas, even if you don't have a band or an orchestra at your disposal.

Imagine MIDI as musical sheet music for the digital age. Musicians write their musical ideas on this digital paper, and when played back through software or instruments, it's like an orchestra following the sheet music, bringing the composition to life. MIDI empowers musicians to explore endless musical possibilities, much like a versatile toolkit in the hands of a skilled craftsman, helping them create beautiful melodies and harmonies effortlessly.

For this section, we will be using Spotify's new tool, [basic pitch](https://github.com/spotify/basic-pitch) 
to convert musical notes to MIDI.


```python
from IPython.display import IFrame
```


```python
IFrame(src='https://basicpitch.spotify.com/about', width=900, height=600)
```

## 6. Front- and Back-End

We will be using gradio as it allows us to build nice graphical user interfaces that we can 
to prototype our models, pipelines, dashboards, and more.


```python
%%writefile app.py

from src.plotting import make_waveform, make_spectogram
from src.helpers import *
import gradio as gr
```


```python
%%writefile -a app.py


with gr.Blocks(theme='gstaff/xkcd') as demo:
    gr.Markdown("# Music Generation and Editing App")
    gr.Markdown("Second Demo of the Day!")

    with gr.Column():
        gr.Markdown("# Step 1 - Describe the music you want 😎 🎸 🎹 🎵")
        with gr.Row(equal_height=True):
            with gr.Column(min_width=900):
                text = gr.Textbox(
                    label="Name", lines=3, interactive=True,
                    info="Audio Prompt for the kind of song you want your model to produce.",
                    value="a fast bachata with violin sounds and few notes from a saxophone",
                    placeholder="Type your song description in here.",
                )
                make_music   = gr.Button("Create Music")
            with gr.Column():
                tokens      = gr.Slider(label="Max Number of New Tokens", value=200, minimum=5, maximum=1000, step=1)
                guidance    = gr.Slider(label="Guidance Scale", value=3, minimum=1, maximum=50, step=1)
                sample_rate = gr.Radio([16000, 32000, 44100], label="Sample Rate", value=32000)
        
        audio_output = gr.Audio()
        make_music.click(fn=make_sound, inputs=[text, guidance, tokens, sample_rate], outputs=audio_output, api_name="create_music")
```


```python
%%writefile -a app.py

        gr.Markdown()
        gr.Markdown("# Step 2 - Visualize your creation 📈 👀 👌")
        with gr.Row():
            with gr.Column():
                create_plots = gr.Button("Visualize Waveform")
                plot1 = gr.Plot()
                create_plots.click(fn=make_waveform, outputs=plot1)
            with gr.Column():
                create_plots = gr.Button("Visualize Spectogram")
                plot2 = gr.Plot()
                create_plots.click(fn=make_spectogram, outputs=plot2)
```


```python
%%writefile -a app.py

        gr.Markdown()
        gr.Markdown("# Step 3 - Add Some Effects to it 📼 🎧 🎷 🎼")
        with gr.Column():
            update_music = gr.Button("Update your Music")
            output_video = gr.Video(label="Output", elem_id="output-video")
            update_music.click(audio_effect, outputs=[output_video])
```


```python
%%writefile -a app.py

        gr.Markdown()
        gr.Markdown("# Step 4 - Create a MIDI Representation! 🎛️ 🎶 🎼")
        gr.HTML(value="""<iframe src="https://basicpitch.spotify.com/" height="1000" width="100%"></iframe>""")

demo.launch()
```

## 8. Exercise

1. Install the diffusers package.


```python

```

2. Import the pipeline for AudioLDM2


```python
from diffusers import ____
```

3. Come up with a few prompts


```python
prompts = [___, ____, ____]
```

4. Download the smallest model


```python
repo_id = "cvssp/audioldm2"
pipe = AudioLDM2Pipeline.from_pretrained(repo_id, torch_dtype=torch.float32)
```

5. Create a pipeline and test that it works.


```python
audio = pipe(
        prompt,
        audio_length_in_s=____,
        guidance_scale=____,
        num_inference_steps=____,
        negative_prompt=____,
        ...
)["audios"]
```

6. Create a server using MLServer and test that it works correctly.


