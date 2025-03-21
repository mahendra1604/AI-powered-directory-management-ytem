# AI-powered-directory-management-ytem
import os
import shutil
import mimetypes
from openai import OpenAI

# Initialize OpenAI client with your API key
client = OpenAI(
  api_key="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"
)

# Function to classify file type using AI
def classify_file_with_ai(file_name, context="Classify the file type"):
    try:
        # Request classification from OpenAI GPT-4 Mini model
        completion = client.chat.completions.create(
            model="gpt-4o-mini",
            store=True,
            messages=[
                {"role": "user", "content": f"{context}: {file_name}"}
            ]
        )
        # Get and return the classification from the response
        return completion.choices[0].message['content'].strip().lower()
    except Exception as e:
        print(f"AI classification failed: {e}")
        return 'unknown'

# Function to categorize files based on MIME type and AI classification
def categorize_file(file_path, custom_rules=None):
    mime_type, _ = mimetypes.guess_type(file_path)
    category = 'Others'
    file_name = os.path.basename(file_path)
    
    # Check for custom rules first
    if custom_rules and file_name in custom_rules:
        category = custom_rules[file_name]
    elif mime_type:
        # Categorize based on MIME type
        if mime_type.startswith('image'):
            category = 'Images'
        elif mime_type.startswith('video'):
            category = 'Videos'
        elif mime_type.startswith('audio'):
            category = 'Audio'
        elif mime_type.startswith('text'):
            category = 'Documents'
        else:
            # If MIME type doesn't fit, use AI classification
            category = classify_file_with_ai(file_name)
    else:
        # If MIME type is unknown, classify using AI
        category = classify_file_with_ai(file_name)
    
    # Default to 'Common' category if the classification is 'unknown'
    if category == 'unknown':
        category = 'Common'
    
    return category.capitalize()

# Function to organize files into directories
def organize_directory(directory, custom_rules=None):
    if not os.path.exists(directory):
        print(f"Directory {directory} does not exist.")
        return

    for root, _, files in os.walk(directory):
        for file in files:
            file_path = os.path.join(root, file)
            category = categorize_file(file_path, custom_rules)
            category_dir = os.path.join(directory, category)
            os.makedirs(category_dir, exist_ok=True)
            shutil.move(file_path, os.path.join(category_dir, file))
            print(f"Moved {file} to {category}/")

    print("Directory organized successfully!")

# Example usage
print("Enter the directory path:")
directory_to_organize = input(" ")
custom_rules = {'example.txt': 'Important', 'notes.docx': 'Work'}
organize_directory(directory_to_organize, custom_rules)
