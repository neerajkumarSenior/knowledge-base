from google.colab import files
import os

# Define the expected PDF path
pdf_path = "/content/book.pdf"

print("Please upload your PDF file. Ensure it is named 'book.pdf'")
print("If you upload a file with a different name, you will need to update the `pdf_path` variable accordingly.")

uploaded = files.upload()

if uploaded:
    for filename in uploaded.keys():
        print(f'User uploaded file "{filename}" with length {len(uploaded[filename])} bytes')

        # Check if the uploaded filename matches the expected pdf_path
        if os.path.basename(pdf_path) != filename:
            print(f"Warning: The uploaded file name '{filename}' does not match the expected PDF name '{os.path.basename(pdf_path)}'.")
            print("Please update the 'pdf_path' variable in subsequent cells if this is a different file.")
            # Optionally update pdf_path if a single file is uploaded and it's intended to be the new source
            pdf_path = os.path.join("/content", filename)
            print(f"Updated pdf_path to: {pdf_path}")

        # Verify if the file is now at the correct location
        if os.path.exists(pdf_path):
            print(f"PDF file successfully available at {pdf_path}")
        else:
            print(f"Error: PDF file not found at expected path {pdf_path} after upload.")
else:
    print("No file was uploaded.")
