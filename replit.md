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
- 2025-11-05: **Lorentzian Range Optimization - Literature-Based Constraints**
  - **Updated Lorentzian optimization constraints** to use literature-specific ranges per species
  - **Modified species definitions** (lines 9698-9699, 9716-9717):
    - gamma_ketoacids_keto: Lorentzian range adjusted to 87-93% (was 88-92%, ±3% of 90%)
    - carboxylic_acids_isolated: Lorentzian range adjusted to 87-93% (was 88-92%, ±3% of 90%)
  - **Updated Adam optimization** (lines 10464-10468):
    - Uses speciesInfo.lorentzianMin and lorentzianMax instead of generic 0-1 limits
    - Ensures each species optimizes within its literature-defined range
  - **Updated gradient descent** (lines 10763-10770):
    - Replaced generic ±5% margin with species-specific min/max ranges
    - Consistent with Adam optimization approach
  - **Impact**: Both Individual and Multiple Deconvolution now respect literature bounds:
    - Ketones: 90-93%
    - Carboxylic Acids (associated): 85-95%
    - Gamma-ketoacids (acid): 90-94%
    - Gamma-ketoacids (keto): 87-93%
    - Esters & Aldehydes: 85-90%
    - Carboxylic Acids (isolated): 87-93%
  - Changes apply to both deconvolution modes (individual and multiple)
  - Workflow restarted and verified functioning correctly

- 2025-11-05: **Replit Environment Setup - Fresh GitHub Import**
  - Successfully configured fresh GitHub import for Replit environment
  - Installed Python 3.11 module for static file serving
  - Created .gitignore with Python and IDE exclusions
  - Configured webserver workflow:
    - Command: `python -m http.server 5000`
    - Port: 5000 with webview output
    - Status: Running successfully
  - Set up autoscale deployment configuration for production
  - Verified application loads correctly with all CDN dependencies
  - Confirmed UI rendering with screenshot (GBM logo, analyzer interface visible)
  - Browser console shows successful initialization
  - Application fully functional and ready for FTIR data analysis

- 2025-11-03: **CRITICAL BUG FIX** - Concentration Consistency Between Deconvolution Modes
  - **Fixed concentration discrepancy**: Individual and Multiple Deconvolution now produce identical concentrations
  - **Root cause**: `calculateReferenceBandArea()` was using the first loaded spectrum instead of the selected depth
  - **Solution**: Modified function to use `deconvDepthSelect.value` to match the currently selected spectrum
  - **Edge case fix**: Added auto-selection of first depth in `populateDepthSelector()` to prevent null spectrum errors
  - **UI improvement**: Changed concentration format from `.toFixed(6)` to `.toFixed(4)` for better readability
  - **Impact**: Both modes now use identical methodology:
    - Same reference band calculation (2000-2040 cm⁻¹)
    - Same Adam optimization algorithm
    - Same concentration formula: C = (Area_Simpson / (ε * Area_Ref)) × 1000
  - Concentration values now display with 4 decimal places (no scientific notation)
  - Verified by architect review - no side effects or security issues

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
