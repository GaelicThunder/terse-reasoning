<p align="center">
  <b>TERSE REASONING</b>
</p>

<p align="center">
  <em>The real latency tax on a local LLM isn't the kernel — it's the model writing its
  thoughts in full prose before answering.</em>
</p>

<p align="center">
  Exact prompt strings + measured numbers from a single GB10 box (ASUS Ascent GX10, 121&nbsp;GiB).
  No kernel work, no new hardware — one string in the system prompt.
</p>

---

## The claim

Free-form thinking decodes at ~17–27 t/s on this hardware. A 2,000-token think block is
~100 s of blank screen. The same reasoning as terse notes is ~15 s — **6–7× faster from a
single template change.**

## The change

One instruction in the reasoning section of the chat/system template: replace *"think in full
prose"* with *"reasoning = notes, not prose. Fragments. No articles, no filler, no hedging."*

See [`templates/notes-style.md`](templates/notes-style.md).

## Measured on this GB10

| | engine |
|---|---|
| hardware | ASUS GB10 · 121 GiB unified memory |
| model | DeepSeek-V4-Flash-0731 |
| stack | vLLM · dsflash · :30001 |

Reasoning-block tokens per reply, before/after the SOUL rewrite (2026-08-31):

| | before | after |
|---|---|---|
| mean | 1,677 | **1,323** |
| max | 43,955 | **13,320** |

> `reasoning_effort=low` is accepted by dsflash (HTTP 200 direct and via gateway) — the
> "low crashes" warning is wrong in practice.

## Try it

```python
# OpenAI-compatible check against your local engine
import openai, time

c = openai.OpenAI(base_url="http://127.0.0.1:30001/v1")  # your vLLM port
t0 = time.time()
r = c.chat.completions.create(
    model="deepseek-v4-flash-0731",
    messages=[
        {"role": "system", "content": open("templates/notes-style.md").read()},
        {"role": "user", "content": "Why does a database connection pool outperform per-request connections?"},
    ],
    # extra_body={"reasoning_effort": "low"}  # dsflash accepts it
)
print(f"{(time.time()-t0):.1f}s · {r.usage.completion_tokens} tokens"
      f" · {r.choices[0].message.content[:120]}")
```

Compare against the same prompt *without* the notes-style string. Post your numbers.

---

## License

MIT
