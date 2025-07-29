<a name="readme-top"></a>

[![Contributors][contributors-shield]][contributors-url]
[![Forks](https://img.shields.io/github/forks/achouhan93/heiDS-ArchEHR-QA-2025.svg?style=for-the-badge)](https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/forks)
[![Stargazers][stars-shield]][stars-url]
[![Issues][issues-shield]][issues-url]
[![MIT License][license-shield]][license-url]
![Visitors](https://api.visitorbadge.io/api/VisitorHit?user=achouhan93&repo=heiDS-ArchEHR-QA-2025&countColor=%237B1E7A)

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

- [About The Project](#-about-the-project)
  - [Abstract](#abstract)
  - [System Workflow](#%EF%B8%8F-system-workflow)
- [Getting Started](#-getting-started)
  - [Environment Setup](#-environment-setup)
  - [Running the Pipeline](#running-the-pipeline)
  - [License](#license)
- [Original README: ArchEHR-QA](#-original-readme-archehr-qa-bionlp-at-acl-2025-shared-task-on-grounded-electronic-health-record-question-answering)
  - [Abstract](#abstract-1)
  - [Objective](#objective)
  - [Data Description](#data-description)
  - [Data Format](#format)
  - [Evaluation](#evaluation)
  - [Submission Format](#submission)
  - [Acknowledgments](#acknowledgments)
  - [References](#references)

---

## 📌 About The Project

This repository contains the implementation of the **heiDS** team, a pipeline developed for the **ArchEHR-QA 2025 Shared Task on Grounded Electronic Health Record Question Answering**, part of the BioNLP Workshop at ACL 2025. The project aims to generate grounded, evidence-based answers to patient questions using clinical note excerpts from EHRs.

- 🔗 [ArchEHR-QA Shared Task Website](https://archehr-qa.github.io/)
- 📚 Dataset Access: [PhysioNet DOI](https://doi.org/10.13026/zzax-sy62)
- Arxiv version of the paper: [here](https://arxiv.org/abs/2506.19512)

In case you find the results and approach useful, kindly consider citing our work along with the shared task paper (see below):
```
@inproceedings{chouhan-gertz-2025-heids,
    title = "hei{DS} at {A}rch{EHR}-{QA} 2025: From Fixed-k to Query-dependent-k for Retrieval Augmented Generation",
    author = "Chouhan, Ashish  and
      Gertz, Michael",
    editor = "Soni, Sarvesh  and
      Demner-Fushman, Dina",
    booktitle = "Proceedings of the 24th Workshop on Biomedical Language Processing (Shared Tasks)",
    month = aug,
    year = "2025",
    address = "Vienna, Austria",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2025.bionlp-share.6/",
    pages = "50--61",
    ISBN = "979-8-89176-276-3",
    abstract = "This paper presents the approach of our team called heiDS for the ArchEHR-QA 2025 shared task. A pipeline using a retrieval augmented generation (RAG) framework is designed to generate answers that are attributed to clinical evidence from the electronic health records (EHRs) of patients in response to patient-specific questions. We explored various components of a RAG framework, focusing on ranked list truncation (RLT) retrieval strategies and attribution approaches. Instead of using a fixed top-k RLT retrieval strategy, we employ a query-dependent-k retrieval strategy, including the existing surprise and autocut methods and two new methods proposed in this work, autocut* and elbow. The experimental results show the benefits of our strategy in producing factual and relevant answers when compared to a fixed-k."
}
```


### Abstract
This paper presents the approach of our team called heiDS for the ArchEHR-QA 2025 shared task. A pipeline using a retrieval augmented generation (RAG) framework is designed to generate answers that are attributed to clinical evidence from the electronic health records (EHRs) of patients  in response to patient-specific questions. We explored various components of a RAG framework, focusing  on ranked list truncation (RLT) retrieval strategies and  attribution approaches. Instead of using a fixed top-k RLT retrieval strategy, we employ a query-dependent-k retrieval strategy, including the existing surprise and autocut methods and two new methods proposed in this work, autocut* and elbow. The experimental results show the benefits of our strategy in producing factual and relevant answers when compared to a fixed-k.

### ⚙️ System Workflow

1. **XML Parsing**: Extract questions and clinical note sentences from patient case XML.
2. **Sentence Indexing**: Encode and index sentences using dense retrieval (FAISS).
3. **Retrieval**: Top-k and query-dependent sentence selection (surprise, autocut, autocut*, elbow).
4. **Re-ranking**: Optional with FlashRank or Cohere APIs.
5. **Generation**: LLM-based answer generation (Models: Mixtral and LLaMA-3.3-70B, Prompting approach: zero-shot and one-shot prompting).
6. **Attribution**: Cite note evidence (e.g., `|2,3|`) using structured formats.

---

## 🚀 Getting Started

### 🔧 Environment Setup

```bash
pip install -qU \
  langchain-community langchain-huggingface huggingface_hub langchain-core faiss-gpu-cu11 \
  flashrank cohere langchain_cohere ruptures evaluate bert_score fuzzywuzzy rouge_score \
  "numpy<2.0.0" "transformers==4.37.0"
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

Clone and navigate to the repository. The pipeline is designed in a jupyter notebook `heiDS_archehr-qa-pipeline.ipynb` in the [notebook](notebook) folder. Follow the execution steps present in the notebook inorder to reproduce the results.

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

- **Retrieval**: Top-k (static), dynamic-k (surprise, autocut, autocut*, elbow), re-ranking (FlashRank, Cohere)
- **Generation**: Prompting styles (zero/one-shot), models ([LLaMA-3.3-70B](https://huggingface.co/meta-llama/Llama-3.3-70B-Instruct)/[Mixtral-8x7B](https://huggingface.co/mistralai/Mixtral-8x7B-v0.1)) and attribution approaches (pre/post)
- **Evaluation**: Precision, Recall, F1 for retrieval; automatic citation inclusion and semantic attribution

---

## License

The code in this work is licensed under the [MIT License](LICENSE). Please refer to the license file for details. The datasets used in this repository are distributed by the ArchEHR-QA 2025 Shared Task Organizers. Their use is governed by the terms and conditions provided by the organizers.Refer to the official [ArchEHR-QA website](https://archehr-qa.github.io/) for dataset licensing and access details.

<!-- MARKDOWN LINKS & IMAGES -->
[contributors-shield]: https://img.shields.io/github/contributors/achouhan93/heiDS-ArchEHR-QA-2025.svg?style=for-the-badge
[contributors-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/graphs/contributors
[stars-shield]: https://img.shields.io/github/stars/achouhan93/heiDS-ArchEHR-QA-2025.svg?style=for-the-badge
[stars-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/stargazers
[issues-shield]: https://img.shields.io/github/issues/achouhan93/heiDS-ArchEHR-QA-2025.svg?style=for-the-badge
[issues-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/issues
[license-shield]: https://img.shields.io/github/license/achouhan93/heiDS-ArchEHR-QA-2025.svg?style=for-the-badge
[license-url]: https://github.com/achouhan93/heiDS-ArchEHR-QA-2025/blob/main/LICENSE

---

## 📖 Original README: ArchEHR-QA: BioNLP at ACL 2025 Shared Task on Grounded Electronic Health Record Question Answering

### Abstract

Responding to patients’ medical inbox messages through patient portals is one of the main contributors to increasing clinician burden. To this end, automatically generating answers to questions from patients considering their medical records is important. The overarching goal of the ArchEHR-QA 2025 shared task is to develop automated responses to patients' questions by generating answers that are grounded in key clinical evidence from their electronic health records (EHRs). The proposed dataset, ArchEHR-QA, comprises hand-curated, realistic patient questions (reflective of patient portal messages), relevant focus areas identified within these questions (as determined by a clinician), corresponding clinician-rewritten versions (crafted to aid in formulating responses), and note excerpts providing essential clinical context. The task is to construct coherent answers to input questions that must be grounded in the provided clinical note excerpts.

### Objective

The volume of messages on the patient portals is on the rise, which includes requests for medical information from clinicians [1, 2]. This is one of the main contributors to desktop medicine and increasing clinician burden. One approach to handling the increasing messaging burden is to assist clinicians in formulating the responses. Thus, the primary objective of the challenge is to automatically respond to input patient questions by constructing coherent answers that must only be based on and grounded in the associated clinical note excerpts.

### Data Description
​
The dataset consists of questions (inspired by real patient questions) and associated EHR data (derived from MIMIC-III [3] and MIMIC-IV [4] databases) containing important clinical evidence to answer these questions. Each instance of the question-note pairs is referred to as a "case". Clinical note excerpts come pre-annotated with sentence numbers which must be used to cite the clinical evidence sentences in system responses. Each sentence is manually annotated with a "relevance" label to mark its importance in answering the given question as `"essential"`, `"supplementary"`, or `"not-relevant"`.

The development set contains a total of 20 patient cases, and test set contains a total of 100 patient cases.

### Format

The dataset is provided as XML and JSON files.

#### Cases

The main data file, `archehr-qa.xml`, contains the cases in the following format:

```xml
<annotations>
    ...
    <case id="5">
        <patient_narrative>
I am 48 years old. On February 20, I passed out, was taken to the hospital, and had two other episodes. I have chronic kidney disease with creatine around 1.5. I had anemia and hemoglobin was 10.3. I was in ICU 8 days and discharged in stable condition. My doctor performed a cardiac catherization. I had no increase in cardiac enzymes and an ECHO in the hospital showed 25% LVEF. Was this invasive, risky procedure necessary.
        </patient_narrative>
        <patient_question>
            <phrase id="0" start_char_index="254">
My doctor performed a cardiac catherization.
            </phrase>
            <phrase id="1" start_char_index="381">
Was this invasive, risky procedure necessary.
            </phrase>
        </patient_question>
        <clinician_question>
Why was cardiac catheterization recommended to the patient?
        </clinician_question>
        <note_excerpt>
History of Present Illness:

...

Brief Hospital Course:

...
        </note_excerpt>
        <note_excerpt_sentences>
            <sentence id="0" paragraph_id="0" start_char_index="0">
History of Present Illness:
            </sentence>
            <sentence id="1" paragraph_id="1" start_char_index="0">
...
            </sentence>
            ...
        </note_excerpt_sentences>
    </case>
    ...
</annotations>
```

Here, the XML elements represent:

* `<case>`: each patient case with its `"id"`.
* `<patient_narrative>`: full patient narrative question.
* `<patient_question>`: key phrases in the narrative identified as focal points related to the patient’s question.
    * `<phrase>`: each annotated phrase with `"id"` and `"start_char_index"`.
* `<clinician_question>`: question posed by a clinician.
* `<note_excerpt>`: clinical note excerpt serving as evidence.
* `<note_excerpt_sentences>`: annotated sentences in the note excerpt.
    * `<sentence>`: each annotated sentence with `"id"`, `"paragraph_id"`, and `"start_char_index"` in the paragraph.

#### Keys and Mappings

The answer keys are present in `archehr-qa_key.json` (only for the development set), which is structured as follows:

```json
[
    {
        "case_id": "5",
        "answers": [
            {
                "sentence_id": "0",
                "relevance": "essential"
            },
            ...
            {
                "sentence_id": "5",
                "relevance": "supplementary"
            },
            ...
            {
                "sentence_id": "7",
                "relevance": "not-relevant"
            },
            ...
        ]
    },
    {
        ...
    }
]
```

Here, each dictionary in the JSON array contains:

* `"case_id"`: case ID from the XML data file.
* `"answers"`: relevance labels for each sentence
    * `"sentence_id"`: sentence id from the pre-annotated sentences in the XML data file
    * `"relevance"`: annotated label indicating the importance of this sentence to answer the given question.

The ID mappings are available in `archehr-qa_mapping.json`, which is structured as follows:

```json
[
    {
        "case_id": "1",
        "document_id": "179164_41762",
        "document_source": "mimic-iii"
    },
    ...
    {
        "case_id": "11",
        "document_id": "22805349",
        "document_source": "mimic-iv"
    },
    ...
]
```

Here, each dictionary in the JSON array has:

* `"case_id"`: case ID from the XML data file.
* `"document_id"`: refers to note ID in the corresponding database -- `{HADM_ID}_{ROW_ID}` for `"mimic-iii"` and `{hadm_id}` for `"mimic-iv"`
* `"document_source"`: version of the mimic database this note was sourced from

 
### Evaluation

See [evaluation](evaluation) for the scoring script.

The submissions will be evaluated for both the quality of generated answers and the use of clinical evidence for grounding. The evidence sentences cited in the generated answers will be evaluated using Precision, Recall, and F1 Scores using a manually annotated ground truth set of evidence sentences. The alignment of sentences in the generated answer with the cited evidence sentence(s) from the clinical notes will be assessed using BLEU [5], ROUGE [6], SARI [7], BERTScore [8], AlignScore [9], and MEDCON [10]. For each submission, an "Overall Score" for alignment is reported as the arithmetic mean of alignment scores using different metrics.

### Submission

Submitted responses for each case must include one sentence per line with the cited note excerpt sentence ID(s) enclosed in pipe symbols ("|") at the end.

> His aortic aneurysm was caused by the rupture of a thoracoabdominal aortic aneurysm, which required emergent surgical intervention. |1| \
> He underwent a complex salvage repair using a 34-mm Dacron tube graft and deep hypothermic circulatory arrest to address the rupture. |2,3| \
> Postoperatively, he required close monitoring in the intensive care unit. \
> The extended recovery time and hospital stay were necessary due to the severity of the rupture and the complexity of the surgery, though his wound is now healing well with only a small open area noted. |8|

The following is an example `submission.json`.
```json
[
  {
    "case_id": "1",
    "answer": "The patient was admitted for emergent repair of thoracoabdominal aneurysm. |1| ..."
  }
]
```

### Acknowledgments

This work was supported by the Division of Intramural Research (DIR) of the National Library of Medicine (NLM), National Institutes of Health, and utilized the computational resources of the NIH HPC Biowulf cluster (https://hpc.nih.gov). The content is solely the responsibility of the authors and does not necessarily represent the official views of the National Institutes of Health.

### References

1. Martinez, K. A., Schulte, R., Rothberg, M. B., Tang, M. C., & Pfoh, E. R. (2024). Patient portal message volume and time spent on the EHR: an observational study of primary care clinicians. Journal of General Internal Medicine, 39(4), 566-572. https://doi.org/10.1007/s11606-023-08577-7
2. Holmgren, A. J., Byron, M. E., Grouse, C. K., & Adler-Milstein, J. (2023). Association between billing patient portal messages as e-visits and patient messaging volume. Jama, 329(4), 339-342. https://doi.org/10.1001/jama.2022.24710
3. Johnson, A., Pollard, T., & Mark, R. (2016). MIMIC-III Clinical Database (version 1.4). PhysioNet. https://doi.org/10.13026/C2XW26
4. Johnson, A., Pollard, T., Horng, S., Celi, L. A., & Mark, R. (2023). MIMIC-IV-Note: Deidentified free-text clinical notes (version 2.2). PhysioNet. https://doi.org/10.13026/1n74-ne17
5. Papineni, K., Roukos, S., Ward, T., & Zhu, W. J. (2002, July). Bleu: a method for automatic evaluation of machine translation. In Proceedings of the 40th annual meeting of the Association for Computational Linguistics (pp. 311-318). https://doi.org/10.3115/1073083.1073135
6. Lin, C. Y. (2004, July). Rouge: A package for automatic evaluation of summaries. In Text summarization branches out (pp. 74-81). https://aclanthology.org/W04-1013/
7. Xu, W., Napoles, C., Pavlick, E., Chen, Q., & Callison-Burch, C. (2016). Optimizing statistical machine translation for text simplification. Transactions of the Association for Computational Linguistics, 4, 401-415. https://aclanthology.org/Q16-1029/
8. Zhang, T., Kishore, V., Wu, F., Weinberger, K. Q., & Artzi, Y. (2020). BERTScore: Evaluating Text Generation with BERT. International Conference on Learning Representations. https://openreview.net/forum?id=SkeHuCVFDr
9. Zha, Y., Yang, Y., Li, R., & Hu, Z. (2023, July). AlignScore: Evaluating Factual Consistency with A Unified Alignment Function. In Proceedings of the 61st Annual Meeting of the Association for Computational Linguistics (Volume 1: Long Papers) (pp. 11328-11348). https://doi.org/10.18653/v1/2023.acl-long.634
10. Yim, W. W., Fu, Y., Ben Abacha, A., Snider, N., Lin, T., & Yetisgen, M. (2023). Aci-bench: a novel ambient clinical intelligence dataset for benchmarking automatic visit note generation. Scientific data, 10(1), 586. https://doi.org/10.1038/s41597-023-02487-3
