# AutoBeat plugin: On-device AI MIDI generation with GPT-2 and JUCE
![autobeat-thumb](https://github.com/user-attachments/assets/a138eb79-4f50-43f9-bc37-a69c45f4d17e)
## Introduction
The use of AI models in generative symbolic music, including MIDI, has become a widely explored topic in the academic world, with good potential for commercial application use. 

One such example is **AutoBeat**. 

AutoBeat is a 12-track MIDI beat generator plugin (MacOS/Win), made with JUCE, which uses an AI model to generate MIDI beats in various electronic music genres and with parameters provided by the user.

Video: https://youtu.be/7CZx4ntO3zM

The model that AutoBeat uses is GPT-2, developed by OpenAI (2019). The inspiration behind the use of a Transformer-based language model like GPT-2 came from this paper: https://arxiv.org/abs/1809.04281 which explains why this architecture could work with symbolic music representation. 

We decided to embed the model directly into the plugin code (more about that below). Therefore, we used the smallest version to keep the plugin size within a reasonable range (around 300 Mb). 

## Fine-tuning
Instead of training the model from scratch, we relied on a fine-tuning process: we provided the already-trained model with our MIDI data and trained it further. That part was done in Python using Google Colab (https://colab.research.google.com).

## Custom tokens/embeddings
One of the challenges of using a language model for symbolic music processing is converting the music data into a format that the model understands. A popular solution is to encode the data into custom _tokens_, character sets that can be added to the model's existing vocabulary.

Unlike most MIDI tokenization techniques (https://arxiv.org/abs/2310.17202), AutoBeat uses a custom, grid-based way to represent MIDI data. After experimentation, we found that this encoding method was better suited to our use case: multi-track electronic music beat generation.

### Implementation
We split every MIDI beat into four instrument groups, according to their function in the beat (e.g. group 1 consists of kick, rimshot, snare, and clap). For every group, we extracted the MIDI notes and encoded them according to their starting time, expressed as steps on a hypothetical 2-bar, 4-steps-per-beat grid, their step offset (so that smaller rhythmic values, as well as rhythmic nuances such as swing, can be represented), their track number and, finally, their velocity. 

On top of that, we added the genre, the beat type, the music density (number of events), and the music intensity (how many events occur on- and offbeat). 

This is an example of a training prompt:

`genre:trap type:beat_loop density:high intensity:high group:1 step:0 track:4 offset:0 velocity:84 step:4 etc.`

## On-device processing
All AI processing in AutoBeat is done on-device. This became possible by using **ggml** (G. Gerganov et al.), a great tensor library for machine learning to enable large models and high performance on commodity hardware (https://ggml.ai).

We used the library's GPT-2 example as a starting point and, after some refactoring and tweaking (and caffeine), we managed to add it to the plugin code. The beat generation process (aka _inference_) runs on a background thread so that it does not block the plugin's workflow. When the user clicks on the generate button, a prompt is constructed and encoded from the plugin parameters (genre, group/tracks, density, intensity) and then is fed into the model. Once the model is done processing and the output is collected, it is decoded into music events and passed on to the audio engine.

### Performance
The beat generation's performance varies among different hardware. On more powerful machines (e.g. Silicon-based MacBooks) it is reasonably fast, with most beats taking no more than 4-5 seconds.
