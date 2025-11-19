# AITOK

This page publishes the tools and data used by Team AITOK when conducting 
additional experiments and evaluating their accuracy following participation 
in NTCIR-18 MedNLP-CHAT.

## Dataset

All data required for accuracy evaluation is located under the results directory.
This includes 112 questions from the German task released in NTCIR-18 MedNLP-CHAT, 
containing only the correct answers for the three risks, along with 117 CSV files 
listing the probability values (TRUE/FALSE) for the three risks across three 
languages (de/en/fr) obtained using 13 different LLMs.

## Tool

[LLMs_for_MedNLP_CHAT_Accuracy_Evaluation.ipynb](LLMs_for_MedNLP_CHAT_Accuracy_Evaluation.ipynb)

We are releasing an accuracy evaluation tool in IPython notebook (ipynb) format that can be run on Google Colaboratory.
To run this tool, launch it in the Google Colaboratory environment, create a results directory, place the ground truth data (xlsx) and LLM output results (csv) there, and then execute each cell.

This tool generates a ROC curve for each output result and calculates the AUC. ROC curve graphs are overlaid to compare and evaluate LLMs.

![ROC Curve](/images/MedicalRisl(de).png)

## License

### Code
The analysis code is released under the MIT License (see `LICENSE`).

### Data
The dataset and results are released under the Creative Commons
Attribution 4.0 International License (CC BY 4.0) (see `LICENSE-data`).
