import fitz  # PyMuPDF
import json
import os

# Define the output JSON path
text_raw_json_path = "/content/text_raw.json"

# Ensure the PDF file exists (pdf_path is from a previous cell)
if not os.path.exists(pdf_path):
    print(f"Error: PDF file not found at '{pdf_path}'. Please upload the PDF file to this path or update the 'pdf_path' variable.")
else:
    raw_extracted_text = []
    try:
        pdf_document = fitz.open(pdf_path)
        total_pages = pdf_document.page_count
        print(f"Total pages in PDF: {total_pages}")

        for page_num in range(total_pages):
            page = pdf_document.load_page(page_num)
            # Extract raw text directly from the page
            text = page.get_text("text") # 'text' extracts plain text

            # Append to our list of dictionaries
            raw_extracted_text.append({"page": page_num + 1, "text": text})

            if (page_num + 1) % 100 == 0 or page_num == total_pages - 1 or page_num == 0:
                print(f"Extracted text from page {page_num+1}/{total_pages}.")

        pdf_document.close()
        print(f"Successfully extracted text from all {total_pages} pages.")

        # Save the raw extracted text to a JSON file
        with open(text_raw_json_path, 'w', encoding='utf-8') as f:
            json.dump(raw_extracted_text, f, ensure_ascii=False, indent=4)

        print(f"Raw extracted text saved to '{text_raw_json_path}'.")
        print(f"Number of pages with extracted text: {len(raw_extracted_text)}")

    except Exception as e:
        print(f"An error occurred during raw text extraction from PDF: {e}")
