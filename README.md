# heiDS at ArchEHR-QA 2025: From Fixed-k to Query-dependent-k for Retrieval Augmented Generation


## Important

* Link for the shared task: [BioNLP @ACL 2025 Shared Task on grounded question answering (QA) from electronic health records (EHRs)](https://archehr-qa.github.io/).
* The dataset for the shared task is available on [PhysioNet](https://doi.org/10.13026/zzax-sy62). 
* The proposed pipeline is present under [notebook](notebook).
* The scoring script is available under [evaluation](evaluation).

## Abstract

As part of the participation of the heiDS team in the ArchEHR-QA 2025 shared task, we present a pipeline utilizing a retrieval-augmented generation (RAG) framework designed to generate answers that are attributed to clinical evidence from patients' electronic health records (EHRs) in response to patient-specific questions. We explored various components of a RAG framework, focusing  on ranked list truncation (RLT) retrieval strategies and  attribution approaches. Instead of using a fixed top-$k$ RLT retrieval strategy, we employ the autocut*, surprise, and elbow methods, which allow for a query-dependent-k retrieval to construct an answer. Our results show the benefits of this strategy in producing factual and relevant answers when compared to a fixed-k.


# Original README: ArchEHR-QA: BioNLP at ACL 2025 Shared Task on Grounded Electronic Health Record Question Answering

## Abstract

Responding to patients’ medical inbox messages through patient portals is one of the main contributors to increasing clinician burden. To this end, automatically generating answers to questions from patients considering their medical records is important. The overarching goal of the ArchEHR-QA 2025 shared task is to develop automated responses to patients' questions by generating answers that are grounded in key clinical evidence from their electronic health records (EHRs). The proposed dataset, ArchEHR-QA, comprises hand-curated, realistic patient questions (reflective of patient portal messages), relevant focus areas identified within these questions (as determined by a clinician), corresponding clinician-rewritten versions (crafted to aid in formulating responses), and note excerpts providing essential clinical context. The task is to construct coherent answers to input questions that must be grounded in the provided clinical note excerpts.

## Objective

The volume of messages on the patient portals is on the rise, which includes requests for medical information from clinicians [1, 2]. This is one of the main contributors to desktop medicine and increasing clinician burden. One approach to handling the increasing messaging burden is to assist clinicians in formulating the responses. Thus, the primary objective of the challenge is to automatically respond to input patient questions by constructing coherent answers that must only be based on and grounded in the associated clinical note excerpts.

## Data Description
​
The dataset consists of questions (inspired by real patient questions) and associated EHR data (derived from MIMIC-III [3] and MIMIC-IV [4] databases) containing important clinical evidence to answer these questions. Each instance of the question-note pairs is referred to as a "case". Clinical note excerpts come pre-annotated with sentence numbers which must be used to cite the clinical evidence sentences in system responses. Each sentence is manually annotated with a "relevance" label to mark its importance in answering the given question as `"essential"`, `"supplementary"`, or `"not-relevant"`.

The development set contains a total of 20 patient cases, and test set contains a total of 100 patient cases.

## Format

The dataset is provided as XML and JSON files.

### Cases

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

### Keys and Mappings

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

 
## Evaluation

See [evaluation](evaluation) for the scoring script.

The submissions will be evaluated for both the quality of generated answers and the use of clinical evidence for grounding. The evidence sentences cited in the generated answers will be evaluated using Precision, Recall, and F1 Scores using a manually annotated ground truth set of evidence sentences. The alignment of sentences in the generated answer with the cited evidence sentence(s) from the clinical notes will be assessed using BLEU [5], ROUGE [6], SARI [7], BERTScore [8], AlignScore [9], and MEDCON [10]. For each submission, an "Overall Score" for alignment is reported as the arithmetic mean of alignment scores using different metrics.

## Submission

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
        "answer": "His aortic aneurysm was caused by the rupture of a thoracoabdominal aortic aneurysm, which required emergent surgical intervention. |1|\nHe underwent a complex salvage repair using a 34-mm Dacron tube graft and deep hypothermic circulatory arrest to address the rupture. |2,3|\nPostoperatively, he required close monitoring in the intensive care unit.\nThe extended recovery time and hospital stay were necessary due to the severity of the rupture and the complexity of the surgery, though his wound is now healing well with only a small open area noted. |8|"
    }
    ...
]
```

## References

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
