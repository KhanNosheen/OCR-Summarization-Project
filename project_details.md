``` python
import pytesseract          # Imports the Tesseract library to perform Optical Character Recognition (OCR) on images
import os                   # Provides functions for interacting with the operating system (e.g., handling file paths)
import cv2                  # Imports OpenCV to perform image processing tasks like resizing or thresholding
import re                   # Imports the Regular Expression module to search for and extract specific patterns of text
from docx import Document   # Imports the Document class to create and manipulate Microsoft Word files

# 1. Setup Tesseract
pytesseract.pytesseract.tesseract_cmd = r'C:\Program Files\Tesseract-OCR\tesseract.exe'    # Path For Offline OCR

# 2. Paths: Using root directory for input and output
input_folder = 'images'
full_output_path = 'combined_results.docx'

# 3. Sorting function for correct sequence (1, 2, ..., 10)
def natural_sort_key(s):
    # Splits the string into a list of numbers and text using regex, then converts numbers to integers for accurate numeric sorting
    return [int(text) if text.isdigit() else text.lower() for text in re.split('([0-9]+)', s)]

# 4. Initialize document
doc = Document()

# 5. Get and sort files
# Creates a list of all files in the directory that end with image extensions, ensuring they are valid for processing
files = [f for f in os.listdir(input_folder) if f.lower().endswith(('.png', '.jpg', '.jpeg'))]
# Sorts that list of files using the custom 'natural_sort_key' function so they are processed in a logical, human-readable order
files.sort(key=natural_sort_key)

# 6. Process images
# Iterates through each file name in the sorted list of images, reads the image, converts it to greyscale, extracts text using Tesseract OCR, and adds that text to a Word document with a page break after each image's text
for filename in files:
    img_path = os.path.join(input_folder, filename)
    image = cv2.imread(img_path)
    
    if image is None:
        # print(f"Skipping {filename}: Could not read.")
        continue
    
    greyscale_img = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    text = pytesseract.image_to_string(greyscale_img)
    
    # doc.add_heading(f"Extracted Text from: {filename}", level=1)
    doc.add_paragraph(text)
    doc.add_page_break()
    print(f"Added {filename} to document.")

# 7. Final Save
doc.save('processed_combined_results.docx')
# Ensure no other program (like Word) has this file open before running
# doc.save(full_output_path)
# print(f"All done! Saved {full_output_path}")

# Summary & MCQs Section
import nltk                 # Imports the Natural Language Toolkit for processing and analyzing human language data
import random               # Imports the random module to generate random selections or perform randomized operations
import re                   # Imports the Regular Expression module for powerful text searching and pattern matching
from docx import Document   # Imports the Document class to create and format Microsoft Word files

# 1. Point NLTK to local data folder (Offline setup)
nltk.data.path = [r'C:\Users\Alhuda HP labtop\AppData\Roaming\nltk_data']

# Imports the collection of "stop words" (common words like 'the', 'is', 'in') to be used for filtering out noise from the text
from nltk.corpus import stopwords

# 2. Load the document and CLEAN it automatically
# Change the path here if document is in a different folder
doc = Document('processed_combined_results.docx')

# Creates an empty list to store only the relevant, filtered content
clean_lines = []

# Iterates through every paragraph found in the Word document
for para in doc.paragraphs:
    # Keeps only the paragraphs that do not contain specific "noise" text and are not just empty spaces
    if "Extracted Text from" not in para.text and para.text.strip() != "":    # "Extracted Text from" is a keyword filter we added during OCR, so we skip it
        # Appends the validated, meaningful text to the cleaned list
        clean_lines.append(para.text)

# Combine into one clean text block
full_text = "\n".join(clean_lines)

# 3. Summarization Logic
def get_clean_summary(text, word_limit=800):
    # Standardize the text by replacing all line breaks with spaces to create a continuous flow
    text = text.replace('\n', ' ').strip()
    
    # Use NLTK to intelligently split the text into a list of individual sentences
    sentences = nltk.sent_tokenize(text)

    # Strategy: If the text is long, sample sentences from the start, middle, and end
    if len(sentences) > 30:
        # 0:10 takes the first 10 sentences
        # len(sentences)//2 finds the middle point, and we take 10 sentences from there
        # -10: takes the final 10 sentences
        summary_sentences = sentences[0:10] + sentences[len(sentences)//2 : len(sentences)//2 + 10] + sentences[-10:]
    else:
        # If the text is short, use all the sentences
        summary_sentences = sentences

    # Combine the chosen sentences into one cohesive paragraph separated by spaces
    summary = " ".join(summary_sentences)
    return summary

# Finally, execute the function on your processed text
final_summary = get_clean_summary(full_text)

# 4. MCQ Generation Logic
def generate_mcqs(text, full_doc_text):                            # Takes the summary text and the full document text to create multiple-choice questions
    sentences = nltk.sent_tokenize(text)                           # Splits the summary into individual sentences to use as potential question bases
    candidates = [s for s in sentences if len(s.split()) > 10]     # Only consider sentences with more than 10 words for better question quality
    random.shuffle(candidates)                                     # Shuffle to get a variety of questions each time
    
    mcq_output = []                                                # This list will hold the formatted multiple-choice questions as strings
    for i in range(min(20, len(candidates))):                      # We will generate up to 20 questions, or as many as there are candidates if fewer than 20
        sentence = candidates[i]
        words = [w for w in sentence.split() if len(w) > 5 and w.isalpha()]   # From the chosen sentence, we extract words that are longer than 5 characters and are purely alphabetic to use as potential answers
        if not words: continue
        
        target = random.choice(words)                                # Randomly select one of those words to be the "blank" in the question
        question = sentence.replace(target, "__________")            # Create the question by replacing the target word with a blank
        
        # Distractors
        all_words = [w for w in re.findall(r'\w+', full_doc_text) if len(w) > 5]
        options = list(set([target] + random.sample(all_words, min(3, len(all_words)))))
        random.shuffle(options)
        
        # Creates the header for the question (e.g., "Q1: [Question Text]")
        q_text = f"Q{i+1}: {question}\n"

        # Loops through the list of options to format each one
        for idx, opt in enumerate(options):
            # If the current option matches the correct target, add bold markers (**) around it
            marker = "**" if opt == target else ""

            # Formats each option as A), B), C), D) and adds it to the q_text block
            # chr(65 + idx) is a clever trick: 65 is the ASCII code for 'A', so 65+0='A', 65+1='B', etc.
            q_text += f"   {chr(65+idx)}) {marker}{opt}{marker}\n"

        # Adds the completed question block to your main list of MCQs
        mcq_output.append(q_text)
        # Returns the final collection of formatted questions
    return mcq_output

# 5. Save to a new Word file with both the summary and the generated MCQs
output_doc = Document()
output_doc.add_heading('Research Summary and MCQs', 0)
output_doc.add_heading('Summary', level=1)
output_doc.add_paragraph(final_summary)
output_doc.add_heading('Practice MCQs', level=1)
for mcq in generate_mcqs(final_summary, full_text):
    output_doc.add_paragraph(mcq)

output_doc.save('Final_Summary_and_MCQs.docx')

# Displaying in console for you to see immediately
print("--- SUMMARY SAVED TO 'Final_Summary_and_MCQs.docx' ---")
```
print(final_summary[:500] + "...") # Preview first 500 chars
print("\n--- MCQs generated and saved! ---")
