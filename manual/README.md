# Yet Another Stable Diffusion Discord Bot Manual

Welcome to the YASD Discord Bot manual! This will help guide you through the various command capabilities that YASD brings to your Discord server.

## Commands

This bot supports both legacy direct message commands and slash commands. Legacy direct message commands are all prefixed with `>`, while slash commands are prefixed with `/` and have some auto-completion features that make them easier to use. Please note that when using a direct message command, extra must be specified in `(foo=bar, zoo=car)` format. For example, to change the sampler and steps in direct message mode, you could prompt `>image a red ball (sampler=euler, steps=10)`.

For all commands that use indexes, the indexes are stored from left to right per row. For example, a series of 4 images will be indexed:

```
|~~~~~|
| 0 1 |
| 2 3 |
|~~~~~|
```

- [Text-to-image](#text-to-image)
- [Riff (Diffusion)](#riff-diffusion)
- [image2image](#image2image)
- [Interpolate](#riff-diffusion)
- [Upload images](#upload-images)
- [Upscale](#upscale)

### Text-to-image
- `/image <prompt>` (slash command)
- `/image <prompt [variation 1, variation 2, ...]>` (slash command)
- `>image <prompt> (foo=bar)` (direct message command)
- `>image <prompt [variation 1, variation 2, ...]> (foo=bar)` (direct message command)

Generate an image from a text prompt. Images may be given variations with an array format by enclosing values in square brackets e.g. "a [red, blue, green, purple] ball".

You can use concepts in your prompt from the [Stable Diffusion Concepts library](https://huggingface.co/sd-concepts-library) by simply entering the URL suffix of the concept library in `<>` brackets in the prompt. For example, to use the [SkyFalls](https://huggingface.co/sd-concepts-library/skyfalls) concept, simply put `<skyfalls>` into the prompt. Note that it is _not_ using the token identifier `<SkyFalls>`, but instead is using the URL suffix `skyfalls`.

Prompt weighting can be achieved using subprompts by adding a colon and then a weight (float). For example, you can create a cat-tiger hybrid with the prompt `tabby cat:0.75 tiger:0.25`. You can also use negative conditioning through subprompts by using a negative float value. For example, if you wanted a human Hatsune Miku, you would use the prompt `hatsune miku:1 anime:-1`.

By default Stable Diffusion limits you to 77 tokens from the CLIP tokenizer in your prompt. This bot is _not_ limited to that many tokens, and you can use as many as you can fit into the subprompt limit by adding `:1` to the end of each prompt.

Options:
- `height`: The height of the output in pixels. Min 384, max 768, steps in 64.
- `sampler`: Which sampler to use when creating the image. Some samplers, such as `euler`, may require fewer steps to get good results, while others can have [a dramatic effect](https://i.redd.it/uy2fp799wmj91.jpg) on image generation itself. Defaults to `k_lms`.
- `scale`: Conditioning scale for prompt (1.0 to 50.0). This is how strongly the prompt conditions the image. Very high scales may induce a saturation like effect. Default 7.5.
- `seed`: Deterministic seed integer used to generate your images. The seed defines the noise that will be used to generate your image, and will cause reproducible results when reusing seeds. For example, all prompt array iterations use a fixed seed to make all images appear similar to one another.  Default random integer.
- `seed_search`: When set, a total of 9 images are made by scanning all seeds from `seed` to `seed + 9`. If no seed is set, the default seed `1` is used.
- `width`: The width of the output in pixels. Min 384, max 768, steps in 64.

### image2image
- `/image2image <url> <prompt>` (slash command)
- `>image2image prompt (foo=bar)` (direct message command)

Diffuse an image that was uploaded with this message.

The only way to make it work with uploading a file is as a direct message command because Discord does not yet allow files to be uploaded with slash commands.

When using the slash command, a URL may be specified to download an image from instead.

Options: Same as for Riff (Diffusion) below.

### Riff (Diffusion)
- `/riff <docarray_id> <index>` (slash command)
- `>riff <docarray_id> <index> (foo=bar)` (direct message command)

Diffuse an image that was generated in the past or which was uploaded to the bot. The `docarray_id` is the short string (e.g. "eoZw11NmT") given with the images, while `index` refers to the index of the images in zeroeth ordering.

Riff buttons using the feature with default settings are automatically added to all `riff` or `image` commands.

Options:
- `height`: The height of the output in pixels. Min 384, max 768, steps in 64.
- `iterations`: The number of times to re-diffuse before generating the final four images. The more iterations, the strong the effect. Default is `1` or no extra iterations. Note that iterations are **ignored** when riffing one aspect ratio to another.
- `latentless`: Use a random latent to generate the image, meaning that the prior image is not used at all. May be used to test a prompt without the image while using the same parameters.
- `prompt`: Prompt the override the prompt saved in the DocArray.
- `sampler`: Which sampler to use when creating the image. Some samplers, such as `euler`, may require fewer steps to get good results, while others can have [a dramatic effect](https://i.redd.it/uy2fp799wmj91.jpg) on image generation itself. Defaults to `k_lms`.
- `scale`: Conditioning scale for prompt (1.0 to 50.0). This is how strongly the prompt conditions the image. Very high scales may induce a saturation like effect. Default 7.5.
- `seed`: Deterministic seed integer used to generate your images. The seed defines the noise that will be used to generate your image, and will cause reproducible results when reusing seeds. Default random integer.
- `strength`: Strength of conditioning (0.01 <= strength <= 0.99). Used to determine how strongly to diffuse the image against the previous state. A strength of 0.99 should effectively eliminate the previous image, while a strength of 0.01 should do almost nothing. Default 0.75. Note that strength is **ignored** when riffing one aspect ratio to another.
- `width`: The width of the output in pixels. Min 384, max 768, steps in 64.

### Interpolate
- `/interpolate <prompt 1> <prompt 2>` (slash command)
- `>interpolate <prompt 1>|<prompt2> (foo=bar)` (direct message command)

This generates a series of 9 images that start with one prompt and end with another prompt while attempting to sample all the space inbetween. The first image is generated through the same sampling pipeline as text-to-image, while the subsequent images are riffed through this while the latent embeddings are spherically interpolated between the two prompts to produce mid-state images.

- `height`: The height of the output in pixels. Min 384, max 768, steps in 64.
- `prompt 1`: Prompt to start the interpolation from.
- `prompt 2`: Prompt to interpolate to.
- `sampler`: Which sampler to use when creating the image. Some samplers, such as `euler`, may require fewer steps to get good results, while others can have [a dramatic effect](https://i.redd.it/uy2fp799wmj91.jpg) on image generation itself. Defaults to `k_lms`.
- `scale`: Conditioning scale for prompt (1.0 to 50.0). This is how strongly the prompt conditions the image. Very high scales may induce a saturation like effect.  Default 7.5.
- `seed`: Deterministic seed integer used to generate your images. The seed defines the noise that will be used to generate your image, and will cause reproducible results when reusing seeds. Default random integer.
- `strength`: Strength of conditioning (0.01 <= strength <= 0.99). Used to determine how strongly to diffuse the image against the previous state. Higher strengths should yield more intense interpolations. Default 0.75.
- `width`: The width of the output in pixels. Min 384, max 768, steps in 64.

### Upload Images
- `@bot_name <description>` (direct message command)

Simply tagging the bot with a message starting with `@bot_name` where `bot_name` is the name of your bot while attaching one or more images will upload all the images to the bot and yield IDs for them to use in `riff`, `upscale`, etc.

### Upscale

- `/upscale <docarray_id> <index>` (slash command)
- `>upscale <docarray_id> <index>` (direct message command)

Upscale an image to 4x the resolution (512x512 -> 2048x2048).

Upscale buttons using the feature are automatically added to all `riff` or `image` commands.


## Administrator-level Configurations

### Bot arguments

- `--allow-queue`: If this is true, the users are allowed to make any quantity of images for themselves at the same time. By default, users are restricted to one image at a time.

- `--default-steps <steps>`: The default number of steps to use on `/image` or `>image`.

- `-g <guild_id>` or `--guild <guild_id>`: Discord guild/server ID to send your slash commands to. If not specified, it may take up to an hour for your slash commands to show up.

- `--hours-on-server-to-use <hours>`: The number of hours a user must have been on the server to use the bot.

- `--nsfw-auto-spoiler`: Automatically add the spoiler tag to any potential NSFW images. Requires you to `pip install -r requirements_nsfw_filter.txt` first, as this requires torch.

- `--nsfw-prompt-detection`: Use BERT through detoxify to detect potentially offensive or NSFW prompts.

- `--nsfw-wordlist <wordlist>`:  Reject any prompts if they contain a word within a wordlist. The wordlist should be strings separated by newlines.

- `--restrict-all-to-channel <channel_id>`: Discord channel ID to restrict all your commands to.

- `--restrict-slash-to-channel <channel_id>`: Discord channel ID to restrict your slash commands to.


### config.yml for Stable Diffusion Executor

If you have installed the Native Installation, you can find a simple configuration file in `dalle/dalle-flow/executors/stable/config.yml`. Here you can alter the default number of samples to generate at the same time along with the resolution of the images generated.

### Multi-GPU setup

In `tmp.flow.yml`, change `CUDA_VISIBLE_DEVICES` for the `stable` executor to be either `RR#:#` or `RR#,#,#,...`, where `#` is the device ID of your GPU. For example, you can select GPUs 0, 1, and 2 with `RR0:3`. Alternatively, you can select them with a command separated list as `RR0,1,2`.

After setting that, adjust the number of replicas. For example, if you wanted 2x executors running on 3x GPUs with the above settings for `CUDA_VISIBLE_DEVICES`, you would set `replicas: 6`.

Requests are round-robined to free GPUs in a multi-GPU setup. You can also place things like the upscaler on their own cards.

### Other models

You can turn on other models in [dalle-flow](https://github.com/jina-ai/dalle-flow) like DALLE-MEGA and GLID3XL by either removing the environmental variables that disable them (Docker) or by removing the flags passed to `flow_parser.py` (Native Installation). Be aware that you'll need a lot of VRAM!

## License

Copyright 2022 Jimmy (AmericanPresidentJimmyCarter)

[MIT](https://choosealicense.com/licenses/mit/)
