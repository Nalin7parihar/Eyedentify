# 👁️ EyeDentify - Face Spoof Detection System

**EyeDentify** is an AI-powered face liveness detection system that distinguishes real faces from spoofed ones (photos, videos, or masks). Built with deep learning and computer vision, it provides real-time detection capabilities through both webcam streaming and image upload interfaces.

![Python](https://img.shields.io/badge/Python-3.12+-blue.svg)
![TensorFlow](https://img.shields.io/badge/TensorFlow-2.20+-orange.svg)
![Streamlit](https://img.shields.io/badge/Streamlit-1.52+-red.svg)
![OpenCV](https://img.shields.io/badge/OpenCV-4.11+-green.svg)

---

## 🎯 Features

- **Real-time Liveness Detection**: Live webcam streaming with instant face spoof detection
- **Image Upload Analysis**: Upload and analyze static images for face authenticity
- **Multi-face Detection**: Simultaneously detect and analyze multiple faces in a single frame
- **High Accuracy**: Custom CNN architecture trained on real and spoofed face datasets
- **User-friendly Interface**: Clean, intuitive Streamlit web interface
- **Confidence Scoring**: Provides probability scores for each detection
- **Visual Feedback**: Color-coded bounding boxes (green for real, red for fake)

---

## 🏗️ Architecture

### System Components

```mermaid
graph LR
    A[Input: Webcam/Image] --> B[Face Detection]
    B --> C[Face Extraction]
    C --> D[Preprocessing]
    D --> E[Liveness Model]
    E --> F[Classification]
    F --> G[Output: Real/Fake]
```

### Models Used

1. **Face Detector**: OpenCV DNN module with SSD (Single Shot Detector)
   - Model: `res10_300x300_ssd_iter_140000.caffemodel`
   - Input: 300x300 RGB images
   - Purpose: Locate faces in frames

2. **Liveness Detection Model**: Custom CNN (LivenessNet)
   - Architecture: Custom convolutional neural network
   - Input: 32x32 RGB face crops
   - Output: Binary classification (Real/Fake)
   - Training: Trained on real faces and presentation attacks

---

## 📁 Project Structure

```
Eyedentify/
├── app.py                              # Main Streamlit application
├── liveness_demo.py                    # Command-line demo script
├── gather_examples.py                  # Training data collection script
├── detector/                           # Face detection models
│   ├── deploy.prototxt                 # SSD model architecture
│   └── res10_300x300_ssd_iter_140000.caffemodel  # Pre-trained weights
├── liveness_model_v3.keras             # Liveness detection model (v3)
├── liveness_model_colab_v3.keras       # Colab-trained variant
├── liveness_model_balanced_v2.keras    # Balanced dataset variant
├── le_model_v3.pickle                  # Label encoder (v3)
├── le_colab_v3.pickle                  # Colab label encoder
├── le_balanced_v2.pickle               # Balanced label encoder
├── requirements.txt                    # Python dependencies
├── pyproject.toml                      # Project configuration
└── README.md                           # This file
```

---

## 🚀 Getting Started

### Prerequisites

- Python 3.12 or higher
- Webcam (for real-time detection)
- pip or uv package manager

### Installation

1. **Clone the repository**
   ```bash
   cd /path/to/Eyedentify
   ```

2. **Create a virtual environment** (recommended)
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # On Windows: .venv\Scripts\activate
   ```

3. **Install dependencies**
   
   Using pip:
   ```bash
   pip install -r requirements.txt
   ```
   
   Or using uv (faster):
   ```bash
   uv pip install -r requirements.txt
   ```

### Running the Application

#### Web Interface (Streamlit)

```bash
streamlit run app.py
```

The application will open in your default browser at `http://localhost:8501`

#### Command-line Demo

For real-time webcam detection:

```bash
python liveness_demo.py \
  --model liveness_model_v3.keras \
  --le le_model_v3.pickle \
  --detector detector
```

Press `q` to quit the demo.

---

## 💻 Usage

### Web Interface

1. **Choose Input Method**:
   - **Real-time Webcam**: Click "Start" to begin live detection
   - **Upload Image**: Upload a JPG/PNG image for analysis

2. **Real-time Webcam Mode**:
   - Grant camera permissions when prompted
   - The system will automatically detect and classify faces
   - Results appear in real-time with confidence scores

3. **Image Upload Mode**:
   - Upload an image containing one or more faces
   - Click "Predict" to analyze
   - View results with bounding boxes and confidence scores

### Command-line Demo

The `liveness_demo.py` script provides a lightweight OpenCV-based interface:

```bash
python liveness_demo.py \
  --model <path-to-model> \
  --le <path-to-label-encoder> \
  --detector <path-to-detector-folder> \
  --confidence 0.5
```

**Arguments**:
- `--model`: Path to the trained liveness detection model
- `--le`: Path to the label encoder pickle file
- `--detector`: Path to the folder containing face detector files
- `--confidence`: Minimum detection confidence (default: 0.5)

---

## 🔧 Training Data Collection

The `gather_examples.py` script helps collect training data from videos:

```bash
python gather_examples.py \
  --input <path-to-video> \
  --output <output-folder> \
  --detector detector \
  --skip 16
```

**Arguments**:
- `--input`: Path to input video file
- `--output`: Directory to save cropped face images
- `--detector`: Path to face detector folder
- `--confidence`: Minimum face detection confidence (default: 0.5)
- `--skip`: Number of frames to skip between detections (default: 16)

**Workflow**:
1. Records videos of real faces and spoofed faces (photos, screens)
2. Extracts face crops using this script
3. Organizes into `real/` and `fake/` folders
4. Trains the liveness model on this dataset

---

## 🧠 How It Works

### Detection Pipeline

1. **Face Detection**:
   - Input frame is resized to 300x300
   - SSD face detector locates all faces
   - Bounding boxes extracted for faces with confidence > 0.5

2. **Preprocessing**:
   - Each face crop is resized to 32x32 pixels
   - Pixel values normalized to [0, 1]
   - Converted to array format for model input

3. **Liveness Classification**:
   - Preprocessed face fed to CNN model
   - Model outputs probability distribution [real, fake]
   - Highest probability determines classification

4. **Visualization**:
   - Green bounding box + "REAL" label for authentic faces
   - Red bounding box + "FAKE" label for spoofed faces
   - Confidence percentage displayed

### Model Variants

The project includes three model variants:

- **v3**: Standard training
- **colab_v3**: Trained on Google Colab (potentially larger dataset)
- **balanced_v2**: Trained on balanced real/fake dataset

You can switch models by changing the file paths in `app.py` (lines 28-29).

---

## 📊 Technical Details

### Dependencies

| Package | Version | Purpose |
|---------|---------|---------|
| streamlit | 1.52+ | Web interface framework |
| tensorflow | 2.20+ | Deep learning model inference |
| opencv-python | 4.11+ | Computer vision operations |
| numpy | 2.4+ | Numerical computations |
| Pillow | 12.1+ | Image processing |
| scikit-learn | 1.8+ | Label encoding |
| imutils | 0.5+ | Video stream utilities |
| matplotlib | 3.10+ | Visualization (training) |

### Performance Considerations

- **Real-time Processing**: Processes every frame for maximum responsiveness
- **Frame Skipping**: Can be enabled in `LivenessDetectionProcessor` to reduce CPU load
- **Async Processing**: Uses WebRTC async processing for smooth streaming
- **Model Caching**: Streamlit caches models to avoid reloading

---

## 🎨 Tips for Best Results

- ✅ Ensure good, even lighting on your face
- ✅ Position your face clearly in view of the camera
- ✅ For testing fake detection, use a printed photo or display a face on another screen
- ✅ If using webcam, allow a moment for the camera to adjust focus and lighting
- ✅ Avoid extreme angles or partial face occlusion

---

## 🔒 Security Applications

EyeDentify can be integrated into:

- **Authentication Systems**: Prevent photo-based spoofing attacks
- **Access Control**: Verify physical presence for secure areas
- **Online Verification**: KYC (Know Your Customer) processes
- **Payment Systems**: Biometric payment authentication
- **Attendance Systems**: Prevent proxy attendance with photos

---

## 🛠️ Troubleshooting

### Common Issues

**Issue**: Camera not detected in Streamlit
- **Solution**: Grant browser camera permissions, check if camera is in use by another app

**Issue**: Model loading errors
- **Solution**: Ensure all `.keras` and `.pickle` files are in the project root

**Issue**: Low detection accuracy
- **Solution**: Improve lighting, ensure face is clearly visible, try different model variant

**Issue**: TensorFlow installation fails
- **Solution**: Ensure Python version is compatible (3.9-3.12), check system architecture

---

## 📝 Development Notes

### Model Training

The liveness models were trained using:
- Custom CNN architecture (LivenessNet)
- Binary cross-entropy loss
- Adam optimizer
- Data augmentation (rotation, flipping, brightness)
- Balanced real/fake datasets

### Future Enhancements

- [ ] Add support for video file upload
- [ ] Implement 3D face analysis
- [ ] Add depth sensor integration
- [ ] Multi-modal detection (texture + motion)
- [ ] Mobile app deployment
- [ ] REST API for integration

---

## 👥 Authors

Made with ❤️ by **Diyana** and **Nalin**

---

## 📄 License

This project is provided as-is for educational and research purposes.

---

## 🙏 Acknowledgments

- OpenCV for face detection models
- TensorFlow/Keras for deep learning framework
- Streamlit for the web interface
- The computer vision research community

---

## 📞 Support

For questions or issues, please create an issue in the repository or contact the authors.

---

**Last Updated**: January 2026
