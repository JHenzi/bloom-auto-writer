# BLOOM Story Writer

This is a proof of concept set of code that will generate stories and other output using the [BLOOM-560m large language model](https://huggingface.co/bigscience/bloom-560m). We are using this model because it can perform (good/fast) CPU inference on a decent i7 machine and can fit the (impressive) model into memory with a reasonable input token size limit.

## Inspiration for this Proof of Concept

The BLOOM auto-story-writer is potentially part one of two projects or a mega project aimed at setting BLOOM to generate stories automatically with just a given writing prompt. Given we had access to the BLOOM model and created a story-writing or text continuation generator at [nlp.henzi.org](https://nlp.henzi.org) and were excited to consider different ways to set the generator up to *iteratively* generate text in "chunks". 

The story writer "Choose your own Fate" on the NLP site we created generates continuations of stories with a human-in-the-loop in a quasi-RLHF type of scenario. The model generates three of the most probable continuations of the text and the user selects the most likely or more interesting (based on coherence, style of writing, potential to prompt a new change or not).

What is of interest in the backend of how we generate the continuations is that we keep in the database for the site a few bits of curious information;

- What the model's parameters were for that continuation, the temperature, top_k, typical_p and other factors
- We keep the input text and tensors that generated the continuation
- We keep track of which story parts were generated, if they are selected or not. *In the future we'd like to train a model to pick the best continuation based on the **content** of the continuation*

Because we keep not only the selections users prefer but those that are discarded we have the potential to do analysis and *actually* look at this as a true **RLHF** project.

## Goals

The genesis of the idea for this "part one" of the project/code is to generate text by looking at the historical selections for the input parameters (temperature, top_k, typical_p, max_length, etc). We have collected, in close to 1,000 generated stories, the different values that the user preferred.

Of interest is that there is mixed consistency when stories are generated iteratively. Users don't always prefer the same temperature and top_k values, sometimes the story requires a drop in temperature or since it started to stagnate or even repeat itself it needed to take on new words (top_k) as probable to escape a semantic/continuity trap.

The goal is to use the data, eventually the live data, to start by using the averages *per step*. Given we have a history of selections *at a particular step* can we rely on the settings *per step number* to use as input parameters to `model.generate`?

### Secondary Goal

Beyond what I would call sophisticated "prompt engineering" (or rather *prompting control*) the secondary goal would be to perform deep learning on the input text that generated the three potential continuations, which was selected and which were not to uncover a potential decision algorithm to pick the best candidate based on similar selections users have made. (e.g. Users typically select a pronoun here and start a quote by a speaker as a pattern).

## Tech

The backend for nlp.henzi.org is written in Python using FastAPI to provide the interface into generating selections. The underlying application uses:

- BLOOM-560m for text generation/tokenization
- FastAPI to provide interactivity to our React web UI
- MongoDB to store the prompt, generated text and user selections of story continuations

One method to obtain the "live" values would be to open an endpoint to 'fetch' the averages. For example if we are generating in this app step #2 of the story we could potentially call the BLOOM backend for the site and inquire as to what the typical values for temperature, top_k, typical_p are for this step and use it in our generation.

Alternatively we could simply use static files of the most recent dump of data to generate the next continuation. While not "live" this could simplify the setup of this application (though requires an input or reference file to be handy at all times).

## Other Thoughts

What we love and experience on the nlp.henzi.org site is the ability to generate stories using *randomized* variables per-run. For each of the three continuations you receive they have different input parameters entirely and they are created per-run on the fly using a tolerable set of ranges for input values on `model.generate`. We have linear distribution to handle spikes in random values (such as a high top_k, which will force a lower temperature value) and mostly see coherent and non-repeating continuations. However, users do **not** prefer just one setting of temperature and top_k or like to vary the text that gets generated at important parts of the story generation process.

Because of this I think using the typical values, averages, etc can help lead to some interesting and potentially coherent results (thus this project). However, this is also potentially a way to uncover different biases and other artifacts in the model by setting generation to run "on it's own" and by varying the input parameters we may see more interesting output.

It should be a goal of this project to publish the generated text and any meta-annotations for the dataset for future research.