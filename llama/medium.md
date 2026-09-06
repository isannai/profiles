# llama-medium (profile)

The engine `.env` for a **llama node running Qwen2.5 14B Q4_K_M**. Wants a
**12GB+ card**.

## It travels with a model

```
MODEL=Qwen2.5-14B-Q4_K_M/Qwen2.5-14B-Instruct-Q4_K_M.gguf
```

The two halves fail in opposite directions. Applying this profile without the
14B weights on disk points the engine at a directory that is not there; pulling
the weights without applying this profile leaves 9GB unused while the small
model keeps serving. Install `Qwen2.5-14B-Q4_K_M` alongside it, or use the
`install-llama-medium` recipe, which does both in that order.

## Why the context is 8192

Not the model's full window, on purpose. At Q4_K_M the weights already take
around 9GB and the KV cache grows with context, so a larger window on a 12GB
card spills into host memory. That reads as "the node got slow" rather than an
out-of-memory error, which is the harder failure to diagnose.

## What it sets

Image `ghcr.io/isannai/llama`, container `llama` on port 7862, context 8192,
every layer on the GPU, tool calls on.

## Install

```
isann profile pull <this asset>
isann profile use --engine llama --name <name>
```

## See also

- `llama-small` - the same engine with a 1.5B model, for a 4GB+ card
- `install-llama-medium` - a recipe that does the profile, the download, and the
  container in one run

## Source

https://github.com/isannai/profiles
