# sd-small (profile)

The engine `.env` for a **Stable Diffusion 1.5 node at 512x512**. The 4GB-class
default.

## Two fields have to agree with the download

SD keeps models per **architecture**, so both of these must match what was
pulled:

```
ARCH=sd15
MODEL=v1-5-pruned-emaonly/v1-5-pruned-emaonly.safetensors
```

Changing `ARCH` swaps the entire assembled view to that architecture's tree, so
an SDXL checkpoint sitting under `ARCH=sd15` is simply never mounted. Install
`v1-5-pruned-emaonly` alongside this profile.

## Sampling defaults apply to every request

`STEPS=20` and `CFG_SCALE=7` are the usual SD 1.5 starting point, and sd.cpp
applies them to **every** call. They are engine settings, not per-request
overrides.

## What it sets

Image `ghcr.io/isannai/sd`, container `sd` on port 7860, sd15 architecture,
outputs to `./outputs`. VAE and LoRA directories wired but empty.

## Install

```
isann profile pull <this asset>
isann profile use --engine sd --name <name>
```

## See also

- `install-sd-small` - a recipe that does the profile, the download, and the
  container in one run

## Source

https://github.com/isannai/profiles
