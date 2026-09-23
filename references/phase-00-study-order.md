# Phase 00 — Recommended Study Order (beginner)

The course lists Phase 00 lessons in one order, but a few foundational topics sit
later than ideal for a beginner. This is my recommended order to learn them in.
The **# on site** column is the lesson's original number on aiengineeringfromscratch.com,
so I can still find it in the Catalog.

| Study order | Lesson | # on site | Type | Note |
|---|---|---|---|---|
| 1 | Dev Environment | 01 | Build | ✅ done |
| 2 | Git & Collaboration | 02 | Learn | ✅ done |
| 3 | Terminal & Shell | 10 | Learn | Pulled early — I use the terminal constantly |
| 4 | Linux for AI | 11 | Learn | Pulled early — core "operate my machine" skills |
| 5 | Editor Setup | 08 | Build | Pulled early — set up VS Code before writing lots of code |
| 6 | Jupyter Notebooks | 05 | Build | Everyday tool for experiments |
| 7 | Python Environments | 06 | Build | Reinforces the uv/venv work from Dev Environment |
| 8 | APIs & Keys | 04 | Build | Needed once I start calling model APIs |
| 9 | Data Management | 09 | Build | Handling datasets |
| 10 | Debugging & Profiling | 12 | Build | Finding and fixing problems |
| 11 | GPU Setup & Cloud | 03 | Build | Deferred — no NVIDIA GPU on my Mac; revisit when training models |
| 12 | Docker for AI | 07 | Build | Deferred — advanced infra; revisit when a lesson needs it |

## Why this order

- **Fundamentals first.** Terminal, Linux, and editor setup are things I use in
  *every* lesson, so learning them early pays off immediately.
- **Defer heavy infra.** GPU/cloud and Docker are advanced and not urgent — I won't
  train real models for many phases, and my Mac has no NVIDIA GPU (it uses Apple's
  MPS instead), so cloud GPUs and CUDA aren't relevant yet.
- **Python Environments is partly review** of the uv/venv setup already done in
  Dev Environment, so it goes after the foundations as reinforcement.

## Caveats

- Each Phase 00 lesson is fairly self-contained, so following the site's original
  order (03 → 12) also works fine — this is an optimization, not a requirement.
- Finishing lessons matters more than perfect ordering. Don't let reordering become
  an excuse to procrastinate.

## From Phase 01 onward

Follow the course's own numbered order. The reordering above is specific to Phase 00's
setup lessons; later phases build on each other and are already sequenced sensibly.
