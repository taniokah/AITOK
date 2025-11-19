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

[results](https://github.com/taniokah/AITOK/tree/main/results)

All data required for accuracy evaluation is located under the results directory.
This includes 112 questions from the German task released in NTCIR-18 MedNLP-CHAT, 
containing only the correct answers for the three risks, along with 117 CSV files 
listing the probability values (TRUE/FALSE) for the three risks across three 
languages (de/en/fr) obtained using 13 different LLMs.

## Tool

[LLMs_for_MedNLP_CHAT_Accuracy_Evaluation.ipynb](LLMs_for_MedNLP_CHAT_Accuracy_Evaluation.ipynb)

We are releasing an accuracy evaluation tool in IPython notebook (ipynb) format that can be run on Google Colaboratory.
To run this tool, launch it in the Google Colaboratory environment, create a results directory, place the ground truth data (xlsx) and LLM output results (csv) there, and then execute each cell.

<img src="https://github.com/taniokah/AITOK/blob/main/images/MedicalRisk(de).png?raw=true" width="480pt" alt="ROC Curve">

This tool generates a ROC curve for each output result and calculates the AUC. ROC curve graphs are overlaid to compare and evaluate LLMs.

## License

### Code
The analysis code is released under the MIT License (see `LICENSE`).

### Data
The dataset and results are released under the Creative Commons
Attribution 4.0 International License (CC BY 4.0) (see `LICENSE-data`).

## Acknowledgments

This work was supported by JSPS KAKENHI Grant Number JP22K12293.  
This containt is managed by Hiroki Tanioka (taniokah[at]gmail.com), since 2025.

