# Leveraging Large Language Models for Generating Research Topic Ontologies: A Multi-Disciplinary Study

This repository offers datasets and scripts to fine-tune and evaluate LLMs for identifying semantic relationships between research topic pairs. It aims to advance the creation of research topic ontologies spanning multiple disciplines.

[![Python 3.12.0](https://img.shields.io/badge/python-3.12.0-blue.svg)](https://www.python.org/downloads/release/python-3120/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

---

## 📜 Abstract
Ontologies and taxonomies of research fields are critical for managing and organising scientific knowledge, as they facilitate efficient classification, dissemination and retrieval of information. 

However, the creation and maintenance of such ontologies are expensive and time-consuming tasks, usually requiring the coordinated effort of multiple domain experts. Consequently, ontologies in this space often exhibit uneven coverage across different disciplines, limited inter-domain connectivity, and infrequent updating cycles.

In this study, we investigate the capability of several LLMs to identify semantic relationships among research topics within three academic domains: **biomedicine**, **physics**, and **engineering**. The models were evaluated under three distinct conditions: **zero-shot prompting**, **chain-of-thought prompting**, and **fine-tuning** on existing domain-specific ontologies. Additionally, we assessed the **cross-domain transferability** of fine-tuned models by measuring their performance when trained in one domain and subsequently applied to a different one. 
To support this analysis, we introduce **PEM-Rel-8K**, a novel dataset consisting of over 8,000 relationships extracted from the most widely adopted taxonomies in the three disciplines considered in this study: **MeSH**, **PhySH**, and **IEEE**. Our experiments demonstrate that fine-tuning LLMs on **PEM-Rel-8K** yields excellent performance across all disciplines.

---

## 📂 Repository Structure

### 📄 `datasets/`
This folder contains all datasets used in this study, including **IEEE-Rel-3K**, **MeSH-Rel-4K**, **PhySH-Rel-875**, and the comprehensive **PEM-Rel-8K** dataset. Access them [here](./datasets).

## 📊 Dataset Construction: PEM-Rel-8K

**PEM-Rel-8K** is a comprehensive dataset of over 8,000 semantic relationships derived from three domain-specific taxonomies:  
- **MeSH** (biomedicine)  
- **PhySH** (physics)  
- **IEEE Thesaurus** (engineering)  

The dataset was systematically constructed to ensure high-quality semantic relationships across these domains. Below is a detailed breakdown of the dataset construction process:

| **Domain**       | **Dataset**      | **Broader** | **Narrower** | **Exact Match** | **Other** | **Total** |
|-------------------|------------------|-------------|--------------|-----------------|-----------|-----------|
| **Engineering**  | **IEEE-Rel-3K** | 800         | 800          | 800             | 800       | 3,200     |
| **Physics**      | **PhySH-Rel-875**| 250         | 250          | 125             | 250       | 875       |
| **Biomedical**   | **MeSH-Rel-4K** | 1,000       | 1,000        | 1,000           | 1,000     | 4,000     |
| **Combined**     | **PEM-Rel-8K**  | 2,050       | 2,050        | 1,925           | 2,050     | 8,075     |

### 🔍 Dataset Details by Domain

#### **1. Engineering Domain: IEEE-Rel-3K**
- **Source**: IEEE Thesaurus (RDF v1.02)
- **Composition**:
    - 800 relationships each for `skos:broader` and `skos:narrower`.
    - 800 relationships for `skos:exactMatch`, manually validated for synonymy.
    - 800 unrelated topic pairs labeled as `other`.

#### **2. Physics Domain: PhySH-Rel-875**
- **Source**: Physical Subject Headings (PhySH)
- **Composition**:
    - 250 relationships each for `skos:broader` and `skos:narrower`.
    - 125 relationships for `skos:exactMatch`, manually curated.
    - 250 unrelated topic pairs labeled as `other`.

#### **3. Biomedical Domain: MeSH-Rel-4K**
- **Source**: Medical Subject Headings (MeSH)
- **Composition**:
    - 1,000 relationships for `skos:broader`, extracted from `mesh:broaderDescriptor`.
    - 1,000 relationships for `skos:narrower`, created by reversing broader triples.
    - 1,000 relationships for `skos:exactMatch`, derived from `mesh:relatedConcept`.
    - 1,000 unrelated topic pairs labeled as `other`.

### 📂 Combined Dataset: PEM-Rel-8K
The three domain-specific datasets were merged to create **PEM-Rel-8K**, supporting cross-domain transferability experiments and fine-tuning of LLMs for ontology generation. The dataset was split into training, validation, and test sets using a 7:1:2 ratio.

| **Dataset**      | **Train** | **Validation** | **Test** | **Total** | **Percentage** |
|-------------------|-----------|----------------|----------|-----------|----------------|
| **IEEE-Rel-3K**   | 2,240     | 320            | 640      | 3,200     | 39.6%          |
| **PhySH-Rel-875** | 613       | 87             | 175      | 875       | 10.8%          |
| **MeSH-Rel-4K**   | 2,800     | 400            | 800      | 4,000     | 49.5%          |
| **PEM-Rel-8K**    | 5,653     | 807            | 1,615    | 8,075     | 100%           |

This structured dataset enables robust evaluation of LLMs for semantic relationship classification across multiple disciplines.

---

### 🖥 `code/`
This folder contains the **scripts** for identifying semantic relationships between research topic pairs. The scripts implement various strategies, including **zero-shot prompting (`STD_prompting`)**, **chain-of-thought prompting (`bCoT_prompting`)**, **fine-tuning (`fine_tuning`)**, and **cross-domain transferability (`cross_domain`)**. Access them [here](./code)


#### Task Overview
The task addressed in this study involves identifying the semantic relationship that holds between a given pair of research concepts, denoted as $c_A$ and $c_B$. This task is formalized as a single-label, multi-class classification problem, where each input pair $(c_A, c_B)$ is assigned to exactly one of the following mutually exclusive categories:

- **skos:broader**: $c_A$ is a more specific concept subsumed by the broader concept $c_B$. For example, *adaptive signal processing* is subsumed by *signal processing*.
- **skos:narrower**: $c_A$ is a broader concept that subsumes the more specific concept $c_B$. For example, *databases* subsumes *distributed databases*. This is the inverse relationship of **skos:broader**.
- **skos:exactMatch**: $c_A$ and $c_B$ are semantically equivalent and can be used interchangeably across a broad range of information retrieval tasks. For example, *haptic interface* and *haptic device*.
- **other**: This category does not define a semantic relationship. Its purpose is to provide the classifier with a mechanism to label negative examples. Without it, the classifier would be forced to assign one of the three predefined semantic relationships even when none actually applies to a given pair of topics.

#### Experiment Strategies

The experimental strategies in this study are categorized into two prompting-based approaches and three fine-tuning-based approaches:

1. **Prompting-based Strategies**:
    - **Standard Prompting**: Each topic pair is processed once using a standardised prompt template.
    - **CoT, Two-way Strategy**: Each topic pair is processed twice:
        - First, the relationship between $c_A$ and $c_B$ is identified.
        - Then, the relationship between $c_B$ and $c_A$ is identified.

2. **Fine-tuning-based Strategies**:
    - **Domain-specific Evaluation**: The LLM is fine-tuned and tested within the same discipline (e.g., fine-tuned on the training set of *MeSH-Rel-4K* and evaluated on the test set of *MeSH-Rel-4K*). This setting is expected to yield the highest performance due to domain alignment.
    - **Cross-domain Evaluation**: The LLM is fine-tuned on one discipline and tested on another (e.g., fine-tuned on the training set of *MeSH-Rel-4K* and evaluated on the test set of *PhySH-Rel-875*) as well as on the entire *PEM-Rel-8K* to assess cross-domain transferability.
    - **Multidisciplinary Evaluation**: The LLM is fine-tuned using the training set of *PEM-Rel-8K* and evaluated on its corresponding test set as well as on the test sets of the three domain-specific datasets to assess its robustness across different disciplines.


    #### Prompt Template
    A standardised prompt template is employed across all strategies and models:

    ```Classify the relationship between `[TOPIC-A]` and `[TOPIC-B]`.```

#### The table below provides an overview of the 12 LLMs used in our experiments
It includes the **Model** name, the alias adopted in this study, the number of trainable **Parameters**, the context **Window** size, and the rank and scaling factor of the low-rank adaptation matrices used in LoRA (**r** and **alpha**).

| **Model**                     | **Alias**         | **Parameters** | **Window** | **r** | **alpha** | **Link**                                                                 |
|-------------------------------|-------------------|----------------|------------|-------|-----------|---------------------------------------------------------------------------|
| mistral-7b-instruct-v0.3      | mistral-7b        | 7.25B          | 32K        | 16    | 16        | [mistral-7b-instruct-v0.3](https://huggingface.co/unsloth/mistral-7b-instruct-v0.3) |
| Mistral-Nemo-Instruct-2407    | mistral-nemo-12b  | 12.2B          | 128K       | 16    | 16        | [Mistral-Nemo-Instruct-2407](https://huggingface.co/unsloth/Mistral-Nemo-Instruct-2407) |
| Mistral-Small-Instruct-2409   | mistral-22b       | 22.2B          | 128K       | 16    | 16        | [Mistral-Small-Instruct-2409](https://huggingface.co/unsloth/Mistral-Small-Instruct-2409) |
| Llama-3.2-3B-Instruct         | llama-3b          | 3.21B          | 128K       | 256   | 128       | [Llama-3.2-3B-Instruct](https://huggingface.co/unsloth/Llama-3.2-3B-Instruct) |
| llama-2-7b-chat               | llama-chat-7b     | 7B             | 4K         | 256   | 128       | [llama-2-7b-chat](https://huggingface.co/unsloth/llama-2-7b-chat)            |
| Meta-Llama-3.1-8B-Instruct    | llama-8b          | 8.03B          | 128K       | 256   | 128       | [Meta-Llama-3.1-8B-Instruct](https://huggingface.co/unsloth/Meta-Llama-3.1-8B-Instruct) |
| gemma-2b-it                   | gemma-2b          | 2.51B          | 8K         | 256   | 128       | [gemma-2b-it](https://huggingface.co/unsloth/gemma-2b-it)                      |
| gemma-2-9b-it                 | gemma-9b          | 9.24B          | 8K         | 256   | 128       | [gemma-2-9b-it](https://huggingface.co/unsloth/gemma-2-9b-it)                  |
| gemma-2-27b-it                | gemma-27b         | 27.2B          | 8K         | 256   | 128       | [gemma-2-27b-it](https://huggingface.co/unsloth/gemma-2-27b-it)                |
| Phi-3.5-mini-instruct         | phi-3             | 3.82B          | 128K       | 256   | 128       | [Phi-3.5-mini-instruct](https://huggingface.co/unsloth/Phi-3.5-mini-instruct)  |
| phi-4                         | phi-4             | 14.7B          | 16K        | 256   | 128       | [phi-4](https://huggingface.co/unsloth/phi-4)                                |
| zephyr-sft                    | zephyr-7b         | 7.24B          | 8K         | 16    | 16        | [zephyr-sft](https://huggingface.co/unsloth/zephyr-sft)                      |

---

## 📜 License
This project is licensed under the [MIT License](https://opensource.org/licenses/MIT).

---

## 🤝 Acknowledgments
We acknowledge the creators of MeSH, PhySH, and the IEEE Thesaurus for providing the foundational taxonomies used in this study.