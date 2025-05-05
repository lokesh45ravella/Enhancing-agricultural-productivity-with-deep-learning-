# Enhancing-agricultural-productivity-with-deep-learning-
Program for Enhancing agricultural productivity with deep learning 
import os
from flask import Flask, request, render_template, jsonify
import torch
import joblib
import numpy as np
from ultralytics import YOLO

app = Flask(_name_)
app.config['UPLOAD_FOLDER'] = 'uploads'
os.makedirs(app.config['UPLOAD_FOLDER'], exist_ok=True)

model = YOLO('models/best.pt')

@app.route('/')
def index():
    return render_template('index.html')

@app.route('/predict_disease', methods=['POST'])
def predict_disease():
    file = request.files['image']
    if not file:
        return jsonify({'error': 'No file uploaded'})
    
    img_path = os.path.join(app.config['UPLOAD_FOLDER'], file.filename)
    file.save(img_path)

    results = model(img_path)
    top_class_idx = results[0].probs.top1
    predicted_class = results[0].names[top_class_idx]
    
    disease_name = predicted_class.replace('tomato_', '')

    return jsonify({'disease': disease_name})

if _name_ == '_main_':
    app.run(debug=False)
    <!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Tomato Leaf Disease & Yield Prediction</title>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@300;400;600&display=swap');
        
        body {
            font-family: 'Montserrat', sans-serif;
            color: black;
            display: flex;
            flex-direction: column;
            align-items: center;
        }
        h1 {
            text-align: center;
            margin-top: 40px;
            margin-bottom: 60px;
        }
        .box {
            width: 50%;
            border: 1px solid #000;
            padding: 20px;
            border-radius: 10px;
        }
        #uploaded-image {
            max-width: 100%;
            height: auto;
            margin-top: 10px;
            display: none; /* Hide by default */
            border-radius: 10px;
            border: 2px solid #000;
        }
    </style>
</head>
<body>
    <h1 style="text-align:center;">Tomato Leaf Disease Prediction</h1>

    <!-- Disease Prediction -->
    <div class="box">
        <form id="disease-form" enctype="multipart/form-data">
            <label>Upload Leaf Image:</label>
            <input type="file" name="image" id="image" required><br><br>
            <button type="submit">Predict Disease</button>
        </form>
        <img id="uploaded-image" alt="Uploaded Image">
        <p id="disease-result"></p>
    </div>

    <script>
        // Display uploaded image
        document.getElementById('image').addEventListener('change', function(event) {
            const file = event.target.files[0];
            if (file) {
                const reader = new FileReader();
                reader.onload = function(e) {
                    const uploadedImage = document.getElementById('uploaded-image');
                    uploadedImage.src = e.target.result;
                    uploadedImage.style.display = 'block'; // Show the image
                };
                reader.readAsDataURL(file);
            }
        });

        // Disease Prediction
        document.getElementById('disease-form').addEventListener('submit', function(e) {
            e.preventDefault();
            const formData = new FormData();
            formData.append('image', document.getElementById('image').files[0]);

            fetch('/predict_disease', {
                method: 'POST',
                body: formData
            })
            .then(response => response.json())
            .then(data => {
                if (data.disease) {
                    document.getElementById('disease-result').innerText = Predicted Disease: ${data.disease};
                } else {
                    document.getElementById('disease-result').innerText = 'Error predicting disease.';
                }
            });
        });
    </script>
</body>
</html>
