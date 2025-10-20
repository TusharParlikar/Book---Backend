# Chapter 18: Machine Learning Model Integration 🧠

**Deploy ML Models in Production & Build Intelligent Backend Systems**

---

## 📚 What You'll Learn

- Integrating pre-trained ML models in Node.js
- Using Python ML models with Flask/FastAPI
- TensorFlow.js for in-browser predictions
- Scikit-learn model deployment
- Building recommendation systems
- Image classification with YOLO/ResNet
- Natural Language Processing (NLP)
- Model versioning and A/B testing

**Time Required:** 10-12 hours  
**Difficulty:** Expert  
**Prerequisites:** B01-B17, Basic Python knowledge

---

## 🎯 Learning Objectives

By the end of this chapter, you will:

✅ Deploy ML models in production  
✅ Integrate Python ML services with Node.js  
✅ Build recommendation engines  
✅ Implement image classification  
✅ Use NLP for sentiment analysis  
✅ Create real-time prediction APIs  
✅ Monitor model performance  
✅ Handle model versioning  

---

## 🏗️ Architecture Patterns

### Pattern 1: Python Microservice

```
React → Node.js API → Python ML Service (Flask/FastAPI)
                    ↓
                Database
```

**Best for:** Complex ML models (TensorFlow, PyTorch, Scikit-learn)

### Pattern 2: TensorFlow.js in Node

```
React → Node.js API (with TensorFlow.js)
                    ↓
                Database
```

**Best for:** Simple models, real-time predictions, no Python dependency

### Pattern 3: Cloud ML APIs

```
React → Node.js API → Google Cloud AI / AWS SageMaker
                    ↓
                Database
```

**Best for:** Pre-trained models, easy scaling

---

## 🐍 Python ML Service with Flask

### Step 1: Create Flask App

**File:** `ml-service/app.py`

```python
from flask import Flask, request, jsonify
import pickle
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer

app = Flask(__name__)

# Load trained model
with open('models/sentiment_model.pkl', 'rb') as f:
    model = pickle.load(f)

with open('models/vectorizer.pkl', 'rb') as f:
    vectorizer = pickle.load(f)

@app.route('/predict/sentiment', methods=['POST'])
def predict_sentiment():
    data = request.get_json()
    text = data.get('text', '')

    # Vectorize text
    features = vectorizer.transform([text])
    
    # Predict
    prediction = model.predict(features)[0]
    probability = model.predict_proba(features)[0]

    return jsonify({
        'sentiment': 'positive' if prediction == 1 else 'negative',
        'confidence': float(max(probability)),
        'scores': {
            'positive': float(probability[1]),
            'negative': float(probability[0])
        }
    })

@app.route('/predict/rating', methods=['POST'])
def predict_rating():
    data = request.get_json()
    features = np.array([[
        data['price'],
        data['reviews_count'],
        data['seller_rating']
    ]])

    rating = model.predict(features)[0]
    
    return jsonify({
        'predicted_rating': float(rating)
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

### Step 2: Train Simple Model

```python
# train_model.py
from sklearn.linear_model import LogisticRegression
from sklearn.feature_extraction.text import TfidfVectorizer
import pickle

# Sample training data
reviews = [
    "This product is amazing!",
    "Terrible quality, waste of money",
    "Best purchase ever",
    "Do not buy this"
]
labels = [1, 0, 1, 0]  # 1 = positive, 0 = negative

# Vectorize
vectorizer = TfidfVectorizer(max_features=1000)
X = vectorizer.fit_transform(reviews)

# Train
model = LogisticRegression()
model.fit(X, labels)

# Save
with open('models/sentiment_model.pkl', 'wb') as f:
    pickle.dump(model, f)

with open('models/vectorizer.pkl', 'wb') as f:
    pickle.dump(vectorizer, f)
```

### Step 3: Docker Setup

**File:** `ml-service/Dockerfile`

```dockerfile
FROM python:3.10-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 5000

CMD ["python", "app.py"]
```

**File:** `ml-service/requirements.txt`

```
flask==2.3.0
scikit-learn==1.3.0
numpy==1.24.0
```

### Step 4: Run with Docker

```bash
cd ml-service
docker build -t ml-service .
docker run -p 5000:5000 ml-service
```

---

## 🔗 Connect Node.js to Python Service

### In Node.js Backend

```javascript
const axios = require('axios');

const ML_SERVICE_URL = process.env.ML_SERVICE_URL || 'http://localhost:5000';

// Predict sentiment
router.post('/reviews/:productId/sentiment', auth, async (req, res) => {
  const { text } = req.body;
  if (!text) {
    return res.status(400).json({ message: 'Review text is required' });
  }

  try {
    // Call Python ML service
    const response = await axios.post(`${ML_SERVICE_URL}/predict/sentiment`, { text });
    const { sentiment, confidence, scores } = response.data;

    // Save review with sentiment
    const review = await Review.create({
      user: req.user._id,
      product: req.params.productId,
      text,
      sentiment,
      sentimentScore: scores.positive,
    });

    res.status(201).json({
      success: true,
      review,
      analysis: { sentiment, confidence, scores }
    });

  } catch (error) {
    console.error('ML service error:', error.message);
    res.status(500).json({ message: 'Sentiment analysis failed due to a server error.' });
  }
});
```

---

## 🎯 Recommendation System

### Collaborative Filtering

**File:** `ml-service/recommender.py`

```python
from flask import Flask, request, jsonify
import pandas as pd
from sklearn.metrics.pairwise import cosine_similarity
import numpy as np

app = Flask(__name__)

# Load user-item matrix
# Format: rows = users, cols = products, values = ratings
ratings_df = pd.read_csv('data/ratings.csv')
user_item_matrix = ratings_df.pivot_table(
    index='user_id',
    columns='product_id',
    values='rating'
).fillna(0)

@app.route('/recommend/collaborative', methods=['POST'])
def collaborative_filtering():
    data = request.get_json()
    user_id = data['user_id']
    n_recommendations = data.get('n', 10)

    # Get user's ratings
    user_ratings = user_item_matrix.loc[user_id].values.reshape(1, -1)

    # Calculate similarity with all users
    similarities = cosine_similarity(user_ratings, user_item_matrix)[0]

    # Find similar users
    similar_users = similarities.argsort()[::-1][1:11]  # Top 10 similar users

    # Get products they liked
    recommendations = []
    for similar_user_idx in similar_users:
        similar_user_id = user_item_matrix.index[similar_user_idx]
        liked_products = user_item_matrix.loc[similar_user_id]
        liked_products = liked_products[liked_products > 3.5]  # Rating > 3.5

        for product_id, rating in liked_products.items():
            # Skip if user already rated this product
            if user_item_matrix.loc[user_id, product_id] == 0:
                recommendations.append({
                    'product_id': int(product_id),
                    'predicted_rating': float(rating),
                    'similarity_score': float(similarities[similar_user_idx])
                })

    # Sort by predicted rating
    recommendations = sorted(
        recommendations,
        key=lambda x: x['predicted_rating'],
        reverse=True
    )[:n_recommendations]

    return jsonify({
        'user_id': user_id,
        'recommendations': recommendations
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001)
```

### Content-Based Filtering

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

@app.route('/recommend/content', methods=['POST'])
def content_based():
    data = request.get_json()
    product_id = data['product_id']
    
    # Load product descriptions
    products_df = pd.read_csv('data/products.csv')
    
    # Vectorize descriptions
    tfidf = TfidfVectorizer(stop_words='english', max_features=500)
    tfidf_matrix = tfidf.fit_transform(products_df['description'])
    
    # Get product index
    idx = products_df[products_df['id'] == product_id].index[0]
    
    # Calculate similarity
    similarities = cosine_similarity(tfidf_matrix[idx], tfidf_matrix)[0]
    
    # Get top similar products
    similar_indices = similarities.argsort()[::-1][1:11]
    
    recommendations = []
    for i in similar_indices:
        recommendations.append({
            'product_id': int(products_df.iloc[i]['id']),
            'name': products_df.iloc[i]['name'],
            'similarity': float(similarities[i])
        })
    
    return jsonify({'recommendations': recommendations})
```

### Node.js Integration

```javascript
// Get personalized recommendations
router.get('/products/recommendations', auth, async (req, res) => {
  try {
    const userRatings = await Rating.find({ user: req.user._id }).lean();

    if (userRatings.length < 3) {
      const trending = await Product.find().sort({ viewCount: -1 }).limit(10).lean();
      return res.json({ recommendations: trending, method: 'trending' });
    }

    const response = await axios.post(`${ML_SERVICE_URL}/recommend/collaborative`, {
      user_id: req.user._id.toString(),
      n: 10
    });

    const productIds = response.data.recommendations.map(r => r.product_id);
    const products = await Product.find({ _id: { $in: productIds } }).lean();

    // Add similarity score to product data
    const recommendations = products.map(p => {
      const recommendationData = response.data.recommendations.find(r => r.product_id.toString() === p._id.toString());
      return { ...p, similarity_score: recommendationData?.similarity_score };
    });

    res.json({
      recommendations,
      method: 'collaborative'
    });

  } catch (error) {
    console.error('Recommendation error:', error.message);
    // Fallback to trending products on error
    try {
      const trending = await Product.find().sort({ viewCount: -1 }).limit(10).lean();
      res.status(500).json({ recommendations: trending, method: 'trending_fallback', error: 'Could not fetch personalized recommendations.' });
    } catch (fallbackError) {
      res.status(500).json({ message: 'Failed to get any recommendations.' });
    }
  }
});
```

---

## 📸 Image Classification

### Using Pre-trained Models

**Install:**

```bash
pip install tensorflow pillow
```

<details>
<summary>💡 Expected Output</summary>

```bash
Collecting tensorflow
  Downloading tensorflow-2.12.0-cp310-cp310-win_amd64.whl (1.5 kB)
Collecting pillow
  Downloading Pillow-9.5.0-cp310-cp310-win_amd64.whl (2.5 MB)
...
Successfully installed tensorflow-2.12.0 pillow-9.5.0
```
</details>

**Python Service:**

```python
from flask import Flask, request, jsonify
from tensorflow.keras.applications import MobileNetV2
from tensorflow.keras.applications.mobilenet_v2 import preprocess_input, decode_predictions
from tensorflow.keras.preprocessing import image
import numpy as np
from PIL import Image
import io

app = Flask(__name__)

# Load pre-trained model
model = MobileNetV2(weights='imagenet')

@app.route('/classify/image', methods=['POST'])
def classify_image():
    if 'image' not in request.files:
        return jsonify({'error': 'No image provided'}), 400

    # Read image
    img_file = request.files['image']
    img_bytes = img_file.read()
    img = Image.open(io.BytesIO(img_bytes))

    # Preprocess
    img = img.resize((224, 224))
    img_array = image.img_to_array(img)
    img_array = np.expand_dims(img_array, axis=0)
    img_array = preprocess_input(img_array)

    # Predict
    predictions = model.predict(img_array)
    decoded = decode_predictions(predictions, top=5)[0]

    results = [{
        'label': label,
        'description': desc,
        'confidence': float(score)
    } for (label, desc, score) in decoded]

    return jsonify({
        'predictions': results,
        'top_prediction': results[0]['description']
    })

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5002)
```

### Node.js Integration

```javascript
const multer = require('multer');
const FormData = require('form-data');

const upload = multer({ storage: multer.memoryStorage() });

router.post('/products/classify-image', upload.single('image'), async (req, res) => {
  if (!req.file) {
    return res.status(400).json({ message: 'No image uploaded' });
  }

  try {
    // Send to ML service
    const formData = new FormData();
    formData.append('image', req.file.buffer, {
      filename: req.file.originalname,
      contentType: req.file.mimetype
    });

    const response = await axios.post(
      `${ML_SERVICE_URL}/classify/image`,
      formData,
      {
        headers: formData.getHeaders()
      }
    );

    res.json({
      success: true,
      classification: response.data
    });

  } catch (error) {
    console.error('Image classification error:', error);
    res.status(500).json({ message: 'Classification failed' });
  }
});
```

---

## 📊 TensorFlow.js in Node.js

For simpler models, run directly in Node.js without Python.

### Install

```bash
npm install @tensorflow/tfjs-node
```

### Load Pre-trained Model

```javascript
const tf = require('@tensorflow/tfjs-node');

let model;

async function loadModel() {
  model = await tf.loadLayersModel('file://./models/price-prediction/model.json');
  console.log('Model loaded successfully');
}

loadModel();

router.post('/predict/price', async (req, res) => {
  const { bedrooms, bathrooms, sqft, location } = req.body;

  // Prepare input
  const input = tf.tensor2d([[bedrooms, bathrooms, sqft, location]], [1, 4]);

  // Predict
  const prediction = model.predict(input);
  const price = prediction.dataSync()[0];

  res.json({
    predicted_price: Math.round(price),
    currency: 'USD'
  });
});
```

---

## 🔍 Natural Language Processing

### Text Summarization

```python
from transformers import pipeline

summarizer = pipeline("summarization", model="facebook/bart-large-cnn")

@app.route('/nlp/summarize', methods=['POST'])
def summarize_text():
    data = request.get_json()
    text = data['text']
    max_length = data.get('max_length', 130)

    summary = summarizer(text, max_length=max_length, min_length=30)[0]

    return jsonify({
        'summary': summary['summary_text'],
        'original_length': len(text.split()),
        'summary_length': len(summary['summary_text'].split())
    })
```

### Named Entity Recognition (NER)

```python
from transformers import pipeline

ner = pipeline("ner", model="dbmdz/bert-large-cased-finetuned-conll03-english")

@app.route('/nlp/entities', methods=['POST'])
def extract_entities():
    data = request.get_json()
    text = data['text']

    entities = ner(text)

    # Group entities
    results = {}
    for entity in entities:
        entity_type = entity['entity']
        if entity_type not in results:
            results[entity_type] = []
        results[entity_type].append({
            'word': entity['word'],
            'score': entity['score']
        })

    return jsonify({'entities': results})
```

---

## 📈 Model Monitoring

### Track Predictions

```javascript
const predictionSchema = new mongoose.Schema({
  model: String,
  version: String,
  input: mongoose.Schema.Types.Mixed,
  output: mongoose.Schema.Types.Mixed,
  confidence: Number,
  latency: Number, // ms
  createdAt: { type: Date, default: Date.now }
});

// Log prediction
async function logPrediction(modelName, version, input, output, confidence, startTime) {
  await Prediction.create({
    model: modelName,
    version,
    input,
    output,
    confidence,
    latency: Date.now() - startTime
  });
}
```

### A/B Testing

```javascript
router.post('/recommend', auth, async (req, res) => {
  // Randomly choose model version
  const useNewModel = Math.random() < 0.5; // 50% traffic to new model

  const modelVersion = useNewModel ? 'v2' : 'v1';
  const endpoint = useNewModel 
    ? `${ML_SERVICE_URL}/recommend/v2` 
    : `${ML_SERVICE_URL}/recommend/v1`;

  const response = await axios.post(endpoint, {
    user_id: req.user._id
  });

  // Log for analysis
  await ABTest.create({
    user: req.user._id,
    experiment: 'recommendation_model',
    variant: modelVersion,
    timestamp: new Date()
  });

  res.json(response.data);
});
```

---

## 🎓 Hands-On Exercise

**Build: Smart Product Recommendation Engine**

1. **Collaborative Filtering**:
   - Collect user ratings
   - Build user-item matrix
   - Calculate similarities
   - Generate recommendations

2. **Content-Based Filtering**:
   - Extract product features
   - TF-IDF vectorization
   - Cosine similarity

3. **Hybrid System**:
   - Combine both approaches
   - Weighted scoring
   - Fallback strategies

4. **Image-Based Search**:
   - Upload product image
   - Extract features with CNN
   - Find similar products

5. **Monitoring Dashboard**:
   - Track recommendation accuracy
   - A/B test different algorithms
   - Monitor latency

---

## 📚 Practice Projects

- **Project #52:** [AI-Powered E-Commerce Assistant](../projects/52-ai-ecommerce-assistant.md) ⭐
- **Project #27:** [Product Recommendation System](../projects/27-product-recommendation.md) ⭐

---

## 🔗 Resources

- [TensorFlow.js](https://www.tensorflow.org/js)
- [Scikit-learn](https://scikit-learn.org/)
- [Hugging Face Transformers](https://huggingface.co/transformers/)
- [FastAPI for ML](https://fastapi.tiangolo.com/)

---

## ✅ Chapter Checklist

- [ ] Set up Python ML service with Flask
- [ ] Deploy ML model in Docker
- [ ] Connect Node.js to Python service
- [ ] Build recommendation system
- [ ] Implement image classification
- [ ] Use NLP for text analysis
- [ ] Set up model monitoring
- [ ] Implement A/B testing
- [ ] Optimize prediction latency

---

**Made with ❤️ by Tushar Parlikar**

[← Previous: AI Chatbot Integration](B17_AI_CHATBOT_INTEGRATION.md) | [Next: Data Analysis with Python →](B19_DATA_ANALYSIS.md)
