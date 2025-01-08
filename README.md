# Fastshare
File sharing program 
import os
from flask import Flask, request, render_template_string, send_from_directory

app = Flask(__name__)

# Folder to save uploaded files
UPLOAD_FOLDER = os.path.join(os.getcwd(), "uploads")
if not os.path.exists(UPLOAD_FOLDER):
    os.makedirs(UPLOAD_FOLDER)

app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# HTML Template
html_template = """
<!DOCTYPE html>
<html>
<head>
    <title>Upload and View Files</title>
</head>
<body>
    <h1>Upload a File</h1>
    <form method="post" enctype="multipart/form-data" action="/upload">
        <input type="file" name="file" />
        <button type="submit">Upload</button>
    </form>
    <h2>Uploaded Files:</h2>
    <ul>
        {% for file in files %}
            <li><a href="{{ url_for('uploaded_file', filename=file) }}" target="_blank">{{ file }}</a></li>
        {% endfor %}
    </ul>
</body>
</html>
"""

@app.route('/')
def home():
    # List all files in the uploads folder
    files = os.listdir(app.config['UPLOAD_FOLDER'])
    return render_template_string(html_template, files=files)

@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return "No file part", 400
    file = request.files['file']
    if file.filename == '':
        return "No file selected", 400
    if file:
        file_path = os.path.join(app.config['UPLOAD_FOLDER'], file.filename)
        file.save(file_path)
        return home()  # Redirect to home to show the updated file list

@app.route('/uploads/<filename>')
def uploaded_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)

if __name__ == "__main__":
    app.run(debug=True, host='0.0.0.0', port=5000)
    
