# Data Sources and Preparation

## 1. Azure Request Trace

Publisher: Microsoft Azure  
Dataset: Azure LLM Inference Dataset 2024  
File: AzureLLMInferenceTrace_conv_1week.csv

Documentation:
https://github.com/Azure/AzurePublicDataset/blob/master/AzureLLMInferenceDataset2024.md

Download:
https://github.com/Azure/AzurePublicDataset/releases/download/dataset-llm-2024/AzureLLMInferenceTrace_conv_1week.csv

### Use in This Project

I saved the first 10,000 records as
`data/azure_conversation_sample.csv`.

The source fields are:

- `TIMESTAMP`: Request arrival timestamp.
- `ContextTokens`: Input token count.
- `GeneratedTokens`: Generated output token count.

The sample covers approximately 392 seconds. It contains no missing
values, exact duplicate records, or zero input/output token counts.
Validation also checks timestamp parsing, finite values, and
nonnegative integer token counts.

I use the first 500 chronologically sorted requests for the configuration
analysis. Load tests rescale arrival spacing while preserving request
order and token counts. Original timestamps remain unchanged.

This is a contiguous sample, not a random or representative sample of
the entire dataset. Azure traffic is used as a proxy workload; it is
not Meta traffic.

## 2. Vidur Performance Profiles

Publisher: Microsoft  
Repository: https://github.com/microsoft/vidur

Pinned commit:
`abae7f63aa857300f5cdc6f5e0d27860cd24721b`

Profile directory:
https://github.com/microsoft/vidur/tree/abae7f63aa857300f5cdc6f5e0d27860cd24721b/data/profiling/compute/a100/meta-llama/Meta-Llama-3-8B

Source files:
- `mlp.csv`
- `attention.csv`

### Preparation

The original downloads contain 1,824 MLP rows and 42,587 attention rows.
Both passed missing-value and finite, nonnegative median-timing checks.

I removed 63 exact duplicate attention rows from the working copy.
The original downloaded CSV files remain unchanged.

I then selected one tensor-parallel worker and, for attention,
the FlashInfer backend. This produced:

- 456 MLP rows.
- 2,644 prefill rows.
- 15,957 decode rows.

Repeated measurements at the same lookup settings are aggregated
using the median. Prefill lookups use an empty KV cache.

### Timing Interpretation

Vidur's profiling code records timings in milliseconds:
https://github.com/microsoft/vidur/blob/abae7f63aa857300f5cdc6f5e0d27860cd24721b/vidur/profiling/common/cuda_timer.py

Its Llama 3 8B configuration specifies 32 transformer layers:
https://github.com/microsoft/vidur/blob/abae7f63aa857300f5cdc6f5e0d27860cd24721b/vidur/config/model_config.py

The component aggregation was informed by:
https://github.com/microsoft/vidur/blob/abae7f63aa857300f5cdc6f5e0d27860cd24721b/vidur/entities/execution_time.py

My notebook implements a simplified profile-informed simulation.
It does not run the full Vidur simulator.

## 3. Derived Configuration Dataset

`data/configuration_results.csv` contains 50 configurations generated
by the simulation, using five maximum batch sizes and ten batching-wait
limits.

`data/capacity_load_tests.csv` records the nine arrival-rate tests for
each configuration, for a total of 450 simulations.

The request records and processing profiles are public inputs.
Supported rates, response delays, Pareto classifications, and strategy
scores are derived results.

No LLM-generated scenario dataset was used. Source attribution here
describes data provenance; it does not describe authorship of the code
or documentation.

The notebook and README document the simulation assumptions,
capacity-screen thresholds, and limitations.