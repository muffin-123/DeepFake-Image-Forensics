# DeepFake-Image-Forensics

### AI-assisted facial image analysis with deep learning and digital-forensics tools

------------------------------------------------------------------------

## Table of Contents

-   [Overview](#-overview)
-   [Problem Statement](#-problem-statement)
-   [Project Objectives](#-project-objectives)
-   [Features](#-features)
-   [Technology Stack](#-technology-stack)
-   [System Architecture](#-system-architecture)
-   [Image Analysis Workflow](#-image-analysis-workflow)
-   [Machine Learning Model](#-machine-learning-model)
-   [Forensic Analysis Components](#-forensic-analysis-components)
-   [Repository Structure](#-repository-structure)
-   [Requirements](#-requirements)
-   [Installation](#-installation)
-   [Running the Project](#-running-the-project)
-   [Using the Application](#-using-the-application)
-   [API Documentation](#-api-documentation)
-   [Model Files and Git LFS](#-model-files-and-git-lfs)
-   [Security and Privacy](#-security-and-privacy)
-   [Limitations](#-limitations)
-   [Troubleshooting](#-troubleshooting)
-   [Contributing](#-contributing)
-   [License](#-license)
-   [Acknowledgments](#-acknowledgments)

## 🔎 Overview

**DeepFake Image Forensics** is a web-based application that helps users
examine facial images for signs of possible manipulation. It combines a
deep-learning image classifier with supporting forensic utilities and
presents the results through a user interface.

The detection component uses a fine-tuned **CLIP ViT-B/32** image
encoder and a binary classification head. The trained model is exported
to **ONNX** for application inference. The wider platform includes image
upload, face detection, per-face analysis, image-quality checks,
forensic visualizations, SHA-256 hashing, analysis history, and PDF
report generation.

## 🎯 Problem Statement

AI-generated and manipulated facial images can be difficult to assess
visually, especially when images are compressed, resized, or shared
without their original context. Manual inspection can be inconsistent
and may not scale well.

This project explores a practical workflow that combines a learned image
classifier with additional image-inspection tools. It aims to make the
analysis process more accessible by bringing predictions and supporting
information together in one application.

## 🧭 Project Objectives

-   Provide an interface for submitting facial images for analysis.
-   Apply a trained deep-learning model to detected face regions.
-   Support analysis of images containing multiple faces.
-   Present predictions and model confidence information clearly.
-   Offer complementary forensic views, including Error Level Analysis
    (ELA) and noise residual visualization.
-   Calculate a SHA-256 hash to help identify the exact file analyzed.
-   Allow users to review previous analyses and access generated
    reports.
-   Provide a maintainable separation between frontend, backend, model
    inference, and storage.

## ✨ Features

### AI-assisted image classification

-   Uses a CLIP ViT-B/32-based classifier trained for real-versus-fake
    facial image classification.
-   Uses an ONNX model for inference in the application.
-   Presents prediction and confidence/probability information returned
    by the model.

### Face-level processing

-   Detects faces in an uploaded image.
-   Supports analysis of multiple detected faces.
-   Displays face regions and face-level results where available.
-   Includes image-quality checks such as blur, brightness,
    overexposure, and minimum face size.

### Supporting forensic analysis

-   **Error Level Analysis (ELA):** creates a visualization of
    differences associated with image recompression.
-   **Noise residual map:** provides a visual representation of image
    noise characteristics.
-   **EXIF metadata inspection:** shows available metadata when it
    exists in the file.
-   **SHA-256 hashing:** generates a content hash that can help identify
    whether the file bytes have changed.
-   **PDF report generation:** brings the available prediction and
    forensic information into a report.

These methods are supporting indicators. None is independently
conclusive evidence of manipulation.

### User and application features

-   Account registration and login.
-   Password hashing and password-recovery flow.
-   Image upload with preview and example images.
-   Analysis history backed by the configured database.
-   Downloadable forensic reports.
-   Webcam capture for frame-based image analysis.
-   User profile and application information pages.
-   Responsive interface and analysis settings.

Feature availability can depend on the current configuration and
environment variables.

## 🧰 Technology Stack

  -----------------------------------------------------------------------
  Layer                               Technologies
  ----------------------------------- -----------------------------------
  Frontend                            React, JavaScript, CSS

  Backend API                         Python, FastAPI, Uvicorn

  Model development                   PyTorch, OpenCLIP / CLIP ViT-B/32

  Model inference                     ONNX Runtime

  Database                            PostgreSQL, Neon

  Image processing                    Face detection, image-quality
                                      checks, ELA, noise analysis, EXIF
                                      inspection

  Integrity                           SHA-256

  Reports                             PDF generation

  Large model storage                 Git LFS
  -----------------------------------------------------------------------

Exact package versions should be taken from the dependency files in the
repository.

## 🏗️ System Architecture

The application is organized into several cooperating parts:

1.  **React frontend** --- provides the user interface for
    authentication, uploads, live capture, result visualization,
    history, and reports.
2.  **FastAPI backend** --- accepts requests, validates input,
    coordinates image analysis, runs inference, and returns results.
3.  **Face-processing pipeline** --- locates faces and applies supported
    quality checks before inference.
4.  **ONNX inference component** --- loads the exported classifier and
    produces predictions for prepared face images.
5.  **Forensic utilities** --- generate ELA/noise visualizations,
    inspect available metadata, and calculate a file hash.
6.  **Database and file storage** --- persist analysis records and store
    files according to the application configuration.
7.  **Report generator** --- assembles available results into a PDF
    report when report generation is enabled.

### High-level flow

``` text
User
 |
 v
React Frontend
 |  image upload / webcam frame
 v
FastAPI Backend
 |
 +--> Validate image
 |
 +--> Calculate SHA-256
 |
 +--> Detect faces and check image quality
 |
 +--> Prepare face crops
 |        |
 |        v
 |   ONNX Model
 |        |
 |        v
 |   Prediction results
 |
 +--> ELA / noise / EXIF analysis
 |
 +--> Optional PDF report
 |
 +--> Save analysis record and associated file references
 |
 v
Frontend displays results
```

This is a conceptual view of the workflow; the exact implementation
details are defined by the source code.

## 🔬 Image Analysis Workflow

A typical analysis follows these stages:

1.  **Input:** The user uploads a supported image or submits a captured
    webcam frame.
2.  **Validation:** The backend checks the file and image properties.
3.  **Hashing:** A SHA-256 hash is calculated for the submitted file.
4.  **Face detection:** Face regions are located in the image.
5.  **Quality checks:** Detected regions may be checked for blur,
    brightness, overexposure, and size.
6.  **Preprocessing:** Face crops are resized and normalized according
    to the model's expected input format.
7.  **Inference:** The ONNX classifier processes the prepared face image
    or images.
8.  **Result preparation:** Predictions and confidence/probability
    values are formatted for the interface.
9.  **Forensic support:** ELA, noise visualization, and available
    metadata may be generated or extracted.
10. **Persistence and reporting:** The result may be stored in the
    analysis history, and a PDF report may be generated if enabled.
11. **Presentation:** The frontend displays the results and available
    supporting information.

If no suitable face is detected, the application may ask the user to
submit a clearer image. Actual validation messages and limits are
defined in the backend.

## 🧠 Machine Learning Model

### Model overview

The model uses **CLIP ViT-B/32** as a pretrained visual encoder with a
classification head adapted to a binary real/fake image-classification
task.

CLIP is a vision-language model. In this project, the visual encoder is
used to extract image representations for classification; the
application should not be described as performing text-image matching
unless that capability is separately implemented.

### Training strategy

The development process used progressive transfer learning:

-   **Phase 1 --- Linear probing:** train the classification head while
    keeping the pretrained visual encoder frozen.
-   **Phase 2 --- Partial fine-tuning:** unfreeze selected transformer
    layers so part of the visual encoder can adapt to the target task.
-   **Phase 3 --- Broader fine-tuning:** fine-tune more of the model and
    compare validation performance.

Validation loss and classification metrics are used to compare
checkpoints. Lower training loss alone does not guarantee better
generalization, so checkpoint selection should rely on validation
results and, ideally, an independent test set.

### Dataset and preprocessing

The model development notes identify **FaceForensics++ (C23)** as the
training data source. The training pipeline used face crops and face
alignment/preprocessing, followed by resizing and CLIP normalization.

For reproducibility, document the exact dataset split, manipulation
types, preprocessing configuration, random seeds, and evaluation
protocol used for the final experiment. Do not publish restricted
datasets or source images unless their terms permit redistribution.

### Inference artifact

The primary exported model is expected at:

``` text
export/cockatoos_vitb32_best.onnx
```

The training checkpoint is kept separately:

``` text
exports/phase3_best.pt
```

The ONNX file is the application inference artifact. The PyTorch
checkpoint is primarily useful for training or further experimentation
and is not normally required to run the ONNX inference path.

### Input expectations

The project configuration has been described as accepting JPG/JPEG/PNG
images and preparing face inputs at **224 × 224** with CLIP
normalization. Verify the preprocessing implementation in the repository
before changing the model input pipeline; different normalization,
channel order, or resize behavior can affect predictions.

### Evaluation

Evaluate the final model on a held-out test set and report metrics such
as:

-   Accuracy
-   Precision
-   Recall
-   F1 score
-   ROC-AUC
-   Confusion matrix
-   Per-manipulation performance, where applicable

This README intentionally does not claim a single set of benchmark
results. Add final metrics only when they are tied to a clearly
identified checkpoint, dataset split, threshold, and evaluation
procedure.

## 🧪 Forensic Analysis Components

### Error Level Analysis (ELA)

ELA recompresses an image and visualizes differences in recompression
error. It can help an analyst inspect inconsistent regions, but results
depend on image history, format, and compression settings. ELA is not a
reliable standalone deepfake detector.

### Noise residual visualization

Noise residuals can help reveal differences in local image texture or
noise. Resizing, denoising, compression, and camera pipelines can also
produce variation, so the map must be interpreted cautiously.

### EXIF metadata

The application can inspect metadata available in an image file. Many
platforms remove metadata, and metadata can be edited. Missing or
unusual metadata alone does not prove manipulation.

### SHA-256 file hash

SHA-256 produces a digest based on the file's bytes. If the file
changes, its hash will normally change. A matching hash can help confirm
byte-for-byte identity when compared with a trusted recorded hash, but
hashing does not prove that the original image itself is authentic.

### PDF reports

The report feature can collect available predictions and forensic
information into a downloadable document. Reports should be treated as
records of the application's analysis, not as a formal certification of
authenticity.

## 📁 Repository Structure

The following is a guide to the expected major components. The exact
layout may vary; use the checked-in files as the source of truth.

``` text
DeepFake-Image-Forensics/
├── app/
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── routers/
│   │   ├── auth.py
│   │   ├── predict.py
│   │   └── history.py
│   ├── services/
│   │   ├── onnx_inference.py
│   │   ├── face_service.py
│   │   ├── ela_service.py
│   │   ├── noise_service.py
│   │   ├── exif_service.py
│   │   ├── pdf_service.py
│   │   ├── hash_service.py
│   │   └── file_validators.py
│   └── static/
│       ├── uploads/
│       └── reports/
├── frontend/
│   ├── public/
│   └── src/
│       ├── pages/
│       ├── components/
│       ├── services/
│       └── styles/
├── export/
│   └── cockatoos_vitb32_best.onnx
├── exports/
│   └── phase3_best.pt
├── diagrams/
├── .gitattributes
├── .gitignore
├── requirements.txt
├── LICENSE
└── README.md
```

Some generated folders and large artifacts may be excluded from version
control. The tree above is illustrative and should be updated if the
repository structure changes.

## 💻 Requirements

Before installing, ensure you have:

-   Python 3.10 or a Python version compatible with the project
    dependencies.
-   Node.js and npm for the React frontend.
-   Git.
-   Git LFS if the ONNX model is stored using Git LFS.
-   A configured PostgreSQL database if database-backed features are
    enabled.
-   Sufficient disk space for Python dependencies and the model
    artifact.

A GPU is not necessarily required for ONNX CPU inference, but
performance depends on the machine, model, and workload.

## ⚙️ Installation

The commands below assume the repository root contains `app/main.py`,
`requirements.txt`, and a `frontend/` directory. Adjust paths if the
checked-in structure differs.

### 1. Clone the repository

``` bash
git clone https://github.com/muffin-123/DeepFake-Image-Forensics.git
cd DeepFake-Image-Forensics
```

### 2. Retrieve Git LFS model files

Install Git LFS from [git-lfs.com](https://git-lfs.com/), then run:

``` bash
git lfs install
git lfs pull
```

Check that the model file exists at `export/cockatoos_vitb32_best.onnx`.
If the repository uses a different model distribution method, follow the
instructions provided by that method.

### 3. Create a Python virtual environment

**Windows PowerShell:**

``` powershell
python -m venv venv
.\venv\Scripts\Activate.ps1
```

If PowerShell blocks activation, use a terminal configured to permit
virtual-environment activation, or activate it from Command Prompt:

``` bat
venv\Scripts\activate.bat
```

### 4. Install backend dependencies

From the repository root:

``` powershell
python -m pip install --upgrade pip
pip install -r requirements.txt
```

If the project has a separate backend requirements file, install the
dependencies from that file instead. Use the dependency versions
maintained by the project.

### 5. Configure environment variables

Create a local `.env` file containing the settings required by the
backend. Check the code and any `.env.example` file to identify the
exact variable names.

Typical configuration may include:

-   Database connection string
-   Authentication or session settings
-   Email service settings for account-related features
-   Frontend/backend URL configuration, if read from environment
    variables

These are examples, not guaranteed variable names. Do not commit
credentials or real secret values. Keep `.env` in `.gitignore` and
provide safe placeholder values in `.env.example`.

### 6. Install frontend dependencies

Open a second terminal:

``` powershell
cd frontend
npm install
```

Inspect `frontend/package.json` to determine the available scripts
before starting the frontend.

## ▶️ Running the Project

### Start the backend

From the repository root, activate the virtual environment and run:

``` powershell
uvicorn app.main:app --host 127.0.0.1 --port 8000 --reload
```

When startup succeeds, open:

-   API root: `http://127.0.0.1:8000`
-   Interactive API documentation: `http://127.0.0.1:8000/docs`

The root URL may not display a webpage unless a root route is
implemented. The `/docs` page is the more useful initial check for a
FastAPI application.

### Start the frontend

In a separate terminal:

``` powershell
cd frontend
npm start
```

If `npm start` is not defined, inspect the `scripts` section in
`frontend/package.json` and run the available development script,
commonly `npm run dev` or `npm start` depending on the project setup.

Ensure the frontend's API base URL points to the backend address you are
using. The frontend and backend must both be running for the full web
application to work locally.

## 🖱️ Using the Application

The expected workflow is:

1.  Open the frontend in a browser.
2.  Register or sign in if authentication is enabled.
3.  Navigate to the image-detection page.
4.  Upload a JPG, JPEG, or PNG image, or use webcam capture if
    available.
5.  Submit the image for analysis.
6.  Review the overall prediction and face-level results.
7.  Inspect available ELA, noise, or metadata information.
8.  Download the report if report generation is enabled.
9.  Visit the history page to review saved analyses, if configured.

Exact button names and page behavior depend on the current frontend
implementation.

## 📚 API Documentation

FastAPI's interactive documentation is normally available at:

``` text
http://127.0.0.1:8000/docs
```

Use it to inspect the actual registered routes, request schemas,
response structures, and available operations. The project includes
authentication, prediction, and history-related functionality; consult
the live documentation and router files for exact endpoint paths rather
than relying on assumed routes.

## 📦 Model Files and Git LFS

The exported ONNX model is large, so it should not be committed as an
ordinary Git blob. If Git LFS is used, the repository should track the
relevant model extension in `.gitattributes`.

To confirm Git LFS tracking:

``` powershell
git lfs ls-files
```

To download tracked objects after cloning:

``` powershell
git lfs pull
```

Git LFS storage and bandwidth quotas may apply. If the model cannot be
distributed through Git LFS, host it through an appropriate release or
model-storage service and document how to obtain it. Do not commit a
large PyTorch training checkpoint unless there is a clear reason to
distribute it.

## 🔐 Security and Privacy

-   Validate uploaded file types and sizes on the server, not only in
    the browser.
-   Treat uploaded images, face crops, metadata, and PDF reports as
    potentially sensitive.
-   Review local storage and database retention policies before
    deploying publicly.
-   Restrict access to user-specific history and reports.
-   Use HTTPS and appropriate authentication/session protections in
    production.
-   Avoid publishing real users' uploaded images or reports in the
    repository.
-   A SHA-256 hash can help identify file changes; it does not encrypt
    an image or hide its contents.

## ⚠️ Limitations

-   The model may perform differently on images from sources or
    manipulation methods not represented in training data.
-   Compression, resizing, lighting, occlusion, and face-detection
    errors can affect the result.
-   ELA, noise maps, and EXIF information are not individually
    definitive indicators of manipulation.
-   Hashing supports file-integrity comparisons but cannot establish the
    truthfulness of the image content.
-   This is an image-focused workflow. Video-level detection is not
    implied by webcam frame capture.
-   The system is a decision-support tool and should not be the sole
    basis for high-impact decisions.

## 🛠️ Troubleshooting

### `Could not import module "main"`

Run the command from the repository root and use the module path
matching the file structure. If the entry point is `app/main.py`, the
command is:

``` powershell
uvicorn app.main:app --reload
```

### Model file is missing or appears as a small text file

The file may be a Git LFS pointer rather than the actual model. Install
Git LFS and run:

``` powershell
git lfs pull
git lfs ls-files
```

Confirm the downloaded model has the expected size before running
inference.

### Hugging Face authentication warning

Some model or configuration downloads may warn that requests are
unauthenticated. Public assets may still download, but rate limits can
apply. Only configure a Hugging Face token if the code actually needs
Hub access; keep the token private and out of source control.

### CLIP / QuickGELU configuration warning

A model configuration warning can indicate that the selected
architecture configuration and pretrained weights do not exactly match.
Verify that the architecture, pretrained checkpoint, and preprocessing
are the same as those used in training. Do not change model settings
blindly, as this can affect predictions.

### Frontend cannot reach the backend

-   Confirm the backend is running.
-   Check the frontend API base URL.
-   Check browser console and backend logs.
-   Verify CORS settings if frontend and backend run on different
    origins.

### Database connection errors

Check the configured connection string, network access, database
availability, and any required environment variables. Do not paste
credentials into public issue reports.

## 🤝 Contributing

Contributions and suggestions are welcome.

1.  Fork the repository.
2.  Create a branch for your change.
3.  Make focused changes and test them.
4.  Update documentation when behavior or setup changes.
5.  Open a pull request with a clear summary.

Please do not submit secrets, personal data, private uploads, generated
reports, datasets with redistribution restrictions, or unnecessarily
large artifacts.

## 📄 License

This repository is intended to use the **MIT License**. See the
[`LICENSE`](LICENSE) file for the full license text.

Third-party dependencies, pretrained weights, and datasets may have
separate terms. Review their licenses and usage restrictions before
redistribution or commercial use. Ensure the repository's actual
`LICENSE` file matches this section.

## 🙏 Acknowledgments

This project builds on open-source tools and research, including:

-   [CLIP](https://github.com/openai/CLIP)
-   [OpenCLIP](https://github.com/mlfoundations/open_clip)
-   [PyTorch](https://pytorch.org/)
-   [ONNX Runtime](https://onnxruntime.ai/)
-   [FastAPI](https://fastapi.tiangolo.com/)
-   [React](https://react.dev/)
-   [PostgreSQL](https://www.postgresql.org/)

Please consult each project's official documentation and license for
details.

------------------------------------------------------------------------
