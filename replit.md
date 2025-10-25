# FTIR UHMWPE Analyzer

## Overview
This is a static web application for analyzing FTIR (Fourier-Transform Infrared Spectroscopy) data for UHMWPE (Ultra-High Molecular Weight Polyethylene) samples. The application provides advanced spectroscopic analysis capabilities including:

- Spectrum visualization and manipulation
- Baseline correction
- Peak deconvolution
- Multiple spectrum analysis
- Data export (Excel, PDF)

## Project Architecture
- **Type**: Single-page static web application
- **Main file**: `index.html` (540KB, fully self-contained)
- **Dependencies**: All loaded via CDN (Plotly, PapaParse, XLSX, jsPDF, Bootstrap, etc.)
- **No build process required**: Pure HTML/CSS/JavaScript

## Recent Changes
- 2025-10-25: Initial import and Replit environment setup
  - Configured Python HTTP server for static file serving
  - Set up workflow to serve on port 5000
  - Configured deployment for autoscale
  - Fixed Multiple Deconvolution table to show all 6 carbonyl species (was showing only 3)
  - Ensured consistency between Individual and Multiple Deconvolution results
  - Reorganized tab structure into 3 main categories:
    * Index Analysis (Oxidation, Crystallinity, TVI, Reports + Full Automatic Analysis button)
    * Carbonyl Deconvolution (Individual and Multiple sub-tabs) - RED header
    * Adsorbed Species Quantification (Vitamin E and Synovial Liquid - placeholders) - GREEN header
  - Added color-coded headers and tabs for better visual organization:
    * Carbonyl Deconvolution: Red gradient (#e74c3c to #c0392b)
      - Main tab button turns red when active
      - Sub-tabs (Individual/Multiple) also use red when active
    * Adsorbed Species Quantification: Green gradient (#27ae60 to #229954)
      - Main tab button turns green when active
      - Sub-tabs (Vitamin E/Synovial Liquid) also use green when active

## Project Structure
```
/
├── index.html              # Main application (self-contained)
├── logo-gbm.png           # Application logo
├── logo.png               # Secondary logo
└── *.png                  # Additional UI assets
```

## Tech Stack
- **Frontend**: Vanilla JavaScript, Bootstrap 5, HTML5, CSS3
- **Charting**: Plotly.js
- **Data Processing**: PapaParse, XLSX.js
- **PDF Generation**: jsPDF, html2canvas
- **Server**: Python HTTP server (for static file serving)

## Development
- Server runs on port 5000
- No compilation or build steps needed
- All changes to index.html are immediately visible on refresh
