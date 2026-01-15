# FTIR UHMWPE Analyzer - Comprehensive Code Analysis

## 1. DATA IMPORT SECTION STRUCTURE

### Location: Lines 2117-2260 (Main Card Block)
- **Card Header**: Line 2117 - `Data Import` section title with upload icon
- **Import Mode Selection**: Lines 2120-2135 - Two-tab system with navigation pills

### Import Mode Options:

#### 1.1 Single File Mode (Multi-depth)
- **Tab ID**: `single-file-tab` (Line 2121)
- **Container**: `#single-file` (Lines 2144-2192)
- **Line Range**: 2144-2192
- **Features**:
  - File input element: `#fileInput` (Line 2149)
  - Accepts: `.xlsx`, `.dpt`, `.spc`, `.txt`, `.csv`, `.0`
  - Hidden form controls:
    - `#autoDetectColumns` (Line 2162)
    - `#skipFirstRow` (Line 2163)
  - Column mapping section (Line 2165-2185) - hidden until needed
    - `#frequencyColumn` dropdown
    - `#depthColumns` checkbox container

#### 1.2 Multiple Depth Files Mode
- **Tab ID**: `multi-files-tab` (Line 2128)
- **Container**: `#multi-files` (Lines 2194-2231)
- **Line Range**: 2194-2231
- **Features**:
  - File input element: `#multiFileInput` (Line 2199)
  - Accepts: `.dpt` files only
  - Expected naming convention: `SAMPLE_NAME_01`, `SAMPLE_NAME_02` (where 01 = 100μm, 02 = 200μm)
  - Auto-detection checkbox: `#autoDepthDetection` (Line 2206)
  - File preview: `#multiFilePreview` (Lines 2215-2231)
    - Shows list of selected files with depth values as badges

### Data Import UI Controls:
- **Load Button**: `#loadDataBtn` (Line 2235) - Triggers `loadSpectrumData()` function
- **Clear Button**: `#clearDataBtn` (Line 2240) - Triggers `clearAllData()` function
- Both buttons initially disabled

### File Input Handlers
- **Handler Initialization**: `initializeFileHandlers()` function (Line 4588)
  - Wires up change events for both file inputs
  - Updates label text and enables load button
  - Stores files in `window.selectedFile` or `window.selectedFiles`

---

## 2. FULL AUTOMATIC ANALYSIS BUTTON

### Location: Lines 2438-2459 and 13396-13560

### Visual Element:
- **Button ID**: `#fullAutoAnalysisBtn` (Line 2438)
- **Container Card**: Lines 2404-2459
- **CSS Styling**: Lines 1867-1895
- **Appearance**: Green success gradient background with white text
- **Text Content**: "Start Complete Analysis"

### CSS Properties (Lines 1867-1895):
```css
#fullAutoAnalysisBtn {
    background: var(--success-color);  /* #2E7D6B */
    color: white;
    font-weight: 700;
    position: relative;
}

#fullAutoAnalysisBtn:hover .progress-bar-overlay {
    left: 100%;  /* Animated progress bar effect */
}

#fullAutoAnalysisBtn .btn-content {
    position: relative;
    z-index: 2;
}
```

### Handler Function: `performFullAutoAnalysis()` (Lines 13396-13560)

#### Execution Flow:
1. **Validation** (Lines 13398-13401):
   - Checks if spectrum data is loaded
   - Shows warning if no data

2. **Button State Management** (Lines 13406-13408):
   - Disables button during analysis
   - Changes text to "Analyzing..."
   - Adds spinning loading icon

3. **Method Selection Sync** (Lines 13415-13428):
   - Reads from full-auto dropdowns:
     - `#fullAutoOxidationMethod`
     - `#fullAutoCrystallinityMethod`
   - Syncs to individual tab selections:
     - `#oxidationMethod`
     - `#crystallinityMethod`

4. **Sequential Processing Steps**:
   - **Step 1** (Lines 13434-13442): Offset Correction
     - Calls `applyOffsetCorrection()`
   - **Step 2** (Lines 13444-13453): Oxidation Analysis
     - Calls `performAutoAnalysis('oxidation')`
   - **Step 3** (Lines 13455-13464): Crystallinity Analysis
     - Calls `performAutoAnalysis('crystallinity')`
   - **Step 4** (Lines 13466-13475): TVI Analysis
     - Calls `performAutoAnalysis('tvi')`
   - **Step 5** (Lines 13477-13485): Depth Profile Generation
     - Calls `updateDepthProfile()`

5. **Results Display** (Lines 13492-13494):
   - Calls `showAnalysisCompleteSummary()`
   - Shows notification with statistics

#### Event Listener:
- **Registration**: Line 13725
- **Trigger**: Click event on `#fullAutoAnalysisBtn`

---

## 3. ANALYSIS FUNCTIONS

### 3.1 Oxidation Analysis

#### Key Functions:
- **Auto-Analysis Trigger**: `performAutoAnalysis('oxidation')` (Lines 7499+)
- **Calculation Function**: Internal logic based on selected method
- **Methods Available**:
  - **Area-Based (OI)**: ASTM F2102-17 standard (default)
  - **Height-Based (KOI)**: Alternative method by Solberg et al.

#### Analysis Parameters (ANALYSIS_PARAMETERS object):
```javascript
oxidation: {
    interestPeak: {min: 1650, max: 1760},  // cm⁻¹
    referencePeak: {min: 2800, max: 3000}   // cm⁻¹
}
```

#### Results Storage:
- **Location**: `window.AppState.analysisResults.oxidation`
- **Structure**:
  ```javascript
  {
    'depthLabel (e.g., "100μm")': {
        index: number,        // OI or KOI value
        ratio: number,        // Normalized ratio
        method: 'area'|'height',
        referenceArea: number,
        interestArea: number
    }
  }
  ```

#### Display Elements:
- **Reference Interval Plot**: `#oxidationReferencePlot` (Lines 2523-2540)
- **Interest Interval Plot**: `#oxidationInterestPlot` (Lines 2542-2559)
- **Results Table**: `#oxidationResults` (Lines 2561-2567)
- **Toggle/Reset Buttons**:
  - `#toggleOxidationRefViewBtn` / `#toggleOxidationIntViewBtn`
  - `#resetOxidationRefZoomBtn` / `#resetOxidationIntZoomBtn`

#### Plotting Functions:
- `plot3DOxidationReference()` / `plotIndividualOxidationReference()` (Lines 7773-7839)
- `plot3DOxidationInterest()` / `plotIndividualOxidationInterest()` (Lines 7843-7909)
- `toggleOxidationView(type)` (Line 7754) - Switches between 3D/individual/overlay

---

### 3.2 Crystallinity Analysis

#### Key Functions:
- **Auto-Analysis Trigger**: `performAutoAnalysis('crystallinity')`
- **Methods Available**:
  - **Area-Based (CI)**: Area-based method (default)
  - **Height-Based (CI)**: Height-based method

#### Analysis Parameters:
```javascript
crystallinity: {
    interestPeak: {min: 900, max: 970},    // cm⁻¹ (900-970 cm⁻¹ crystalline region)
    referencePeak: {min: 2800, max: 3000}  // cm⁻¹ (CH₂ stretching)
}
```

#### Results Storage:
- **Location**: `window.AppState.analysisResults.crystallinity`
- **Structure**: Same as oxidation analysis

#### Display Elements:
- **Reference Interval Plot**: `#crystallinityReferencePlot` (Lines 2734-2751)
- **Interest Interval Plot**: `#crystallinityInterestPlot` (Lines 2753-2770)
- **Results Table**: `#crystallinityResults` (Lines 2772-2778)
- **Toggle/Reset Buttons**:
  - `#toggleCrystallinityRefViewBtn` / `#toggleCrystallinityIntViewBtn`
  - `#resetCrystallinityRefZoomBtn` / `#resetCrystallinityIntZoomBtn`

#### Plotting Functions:
- `plot3DCrystallinityReference()` / `plotIndividualCrystallinityReference()` (Lines 7949-8015)
- `plot3DCrystallinityInterest()` / `plotIndividualCrystallinityInterest()` (Lines 8019-8085)

---

### 3.3 TVI (Thermal Oxidative Degradation Index) Analysis

#### Key Functions:
- **Auto-Analysis Trigger**: `performAutoAnalysis('tvi')`
- **Calculation**: TVI index from ratio of carbonyl and reference peaks

#### Analysis Parameters:
```javascript
tvi: {
    interestPeak: {min: 1200, max: 1300},  // cm⁻¹ (C-O stretching)
    referencePeak: {min: 2800, max: 3000}  // cm⁻¹ (CH₂ stretching)
}
```

#### Results Storage:
- **Location**: `window.AppState.analysisResults.tvi`

#### Irradiation Dose Calculation:
- **Formula**: `Dose (kGy) = 4196.8 * meanTVI - 12.331`
- **Function**: `updateIrradiationDoseDisplay(meanTvi)` (Lines 11775-11823)
- **Display**: `#irradiationDoseResult` element
- **Comparison**: Can be compared with manufacturer-provided dose

#### Display Elements:
- **Reference Interval Plot**: `#tviReferencePlot` (Lines 2909-2926)
- **Interest Interval Plot**: `#tviInterestPlot` (Lines 2928-2945)
- **Results Table**: `#tviResults` (Lines 2947-2953)
- **Irradiation Dose Display**: `#irradiationDoseResult` (Lines 2994-2996)
- **Manufacturer Dose Input**: `#manufacturerDose` (optional, for comparison)

#### Plotting Functions:
- `plot3DTVIReference()` / `plotIndividualTVIReference()` (Lines 8125-8191)
- `plot3DTVIInterest()` / `plotIndividualTVIInterest()` (Lines 8195-8261)

---

## 4. RESULT STORAGE AND ACCESS

### AppState Structure (Lines 4517-4540)

```javascript
window.AppState = {
    rawData: null,                          // Original file data
    processedData: null,                    // After processing
    spectraData: {},                        // {depthLabel: {frequency, transmittance, ...}}
    maskedDepths: new Set(),                // Depths to ignore
    analysisResults: {
        oxidation: {},                      // {depthLabel: {index, ratio, method, ...}}
        crystallinity: {},                  // {depthLabel: {index, ratio, ...}}
        tvi: {},                            // {depthLabel: {index, ratio, ...}}
        vitaminE: {},
        dualIndex: {}
    },
    processingStatus: {
        offsetCorrected: false,
        baselineCorrected: false
    },
    currentFileName: '',
    loadedFiles: []
}
```

### getAllAnalysisResults() Function (Lines 11721-11751)

**Purpose**: Consolidate all analysis results into a unified format

**Parameters**: None (reads from `window.AppState`)

**Returns**:
```javascript
{
    depths: [100, 200, 300, ...],           // Numeric depth values
    oxidation: [1.234, 2.345, ...],         // Index values per depth
    crystallinity: [5.678, 6.789, ...],     // Index values per depth
    tvi: [0.123, 0.234, ...]                // Index values per depth
}
```

**Logic**:
1. Extracts all unique depths from all analysis types
2. Filters out masked depths (from `window.AppState.maskedDepths`)
3. Returns values in depth order (ascending)
4. Returns `null` for missing values

**Usage Locations**:
- Line 7499: Depth profile generation
- Line 7635: Results export
- Line 11873: PDF report generation
- Line 11995: CSV export
- Line 12056: JSON export
- Line 12525: Excel export
- Line 13148: PDF report with plots

---

### Analysis Results Access Patterns

#### Direct Access:
```javascript
// Get specific depth result
window.AppState.analysisResults.oxidation['100μm'].index  // Returns OI value

// Get all depths for analysis type
Object.keys(window.AppState.analysisResults.oxidation)     // Returns depth labels

// Check processing status
window.AppState.processingStatus.offsetCorrected
window.AppState.processingStatus.baselineCorrected
```

#### Via getAllAnalysisResults():
```javascript
const results = getAllAnalysisResults();
results.depths[0]           // First depth value
results.oxidation[0]        // Oxidation index at first depth
results.crystallinity[0]    // Crystallinity index at first depth
results.tvi[0]              // TVI index at first depth
```

---

## 5. FILE INPUT HANDLING PATTERNS

### Single File Upload (Lines 4607-4622)
```javascript
function handleFileSelection(event) {
    const file = event.target.files[0];
    window.AppState.currentFileName = file.name;
    window.AppState.loadedFiles = [file.name];
    window.selectedFile = file;  // Store for processing
    // Enable load button
}
```

**Supported Formats**:
- `.xlsx` - Excel files
- `.csv` - CSV files
- `.txt` - Text files
- `.dpt` - Origin DPT files
- `.spc` - Galactic SPC files
- `.0` - Binary spectral format

### Multiple File Upload (Lines 4624-4667)
```javascript
function handleMultiFileSelection(event) {
    const files = Array.from(event.target.files);
    
    // Parse and sort by depth number in filename
    const parsedFiles = files.map(file => {
        const depthMatch = file.name.match(/(\d+)\.dpt$/i);
        const depth = depthMatch ? parseInt(depthMatch[1]) * 100 : 0;
        return { file, depth, originalName: file.name };
    }).sort((a, b) => a.depth - b.depth);
    
    window.AppState.loadedFiles = files.map(f => f.name);
    window.selectedFiles = parsedFiles;  // Store for processing
}
```

**Expected Naming**: `SAMPLE_NAME_01.dpt`, `SAMPLE_NAME_02.dpt`, etc.

### File Loading (Lines 4739-4806)
```javascript
async function loadSpectrumData() {
    if (window.selectedFiles) {
        // Multi-file mode
        data = await loadMultipleDPTFiles(window.selectedFiles);
    } else {
        // Single file mode - determine format by extension
        const ext = getFileExtension(window.selectedFile.name);
        
        switch(ext) {
            case 'xlsx': data = await loadExcelFile(...);
            case 'csv': data = await loadCSVFile(...);
            case 'txt': data = await loadTextFile(...);
            case 'dpt': data = await loadDPTFile(...);
            case 'spc': data = await loadSPCFile(...);
            case '0': data = await loadBinaryFile(...);
        }
    }
}
```

### Spectrum Data Storage (Lines 5143-5154)
```javascript
// After loading, spectra stored as:
window.AppState.spectraData = {
    '100μm': {
        frequency: [800, 801, ...],       // cm⁻¹ values
        transmittance: [0.95, 0.94, ...], // Absorbance values
        originalTransmittance: [...],     // Backup copy
        depth: 100,                        // Numeric depth
        color: '#FF6B6B',                 // Assigned color
        sampleName: 'SAMPLE_A',
        originalName: 'SAMPLE_A_01.dpt'
    },
    '200μm': { ... }
}
```

---

## 6. EXCEL EXPORT FUNCTIONALITY

### Location: Lines 12653-12787

### Export Function: `exportToExcel(results, spectraData, filename, options)`

**Parameters**:
- `results` - From `getAllAnalysisResults()`
- `spectraData` - `window.AppState.spectraData`
- `filename` - Base filename (without extension)
- `options` - Object with flags:
  - `includeRawData`: Include raw spectral data
  - `includeStatistics`: Include statistical summaries
  - `includeParameters`: Include analysis parameters

### Generated Sheets:

#### Sheet 1: "Analysis Results" (Lines 12670-12690)
**Content**:
- Header information (sample file, export date, methods used)
- Column headers: Depth, OI/KOI, CI, TVI
- Data rows with analysis results for each depth

#### Sheet 2: "Statistics" (Lines 12692-12717) - *optional*
**Content**:
- Statistical summaries:
  - Mean, Std Dev, Min, Max, Count
  - For each analysis type (oxidation, crystallinity, tvi)
- Maximum values section:
  - Shows which depth has maximum value for each type

#### Sheet 3: "Parameters" (Lines 12719-12737) - *optional*
**Content**:
- Analysis Parameters table
- For each analysis type: interest peak min/max, reference peak min/max

#### Sheet 4: "Raw Data" (Lines 12739-12763) - *optional*
**Content**:
- Complete spectral data
- Column 1: Frequency (cm⁻¹)
- Subsequent columns: Transmittance for each depth
- Every frequency point and corresponding values

### Method Detection (Lines 12660-12668)
```javascript
// Auto-detect which methods were used
const firstOxResult = Object.values(window.AppState.analysisResults.oxidation)[0];
const oxMethod = firstOxResult.method || 'area';  // 'area' or 'height'
const oxHeader = oxMethod === 'height' ? 'KOI (Height-Based)' : 'OI (ASTM F2102)';
```

### Export Dialog: `createExportModal()` (Lines 12437-12523)
**Location**: Lines 12437-12523

**Features**:
- Modal dialog with export format selection
- Options for what to include:
  - Raw Data checkbox
  - Statistics checkbox
  - Parameters checkbox
- Format buttons: Excel, CSV, JSON
- Result: Triggers `exportData(format)` → `exportToExcel(...)`

---

## 7. PAGE STRUCTURE FOR MULTI-FILE OPTION

### Current Layout:
```
┌─────────────────────────────────────────┐
│          FTIR Analyzer Header            │
├─────────────────────────────────────────┤
│ ┌─── Data Import Section ────────────┐  │
│ │ [Single File] [Multiple Files]     │  │
│ │ • Single File Mode (Multi-depth)   │  │
│ │   └─ File input, Column mapping    │  │
│ │ • Multiple Files Mode (.dpt)       │  │
│ │   └─ File multi-select             │  │
│ │   └─ Depth config                  │  │
│ │   └─ File preview list             │  │
│ │ [Load Data] [Clear All]            │  │
│ └────────────────────────────────────┘  │
├─────────────────────────────────────────┤
│ ┌─── Spectrum Overview ──────────────┐  │
│ │ [Toggle] [Reset]                   │  │
│ │ ┌─ Spectra Plot ────┐ ┌─ Info ─┐ │  │
│ │ │                    │ │ Loaded │ │  │
│ │ │                    │ │ Spectra│ │  │
│ │ │                    │ │ Depth  │ │  │
│ │ │                    │ │ Info   │ │  │
│ │ │                    │ │ Mask   │ │  │
│ │ │                    │ │ Depths │ │  │
│ │ └────────────────────┘ └────────┘ │  │
│ └────────────────────────────────────┘  │
├─────────────────────────────────────────┤
│ ┌─── Analysis Section ──────────────┐  │
│ │ [Oxidation] [Crystallinity] [TVI] │  │
│ │ ┌─ Full Auto Button ────────────┐ │  │
│ │ │ Start Complete Analysis       │ │  │
│ │ └───────────────────────────────┘ │  │
│ │                                    │  │
│ │ [Per-Analysis Tabs and Plots]     │  │
│ └────────────────────────────────────┘  │
└─────────────────────────────────────────┘
```

### Multi-File Tab Location:
- **Parent Container**: `#importModeContent` (Lines 2136-2254)
- **Tab Pane**: `#multi-files` (Lines 2194-2254)
- **Tab Trigger**: `#multi-files-tab` (Lines 2128-2133)

### Multi-File UI Elements Hierarchy:
```
#multi-files (tab pane)
├── .row
│   ├── .col-md-8
│   │   ├── label (Select Multiple Depth Files)
│   │   ├── .custom-file-input-wrapper
│   │   │   ├── #multiFileInput (file[multiple], accept=".dpt")
│   │   │   └── .custom-file-label
│   │   └── .form-text (instructions)
│   └── .col-md-4
│       ├── label (Depth Configuration)
│       ├── .form-check
│       │   ├── #autoDepthDetection (checkbox)
│       │   └── .form-check-label
│       └── .form-text
│
└── #multiFilePreview (initially hidden)
    └── .row
        ├── h6 (Selected Files Preview)
        └── .alert.alert-info
            └── #multiFileList
                └── (dynamically populated file list)
```

---

## SUMMARY TABLE

| Component | Location | Key ID | Function |
|-----------|----------|--------|----------|
| **Data Import Header** | Line 2117 | - | Display "Data Import" section title |
| **Single File Tab** | Lines 2121-2142 | `single-file-tab` | Switch to single file import mode |
| **Multi File Tab** | Lines 2128-2133 | `multi-files-tab` | Switch to multiple file import mode |
| **Single File Input** | Lines 2144-2192 | `fileInput` | Select single file with multi-depth data |
| **Multi File Input** | Lines 2194-2231 | `multiFileInput` | Select multiple .dpt depth files |
| **Load Button** | Line 2235 | `loadDataBtn` | Trigger `loadSpectrumData()` |
| **Full Auto Button** | Line 2438 | `fullAutoAnalysisBtn` | Trigger `performFullAutoAnalysis()` |
| **Full Auto Handler** | Lines 13396-13560 | - | Execute sequential analysis pipeline |
| **Oxidation Analysis** | Lines 2523-2567 | `oxidationResults` | Display OI/KOI results |
| **Crystallinity Analysis** | Lines 2734-2778 | `crystallinityResults` | Display CI results |
| **TVI Analysis** | Lines 2909-2953 | `tviResults` | Display TVI and irradiation dose |
| **getAllAnalysisResults()** | Lines 11721-11751 | - | Consolidate all results for export |
| **exportToExcel()** | Lines 12653-12787 | - | Generate Excel file with multiple sheets |
| **AppState** | Lines 4517-4540 | `window.AppState` | Global application state object |

---

## TECHNICAL NOTES

1. **Depth Format**: All depth values stored as `"XXXμm"` strings (e.g., `"100μm"`, `"200μm"`)

2. **Spectral Data Backup**: Original transmittance data stored in `originalTransmittance` to prevent corruption during baseline correction

3. **Masking System**: Uses `Set` for efficient depth filtering in `window.AppState.maskedDepths`

4. **Method Switching**: Full auto analysis syncs method selections from master controls to individual tab dropdowns before running each analysis

5. **Irradiation Dose**: Calculated from mean TVI using formula: `4196.8 * TVI - 12.331` (kGy)

6. **File Format Auto-Detection**: Uses file extension to determine parsing strategy (Excel vs CSV vs DPT, etc.)

7. **Multi-file Sorting**: DPT files automatically sorted by depth number in filename for correct depth ordering

8. **Export Options**: Excel export generates 4 sheets with metadata, statistics, parameters, and raw data
