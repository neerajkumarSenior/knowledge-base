# Install PyMuPDF (for PDF handling), pytesseract (Python wrapper for Tesseract), pandas, and paddleocr
!pip install PyMuPDF pandas pytesseract paddleocr

# Install Tesseract OCR engine and language packs for Hindi and Sanskrit
!sudo apt update
!sudo apt install -y tesseract-ocr
!sudo apt install -y tesseract-ocr-hin # Hindi language pack
!sudo apt install -y tesseract-ocr-san # Sanskrit language pack
