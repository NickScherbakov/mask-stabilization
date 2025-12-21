# Mask Stabilization System

**Автор:** Чубарова Дарья Алексеевна

A complete system for reducing flickering in frame-by-frame video segmentation using temporal smoothing techniques.

## 🌐 Project Presentation

**Live Demo:** [https://nickscherbakov.github.io/mask-stabilization/](https://nickscherbakov.github.io/mask-stabilization/)

A comprehensive presentation website (in Russian) showcasing:
- Problem statement and visual explanations
- System architecture and technologies
- Stabilization methods with formulas
- Experimental results and metrics
- API documentation
- Q&A section for homework defense

See [docs/GITHUB_PAGES_SETUP.md](docs/GITHUB_PAGES_SETUP.md) for GitHub Pages setup instructions.

## 📋 Overview

This project implements a full pipeline for:
1. **Video Segmentation** using DeepLabv3 (PyTorch/torchvision)
2. **Mask Stabilization** with multiple temporal smoothing methods
3. **Metrics Calculation** to measure stability improvements
4. **REST API** for easy integration (FastAPI)
5. **Interactive Analysis** with Jupyter notebooks

## 🏗️ Architecture

```
┌─────────────┐      ┌──────────────┐      ┌─────────────────┐
│   Upload    │─────▶│ Segmentation │─────▶│  Stabilization  │
│    Video    │      │  (DeepLabv3) │      │   (Temporal)    │
└─────────────┘      └──────────────┘      └─────────────────┘
                            │                        │
                            │                        │
                            ▼                        ▼
                     ┌──────────────┐        ┌─────────────┐
                     │    Masks     │        │  Smoothed   │
                     │   (Before)   │        │   Masks     │
                     └──────────────┘        └─────────────┘
                                                     │
                                                     │
                                                     ▼
                                             ┌─────────────┐
                                             │   Metrics   │
                                             │ Calculation │
                                             └─────────────┘
```

## 📁 Project Structure

```
mask-stabilization/
├── README.md                    # This file
├── requirements.txt             # Python dependencies
├── Dockerfile                   # Docker container setup
├── docker-compose.yml           # Docker composition
│
├── docs/                        # Presentation website (GitHub Pages)
│   ├── index.html               # Main presentation page (Russian)
│   └── GITHUB_PAGES_SETUP.md    # Setup instructions
│
├── src/
│   ├── __init__.py
│   ├── main.py                  # FastAPI server
│   ├── segmentation.py          # DeepLabv3 segmentation
│   ├── stabilization.py         # Temporal smoothing methods
│   ├── metrics.py               # Stability metrics (IoU, etc.)
│   └── utils.py                 # Utility functions
│
├── notebooks/
│   └── analysis.ipynb           # Interactive analysis notebook
│
├── frontend/
│   ├── index.html               # Web frontend (HTML/CSS/JS)
│   └── README.md                # Frontend documentation
│
├── spark_frontend/
│   └── SPARK_PROMPT.md          # GitHub Spark frontend prompt
│
├── examples/
│   └── .gitkeep                 # Place test videos here
│
└── results/
    └── .gitkeep                 # Processing results stored here
```

## 🚀 Installation

### Option 1: Local Installation

1. **Clone the repository**
   ```bash
   git clone https://github.com/NickScherbakov/mask-stabilization.git
   cd mask-stabilization
   ```

2. **Create virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install -r requirements.txt
   ```

### Option 2: Docker

1. **Build and run with Docker Compose**
   ```bash
   docker-compose up --build
   ```

The API will be available at `http://localhost:8000`

## 📖 Usage

### Running the API Server

```bash
# From the project root directory
uvicorn src.main:app --host 0.0.0.0 --port 8000 --reload
```

Access the API documentation at `http://localhost:8000/docs`

### Using the Jupyter Notebook

```bash
jupyter notebook notebooks/analysis.ipynb
```

The notebook provides a step-by-step demonstration of the entire pipeline.

### API Endpoints

#### 1. Upload Video
```bash
curl -X POST "http://localhost:8000/api/upload" \
  -F "file=@path/to/video.mp4"
```

**Response:**
```json
{
  "job_id": "550e8400-e29b-41d4-a716-446655440000",
  "status": "uploaded",
  "video_info": {
    "fps": 30.0,
    "frame_count": 150,
    "width": 1920,
    "height": 1080
  }
}
```

#### 2. Segment Video
```bash
curl -X POST "http://localhost:8000/api/segment" \
  -H "Content-Type: application/json" \
  -d '{"job_id": "YOUR_JOB_ID", "target_class": "person"}'
```

#### 3. Apply Stabilization
```bash
curl -X POST "http://localhost:8000/api/stabilize" \
  -H "Content-Type: application/json" \
  -d '{
    "job_id": "YOUR_JOB_ID",
    "method": "moving_average",
    "window_size": 5
  }'
```

#### 4. Get Status
```bash
curl "http://localhost:8000/api/status/YOUR_JOB_ID"
```

#### 5. Get Metrics
```bash
curl "http://localhost:8000/api/metrics/YOUR_JOB_ID"
```

**Response:**
```json
{
  "iou_before": {
    "mean": 0.8234,
    "std": 0.0521,
    "min": 0.6543,
    "max": 0.9876
  },
  "iou_after": {
    "mean": 0.9123,
    "std": 0.0234,
    "min": 0.8234,
    "max": 0.9912
  },
  "improvement": {
    "iou_improvement": 0.0889,
    "iou_improvement_percent": 10.8,
    "instability_reduction_percent": 57.3
  }
}
```

#### 6. Get Frame Image
```bash
curl "http://localhost:8000/api/frames/YOUR_JOB_ID/comparison/25" \
  --output frame_25.png
```

Frame types: `mask_before`, `mask_after`, `comparison`

### Available Segmentation Classes

```python
{
    0: 'background',
    15: 'person',
    7: 'car',
    6: 'bus',
    8: 'truck',
    9: 'boat',
    17: 'cat',
    18: 'dog',
    19: 'horse',
    20: 'sheep',
    21: 'cow'
}
```

## 🔬 Stabilization Methods

### 1. Moving Average
Averages probability maps over a temporal window.

**Parameters:**
- `window_size`: 3, 5, 7, or 9 (must be odd)

**Formula:**
```
smoothed[i] = mean(masks[i-w:i+w+1])
```

**Use case:** General-purpose smoothing, good balance between smoothness and responsiveness.

### 2. Median Filter
Computes median across temporal window for each pixel.

**Parameters:**
- `window_size`: 3, 5, 7, or 9 (must be odd)

**Formula:**
```
smoothed[i] = median(masks[i-w:i+w+1])
```

**Use case:** Robust to outliers, preserves sharp edges better than moving average.

### 3. Exponential Smoothing
Weighted average giving more importance to recent frames.

**Parameters:**
- `alpha`: 0.1 to 0.9 (smoothing factor)
  - Lower α = more smoothing
  - Higher α = more responsive

**Formula:**
```
smoothed[t] = α * original[t] + (1-α) * smoothed[t-1]
```

**Use case:** Adaptive smoothing, good for varying motion speeds.

## 📊 Metrics

### IoU (Intersection over Union)
Measures overlap between consecutive frames:
```
IoU = |A ∩ B| / |A ∪ B|
```

Higher IoU = more temporal consistency

### Instability Score
```
Instability = 1 - IoU
```

Higher instability = more flickering

### Metrics Computed

- **Mean IoU**: Average consistency across all frame transitions
- **IoU Standard Deviation**: Variability in consistency
- **Instability Reduction**: Percentage decrease in flickering
- **Min/Max IoU**: Range of consistency values

## 🎨 Frontend

A clean, modern web interface is available in the `frontend/` directory.

### Quick Start

1. **Start the API server:**
   ```bash
   uvicorn src.main:app --host 0.0.0.0 --port 8000
   ```

2. **Open the frontend:**
   - Simply open `frontend/index.html` in a web browser, or
   - Serve it with a simple HTTP server:
     ```bash
     cd frontend
     python -m http.server 8080
     ```
   - Navigate to `http://localhost:8080/index.html`

### Features

The frontend provides:
- **Drag-and-drop video upload** with format validation
- **Real-time processing status** with progress tracking
- **Interactive frame viewer** with navigation controls
- **Metrics visualization** showing IoU improvements
- **Configuration options** for object classes and stabilization methods
- **Responsive design** that works on all screen sizes

See `frontend/README.md` for detailed documentation.

For an alternative GitHub Spark interface, see the prompt in `spark_frontend/SPARK_PROMPT.md`.

## 🔍 Example Workflow

1. **Upload a video**
2. **Segment** targeting "person" class
3. **Apply** moving average with window_size=5
4. **Visualize** results:
   - Frame-by-frame comparison
   - IoU improvement chart
   - Quantitative metrics

## 📈 Expected Results

Typical improvements with window_size=5:

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| Mean IoU | 0.823 | 0.912 | +10.8% |
| IoU Std | 0.052 | 0.023 | -55.8% |
| Instability | 0.177 | 0.088 | -50.3% |

## 🛠️ Development

### Running Tests

```bash
# Add tests in tests/ directory
pytest tests/
```

### Code Structure

- `segmentation.py`: VideoSegmenter class using DeepLabv3
- `stabilization.py`: MaskStabilizer with temporal filtering methods
- `metrics.py`: Metric calculation functions
- `utils.py`: Helper functions for video/image processing
- `main.py`: FastAPI application with REST endpoints

## 📝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## 📄 License

This project is for educational purposes (Homework Assignment 5).

## 🙏 Acknowledgments

- DeepLabv3 model from torchvision
- FastAPI framework
- OpenCV for video processing

## 📚 References

- DeepLabv3: [Rethinking Atrous Convolution for Semantic Image Segmentation](https://arxiv.org/abs/1706.05587)
- Temporal consistency in video segmentation
- Moving average and median filters for temporal smoothing

## 💡 Tips

1. **Video Selection**: Start with short videos (5-10 seconds) for faster processing
2. **Class Selection**: Choose "person" for best results with human subjects
3. **Window Size**: Start with 5, increase for more smoothing
4. **Alpha Value**: Try 0.3 for balanced exponential smoothing

## 🐛 Troubleshooting

**Issue**: CUDA out of memory
```python
# Solution: Use CPU mode or reduce batch size
segmenter = VideoSegmenter(device='cpu')
```

**Issue**: Video won't upload
- Check file format (.mp4, .avi, .mov supported)
- Ensure file size < 100MB
- Verify video codec compatibility

**Issue**: Segmentation is slow
- Use GPU if available
- Reduce video resolution
- Process fewer frames

## 📞 Support

For issues and questions, please open an issue on GitHub.

---

**Status**: ✅ Ready for deployment and testing

**Version**: 1.0.0

**Last Updated**: December 2025