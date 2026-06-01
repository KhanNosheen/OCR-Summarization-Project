# OCR and Summarization Automation Pipeline

## Project Overview
This repository contains an OCR and summarization project. 

### Project Access
Please note: GitHub occasionally experiences issues rendering Jupyter Notebooks (`.ipynb`). 
If the notebook preview is not loading, please [**view the project code and details here**](project_details.md) for a reliable, fully formatted version.

This project is an offline-first automation tool designed to process scanned document images, perform Optical Character Recognition (OCR), generate concise summaries, and create multiple-choice questions (MCQs) for study purposes.

## Key Features
* **Offline OCR**: Processes images locally using Tesseract OCR, ensuring data privacy.
* **Intelligent Summarization**: Uses a balanced sampling heuristic (beginning, middle, end) to generate meaningful summaries of long documents.
* **Automated MCQ Generation**: Dynamically creates study quizzes by extracting keywords and generating plausible distractors from the document text.
* **Portable Output**: Exports all findings into a clean, formatted Word document.

## Technologies Used
* Python
* OpenCV (Image processing)
* Pytesseract (OCR)
* NLTK (Natural Language Processing)
* python-docx (Document generation)

## How It Works
1. **OCR Stage**: Scans folder of images, performs greyscale conversion, and extracts text.
2. **Analysis Stage**: Filters out noise and generates a summary using NLTK sentence tokenization.
3. **Quiz Stage**: Randomly selects keywords to create blanks and shuffles distractor words to create unique MCQs.

## Sample Results
I have included `Final_Summary_and_MCQs.docx` in this repository as an example of the generated output for recruiters to review.
