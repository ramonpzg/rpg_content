# 2 Content Creators

## Table of Contents

1. Overview
2. Tools
3. ML Models
    - Text
    - Images
    - Audio
    - Videos
4. Final Thoughts
5. Exercise

## 1. Overview

The world of generative AI is a rabbit hole of models capable of spectacular feats of automated 
content creation. In this workshop, we'll explore some of the most promising genres of generative 
models with a focus on their creative applications. 

First we'll look at transformer models - which can conjure up synthetic images, video, and audio 
that capture the style and essence of their training data.

Next up are diffusion models - transformers artsy cousin specialized in realistic image synthesis. 
We'll talk about how they work their magic and try our hand at guiding them to generate new scenes and 
characters.

Finally, we'll have a look at the different models we can use for Audio generation and Video creation.

Let's dive in.

## 2. Tools

- transformers
- ctransformers
- diffusers

## 3. ML Models

Please note that the models below can run in modern-day computers without extravagant hardware. The caveat 
is that it will most-likely take too lond to run without a decent size GPU. Therefore, we'll walk through this 
part together, and, where possible, fetch content from different models freely available to generate the input 
for our website.

### 3.1 Text

Large language models like GPT-3 are neural networks trained on massive text datasets to generate 
language. They are useful for:

- Text generation - Creating novels, articles, code, dialogue automatically
- Text summarization - Condensing long text into concise summaries 
- Translation - Converting text between languages
- Question answering - Answering natural language questions

Large language models absorb linguistic patterns like a sponge soaks up water. After digesting huge 
volumes of text, they can then wring out new writings dripping with style and substance. Their potential 
is as vast as the ocean of data they learn from.

For this next section, one of the things you'll need to do is to download the checkpoint of the fine-tuned 
model we will be using.

This model was trained by [Mistral AI](https://mistral.ai/) and 
[fine-tuned on the Open Orca dataset](https://huggingface.co/TheBloke/Mistral-7B-OpenOrca-GGUF) by TheBloke. 
It has different variations of it, and the unique part of it is that it uses an implementation in C, which 
makes it much faster than the Python implementation.

⚠️ Please note that the checkpoint is about 4GB in size.

```sh
huggingface-cli download TheBloke/Mistral-7B-OpenOrca-GGUF mistral-7b-openorca.Q4_K_M.gguf --local-dir . --local-dir-use-symlinks False
```

To avoid having to run this on your laptop, you can use this [huggingface space](https://huggingface.co/spaces/osanseviero/mistral-super-fast) to generate some text.


```python
from ctransformers import AutoModelForCausalLM
```

You can set `gpu_layers` to the number of an adequate number your GPU might be able to handle. Otherwise, 
setting it to 0 means CPU only.


```python
model_ckp  = "/home/ramonperez/Tresors/seldon/tutorials/synthetic_data/mistral-7b-openorca.Q4_K_M.gguf"
model_name = "TheBloke/Mistral-7B-OpenOrca-GGUF"

llm = AutoModelForCausalLM.from_pretrained(model_name, model_file=model_ckp, model_type="mistral", gpu_layers=4)
```


```python
print(llm("Siberian huskies are a fun breed of dogs that"))
```

### 3.2 Images

Large vision transformer models like DALL-E are neural networks trained on large amounts of images 
to generate and manipulate visual content. They are useful for:

- Image generation - Creating original digital artwork, photos, and designs 
- Image editing - Changing attributes like style, lighting, objects in images
- Text-to-image - Generating images from textual descriptions
- Image captioning - Adding descriptive captions to images

Large vision models absorb visual patterns like a painter studies the world around them. After observing 
enough training data, they can conjure new images imbued with lighting, perspective and detail.

Images are a type of unstructured data 


```python
import torch
from diffusers import AutoPipelineForText2Image
```


```python
device = "cuda"
dtype = torch.float32
pipeline =  AutoPipelineForText2Image.from_pretrained("warp-ai/wuerstchen", torch_dtype=dtype)#.to(device)
```


```python
# caption = "Anthropomorphic siberian husky dressed as a fire fighter"
```


```python
prompt = 'Black and white cartoon of a siberian husky in a suit sitting at a bar at the top floor of a building in New York City overlooking central park'
```


```python
output = pipeline(
    prompt=prompt, height=768, width=1024, output_type="pil",
    prior_guidance_scale=4.0, decoder_guidance_scale=0.0
).images
```


```python
output[0]
```

### 3.3 Audio

Audio transformer models like [Jukebox](https://openai.com/research/jukebox) are neural networks trained 
on large datasets of audio to generate and modify music and speech. They are useful for:

- Music generation - Creating original songs, instrumentals, and compositions
- Voice cloning - Mimicking the tone and speech patterns of a speaker
- Text-to-speech - Converting text to natural sounding speech 
- Speech enhancement - Improving audio quality, removing background noise

Audio transformers absorb acoustic patterns like a musician develops an ear for music. After "listening" to 
enough training samples, they can produce novel rhythms, melodies, and vocals - like an artificial imagination 
for sound.

For our use case, we will be using [MusicGen](https://huggingface.co/facebook/musicgen-small), a model developed by 
a group of researchers at **Meta AI**.

It can help us create novel sounds with only a few words. Let's try it out. 🎹


```python
from transformers import AutoProcessor, MusicgenForConditionalGeneration
from IPython.display import Audio
import torch
from pedalboard.io import AudioFile
```


```python
processor = AutoProcessor.from_pretrained("facebook/musicgen-small")
model = MusicgenForConditionalGeneration.from_pretrained("facebook/musicgen-small")
```


```python
inputs = processor(
    text=["fast bachata with high quality in the style of juan luis guerra", "Piano and violin orchestra with slow and fast tempo"],
    padding=True,
    return_tensors="pt",
)
```


```python
sampling_rate = model.config.audio_encoder.sampling_rate
sampling_rate
```


```python
audio_values = model.generate(**inputs, do_sample=True, guidance_scale=3, max_new_tokens=256)
```


```python
audio_values.shape
```


```python
Audio(audio_values[1].numpy(), rate=sampling_rate)
```


```python
with AudioFile("../assets/audio/orchestra.wav", samplerate=sampling_rate, num_channels=1) as f:
    f.write(audio_values[1].numpy())
```

### 3.4 Video

Video transformer models are neural networks trained on large video datasets to generate and edit 
digital video content. They are useful for:

- Video generation - Creating original video clips, effects, and animations
- Video prediction - Generating plausible future video frames 
- Text-to-video - Converting text descriptions into video
- Video enhancement - Increasing resolution, framerate, quality of video

Video transformers absorb motion and temporal patterns like a painter studies light and perspective. 
After observing enough footage, they can produce novel scenes and effects - like an artificial director's 
eye for sequencing images. Their potential is as far-reaching as our visual imagination and cinematic history.

The model we'll be using is a transformer model created by a group of researchers from Alibaba and its 
details can be found [here](https://huggingface.co/damo-vilab).

A hugging face space can be accessed [here](https://huggingface.co/spaces/damo-vilab/modelscope-text-to-video-synthesis).


```python
import torch
from diffusers import DiffusionPipeline, DPMSolverMultistepScheduler
from diffusers.utils import export_to_video

# this setup will work on CPU
pipe = DiffusionPipeline.from_pretrained("damo-vilab/text-to-video-ms-1.7b", torch_dtype=torch.float32)

# this setup will work on GPU
# pipe = DiffusionPipeline.from_pretrained("damo-vilab/text-to-video-ms-1.7b", torch_dtype=torch.float16, variant="fp16").to('cuda')

pipe.scheduler = DPMSolverMultistepScheduler.from_config(pipe.scheduler.config)
```




```python
prompt = "A Siberian Husky is surfing"
prompt = "A Siberian Husky is going down the street in her skateboard"
video_frames = pipe(prompt, num_inference_steps=25).frames
```


```python
video_path = export_to_video(video_frames, output_video_path='../assets/videos/husky_skating.mp4')
video_path
```

## 4. Final Thoughts

We have only scratched the surface of what we can do with the current architectures, and we can be confident 
it is only a matter of time until a new one, or a new combination of old ones, arrives.

To recap:
- We used pre-trained models for different modalities to generate data for the website we created earlier.
- Each model comes with its pros and cons and some have better default settings than others.
- By training on thousands of photos, diffusion models learned to render highly realistic synthetic images from noise.

## 5. Exercise

Here are two exercise templates using the transformers library for a generative models workshop:

Exercise 1 - Text Generation

Import GPT2 and generate a joke:


```python
from transformers import ____, ____

model = GPT2LMHeadModel.from_pretrained('__') 
tokenizer = _____.____('__')

prompt = 'Two elephants walk into a bar' 
input_ids = tokenizer._____(_____, return_tensors='pt')

______ = model.generate(_____, max_length=____)
text = tokenizer._____(______[0])

print(text)
```

Compare the quality of the joke with that of a joke from pyjokes.


```python
import pyjokes

____
```

Exercise 2 - Text Summarization

Import T5 and summarize text from wikipedia or elsewhere.


```python
from transformers import ____Model, ____Tokenizer

model = ____Model.from_pretrained('__')
tokenizer = ____Tokenizer.from_pretrained('__') 

text = """Your long text to summarize"""

inputs = _____([text], return_tensors='__')
summary = model._____(_____)

print(_____._____(_____[0]))
```
