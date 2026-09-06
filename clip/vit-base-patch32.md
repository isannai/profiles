# vit-base-patch32

The stock `clip` profile. CPU only, one model, and almost nothing to tune. Most
nodes apply it once and never look at it again.

## What clip is for

It does not generate anything. Given an image and a piece of text it returns how
well the two agree, as a number. On an iSANN node that is what the faucet's
image track uses to grade a prober's answer: the prober asks for a picture, the
node produces one, and clip decides whether the picture is actually what was
asked for.

Outside the faucet the same score is useful for zero-shot classification,
text-to-image search, and deduplication.

## Applying it

```bash
isann model pull https://huggingface.co/openai/clip-vit-base-patch32 \
  --engine clip --kind model --name clip-vit-base-patch32
isann profile use --engine clip --name vit-base-patch32
isann docker create clip -prepare
isann docker probe clip
```

`MODEL` here names the folder that `model pull --name` created. They have to
match exactly or the container starts and then cannot find the weights.

## What you might change

| Key | Default | When to touch it |
|---|---|---|
| `THREADS` | `4` | Drop to `2` on a memory-tight node. Single process, and each worker costs about 840MB |
| `CLIP_THRESHOLD` | `0.25` | Only affects mode 1 (absolute similarity). Mode 2 is tuned per request by `margin` and ignores it |
| `VALIDATOR_VERSION` | `1.0.0` | Bump it whenever you change `CLIP_MODEL`. It rides along on every verdict, and a different model shifts the score distribution, so old verdicts stop being comparable |
| `PORT` | `7866` | Only if something else already holds the port. Bound to 127.0.0.1 either way |

Everything else is structural. `MODELS_DIR` in particular is a fixed key that
points at the view isannd assembles from the store, and editing it breaks the
mount.

## Why there is no GPU setting

CLIP ViT-B/32 is small enough that CPU inference is fast, and the node's GPU is
better spent on the engine actually generating. The compose file has no
`deploy.resources` block at all.

## Why no registry image

Unlike the other engines, clip builds from `src/` locally: the source is 13KB,
so running a GHCR repo for it would buy nothing. That is why `IMAGE_NAME` has no
registry prefix and the image is never pulled.
