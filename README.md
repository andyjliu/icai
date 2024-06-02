
# Inverse Constitutional AI

Repository containing code for the paper "Inverse Constitutional AI: Compressing Preferences into Principles" (pdf coming soon). The figure below provides an overview of the *Inverse Constitutional AI* (ICAI) problem we introduce: starting from a set of pairwise preference feedback, we derive a set of natural language principles (a *constitution*) that explain the preference data.
For validation, we re-construct the original preferences with an LLM judging according to the generated constitution. The constitution represents a (highly compact) compression of the preferences.

<p align="center">
<img src="./docs/img/02_complete_overview.png" width="1000px" align="center">
</p>

# Motivation

Feedback data plays an important role in fine-tuning and evaluating state-of-the-art AI models. Often pairwise text preferences are used: given two texts, human (or AI) annotators select the “better” one. Such feedback data is widely used to align models to human preferences (e.g., reinforcement learning from human feedback), or to rank models according to human preferences (e.g., Chatbot Arena). Despite its wide-spread use, prior work has demonstrated that human-annotated pairwise text preference data often exhibits unintended biases. For example, human annotators have been shown to prefer assertive over truthful texts in certain contexts. Models trained or evaluated on this data may implicitly encode these biases in a manner hard to identify. To be able to better understand existing pairwise text preference data, we formulate its interpretation as a compression task: the *Inverse Constitutional AI* problem. Read the [full paper]() (coming soon) for more background.

# Algorithm

We introduce a first *Inverse Constitutional AI* (ICAI) algorithm that generates a set of principles based on a feedback dataset. See the figure below for an overview of the algorithm. Given a dataset of pairwise rankings, in Step 1 candidate principles are generated using an LLM. In Step 2, these principles are clustered using an embedding model. In Step 3, similar principles are “de-duplicated” by sampling one principle per cluster. In Step 4, each principle is tested to evaluate its ability to help an LLM reconstruct the original annotations. Finally in Step 5, the principles are filtered according to the testing results and a set of filtered principles are returned as the final constitution. Optionally, this last step is augmented with additional clustering and subsampling steps to ensure diverse principles. The implementation is provided in this repository.

<p align="center">
<img src="./docs/img/03_algorithm.png" width="800px" align="center">
</p>

## Installation

1. *Pip install the package*

    - **Non-contributors**
        ```
        pip install git+https://github.com/rdnfn/icai.git
        ```
    - **Contributors:** clone repo locally, e.g.
        ```
        git clone git@github.com:rdnfn/icai.git
        ```
        Then (inside repo folder) install package in editable mode:
        ```
        pip install -e .
        ```
2. *Set up API secrets:* inside the main directory of the cloned repo (or wherever you like really) set up a secrets.toml file like below. You only need to include keys for APIs you want to use.
    ```toml
    OPENAI_API_KEY="<YOURKEY>"
    ANTHROPIC_API_KEY="<YOURKEY>"
    ```
3. *Download data*, or use your own feedback data. You can use the data notebook (see Quickstart) to download the supported data sources. Currently supported data sources:
    - https://github.com/anthropics/hh-rlhf
    - https://huggingface.co/datasets/lmsys/chatbot_arena_conversations

## Quickstart

Given a feedback dataset (use the [data notebook](https://github.com/rdnfn/icai/blob/main/notebooks/01_data_prepocessing.ipynb) to download one), you can run your first Inverse Constitutional AI (ICAI) experiment using the `icai-exp` command:

```
icai-exp secrets_path="./secrets.toml" data_path="data/processed/example/example.csv"
```

To get the available experiment parameters and instructions on how to adjust them, run

```
icai-exp --help
```
>[!NOTE]
> **If you want more control**:
> the `icai-exp` command executes the `run` function inside [`./src/inverse_cai/experiment/core.py`](https://github.com/rdnfn/icai/blob/main/src/inverse_cai/experiment/core.py#L111). Edit that file, and the other parts of the `inverse_cai` Python package it uses, to fully adapt this code to your needs.

### Inspecting results

By default all experiment results are saved in the `./outputs/<DATE>_<TIME>` directory. These outputs contain a full record of API calls, as well as intermediate states of the algorithm (proposed principles, clusters, distilled principles, etc.). Each result output follows the structure below:

```text
./outputs
└── 2024-03-14
    └── 16-09-43
        ├── api_calls.jsonl
        ├── core.log
        ├── main.log
        └── results
            ├── 010_principles_per_comparison.csv
            ├── 011_principles_list.json
            ├── 020_principle_clusters.json
            ├── 030_distilled_principles_per_cluster.json
            ├── 040_votes_per_comparison.csv
            ├── 041_votes_per_cluster.json
            ├── 050_filtered_principles.json
            ├── 060_constitution.json
            ├── 092_results_training.csv
            └── 093_results_testset.json
```

## Run experiment from config file

In the `exp/configs` folder there is a number of configs to recreate experiments. You can run these experiments using the command:

```
icai-exp -cd ./exp/configs/<EXPERIMENT-DIR>
```

For example:

```
icai-exp -cd ./exp/configs/001_synthetic_orthogonal
```

>[!NOTE]
> **To re-run paper experiments**:
> Look at the README file inside the `exp/configs`. This file gives detailed instructions on which configurations to run, and how to generate the corresponding plots.

## Development

### Running test cases

To run the test cases for the code, use the following command:

```
pytest ./tests
```

### Simplest way to run experiment script
This doesn't do any meaningful experimental work but allows running the experiment script for testing purposes.

```
icai-exp generate_constitution=false annotator.constitution=null annotator.other_annotator_configs="[]"
```


### License

All code in this repo is licensed under [Apache-2.0](./LICENSE).
