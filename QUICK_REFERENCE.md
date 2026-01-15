# FTIR Analyzer - Quick Reference Summary

## 1. DATA IMPORT SECTION LOCATION
- **Main Card**: Lines 2117-2260
- **Single File Tab**: Lines 2144-2192 (ID: `single-file`)
- **Multi-File Tab**: Lines 2194-2231 (ID: `multi-files`)
- **File Inputs**: `#fileInput` and `#multiFileInput`
- **Load/Clear Buttons**: Lines 2235-2246

## 2. FULL AUTOMATIC ANALYSIS BUTTON

### Visual Location: Lines 2438-2459
- **Button ID**: `#fullAutoAnalysisBtn`
- **Background**: Green success gradient (`var(--success-color)`)
- **Icon**: `fa-play` spinner

### Handler Function: `performFullAutoAnalysis()`
- **Location**: Lines 13396-13560
- **Event Registration**: Line 13725

### Execution Sequence:
```
1. Apply Offset Correction          (applyOffsetCorrection)
2. Analyze Oxidation               (performAutoAnalysis('oxidation'))
3. Analyze Crystallinity           (performAutoAnalysis('crystallinity'))
4. Analyze TVI                     (performAutoAnalysis('tvi'))
5. Generate Depth Profile          (updateDepthProfile)
6. Show Results Summary             (showAnalysisCompleteSummary)
```

## 3. KEY ANALYSIS FUNCTIONS

### Oxidation Analysis
- **Methods**: Area-Based (OI - ASTM F2102) or Height-Based (KOI)
- **Interest Peak**: 1650-1760 cm⁻¹
- **Reference Peak**: 2800-3000 cm⁻¹
- **Results Location**: `window.AppState.analysisResults.oxidation`
- **Plot IDs**: `#oxidationReferencePlot`, `#oxidationInterestPlot`

### Crystallinity Analysis
- **Methods**: Area-Based (CI) or Height-Based (CI)
- **Interest Peak**: 900-970 cm⁻¹ (crystalline region)
- **Reference Peak**: 2800-3000 cm⁻¹
- **Results Location**: `window.AppState.analysisResults.crystallinity`
- **Plot IDs**: `#crystallinityReferencePlot`, `#crystallinityInterestPlot`

### TVI (Thermal Oxidative Degradation) Analysis
- **Interest Peak**: 1200-1300 cm⁻¹ (C-O stretching)
- **Reference Peak**: 2800-3000 cm⁻¹
- **Results Location**: `window.AppState.analysisResults.tvi`
- **Plot IDs**: `#tviReferencePlot`, `#tviInterestPlot`
- **Irradiation Dose Formula**: `4196.8 × meanTVI - 12.331` (kGy)

## 4. RESULT STORAGE & ACCESS

### AppState Structure (Lines 4517-4540)
```javascript
window.AppState = {
    spectraData: {           // {depthLabel: {frequency, transmittance, ...}}
    analysisResults: {       // {analysisType: {depthLabel: {index, ratio, ...}}}
    maskedDepths: Set(),     // Depths to exclude
    currentFileName: '',
    loadedFiles: []
}
```

### Get All Results Function
**Location**: Lines 11721-11751
**Function**: `getAllAnalysisResults()`
**Returns**: 
```javascript
{
    depths: [100, 200, 300, ...],      // Numeric values
    oxidation: [1.234, 2.345, ...],
    crystallinity: [5.678, 6.789, ...],
    tvi: [0.123, 0.234, ...]
}
```

## 5. FILE INPUT HANDLING

### Single File Mode
- **Supported Formats**: .xlsx, .csv, .txt, .dpt, .spc, .0
- **Storage**: `window.selectedFile`
- **Handler**: `handleFileSelection()` (Line 4607)

### Multiple File Mode
- **Format**: .dpt only
- **Naming Convention**: `SAMPLE_NAME_01.dpt`, `SAMPLE_NAME_02.dpt`
- **Auto-Parsing**: Extracts depth from filename suffix (01=100μm, 02=200μm, etc.)
- **Storage**: `window.selectedFiles[]` (sorted by depth)
- **Handler**: `handleMultiFileSelection()` (Line 4624)

### Data Processing
- **Single File**: `loadSpectrumData()` → format-specific loader → `processLoadedData()`
- **Multi-File**: `loadMultipleDPTFiles()` → `processMultiFileData()`
- **Result**: Populates `window.AppState.spectraData`

## 6. EXCEL EXPORT FUNCTIONALITY

### Location: Lines 12653-12787
**Function**: `exportToExcel(results, spectraData, filename, options)`

### Generated Sheets:

| Sheet | Contains |
|-------|----------|
| "Analysis Results" | Depth, OI/KOI, CI, TVI values |
| "Statistics" | Mean, StdDev, Min, Max, Count + Max value summary |
| "Parameters" | Analysis peak intervals (if `includeParameters: true`) |
| "Raw Data" | Frequency + all depth transmittance values (if `includeRawData: true`) |

### Export Trigger
- **Export Modal**: `createExportModal()` (Line 12437)
- **Button Listener**: Event on export buttons in modal
- **Execution**: `exportData(format)` → format-specific function

## 7. PAGE STRUCTURE FOR MULTI-FILE

### Tab Navigation
- **Container**: `#importModeNav` (Lines 2121-2135)
- **Tabs**: 
  - `#single-file-tab` → shows `#single-file` pane
  - `#multi-files-tab` → shows `#multi-files` pane

### Multi-File Tab Contents (Lines 2194-2231)
```
Multi-File Upload Section
├── File Input Section (.col-md-8)
│   ├── #multiFileInput (file[multiple], .dpt only)
│   └── Instructions
├── Depth Config (.col-md-4)
│   └── #autoDepthDetection (checkbox)
└── File Preview (#multiFilePreview)
    └── #multiFileList (dynamically populated)
```

### File Preview Display
- **Function**: `displayMultiFilePreview()` (Line 4669)
- **Grouped By**: Sample name (extracted from filename)
- **Shows**: Filename and depth badge for each file

## 8. CURRENT PAGE SECTIONS

### Visible After Data Load:
1. **Spectrum Overview** (`#spectrumOverview`) - Lines 2262-2400
   - Overview plot, spectrum list, depth info, masking controls

2. **Processing Controls** (`#processingControls`) - Lines 2402-2433
   - Offset correction, baseline, processing status

3. **Analysis Section** (`#analysisSection`) - Lines 2435+
   - Main tabs: Index Analysis, Deconvolution, Absorbed Species
   - Contains full auto button and sub-analyses

## 9. QUICK LOOKUP TABLE

| Need to Find... | Location |
|-----------------|----------|
| Import UI HTML | Lines 2117-2260 |
| Full Auto Button | Lines 2438-2459, 13396-13560 |
| Oxidation Results | Lines 2523-2567, `window.AppState.analysisResults.oxidation` |
| Crystallinity Results | Lines 2734-2778, `window.AppState.analysisResults.crystallinity` |
| TVI Results | Lines 2909-2953, `window.AppState.analysisResults.tvi` |
| Analysis Parameters | `ANALYSIS_PARAMETERS` object (search: "interestPeak") |
| Method Selection Sync | Lines 13415-13428 |
| Results Consolidation | `getAllAnalysisResults()` (Line 11721) |
| Excel Export | Lines 12653-12787 |
| File Handlers Setup | `initializeFileHandlers()` (Line 4588) |
| Spectrum Data Store | `window.AppState.spectraData` (Line 4523) |

## 10. IMPORTANT NOTES

✓ **Depth Labeling**: All depths stored as "XXXμm" (e.g., "100μm")

✓ **Multi-file Sorting**: Automatic by depth number in filename

✓ **Data Backup**: Original transmittance in `originalTransmittance` (prevents corruption)

✓ **Method Auto-Sync**: Full auto analysis propagates method selections to individual tabs

✓ **Masking**: Depths can be masked via UI; excluded from all analysis

✓ **Excel Export**: Multi-sheet output with optional raw data and statistics

✓ **Irradiation Dose**: Calculated from TVI using: `4196.8 × TVI - 12.331`

✓ **File Auto-Detection**: Format determined by file extension
