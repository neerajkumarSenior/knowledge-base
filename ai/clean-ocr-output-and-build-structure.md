import json
import re
import pandas as pd
import os

# Input and Output file paths
ocr_raw_path = "/content/ocr_raw.json"
ocr_clean_path = "/content/ocr_clean.json"
chapter_index_path = "/content/chapter_index.csv"

# --- Text Cleaning Function ---
def clean_text(text):
    if not isinstance(text, str): # Ensure input is string
        return ""

    # 1. Remove null bytes and other non-printable ASCII characters
    text = text.replace('\x00', '')
    text = re.sub(r'[\x00-\x1F\x7F-\x9F]', '', text) # Remove most non-printable ASCII

    # 2. Normalize whitespace: replace multiple spaces/tabs/newlines with a single space or newline
    # First, consolidate multiple spaces into one
    text = re.sub(r'[ \t]+', ' ', text)
    # Consolidate multiple newlines into one or two, to preserve paragraph breaks
    text = re.sub(r'\n\s*\n+', '\n\n', text)
    text = re.sub(r'[\r]+', '', text) # Remove carriage returns

    # 3. Remove common OCR garbage characters and non-Devanagari/non-alphanumeric that are likely noise
    # This regex allows Devanagari script (U+0900-U+097F), basic Latin characters, digits,
    # and common punctuation marks that are typically used in text.
    # It's aggressive; adjust if more varied characters are expected.
    # Note: Specific OCR errors for broken/half/joined words are very hard to fix generically without a language model or dictionary.
    # This part focuses on removing clear 'garbage'.
    text = re.sub(r'[^\u0900-\u097F\u0020-\u007F\d\s.,;:\[\]?!\-_—()\"\']', '', text)

    # 4. Remove isolated numbers that are likely page numbers within content
    # This is a heuristic and might remove legitimate single-digit numbers
    text = re.sub(r'\b\d+\b', '', text)

    # 5. Remove duplicate consecutive lines (e.g., repeated headers/footers OCR'd as part of content)
    lines = text.split('\n')
    cleaned_lines = []
    for i, line in enumerate(lines):
        stripped_line = line.strip()
        if stripped_line:
            if not cleaned_lines or stripped_line != cleaned_lines[-1].strip():
                cleaned_lines.append(line)
    text = '\n'.join(cleaned_lines)

    # 6. Basic Devanagari specific corrections (highly heuristic, might need refinement)
    # Example: often '‍' (zero width joiner) or '‌' (zero width non-joiner) can cause issues, or extra spaces around virama
    text = text.replace('‍', '').replace('‌', '')
    text = re.sub(r'(\S) (\u094d)', r'\1\2', text) # Remove space between consonant and virama
    text = re.sub(r'(\u094d) (\S)', r'\1\2', text) # Remove space between virama and next consonant

    # 7. Trim leading/trailing whitespace from the whole text
    text = text.strip()

    return text

# Load raw OCR data
print(f"Loading raw OCR data from {ocr_raw_path}...")
with open(ocr_raw_path, 'r', encoding='utf-8') as f:
    raw_ocr_data = json.load(f)
print(f"Loaded {len(raw_ocr_data)} pages.")

# Apply cleaning
cleaned_ocr_data = []
for entry in raw_ocr_data:
    cleaned_text = clean_text(entry['text'])
    cleaned_ocr_data.append({"page": entry['page'], "text": cleaned_text})

# Save cleaned OCR data
print(f"Saving cleaned OCR data to {ocr_clean_path}...")
with open(ocr_clean_path, 'w', encoding='utf-8') as f:
    json.dump(cleaned_ocr_data, f, ensure_ascii=False, indent=4)
print("Cleaned OCR data saved successfully.")


# --- Chapter Index Generation ---
print("Generating chapter index...")
chapter_index = []
current_chapter = "Introduction"
current_chapter_start_page = 1

# Patterns for identifying chapters and sections in Hindi/Sanskrit
# These are examples; adjust based on actual book's formatting
chapter_patterns = [
    re.compile(r'(विषय सूची)', re.IGNORECASE | re.UNICODE),
    re.compile(r'(अध्याय\s*\d+)', re.IGNORECASE | re.UNICODE), # e.g., अध्याय 1, अध्याय 2
    re.compile(r'(अध्याय\s*[०-९]+)', re.IGNORECASE | re.UNICODE), # e.g., अध्याय १, अध्याय २ (Devanagari digits)
    re.compile(r'(भूमिका)', re.IGNORECASE | re.UNICODE),
    re.compile(r'(प्रस्तावना)', re.IGNORECASE | re.UNICODE),
    re.compile(r'^(.*?):\s*(\d+)$', re.IGNORECASE | re.UNICODE) # Generic heading followed by page number (less reliable)
]

for entry in cleaned_ocr_data:
    page_number = entry['page']
    page_text = entry['text']

    found_chapter = None
    for pattern in chapter_patterns:
        match = pattern.search(page_text)
        if match:
            found_chapter = match.group(0).strip()
            break

    if found_chapter:
        # If it's the first chapter or a new chapter is detected
        # Add the previous chapter entry if it's not the default 'Introduction' or already added
        if current_chapter != "Introduction" or (current_chapter == "Introduction" and not chapter_index):
             # Avoid adding 'Introduction' multiple times if it's implicitly the first block
            if not chapter_index or chapter_index[-1]['chapter'] != current_chapter:
                chapter_index.append({"chapter": current_chapter, "start_page": current_chapter_start_page})

        current_chapter = found_chapter
        current_chapter_start_page = page_number

# Add the last identified chapter
chapter_index.append({"chapter": current_chapter, "start_page": current_chapter_start_page})

# Convert to DataFrame and save to CSV
df_chapter_index = pd.DataFrame(chapter_index)
print(f"Saving chapter index to {chapter_index_path}...")
df_chapter_index.to_csv(chapter_index_path, index=False, encoding='utf-8')
print("Chapter index saved successfully.")

print("Cleaning and chapter indexing complete.")
