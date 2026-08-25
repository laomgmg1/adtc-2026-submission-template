# Technical Report — Africasai

**Team ID:** africasai
**Domain:** coding_assistants  
**Model:** Qwen2.5-1.5B-Instruct-Q4_K_M

---

## Problem

Africasai explores how a useful coding assistant can be deployed locally on
resource-constrained computers for programming education.

The target users are students and aspiring developers who may not have access
to powerful computers, dedicated GPUs, or reliable internet connectivity.

Our goal is therefore not simply to maximize model capability. We want to
find a practical quality/performance balance that allows an AI coding
assistant to run locally, with low memory consumption and without requiring
cloud inference.

This is particularly relevant in African educational environments where
hardware and connectivity constraints can make cloud-dependent AI tools
difficult to access consistently.

---

## Design Decisions

- **Base model:** Qwen2.5-1.5B-Instruct
- **Quantization:** GGUF Q4_K_M
- **Runtime:** llama.cpp
- **Inference:** CPU-based local inference

We evaluated several model sizes during development, including SmolLM2-135M,
Qwen2.5-1.5B, Qwen2.5-3B, and the Ternary-Bonsai family.

The 1.5B Qwen model provided the best measured balance between response
quality, inference speed, and memory usage in our available environment.

The Qwen2.5-3B Q4_K_M model consumed approximately 3.48 GB peak RSS and
generated approximately 3.8 tokens/second in our profiling environment.
This made it substantially slower than the 1.5B model.

We also investigated Ternary-Bonsai models. The 8B Q2_0 model was too slow
for our target environment, while compatibility with the mainline llama.cpp
runtime was an additional consideration.

The final candidate was therefore Qwen2.5-1.5B-Instruct Q4_K_M.

---

## Constraints

- **Target:** approximately 8 GB RAM
- **Inference:** CPU-only
- **Development environment:** Ubuntu 24.04.4 LTS
- **Development CPU:** Intel(R) Xeon(R) Platinum 8573C
- **Available CPU cores:** 2 in the development Codespace
- **GPU:** None
- **Runtime:** llama.cpp
- **Connectivity:** The intended system is designed around local inference,
  reducing dependence on continuous internet connectivity.

The development environment does not exactly match the final ADTC evaluation
hardware. In particular, our Codespace exposed 2 CPU cores, whereas the
ADTC evaluation environment uses 4 cores. Our measurements should therefore
be treated as conservative development benchmarks.

---

## Benchmarks

We evaluated several local models and configurations on the participant environment before selecting the final submission model. The goal was to balance reasoning quality, accuracy, inference speed, memory usage, and thermal headroom under a CPU-only environment with approximately 8 GB RAM.

### Experimental Environment

| Metric | Development environment |
|---|---|
| CPU | Intel Xeon Platinum 8573C |
| Available RAM | 7.8 GB |
| GPU | None |
| OS | Ubuntu 24.04.4 LTS |
| Inference runtime | llama.cpp |
| CPU threads tested | 2 |
| Random seed | 42 |

The development environment exposes 4 CPU cores, but the ADTC participant environment provides 2 cores. We therefore explicitly tested inference using `-t 2` to better represent the target environment.

### Model Comparison

| Model | Quantization | Parameters | Peak RAM | Generation | TTFT | ARC-Easy |
|---|---|---:|---:|---:|---:|---:|
| SmolLM2-135M-Instruct | Q4_K_M | 135M | 213 MB | 62.02 t/s | 2.22 s | — |
| Qwen2.5-1.5B-Instruct | Q4_K_M | 1.5B | 1.85 GB | 8.08 t/s | 13.08 s | **76%** |
| Qwen2.5-3B-Instruct | Q4_K_M | 3B | 3.48 GB | 3.84 t/s | 28.74 s | — |
| Ternary-Bonsai-8B | Q2_0 g64 | 8B | — | ~0.3 t/s | — | — |

### Qwen2.5-1.5B-Instruct

The final candidate was evaluated using the ADTC profiler with the `arc_easy` benchmark on 50 samples.

- **Generation:** 8.08 tokens/s
- **First-token latency:** 13.08 seconds
- **Peak RSS:** 1.85 GB
- **Steady-state RSS:** 1.77 GB
- **Peak CPU utilization (p99):** 77.8%
- **Thermal throttling:** Not observed
- **ARC-Easy accuracy:** 0.76
- **Context length:** 32,768 tokens

The model therefore remained comfortably below the available memory budget, although generation speed was below our initial 15 tokens/s target.

### Qwen2.5-3B-Instruct

We also evaluated the 3B Q4_K_M model under the same CPU-only configuration.

- **Generation:** 3.84 tokens/s
- **First-token latency:** 28.74 seconds
- **Peak RSS:** 3.48 GB
- **Steady-state RSS:** 3.29 GB
- **Peak CPU utilization (p99):** 94%
- **Thermal throttling:** Not observed

Although the 3B model remained within the memory budget, its substantially lower generation speed and higher CPU utilization made it less suitable for the target hardware.

### Ternary-Bonsai Experiments

We investigated Ternary-Bonsai models as an alternative approach to reducing memory requirements through aggressive low-bit representations.

The 8B Q2_0 g64 model was approximately 2.2 GB on disk and successfully loaded with the mainline llama.cpp build. However, inference on the participant environment was extremely slow, reaching approximately **0.3 tokens/s** in our short test.

We used a fork of llama.cpp to run the Ternary-Bonsai-8B-Q2_0.gguf model which gave an output of about **2.8 tokens/s** in contrast to the 0.3 tokens/s observed on the Ternary-Bonsai-8B-Q2_0_g64.gguf. Despite this improvement the model was too slow, thus rejected as a candidate.

Given the inference speed, the 8B model was rejected for the final submission despite its attractive quantized memory footprint.

### Selection

The final model was selected based on the overall trade-off rather than maximum model size.

**Qwen2.5-1.5B-Instruct-Q4_K_M** provided the strongest practical balance among the candidates tested:

- substantially better capability than the 135M baseline;
- **76% ARC-Easy accuracy** in our 50-sample evaluation;
- approximately **1.85 GB peak RAM**;
- no observed thermal throttling;
- significantly faster inference than the 3B and 8B alternatives.

The experiment also demonstrated that simply increasing parameter count was not sufficient for our target environment. Under CPU-only inference, the 3B and 8B candidates imposed a substantial throughput penalty.

These are self-reported development benchmarks. Official competition scores are measured by the ADTC profiler on the standard evaluation environment.

---

## Benchmarks

| Metric | Value |
|---|---:|
| Machine | Intel Xeon Platinum 8573C, 2 cores |
| Available RAM | 7.8 GB |
| RAM at peak | 1.85 GB |
| RAM steady state | 1.77 GB |
| Time to first token | 13.08 s |
| Generation speed | 8.08 t/s |
| CPU utilization p99 | 77.8% |
| Thermal throttling | None observed |
| Accuracy benchmark | ARC-Easy |
| Accuracy samples | 50 |
| ARC-Easy accuracy | 76% |

These are self-reported development benchmarks produced through the ADTC
profiler. Official scores are measured by the ADTC profiler on the standard
evaluation machine.

---

## Evaluation

The Qwen2.5-1.5B-Instruct Q4_K_M model achieved 0.76 normalized accuracy
on a 50-sample ARC-Easy evaluation.

The model used 1.85 GB peak resident memory in our development environment,
leaving substantial memory headroom within an approximately 8 GB system.

The model generated approximately 8.08 tokens/second on the 2-core
development environment, without observed thermal throttling.

The results suggest that a relatively small quantized model can provide a
useful level of language-model capability while remaining practical for
local CPU inference.

---

## What We Learned

Our experiments showed that model size alone is not a sufficient measure of
practical usefulness.

Increasing from 1.5B to 3B parameters improved qualitative response
behaviour in some of our tests, but substantially reduced inference speed
on our constrained CPU environment.

Quantization provided a way to make larger language models feasible, but
the resulting quality/performance trade-off still needs to be evaluated
against the actual hardware available to users.

For our target, the 1.5B Q4_K_M configuration offered the most attractive
balance we measured.

---

## Future Work

Future versions of Africasai will explore:

- offline local retrieval-augmented generation;
- support for English and an African language;
- deterministic tools for tasks where exact computation is preferable to
  language-model generation;
- more aggressive context and memory optimization;
- larger coding-specific evaluation sets;
- deployment on low-cost consumer hardware;
- and broader evaluation across African educational use cases.
