# STEP 2 — Clean OCR + Build Structure

Input:

ocr_raw.json

Format:

{
"page":1,
"text":"..."
}

Task:

Clean OCR output.

Remove:

- repeated headers
- repeated footers
- page numbers inside content
- extra spaces
- OCR garbage characters
- duplicate lines

Fix:

- broken Hindi words
- Sanskrit OCR issues
- half characters
- joined words

Do NOT summarize

Do NOT chunk

Do NOT change meaning

Keep page number unchanged

Output:

{
"page":1,
"text":"clean text"
}

Save:

ocr_clean.json


Next:

Search entire text for:

- विषय सूची
- अध्याय
- भूमिका
- headings
- chapter names
- page references

Generate:

chapter_index.csv

Format:

chapter,start_page
