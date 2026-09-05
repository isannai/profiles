# Profiles

A **profile** is a named `.env` for one engine: the settings the container reads
when it starts. Model file, context size, GPU layers, port, and so on.

Profiles are per engine, so the same name can exist for `sd` and for `llama`
without clashing.

```
<engine>/<name>.env     one file = one profile (this repo)
         |
         | isann profile pull --engine <engine>
         v
<install-root>/artifacts/addon/profiles/<engine>/<name>.env    the library
<install-root>/artifacts/addon/engines/<engine>/.env           the ACTIVE one
```

The top-level folder is the engine name, matching the `--engine` flag:

```
llama/example.env
sd/example.env
vllm/example.env
```

The same profile name can exist under several engines without clashing, because
the engine is part of the address everywhere - in this repo, in the local
library, and on the command line.

`profile use` copies the library file over the engine's `.env`. Nothing takes
effect until the engine is restarted.

## File format

Plain `KEY=value`, one per line, `#` for comments - the format
`docker compose --env-file` and the engine's `run.sh` both read.

These are the keys in `llama/example.env`, comments stripped (the file itself
documents every one of them inline):

```env
PROFILE_NAME=default
PROFILE_APPLIED_AT=
IMAGE_NAME=ghcr.io/isannai/llama
IMAGE_TAG=latest
CONTAINER_NAME=llama
PORT=7862
IP=10.10.1.13
MODELS_DIR=./.temp/models/defaults
MODEL=Qwen2.5-1.5B-Q4_K_M/Qwen2.5-1.5B-Instruct-Q4_K_M.gguf
SERVED_MODEL_NAME=
CTX_SIZE=8192
GPU_LAYERS=99
SPLIT_MODE=
PARALLEL=1
SLOTS=
THREADS=
KV_TYPE=
TOOL_CALLS=on
CHAT_TEMPLATE=
NO_MMAP=
MLOCK=
LORA_DIR=./.temp/models/loras
LORA_ADAPTERS=
EXTRA_LLAMA_ARGS=
```

An empty value means "use the engine's own default" - it is not the same as
removing the line.

The key set differs per engine. `sd/example.env` has `ARCH`, `VAE_DIR`,
`VAE_FILE`, `STEPS`, `CFG_SCALE`, `SAMPLE_METHOD`, `OUTPUTS_DIR` instead;
`vllm/example.env` has `SHM_SIZE`, `MAX_MODEL_LEN`, `GPU_MEMORY_UTILIZATION`,
`MAX_NUM_SEQS`, `TOOL_PARSER` and the LoRA module keys.

`PROFILE_NAME` and `PROFILE_APPLIED_AT` are stamped when the profile is applied;
you do not maintain them by hand.

The authoritative list of keys for an engine is the `.env` shipped in that
engine's folder (`artifacts/addon/engines/<engine>/.env`) - the files here are
copies of those, so every key is documented inline with its default.

### Keys that are NOT portable between nodes

A profile is meant to be shared, but three groups of keys describe the machine,
not the taste:

| Key | Why it is node-specific |
|---|---|
| `PORT` | Host port. Two engines on one node must differ. |
| `IP` | Static address on the `isann` docker network. Collides if reused. |
| `CONTAINER_NAME` | Must be unique per node. |
| `MODELS_DIR` `LORA_DIR` `VAE_DIR` | FIXED view paths assembled by isannd - do not edit. |

Pulling a profile overwrites these with the publisher's values. Check them after
`profile use`, before starting the engine.

Portable keys are the ones worth sharing: `MODEL`, `CTX_SIZE`, `GPU_LAYERS`,
`PARALLEL`, `KV_TYPE`, `TOOL_CALLS`, `STEPS`, `CFG_SCALE`, and so on.

## Install

Use the **raw** URL. A `github.com/.../blob/...` (or `/tree/...`) address serves
an HTML page, not the file - and because a `.env` is never parsed, that page
would be stored as your profile without complaint.

`--engine` is required: it decides which engine's library the file lands in.

```bash
# from this repo
isann profile pull \
  https://raw.githubusercontent.com/isannai/profiles/main/llama/example.env \
  --engine llama --name example

isann profile pull \
  https://raw.githubusercontent.com/isannai/profiles/main/sd/example.env \
  --engine sd --name example

isann profile pull \
  https://raw.githubusercontent.com/isannai/profiles/main/vllm/example.env \
  --engine vllm --name example

# pin to a commit so the content can never change under you
isann profile pull \
  https://raw.githubusercontent.com/isannai/profiles/<commit-sha>/llama/example.env \
  --engine llama --name example

# pin the content by hash instead
isann profile pull <url> --engine llama --name example --hash sha256:<hex>

# from a local file
isann profile pull file:///d:/profiles/example.env --engine llama --name example
```

`--name` defaults to the URL's file name without its extension, so
`example.env` installs as `example`.

## Use

```bash
isann profile list --engine llama          # what is installed
isann profile use  --engine llama --name example
isann docker prepare llama
isann docker create llama                  # restart to pick up the new .env
isann docker wait  --engine llama          # block until it actually responds
```

The restart is the part people forget. `profile use` only writes the file.

Some engines re-read `.env` on `stop`/`start` because their entrypoint sources
the bind-mounted file at each start - `llama` is one. Changing the container spec
itself (image, ports, mounts) always needs a full recreate.

## Create and edit locally

```bash
isann profile set  --engine llama --name example MODEL=... CTX_SIZE=4096 GPU_LAYERS=20
isann profile copy --engine llama --from example --to example-cpu
isann profile rm   --engine llama --name example -y
```

`profile set` creates the profile if it does not exist and merges the given keys
if it does. `profile copy` rewrites `PROFILE_NAME` in the copy.

The active profile is protected from `rm` unless you pass `-force`.

## Inspect

```bash
isann profile inspect example           # raw .env + its provenance
isann profile list                     # all engines
```

`inspect` shows where the profile came from (`source_url`), which matters once
you have pulled from several places.

## Publish to the hub

```bash
isann profile push example --engine llama --version 1.0.0 --summary "4GB VRAM, 8k ctx"
```

Run `isann auth unlock` first - the upload is signed with your owner wallet.

## Layout of this repo

```
llama/
  example.env
sd/
  example.env
vllm/
  example.env
```

One folder per engine, named exactly as the `--engine` value. Adding a profile
means dropping a `.env` into the right engine folder - there is no index file to
maintain.
