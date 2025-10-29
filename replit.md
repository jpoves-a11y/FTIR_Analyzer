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
- 2025-10-29: **Favicon Added**
  - Custom favicon/browser tab icon added to the application
  - Image saved as `favicon.png` in the root directory
  - HTML includes both standard favicon and Apple touch icon references
  - Compatible with GitHub Pages deployment

- 2025-10-29: **UI Improvements for Carbonyl Deconvolution**
  - **Inverted X-axis**: Wavenumber axis now displays from high to low (left to right) for better spectroscopic convention
  - **Horizontal Species Quantification Table**: Moved table below the deconvolution graph and redesigned as horizontal layout
  - **Compact table design**: Reduced column widths and font sizes for better data density
    - Column width optimized (95-110px) to fit all species names
    - Font size reduced to 0.75rem for better space utilization
    - Reduced padding (6px vertical, 2px horizontal) for compact display
  - **Concentration units**: Changed from mol/cm³ to mmol/cm³ for more readable values
  - Table shows all 6 carbonyl species in columns with parameters in rows:
    - Frequency, Percentage, Simpson Area, Intensity, Width, Lorentzian %, Concentration (mmol/cm³), ε
  - Enhanced readability with color-coded species headers matching graph traces
  - Improved data visualization and comparison between species

- 2025-10-29: Replit environment setup completed for GitHub import
  - Verified Python 3.11 installation
  - Confirmed workflow configuration on port 5000 with webview output
  - Verified .gitignore for Python environment files
  - Confirmed autoscale deployment configuration for production
  - Tested application - fully functional with all CDN dependencies loading correctly
  - Application successfully processes FTIR data files and displays analysis results

- 2025-10-28: **MAJOR UPDATE** - Full Adam optimization for Lorentzian-Gaussian parameter in deconvolution
  - **Integrated Adam optimizer** for Lorentzian-Gaussian mixing parameter (previously only intensity and width were optimized)
  - **500 iterations** with momentum-based optimization for all three parameters: intensity, width, and Lorentzian fraction
  - **Reduced margin constraint**: Lorentzian parameter iterates within ±5% of initial value (e.g., 90% → 85.5%-94.5%)
  - **Adam hyperparameters**: β1=0.9, β2=0.999, ε=1e-8, learning_rate=0.05
  - **Numerical gradient**: Uses finite differences (δ=0.005) for precise gradient calculation
  - **Real-time logging**: Shows Lorentzian values every 50 iterations during optimization
  - **Final report**: Displays initial → optimized Lorentzian percentages for all 6 carbonyl species
  - **Global parameter storage**: Optimized values saved in window.optimizedParams for consistent use across visualizations
  - **Updated visualization**: Added Lorentzian percentage (L: XX.X%) to species quantification table
  - **Enhanced tooltips**: Plotly graphs show optimized Lorentzian percentage in hover information
  - **Console analytics**: Complete logs showing Lorentzian values in area calculations
  - Benefits both Individual and Multiple Deconvolution analyses
  - Ensures mathematically consistent peak shape optimization across all carbonyl species
  
- 2025-10-28: Replit environment setup completed
  - Installed Python 3.11 for serving static files
  - Configured Python HTTP server workflow on port 5000 with webview output
  - Added .gitignore for Python environment files
  - Configured autoscale deployment for production (uses python -m http.server)
  - Verified application loads correctly with all CDN dependencies
  - Application fully functional and ready to use

- 2025-10-25: Initial development
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
