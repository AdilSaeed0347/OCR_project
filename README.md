# OCR_project
This is short project of OCR (Optical Character Recognition) which is extracting English text from Png , Jpeg , Jpg , Bm    and  Docx format.It will give you two options whther you want to copy text to clipboard or save it in txt fromat bu giving option 1 and 2 .


code

!pip install pytesseract
!apt-get install tesseract-ocr -y
!apt-get install libtesseract-dev -y

!pip install pytesseract python-docx opencv-python-headless
!apt-get install tesseract-ocr -y
!apt-get install -y tesseract-ocr-urd

import cv2
import pytesseract
from PIL import Image
import numpy as np
from google.colab import files
from docx import Document
import textwrap
import os
from IPython.display import display, HTML, Javascript


uploaded = files.upload()


filename = list(uploaded.keys())[0]
ext = os.path.splitext(filename)[1].lower()

text_eng = ""


if ext in ['.png', '.jpg', '.jpeg', '.bmp']:
    print(" Image file detected!")


    image = Image.open(filename)
    image_cv = cv2.cvtColor(np.array(image), cv2.COLOR_RGB2BGR)

    gray = cv2.cvtColor(image_cv, cv2.COLOR_BGR2GRAY)
    gray = cv2.equalizeHist(gray)

    thresh = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C,
                                   cv2.THRESH_BINARY, 11, 2)


    config = r'--oem 3 --psm 6'
    text_eng = pytesseract.image_to_string(thresh, lang='eng', config=config)

elif ext == '.docx':
    print(" Word document detected!")

    doc = Document(filename)
    extracted_paragraphs = [para.text for para in doc.paragraphs]
    text_eng = '\n'.join(extracted_paragraphs)

elif ext == '.pdf':
    print(" PDF file detected!")
    text_eng = "PDF OCR not implemented in this version."

else:
    print(" Unsupported file type!")


if text_eng.strip():
    wrapped_text = textwrap.fill(text_eng.strip(), width=100)

    display(HTML(f"""
    <div style="border:1px solid #ccc; padding:10px; max-height:400px; overflow:
    auto;white-space:pre-wrap; font-family:monospace; background-color:#f9f9f9;">
     English Text Extracted:
    {wrapped_text}
    </div>
    """))


    def ask_save_copy():
        print("\nDo you want to save the text to a file or copy it?")
        print("1. Save to file")
        print("2. Copy to clipboard")

        action = input("Enter 1 to save or 2 to copy: ")

        if action == '1':
            with open("english_text.txt", "w", encoding="utf-8") as f:
                f.write(text_eng)
            files.download("english_text.txt")
            print(" Text saved and ready for download.")
        elif action == '2':
            escaped_text = text_eng.replace("'", "\\'").replace("\n", "\\n").replace("\r", "\\r").replace("\t", "\\t")
            display(Javascript(f'''
                navigator.clipboard.writeText(`{escaped_text}`).then(function() {{
                    console.log("Text copied to clipboard!");
                }}).catch(function(err) {{
                    console.log("Failed to copy text:", err);
                }});
            '''))
            print(" Text copied to clipboard!")
        else:
            print(" Invalid option. Please enter 1 or 2.")

    ask_save_copy()

else:
    print(" No English text found to display.")
