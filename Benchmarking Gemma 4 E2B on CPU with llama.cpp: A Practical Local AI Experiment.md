

## 1. Why This Experiment?

I want to run a local language model for everyday office work without depending entirely on a cloud-based AI service.

My intended use cases include:

- A local chatbot for general questions.
- Drafting Malay and English letters, memoranda, and emails.
- Preparing technical documentation.
- Basic spreadsheet analysis involving hundreds of rows, rather than thousands or millions.
- Learning how to deploy and tune local AI models on different computers.

The challenge is that not every computer has the same CPU. Some machines have older processors with only a few cores, while others have newer CPUs with more cores and threads.

Instead of assuming that one configuration works everywhere, I want to establish a baseline and use the same benchmarking method across different machines.

This article records my initial experiments with **Gemma 4 E2B using `llama-server` from llama.cpp**.

## 2. Test Environment

The initial experiment was performed on a resource-constrained machine.

| Component | Specification |
|---|---|
| CPU | AMD Athlon 3000G |
| CPU architecture | 2 physical cores, 4 logical threads |
| Graphics | Radeon Vega Graphics |
| Runtime environment | WSL |
| WSL memory limit | 5 GB |
| Inference backend | llama.cpp `llama-server` |
| Inference mode | CPU-only |
| Parallel slots | 1 |

The main model file was:

```text
gemma-4-E2B-it-qat-UD-Q4_K_XL.gguf
```

The model is an instruction-tuned Gemma 4 E2B GGUF using the QAT Q4_K_XL quantization variant.

The objective is not to establish a universal performance figure for this model. It is to understand how configuration choices affect inference on this particular machine.

*Note: The exact llama.cpp version/build identifier was not recorded in this initial benchmark. Future tests should record it because performance and available options can change between builds.*

## 3. Establishing a Baseline

I started with a conservative configuration:

```bash
llama-server \
  -m ./gemma-4-E2B-it-qat-UD-Q4_K_XL.gguf \
  -t 2 \
  -tb 2 \
  -c 2048 \
  -b 64 \
  -ub 16 \
  --host 127.0.0.1 \
  --port 5001
```

The initial reported average generation speed was approximately **4.6 tokens per second**.

I then tested different thread counts and batch sizes. The following table records the results observed during the experiments.

| Test | Main configuration change | Reported speed |
|---|---|---:|
| A | `-t 2 -tb 2 -c 2048 -b 64 -ub 16` | 4.6 t/s |
| B | `-t 3 -tb 4 -c 2048 -b 64 -ub 16` | 5.9 t/s |
| B, repeat | Same thread configuration | 6.1 t/s |
| C | `-b 32 -ub 8`, with `-t 3 -tb 4` | 6.0 t/s |
| D | `-t 4 -tb 4`, with the original batch sizes | 5.8 t/s |

These results suggested that using three generation threads and four batch-processing threads was worth investigating further on this CPU.

Increasing the thread count did not automatically improve performance. The four-thread test was slower than the three-thread result in these particular trials.

However, these were exploratory measurements rather than a controlled benchmark. They should not be interpreted as proof that three threads will always be optimal.

## 4. Testing Additional Server Parameters

Next, I added several parameters:

```text
-np 1
-ngl 0
--temp 0.7
--top-p 0.8
--top-k 20
--metrics
--jinja
--cache-type-k q8_0
--cache-type-v q8_0
--reasoning-format none
```

The reported speed was approximately **6.4 tokens per second**.

Because several options were added together, this result does not tell us which individual parameter, if any, improved generation speed.

I then tested Flash Attention explicitly:

```text
-fa on
```

The reported speed for that trial was approximately **6.2 tokens per second**.

This trial did not demonstrate an improvement, so I left Flash Attention out of the preferred configuration for now. A more controlled test would be needed to establish whether it helps on another CPU or build.

### What these parameters are for

- `-np 1`: Configures one parallel sequence slot for this single-user experiment.
- `-ngl 0`: Keeps model inference on the CPU rather than offloading model layers to a GPU.
- `--cache-type-k q8_0` and `--cache-type-v q8_0`: Select quantized data types for the key and value caches.
- `--metrics`: Enables the server's metrics endpoint.
- `--jinja`: Enables the relevant Jinja chat-template handling.
- `--temp`, `--top-p`, and `--top-k`: Control sampling behaviour, not a guaranteed performance improvement.
- `--reasoning-format none`: Configures reasoning-format handling for the server.

The available options and their precise behaviour should be checked against the [official llama.cpp server documentation](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md).

## 5. Best Reported Configuration So Far

The strongest reported result from the exploratory trials was approximately **6.6 tokens per second**.

The configuration was:

```bash
llama-server \
  -m ./gemma-4-E2B-it-qat-UD-Q4_K_XL.gguf \
  -t 3 \
  -tb 4 \
  -c 2048 \
  -b 128 \
  -ub 32 \
  -np 1 \
  -ngl 0 \
  --temp 0.7 \
  --top-p 0.8 \
  --top-k 20 \
  --metrics \
  --jinja \
  --cache-type-k q8_0 \
  --cache-type-v q8_0 \
  --reasoning-format none \
  --host 127.0.0.1 \
  --port 5001
```

This is my **provisional baseline**, not a claim that the configuration is universally optimal.

The larger batch settings were present in the best reported trial. More repeatable testing is required to establish whether they improve performance consistently, particularly because batch settings can affect prompt processing differently from token generation.

The `--no-mmap` option has not been benchmarked in this experiment and should not be considered part of the tested configuration.

## 6. A Real Office-Work Test

A later trial used a Malay prompt requesting a formal memorandum about periodic maintenance for internally developed systems using Laravel and Debian Linux.

The generated memorandum covered:

1. Security patching.
2. Package updates.
3. Log and performance monitoring.
4. Risks associated with inadequate maintenance.
5. A recommendation to include maintenance in the department's operational plan.

The reported statistics were:

| Metric | Result |
|---|---:|
| Prompt tokens | 126 |
| Generated tokens | 391 |
| Reported average generation speed | 6.0 t/s |
| Approximate generation time from token count and speed | 65 seconds |

The time is an estimate calculated from the reported token count and speed. It is not a direct end-to-end timing measurement.

### Observations about output quality

The memorandum had a recognisable formal structure and its recommendations were generally relevant to the requested topic.

However, it also generated a specific date, 26 May 2024, even though the prompt did not provide that date. This is an important reminder that local language models can introduce unsupported details.

For actual office use, the application should provide authoritative information such as dates, recipient details, reference numbers, and departmental names. Generated letters and technical recommendations should still be reviewed before they are issued or acted upon.

The experiment therefore evaluates two separate dimensions:

- **Performance:** How quickly the model generates output.
- **Quality:** Whether the generated output is accurate, relevant, appropriately formatted, and safe to use.

A higher tokens-per-second figure does not necessarily mean a better office assistant.

## 6.1 Testing Document Understanding and Structured Data Analysis

After testing the model's ability to draft an office memorandum, I moved on to two additional tasks using plain-text (`.txt`) documents.

The objective was to evaluate whether Gemma 4 E2B could extract facts from a document, follow instructions, identify missing information, and analyse a small dataset without inventing values.

### Test 1 — Document Understanding

The first test used a simulated internal system maintenance procedure. The document covered maintenance schedules, change-management requirements, database backups, restore testing, and incident records.

I asked the model nine questions, including which details were explicitly documented and which were missing.

| Metric | Result |
|---|---|
| Prompt tokens evaluated | 1,051 |
| Generated tokens | 638 |
| Reported average generation speed | 5.0 t/s |
| Questions answered | 9 |
| Overall observation | Satisfactory for this test |

The model answered all nine questions correctly in this trial. It produced a structured frequency table, identified the required change-record fields, explained why a successful backup job does not guarantee that restoration will work, and avoided inventing details that were absent from the source document.

This was an encouraging result for document-grounded question answering. However, it was a single qualitative test rather than a statistically validated accuracy benchmark.

### Test 2 — ICT Asset Data Analysis

The second test used a simulated ICT asset inventory containing eight records, including desktops, laptops, monitors, printers, and a scanner. Each record included a quantity, unit cost where available, and status.

The model was asked to calculate the total number of units, calculate record-level costs, identify missing prices, summarise assets requiring inspection, and prepare a short management summary.

I first ran the test with a context size of 2,048 tokens.

| Metric | Result |
|---|---|
| Prompt tokens evaluated | 1,462 |
| Generated tokens | 585 |
| Reported average generation speed | 5.0 t/s |
| Output completion | Incomplete |

The response stopped before all questions were answered. The reported context usage reached approximately 2,048 tokens, suggesting that the context limit may have contributed to the incomplete response. This is an observation from the reported run, not proof that context capacity was the only cause.

I then repeated the test with a context size of 4,096 tokens.

| Metric | Result |
|---|---|
| Prompt tokens evaluated | 1,098 |
| Generated tokens | 1,696 |
| Reported average generation speed | 5.3 t/s |
| Output completion | All 10 questions answered |

With the larger context, the model completed the requested analysis. It correctly identified 22 total units, calculated the individual costs for records with known prices, and identified two records marked *Perlu diperiksa*, covering four units in total. It also correctly noted that this status alone does not prove that an asset is broken.

However, the final cost calculation was incorrect. The model reported **RM33,850**, whereas the correct sum of the seven calculable record costs is **RM33,050**. The individual record-level calculations were correct, but the final addition was not.

The record with a missing unit cost was correctly excluded from the total.

### What These Tests Tell Me

These experiments highlight three practical findings:

1. **Document understanding looks promising.** In the first trial, the model extracted facts and respected missing information in a simulated maintenance procedure.
2. **Context capacity matters for longer tasks.** The 2,048-token run produced an incomplete response, while the 4,096-token run completed the task. The tests were not identical in token usage, so this should be treated as an initial observation rather than a controlled performance comparison.
3. **Correct-looking calculations still need verification.** The asset-analysis test contained correct individual calculations but an incorrect final sum. A fluent explanation and a completed response do not guarantee numerical accuracy.

For practical office applications, I would use the language model to interpret documents, explain results, and prepare summaries. For financial totals and other exact calculations, I would rely on deterministic tools such as PHP, SQL, or spreadsheet formulas, then provide the verified results to the model for explanation.

For now, a context size of 4,096 tokens is a more useful working configuration for this type of longer document task on my test machine. I still need to monitor memory usage and repeat the tests before drawing broader conclusions about performance or reliability.

## 7. Why the Results Are Still Preliminary

The measurements above were collected during interactive experiments. The prompts and output lengths were not identical in every trial, and some trials changed several parameters at once.

Consequently, the results are useful for narrowing down configurations, but not for drawing definitive conclusions about individual options.

A more reliable benchmark should:

1. Use the same prompt for every comparison.
2. Set the same maximum output-token limit.
3. Allow the model and system to reach a comparable starting state.
4. Repeat each configuration at least three times.
5. Record the median and average generation speed.
6. Record prompt-processing speed separately from generation speed where possible.
7. Monitor RAM usage, CPU utilisation, thermal behaviour, and swapping.
8. Check output quality using the same practical tasks.

The llama.cpp server supports a Prometheus-compatible metrics endpoint when `--metrics` is enabled. Its reported metrics can help distinguish prompt-processing throughput from generation throughput. See the [server metrics documentation](https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md).

## 8. A Repeatable Benchmark Plan for Other PCs

The long-term objective is to repeat this experiment across computers with different CPU generations, core counts, and memory limits.

For each machine, I will record:

| Field | What to record |
|---|---|
| Machine ID | A simple label, such as `PC-01` |
| CPU | Exact processor model |
| CPU topology | Physical cores and logical threads |
| Memory | Installed RAM and the actual limit available to the inference environment |
| Operating environment | Linux, WSL, or another supported environment |
| llama.cpp build | Version, build details, and relevant compilation options |
| Model | Exact model filename and quantization |
| Context size | Value of `-c` |
| Threads | Values of `-t` and `-tb` |
| Batch settings | Values of `-b` and `-ub` |
| Cache settings | K and V cache types |
| Prompt and output | Prompt tokens and generated tokens |
| Performance | Generation tokens/s and prompt tokens/s, when available |
| Resource use | Peak RAM, CPU utilisation, swapping, and temperature where available |
| Quality | Accuracy, language quality, formatting, and unsupported claims |

### Recommended test sequence

Start with the provisional baseline and change one variable at a time.

**Stage 1 — Thread configuration**

Compare suitable values for `-t` and `-tb` based on the CPU's topology. For a CPU with two physical cores and four logical threads, for example, test a small range of thread settings instead of assuming that all logical threads will always help.

**Stage 2 — Context size**

Compare `-c 1024` with `-c 2048` while keeping other parameters constant. A smaller context may reduce memory requirements, but it also limits how much conversation or source material can fit into the context.

The `-c 1024` configuration is a proposed future test, not a completed measurement in this experiment.

**Stage 3 — Batch sizes**

Test `-b` and `-ub` independently where practical. Record prompt-processing throughput as well as generation speed, because the effect may differ between these workloads.

**Stage 4 — KV cache**

Only after establishing a stable baseline, consider comparing `q8_0` with other supported cache types, such as `q4_0`. Measure memory use, speed, stability, and output quality. Do not assume that a smaller cache will necessarily make inference faster.

**Stage 5 — Practical workload**

Run the same set of tasks on each machine:

- A short chatbot question.
- A formal Malay letter or memorandum.
- An English email.
- A structured technical document.
- A small spreadsheet-analysis task.

This will help determine which configurations are suitable for actual use rather than merely optimising a single benchmark prompt.

## 9. Practical Lessons So Far

The initial experiment suggests several useful lessons:

- A local model can produce a structured Malay office memorandum on a CPU-only machine.
- The best reported speed so far was 6.6 tokens per second, but the benchmark needs more controlled repetitions.
- More CPU threads do not guarantee better generation speed.
- Sampling parameters should be chosen for the desired response behaviour, not treated as performance switches.
- Context size, batch configuration, and cache types should be tested separately.
- RAM usage and thermal stability matter, particularly on older or resource-constrained computers.
- Output accuracy must be evaluated separately from generation speed.
- For spreadsheet tasks, an application can process the actual spreadsheet with a suitable data library and pass relevant results to the model instead of sending an entire workbook into the prompt.

## 10. Conclusion

This experiment establishes a starting point for running Gemma 4 E2B with llama.cpp on modest CPU hardware.

The current provisional configuration uses three generation threads, four batch threads, a context size of 2048, and batch settings of 128 and 32. It achieved a reported best speed of approximately 6.6 tokens per second in the exploratory trials.

The next goal is not simply to make one computer faster. It is to build a repeatable method for finding practical configurations across several computers, including older CPUs and newer machines with more cores.

The long-term measure of success is a useful local assistant for everyday office work: responsive enough for conversation, capable of producing good first drafts, and reliable enough to support documentation and basic data analysis with appropriate human review.

---

**Benchmark status:** Initial exploratory phase completed. Further parameter testing is paused until a new machine or a controlled test session is selected.

This article was created with the help of AI
