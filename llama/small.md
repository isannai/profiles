# llama-small (profile)

The engine `.env` for a **llama node running Qwen2.5 1.5B Q4_K_M**. The
4GB-class default: at Q4_K_M the weights are around 1.1GB, so a 4GB card holds
the whole model and its KV cache with room to spare (`GPU_LAYERS=99` puts every
layer on the GPU).

## It travels with a model

`MODEL` names a **directory** under the assembled view plus the file inside it,
and it has to match what the download created, character for character:

```
MODEL=Qwen2.5-1.5B-Q4_K_M/Qwen2.5-1.5B-Instruct-Q4_K_M.gguf
         ^ --name                ^ the file inside it
```

A mismatch starts the container and loads nothing, with little in the log to say
why. Install `Qwen2.5-1.5B-Q4_K_M` alongside this profile.

## What it sets

Image `ghcr.io/isannai/llama`, container `llama` on port 7862, context 8192,
every layer on the GPU, tool calls on. LoRA directory wired but empty.

## Install

```
isann profile pull <this asset>
isann profile use --engine llama --name <name>
```

## See also

- `llama-medium` - the same engine with a 14B model, for a 12GB+ card
- `install-llama-small` - a recipe that does the profile, the download, and the
  container in one run

## Source

https://github.com/isannai/profiles
