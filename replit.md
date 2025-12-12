# FTIR UHMWPE Analyzer

## Overview
A comprehensive web-based tool for analyzing Fourier-Transform Infrared (FTIR) spectroscopy data of Ultra-High Molecular Weight Polyethylene (UHMWPE) samples. This application provides advanced spectroscopic analysis capabilities for material scientists and researchers studying biomedical polymer degradation.

## Project Structure
```
FTIR-UHMWPE-Analyzer/
├── index.html          # Main application (single-page HTML with embedded CSS/JS)
├── server.py           # Python HTTP server for serving static files
├── logo-gbm.png        # Application logo
├── favicon.png         # Browser favicon
├── README.md           # Detailed documentation
└── replit.md           # This file
```

## Technology Stack
- **Frontend**: Static HTML/CSS/JavaScript (single-page application)
- **Backend**: Python 3.11 simple HTTP server
- **External Libraries** (loaded via CDN):
  - Plotly.js 2.26.0 - Interactive charting
  - PapaParse 5.4.1 - CSV parsing
  - SheetJS 0.18.5 - Excel file handling
  - jsPDF 2.5.1 - PDF generation
  - Bootstrap 5.3.0 - UI framework
  - Font Awesome 6.4.0 - Icons

## Running the Application
The server runs on `0.0.0.0:5000` with cache control headers disabled for development.

Start command:
```bash
python server.py
```

## Key Features
- Multi-format support (CSV, TXT, Excel, DPT, SPC, OPUS)
- Multi-depth spectral analysis
- Automatic baseline correction
- Peak deconvolution with Gaussian-Lorentzian profiles
- Species quantification using Beer-Lambert law
- PDF and Excel report generation

## Architecture
- Single-page static web application
- No build process required
- All functionality contained within index.html
- Server provides simple HTTP hosting with cache control

## Recent Changes
- December 12, 2025: Initial Replit environment setup with Python 3.11
