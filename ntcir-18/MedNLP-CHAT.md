# AITOK in NTCIR-18 MedNLP-CHAT

[NTCIR-1 MedNLP-CHAT](https://sociocom.naist.jp/mednlp-chat/)

This page publishes the tools and data used by Team AITOK when conducting 
additional experiments and evaluating their accuracy following participation 
in NTCIR-18 MedNLP-CHAT.

- [NTCIR-18 MedNLP-CHAT
Determining Medical, Ethical and Legal Risks
in Patient-Doctor Conversations:
Task Overview](https://research.nii.ac.jp/ntcir/workshop/OnlineProceedings18/pdf/ntcir/01-NTCIR18-OV-MEDNLP-AramakiE.pdf)
- [AITOK at the NTCIR-18 MedNLP-CHAT to Identify Medical,
Ethical and Legal Risks in Patient-Doctor Conversations](https://research.nii.ac.jp/ntcir/workshop/OnlineProceedings18/pdf/ntcir/08-NTCIR18-MEDNLP-TaniokaH.pdf)

## Dataset

[Sample](https://sociocom.naist.jp/mednlp-chat/wp-content/uploads/sites/10/2024/12/ntcir18_mednlp-chat_german_sample_02_12_24.csv_.zip)

The data consists of a question, an answer, and labels for the answer.
The labels for each answer are objective labels (‘medicalRisk’, ‘ethicalRisk’, and ‘legalRisk’) judged by experts considering German laws and medical guidelines. 

Data size: Each language comprises approximately 200 pairs of Question, Answer, and Answer labels. Out of the 200 language pairs, 100 (each) are released as a training set. 
Questions and answers are created by humans, referencing responses from a chatbot.
Answer labels represent the evaluation of the answers, which will be estimated in this task. There are six labels comprising three objective labels (medicalRisk, ethicalRisk, and legalRisk) assigned by experts based on Japanese/German laws and medical guidelines. 

Languages: 
Step 1: Data is created.
Step 2: It is translated into the other languages. The training and test data will be translated to English and French manually by professional translators. 

## Tool

[LLMs_for_MedNLP_CHAT_Accuracy_Evaluation.ipynb](https://github.com/taniokah/AITOK/blob/main/ntcir-18/LLMs_for_MedNLP_CHAT_Accuracy_Evaluation.ipynb)

We are releasing an accuracy evaluation tool in IPython notebook (ipynb) format that can be run on Google Colaboratory.
To run this tool, launch it in the Google Colaboratory environment, create a results directory, place the ground truth data (xlsx) and LLM output results (csv) there, and then execute each cell.

<img src="https://github.com/taniokah/AITOK/blob/main/images/MedicalRisk(de).png?raw=true" width="480pt" alt="ROC Curve">Fig 1

This tool generates a ROC curve for each output result and calculates the AUC. ROC curve graphs are overlaid to compare and evaluate LLMs like Fig 1.

## Results

[results](results)

All data required for accuracy evaluation is located under the results directory.
This includes 112 questions from the German task released in NTCIR-18 MedNLP-CHAT, 
containing only the correct answers for the three risks, along with 117 CSV files 
listing the probability values (TRUE/FALSE) for the three risks across three 
languages (de/en/fr) obtained using 13 different LLMs.

## License

### Code
The analysis code is released under the MIT License (see `LICENSE`).

### Data
The dataset and results are released under the Creative Commons
Attribution 4.0 International License (CC BY 4.0) (see `LICENSE-data`).

## Acknowledgments

This work was supported by JSPS KAKENHI Grant Number JP22K12293.  
This containt is managed by Hiroki Tanioka (taniokah[at]gmail.com), since 2025.

