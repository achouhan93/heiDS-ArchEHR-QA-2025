<a name="readme-top"></a>

[![Contributors][contributors-shield]][contributors-url]
[![Forks](https://img.shields.io/github/forks/achouhan93/heiDS-ArchEHR-QA-2025.svg?style=for-the-badge)](https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/forks)
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]

<!-- PROJECT LOGO -->
<br />
<div align="center">
  <a href="https://github.com/achouhan93/heiDS-ArchEHR-QA-2025">
    <img src="images/cutting.png" alt="Logo" width="80" height="80">
  </a>

  <h3 align="center">heiDS at ArchEHR-QA 2025: From Fixed-k to Query-dependent-k for Retrieval Augmented Generation</h3>

  <p align="center">
    Ashish Chouhan and Michael Gertz

    Data Science Group, Institute of Computer Science

    Heidelberg University, Germany

    Contact us at: [`{chouhan, gertz}@informatik.uni-heidelberg.de`](mailto:chouhan@informatik.uni-heidelberg.de) 
    <a href="https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/issues">Report Bug</a> · 
    <a href="https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/issues">Request Feature</a>
  </p>
</div>

---

## Table of Contents

- [About The Project](#about-the-project)
  - [Abstract](#abstract)
  - [System Workflow](#system-workflow)
- [Getting Started](#getting-started)
  - [Environment Setup](#environment-setup)
  - [Running the Pipeline](#running-the-pipeline)
- [Experimentation](#experimentation)
- [Submission Format](#submission-format)
- [Acknowledgments](#acknowledgments)
- [References](#references)
- [License](#license)

---

## 📌 About The Project

This repository contains the implementation of the **heiDS pipeline**, a system developed for the **ArchEHR-QA 2025 Shared Task on Grounded Electronic Health Record Question Answering**, part of the BioNLP Workshop at ACL 2025.

### Abstract

The ArchEHR-QA task focuses on generating factual, evidence-grounded answers to patient-specific questions based on structured electronic health records. The **heiDS-QA** system addresses this through robust note retrieval and large language model-based generation. In particular, our submission explores dynamic, query-dependent selection of top-\(k\) evidence sentences, instead of static retrieval, to improve grounding and answer precision.

### System Workflow

1. Parse EHR case XML and extract questions and note fragments.
2. Embed and index sentences using dense retrieval (FAISS).
3. Retrieve top-k sentences with options for dynamic selection and re-ranking.
4. Generate grounded answers using instruction-tuned LLMs.
5. Attribute answer content to evidence using structured citation (e.g., `|1|`).

---

## 🚀 Getting Started

### 🔧 Environment Setup

```bash
pip install -qU \
  langchain-community langchain-huggingface huggingface_hub langchain-core faiss-gpu-cu11 \
  flashrank cohere langchain_cohere ruptures evaluate bert_score fuzzywuzzy rouge_score \
  "numpy<2.0.0" "transformers==4.37.0" context-cite
```

```bash
python -m spacy download en_core_web_sm
```

Set credentials (for use in Colab):

```python
from google.colab import userdata
hf_auth = userdata.get('HUGGINGFACEHUB_API_TOKEN')
cohere_key = userdata.get('COHERE_API_KEY')
```

---

## Running the Pipeline

Clone and navigate to the repository. The main script is `heiDS_archehr-qa-pipeline.ipynb`.

### Development Evaluation

```python
pipeline_config = PipelineConfig(
    xml_file='./dataset/archehr-qa.xml',
    mode='dev',
    output_file='./results/dev_submission.json',
    index_dir='./faiss-index-IP',
    sim_k=54,
    dynamic_k=True,
    dynamic_mode='surprise'
)
```

### Test Submission

```python
pipeline_config.mode = 'test'
pipeline_config.output_file = './results/test_submission.json'
main(pipeline_config, generation_config)
```

---

## Experimentation

- **Retrieval**: Top-k (static), dynamic-k (surprise, autocut, elbow), re-ranking (FlashRank, Cohere)
- **Generation**: Prompting styles (zero/one-shot), generation and citation methods (pre/post)
- **Evaluation**: Precision, Recall, F1 for retrieval; automatic citation inclusion and semantic attribution

---

## Submission Format

```json
[
  {
    "case_id": "1",
    "answer": "The patient was admitted for emergent repair of thoracoabdominal aneurysm. |1| ..."
  }
]
```

---

## Acknowledgments
- LLMs accessed via HuggingFace Inference API and Cohere
- ArchEHR-QA shared task organizers and official evaluation scripts

---

## References

- [ArchEHR-QA](https://archehr-qa.github.io)
- [MIMIC-III](https://doi.org/10.13026/C2XW26)
- [MIMIC-IV](https://doi.org/10.13026/1n74-ne17)
- [Mixtral](https://huggingface.co/mistralai/Mixtral-8x7B-Instruct-v0.1)

---

## License

The software in this repository is released under the [MIT License](LICENSE). Please refer to the license file for details.

---

<!-- MARKDOWN LINKS & IMAGES -->
[contributors-shield]: https://img.shields.io/github/contributors/heidelberg-nlp/heiDS-QA.svg?style=for-the-badge
[contributors-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/graphs/contributors
[stars-shield]: https://img.shields.io/github/stars/heidelberg-nlp/heiDS-QA.svg?style=for-the-badge
[stars-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/stargazers
[issues-shield]: https://img.shields.io/github/issues/heidelberg-nlp/heiDS-QA.svg?style=for-the-badge
[issues-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/issues
[license-shield]: https://img.shields.io/github/license/heidelberg-nlp/heiDS-QA.svg?style=for-the-badge
[license-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/blob/main/LICENSE