# 🚀 Quick Start Guide - EyeDentify

Get up and running with EyeDentify in 5 minutes!

---

## Prerequisites

- **Python**: 3.12 or higher
- **Webcam**: For real-time detection (optional)
- **Operating System**: Linux, macOS, or Windows

---

## Installation Steps

### 1. Navigate to Project Directory

```bash
cd /home/nalin7parihar/Documents/Eyedentify
```

### 2. Create Virtual Environment

```bash
python -m venv .venv
```

### 3. Activate Virtual Environment

**Linux/macOS**:
```bash
source .venv/bin/activate
```

**Windows**:
```bash
.venv\Scripts\activate
```

### 4. Install Dependencies

**Option A: Using pip**
```bash
pip install -r requirements.txt
```

**Option B: Using uv (faster)**
```bash
pip install uv
uv pip install -r requirements.txt
```

### 5. Verify Installation

```bash
python -c "import streamlit; import tensorflow; import cv2; print('✓ All dependencies installed!')"
```

---

## Running the Application

### Web Interface (Recommended)

```bash
streamlit run app.py
```

The app will open automatically at `http://localhost:8501`

### Command-line Demo

```bash
python liveness_demo.py \
  --model liveness_model_v3.keras \
  --le le_model_v3.pickle \
  --detector detector
```

Press `q` to quit.

---

## Usage Examples

### Example 1: Real-time Webcam Detection

1. Run the Streamlit app
2. Select **"Real-time Webcam"**
3. Click **"Start"**
4. Grant camera permissions
5. Show your face to the camera
6. See real-time detection results

**Testing Fake Detection**:
- Hold up a printed photo of a face
- Display a face on your phone screen
- The system should label it as "FAKE"

### Example 2: Upload Image Analysis

1. Run the Streamlit app
2. Select **"Upload Image"**
3. Click **"Browse files"** and select an image
4. Click **"Predict"**
5. View the annotated result

**Sample Images to Test**:
- Selfie from your phone (should be "REAL")
- Screenshot of a face (should be "FAKE")
- Group photo with multiple faces

### Example 3: Batch Processing (Python Script)

Create a file `batch_process.py`:

```python
import cv2
import glob
from app import load_face_detector, load_liveness_model, process_frame

# Load models once
print("Loading models...")
detector = load_face_detector()
model, le = load_liveness_model()

# Process all images in a folder
image_paths = glob.glob("test_images/*.jpg")

for path in image_paths:
    print(f"\nProcessing: {path}")
    frame = cv2.imread(path)
    result_img, results = process_frame(frame)
    
    for i, result in enumerate(results):
        print(f"  Face {i+1}: {result['label'].upper()} ({result['confidence']:.2f}%)")
    
    # Save annotated image
    output_path = path.replace("test_images", "results")
    cv2.imwrite(output_path, result_img)
    print(f"  Saved to: {output_path}")
```

Run it:
```bash
mkdir -p results
python batch_process.py
```

---

## Common Issues & Solutions

### Issue: "ModuleNotFoundError: No module named 'streamlit'"

**Solution**: Activate virtual environment and install dependencies
```bash
source .venv/bin/activate
pip install -r requirements.txt
```

### Issue: "Camera not detected" in browser

**Solution**: 
1. Grant camera permissions in browser
2. Check if camera is in use by another app
3. Try a different browser (Chrome/Firefox recommended)

### Issue: "Error loading models"

**Solution**: Verify model files exist
```bash
ls -lh *.keras *.pickle
```

Expected files:
- `liveness_model_v3.keras`
- `le_model_v3.pickle`
- `detector/deploy.prototxt`
- `detector/res10_300x300_ssd_iter_140000.caffemodel`

### Issue: TensorFlow installation fails

**Solution**: Check Python version compatibility
```bash
python --version  # Should be 3.9-3.12
```

For Python 3.13+, TensorFlow may not be available yet. Use Python 3.12.

---

## Testing Your Installation

### Quick Test Script

Create `test_installation.py`:

```python
import streamlit as st
import tensorflow as tf
import cv2
import numpy as np
from PIL import Image

print("✓ Streamlit version:", st.__version__)
print("✓ TensorFlow version:", tf.__version__)
print("✓ OpenCV version:", cv2.__version__)
print("✓ NumPy version:", np.__version__)
print("✓ Pillow version:", Image.__version__)

# Test model loading
from app import load_face_detector, load_liveness_model

try:
    detector = load_face_detector()
    print("✓ Face detector loaded successfully")
except Exception as e:
    print("✗ Face detector failed:", e)

try:
    model, le = load_liveness_model()
    print("✓ Liveness model loaded successfully")
    print("  Classes:", le.classes_)
except Exception as e:
    print("✗ Liveness model failed:", e)

print("\n🎉 Installation test complete!")
```

Run it:
```bash
python test_installation.py
```

---

## Next Steps

### Learn More
- Read [`README.md`](file:///home/nalin7parihar/Documents/Eyedentify/README.md) for full documentation
- Check [`ARCHITECTURE.md`](file:///home/nalin7parihar/Documents/Eyedentify/ARCHITECTURE.md) for technical details
- Explore [`API_REFERENCE.md`](file:///home/nalin7parihar/Documents/Eyedentify/API_REFERENCE.md) for API docs

### Customize
- Try different model variants (v3, colab_v3, balanced_v2)
- Adjust detection thresholds
- Modify UI appearance in `app.py`

### Extend
- Add video file upload support
- Integrate with authentication systems
- Deploy to cloud (Streamlit Cloud, Docker)

---

## Getting Help

If you encounter issues:

1. Check the [Common Issues](#common-issues--solutions) section
2. Review error messages carefully
3. Verify all dependencies are installed
4. Ensure model files are present

---

**Happy detecting! 👁️**

---

**Last Updated**: January 2026
