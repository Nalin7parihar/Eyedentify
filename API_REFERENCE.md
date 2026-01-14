# 📚 EyeDentify - API Reference

This document provides detailed API documentation for the core functions and classes in the EyeDentify project.

---

## Table of Contents

1. [Main Application (app.py)](#main-application-apppy)
2. [Command-line Demo (liveness_demo.py)](#command-line-demo-liveness_demopy)
3. [Data Collection (gather_examples.py)](#data-collection-gather_examplespy)

---

## Main Application (app.py)

### Functions

#### `load_face_detector()`

Loads the OpenCV DNN face detector model.

**Returns**:
- `cv2.dnn.Net`: Loaded face detector network

**Caching**: 
- Decorated with `@st.cache_resource` for singleton behavior
- Loaded once per Streamlit session

**Example**:
```python
detector = load_face_detector()
```

**Implementation Details**:
- Loads SSD model from `detector/deploy.prototxt` and `detector/res10_300x300_ssd_iter_140000.caffemodel`
- Uses Caffe framework format

---

#### `load_liveness_model()`

Loads the liveness detection model and label encoder.

**Returns**:
- `tuple`: (model, label_encoder)
  - `model` (keras.Model): Trained liveness detection model
  - `label_encoder` (sklearn.LabelEncoder): Maps class indices to labels

**Caching**: 
- Decorated with `@st.cache_resource`

**Example**:
```python
model, le = load_liveness_model()
```

**Model Files**:
- Model: `liveness_model_v3.keras`
- Label Encoder: `le_model_v3.pickle`

---

#### `process_frame(frame)`

Main processing pipeline that detects faces and classifies them as real or fake.

**Parameters**:
- `frame` (numpy.ndarray): Input image in BGR format, shape (H, W, 3)

**Returns**:
- `tuple`: (result_image, results)
  - `result_image` (numpy.ndarray): Annotated image with bounding boxes and labels
  - `results` (list): List of detection dictionaries

**Result Dictionary Structure**:
```python
{
    "label": str,           # "real" or "fake"
    "confidence": float,    # Probability percentage (0-100)
    "position": tuple       # (startX, startY, endX, endY)
}
```

**Example**:
```python
import cv2
frame = cv2.imread("image.jpg")
annotated_frame, detections = process_frame(frame)

for detection in detections:
    print(f"Label: {detection['label']}")
    print(f"Confidence: {detection['confidence']:.2f}%")
    print(f"Position: {detection['position']}")
```

**Processing Steps**:
1. Resize frame if width > 600px
2. Create blob for face detection (300x300)
3. Run face detector
4. For each detected face (confidence > 0.5):
   - Extract face ROI
   - Resize to 32x32
   - Normalize to [0, 1]
   - Run liveness model
   - Draw bounding box and label
5. Return annotated frame and results

**Error Handling**:
- Returns original frame and empty list if models not loaded
- Skips individual faces that fail processing
- Validates bounding boxes are within frame bounds

---

### Classes

#### `LivenessDetectionProcessor`

WebRTC video processor for real-time liveness detection.

**Inheritance**: 
- Extends `streamlit_webrtc.VideoProcessorBase`

**Attributes**:
- `results_queue` (queue.Queue): Thread-safe queue for latest results (maxsize=1)
- `frame_skip` (int): Number of frames to skip (0 = process all frames)

**Methods**:

##### `__init__()`

Initializes the video processor.

```python
processor = LivenessDetectionProcessor()
```

##### `recv(frame)`

Processes incoming WebRTC video frames.

**Parameters**:
- `frame` (av.VideoFrame): WebRTC video frame

**Returns**:
- `av.VideoFrame`: Processed frame with annotations

**Example**:
```python
def recv(self, frame):
    img = frame.to_ndarray(format="bgr24")
    result_img, results = process_frame(img)
    
    # Update results queue
    while not self.results_queue.empty():
        self.results_queue.get_nowait()
    self.results_queue.put(results, block=False)
    
    return av.VideoFrame.from_ndarray(result_img, format="bgr24")
```

**Frame Skipping**:
- If `frame_skip > 0`, decrements counter and returns original frame
- Resets to 0 after skipping to process next frame

##### `get_latest_results()`

Retrieves the most recent detection results.

**Returns**:
- `list`: Latest detection results, or empty list if none available

**Example**:
```python
results = processor.get_latest_results()
for result in results:
    print(f"{result['label']}: {result['confidence']:.2f}%")
```

**Thread Safety**:
- Uses non-blocking queue operations
- Returns empty list on exception

---

## Command-line Demo (liveness_demo.py)

### Command-line Arguments

```bash
python liveness_demo.py \
  --model <path> \
  --le <path> \
  --detector <path> \
  --confidence <float>
```

**Arguments**:

| Argument | Short | Type | Required | Default | Description |
|----------|-------|------|----------|---------|-------------|
| `--model` | `-m` | str | Yes | - | Path to trained liveness model (.keras) |
| `--le` | `-l` | str | Yes | - | Path to label encoder (.pickle) |
| `--detector` | `-d` | str | Yes | - | Path to face detector directory |
| `--confidence` | `-c` | float | No | 0.5 | Minimum detection confidence (0.0-1.0) |

**Example**:
```bash
python liveness_demo.py \
  -m liveness_model_v3.keras \
  -l le_model_v3.pickle \
  -d detector \
  -c 0.6
```

### Main Loop

**Pseudocode**:
```python
while True:
    # Read frame from webcam
    frame = video_stream.read()
    
    # Resize to width=600
    frame = imutils.resize(frame, width=600)
    
    # Detect faces
    blob = cv2.dnn.blobFromImage(...)
    detections = net.forward()
    
    # Process each detection
    for detection in detections:
        if confidence > threshold:
            # Extract face
            # Preprocess (resize to 32x32, normalize)
            # Predict liveness
            # Draw bounding box and label
    
    # Display frame
    cv2.imshow("Frame", frame)
    
    # Exit on 'q' key
    if cv2.waitKey(1) & 0xFF == ord('q'):
        break
```

**Controls**:
- Press `q` to quit

---

## Data Collection (gather_examples.py)

### Command-line Arguments

```bash
python gather_examples.py \
  --input <path> \
  --output <path> \
  --detector <path> \
  --confidence <float> \
  --skip <int>
```

**Arguments**:

| Argument | Short | Type | Required | Default | Description |
|----------|-------|------|----------|---------|-------------|
| `--input` | `-i` | str | Yes | - | Path to input video file |
| `--output` | `-o` | str | Yes | - | Path to output directory for face crops |
| `--detector` | `-d` | str | Yes | - | Path to face detector directory |
| `--confidence` | `-c` | float | No | 0.5 | Minimum face detection confidence |
| `--skip` | `-s` | int | No | 16 | Frames to skip between detections |

**Example**:
```bash
python gather_examples.py \
  -i videos/real_faces.mp4 \
  -o dataset/real \
  -d detector \
  -s 16
```

### Processing Logic

**Frame Selection**:
```python
read = 0
while True:
    frame = video.read()
    read += 1
    
    # Skip frames
    if read % args["skip"] != 0:
        continue
    
    # Process this frame
    detect_and_save_face(frame)
```

**Face Extraction**:
1. Detect all faces in frame
2. Select face with highest confidence
3. Extract bounding box
4. Save as `{number}.png` in output directory

**Smart Numbering**:
```python
# Find highest existing number in output directory
existing_files = [f for f in os.listdir(output_dir) if f.endswith('.png')]
existing_numbers = [int(f.replace('.png', '')) for f in existing_files]
saved = max(existing_numbers) + 1 if existing_numbers else 0
```

**Output**:
- Saves face crops as numbered PNG files: `0.png`, `1.png`, `2.png`, ...
- Prints progress: `[INFO] saved /path/to/output/123.png to disk`

---

## Model Input/Output Specifications

### Face Detector (SSD)

**Input**:
```python
blob = cv2.dnn.blobFromImage(
    image,
    scalefactor=1.0,
    size=(300, 300),
    mean=(104.0, 177.0, 123.0)
)
# Shape: (1, 3, 300, 300)
```

**Output**:
```python
detections = net.forward()
# Shape: (1, 1, N, 7)
# Where N = number of detections
# Each detection: [batch_id, class_id, confidence, x1, y1, x2, y2]
```

**Accessing Detections**:
```python
for i in range(detections.shape[2]):
    confidence = detections[0, 0, i, 2]
    box = detections[0, 0, i, 3:7]  # [x1, y1, x2, y2] normalized to [0, 1]
```

### Liveness Model

**Input**:
```python
face = cv2.resize(face, (32, 32))
face = face.astype("float") / 255.0
face = img_to_array(face)
face = np.expand_dims(face, axis=0)
# Shape: (1, 32, 32, 3)
# Value range: [0.0, 1.0]
```

**Output**:
```python
predictions = model.predict(face)
# Shape: (1, 2)
# predictions[0] = [P(real), P(fake)]

# Get predicted class
class_index = np.argmax(predictions[0])
label = label_encoder.classes_[class_index]  # "real" or "fake"
confidence = predictions[0][class_index] * 100  # Percentage
```

---

## Configuration

### Model Selection

To use different model variants, modify [`app.py`](file:///home/nalin7parihar/Documents/Eyedentify/app.py):

```python
# Lines 28-29
def load_liveness_model():
    # Change these paths to use different models
    model = load_model("liveness_model_v3.keras")
    le = pickle.loads(open("le_model_v3.pickle", "rb").read())
    return model, le
```

**Available Models**:
- `liveness_model_v3.keras` + `le_model_v3.pickle` (default)
- `liveness_model_colab_v3.keras` + `le_colab_v3.pickle`
- `liveness_model_balanced_v2.keras` + `le_balanced_v2.pickle`

### Detection Thresholds

**Face Detection Confidence**:
```python
# Line 118 in app.py
if confidence > 0.5:  # Change threshold here
    # Process this face
```

**Frame Size Limit**:
```python
# Lines 96-97 in app.py
if image_cv.shape[1] > 600:  # Change max width here
    image_cv = cv2.resize(image_cv, (600, ...))
```

---

## Error Handling

### Model Loading Errors

```python
try:
    face_detector = load_face_detector()
    liveness_model, label_encoder = load_liveness_model()
    model_loaded = True
except Exception as e:
    model_loaded = False
    st.error(f"Error loading models: {e}")
```

**Common Errors**:
- `FileNotFoundError`: Model files missing
- `ImportError`: TensorFlow not installed
- `ValueError`: Incompatible model format

### Processing Errors

```python
try:
    face = cv2.resize(face, (32, 32))
    # ... preprocessing ...
    preds = liveness_model.predict(face)[0]
except Exception as e:
    # Skip this face, continue with next
    continue
```

**Graceful Degradation**:
- Individual face failures don't crash the application
- Empty results returned if no faces detected
- Original frame returned if processing fails

---

## Performance Tips

### 1. Batch Processing

For multiple images, reuse loaded models:

```python
detector = load_face_detector()
model, le = load_liveness_model()

for image_path in image_paths:
    frame = cv2.imread(image_path)
    result_img, results = process_frame(frame)
    # Process results
```

### 2. Frame Skipping

For real-time processing on slow hardware:

```python
class LivenessDetectionProcessor(VideoProcessorBase):
    def __init__(self):
        self.frame_skip = 2  # Process every 3rd frame
```

### 3. GPU Acceleration

Enable GPU for TensorFlow:

```python
import tensorflow as tf

# Check GPU availability
print(tf.config.list_physical_devices('GPU'))

# Enable memory growth
gpus = tf.config.list_physical_devices('GPU')
if gpus:
    tf.config.experimental.set_memory_growth(gpus[0], True)
```

---

## Integration Examples

### REST API Wrapper

```python
from fastapi import FastAPI, File, UploadFile
import cv2
import numpy as np

app = FastAPI()

@app.post("/detect")
async def detect_liveness(file: UploadFile = File(...)):
    # Read uploaded image
    contents = await file.read()
    nparr = np.frombuffer(contents, np.uint8)
    frame = cv2.imdecode(nparr, cv2.IMREAD_COLOR)
    
    # Process
    result_img, results = process_frame(frame)
    
    return {"detections": results}
```

### Batch Processing Script

```python
import glob

image_paths = glob.glob("images/*.jpg")
results_all = []

for path in image_paths:
    frame = cv2.imread(path)
    _, results = process_frame(frame)
    results_all.append({
        "image": path,
        "detections": results
    })

# Save results
import json
with open("results.json", "w") as f:
    json.dump(results_all, f, indent=2)
```

---

## Troubleshooting

### Issue: Low Confidence Scores

**Solution**: Adjust preprocessing or retrain model with more diverse data

```python
# Experiment with different normalization
face = face.astype("float") / 255.0  # Current
# vs
face = (face.astype("float") - 127.5) / 127.5  # Alternative
```

### Issue: Slow Processing

**Solution**: Profile and optimize bottlenecks

```python
import time

start = time.time()
detections = face_detector.forward()
print(f"Face detection: {time.time() - start:.3f}s")

start = time.time()
preds = liveness_model.predict(face)
print(f"Liveness prediction: {time.time() - start:.3f}s")
```

---

**Last Updated**: January 2026
