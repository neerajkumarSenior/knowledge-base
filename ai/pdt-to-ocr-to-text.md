# STEP 1

Task:

Load scanned PDF book.

Book:
"Kya Balu Ki Bheet Par Khada Hai Hindu Dharm"

Requirements:

1. Detect total pages
2. Convert PDF pages to images
3. Run OCR for Hindi + Sanskrit
4. Keep original page number
5. Do NOT clean text
6. Do NOT chunk
7. Do NOT detect chapters
8. Extract raw text only

Output format:

{
"page":1,
"text":"OCR extracted raw text"
}

Save file:

ocr_raw.json

Process all pages.  


import fitz  # PyMuPDF
import os
import json
from PIL import Image
import pytesseract

# Set the path to the Tesseract executable (globally, if needed)
pytesseract.pytesseract.tesseract_cmd = r'/usr/bin/tesseract'

# Define paths (pdf_path is already set from previous cell)
# pdf_path = "/content/Kya Balu Ki Bheet Par Khada Hai Hindu Dharm.pdf"
output_image_dir = "/content/images"
ocr_output_json_path = "/content/ocr_raw.json"

# --- PDF to Image Conversion ---
# Ensure the PDF file exists
if not os.path.exists(pdf_path):
    print(f"Error: PDF file not found at '{pdf_path}'. Please upload the PDF file to this path or update the 'pdf_path' variable.")
else:
    # Create the output directory for images if it doesn't exist
    os.makedirs(output_image_dir, exist_ok=True)
    print(f"Output image directory '{output_image_dir}' ensured.")

    try:
        pdf_document = fitz.open(pdf_path)
        total_pages = pdf_document.page_count
        print(f"Total pages in PDF: {total_pages}")

        for page_num in range(total_pages):
            page = pdf_document.load_page(page_num)
            
            # Render page to a high-resolution pixmap (e.g., 300 DPI)
            pix = page.get_pixmap(matrix=fitz.Matrix(300/72, 300/72))

            # Define output image path
            image_filename = os.path.join(output_image_dir, f"page_{page_num+1:04d}.png")
            
            # Save the pixmap as a PNG image
            pix.save(image_filename)
            
            if (page_num + 1) % 50 == 0 or page_num == total_pages - 1 or page_num == 0:
                print(f"Converted page {page_num+1}/{total_pages} to image.")

        pdf_document.close()
        print(f"Successfully converted all {total_pages} pages to high-resolution images in '{output_image_dir}'.")

    except Exception as e:
        print(f"An error occurred during PDF to image conversion: {e}")


# --- OCR Processing ---
ocr_results = []
paddle_ocr_instance = None
paddleocr_initialized = False

# Try to initialize PaddleOCR
try:
    # Attempt to import and initialize PaddleOCR within the try block
    from paddleocr import PaddleOCR, build_logger
    logger = build_logger(name='ocr_logger', log_file='ocr.log', log_level='ERROR')
    paddle_ocr_instance = PaddleOCR(use_angle_cls=True, lang=['en', 'hi'], show_log=False, logger=logger)
    paddleocr_initialized = True
    print("PaddleOCR initialized successfully.")
except RuntimeError as e:
    # Catch the specific PDX initialization error
    if "PDX has already been initialized" in str(e):
        print("Warning: PaddleOCR initialization error (PDX already initialized). Falling back to Tesseract only.")
        print("To use PaddleOCR, please restart the kernel and run the cells again.")
    else:
        print(f"Error initializing PaddleOCR: {e}. Falling back to Tesseract only.")
except ImportError:
    # Catch if paddleocr itself failed to import (e.g., not installed correctly)
    print("Warning: PaddleOCR library not found or could not be imported. Falling back to Tesseract only.")
except Exception as e:
    # Catch any other unexpected errors during PaddleOCR setup
    print(f"An unexpected error occurred during PaddleOCR setup: {e}. Falling back to Tesseract only.")


if not os.path.exists(output_image_dir) or not os.listdir(output_image_dir):
    print(f"Error: Image directory '{output_image_dir}' not found or empty. Please ensure images are generated.")
else:
    image_files = sorted([f for f in os.listdir(output_image_dir) if f.endswith(('.png', '.jpg', '.jpeg'))])
    total_images = len(image_files)
    print(f"Found {total_images} images to process in '{output_image_dir}'.")

    for i, image_file in enumerate(image_files):
        image_path = os.path.join(output_image_dir, image_file)
        page_number = int(image_file.split('_')[-1].split('.')[0]) # Extract page number from filename

        if (i + 1) % 50 == 0 or i == total_images - 1 or i == 0:
            print(f"Performing OCR for image {i + 1}/{total_images} (Page {page_number})...")

        extracted_text = ""
        if paddleocr_initialized:
            try:
                # Try PaddleOCR first
                result = paddle_ocr_instance.ocr(image_path, cls=True)
                # Flatten the result into a single string if text is found
                if result and result[0]:
                    paddle_text = [line[1][0] for line in result[0]]
                    extracted_text = "\n".join(paddle_text)
                
                if not extracted_text.strip(): # Fallback if PaddleOCR returns empty or only whitespace
                    print(f"PaddleOCR returned empty text for image {image_file} (Page {page_number}), trying Tesseract.")
                    img = Image.open(image_path)
                    tesseract_text = pytesseract.image_to_string(img, lang='hin+san')
                    extracted_text = tesseract_text.strip()

            except Exception as e:
                # If PaddleOCR fails, try Tesseract as a fallback
                print(f"PaddleOCR failed for image {image_file} (Page {page_number}), falling back to Tesseract: {e}")
                try:
                    img = Image.open(image_path)
                    tesseract_text = pytesseract.image_to_string(img, lang='hin+san')
                    extracted_text = tesseract_text.strip()
                except Exception as e_tesseract:
                    print(f"Tesseract also failed for image {image_file} (Page {page_number}): {e_tesseract}")
                    extracted_text = ""
        else: # If PaddleOCR was not initialized, go straight to Tesseract
            try:
                img = Image.open(image_path)
                tesseract_text = pytesseract.image_to_string(img, lang='hin+san')
                extracted_text = tesseract_text.strip()
            except Exception as e_tesseract:
                print(f"Tesseract failed for image {image_file} (Page {page_number}): {e_tesseract}")
                extracted_text = ""

        # Store the page number and extracted text
        ocr_results.append({"page": page_number, "text": extracted_text.strip()})

    # Save the list of OCR results to a JSON file
    with open(ocr_output_json_path, 'w', encoding='utf-8') as f:
        json.dump(ocr_results, f, ensure_ascii=False, indent=4)

    print(f"OCR results saved to '{ocr_output_json_path}'.")
    print(f"Number of pages with OCR results: {len(ocr_results)}")
