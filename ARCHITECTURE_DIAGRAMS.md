# FTIR Analyzer Architecture & Flow Diagrams

## 1. DATA IMPORT FLOW DIAGRAM

```
┌────────────────────────────────────────────────────────────────┐
│              Data Import Section (Lines 2117-2260)             │
└────────────────────────────────────────────────────────────────┘
                              │
                ┌─────────────┼─────────────┐
                │                           │
        ┌───────▼──────┐        ┌──────────▼──────┐
        │  Single File │        │ Multiple Files  │
        │  (Multi-Depth)        │  (.dpt files)   │
        └───────┬──────┘        └──────┬──────────┘
                │                      │
        ┌───────▼──────────┐   ┌──────▼──────────┐
        │ File Selection   │   │ Multi-Select    │
        │ #fileInput       │   │ #multiFileInput │
        │ (any format)     │   │ (dpt only)      │
        └───────┬──────────┘   └──────┬──────────┘
                │                      │
        ┌───────▼──────────┐   ┌──────▼──────────┐
        │ Column Mapping   │   │ File Preview    │
        │ (if needed)      │   │ Grouped by      │
        │ #columnMapping   │   │ Sample Name     │
        └────────┬─────────┘   └──────┬──────────┘
                │                      │
                └──────────┬───────────┘
                           │
                    ┌──────▼──────────┐
                    │  Load Data Btn  │
                    │ #loadDataBtn    │
                    │ (Line 2235)     │
                    └──────┬──────────┘
                           │
        ┌──────────────────▼──────────────────┐
        │    loadSpectrumData()               │
        │    (Lines 4739-4806)                │
        └──────────┬───────────────────────────┘
                   │
        ┌──────────▼──────────────────────────┐
        │  Determine File Type                │
        │  (extension-based routing)          │
        └──────────┬───────────────────────────┘
                   │
        ┌──────────┴──────────┬─────────────┐
        │                     │             │
    ┌───▼────┐ ┌────▼───┐ ┌──▼──┐ ┌──▼──┐
    │ XLSX   │ │ CSV    │ │ DPT │ │ SPC │
    │ Excel  │ │ Text   │ │ Bin │ │ Gal │
    └───┬────┘ └───┬────┘ └──┬──┘ └──┬──┘
        │          │         │       │
        └──────────┬─────────┴───────┘
                   │
        ┌──────────▼──────────────────────────┐
        │  Process Data                       │
        │  processLoadedData() or             │
        │  processMultiFileData()             │
        └──────────┬───────────────────────────┘
                   │
        ┌──────────▼──────────────────────────┐
        │  Store in AppState                  │
        │  window.AppState.spectraData = {    │
        │    '100μm': {                       │
        │      frequency: [...],              │
        │      transmittance: [...],          │
        │      color: '#...',                 │
        │      ...                            │
        │    }                                │
        │  }                                  │
        └──────────┬───────────────────────────┘
                   │
        ┌──────────▼──────────────────────────┐
        │  Show Overview Section              │
        │  displaySpectrumOverview()          │
        │  - Plot spectra                     │
        │  - List spectra                     │
        │  - Show depth info                  │
        │  - Enable masking                   │
        └──────────┬───────────────────────────┘
                   │
        ┌──────────▼──────────────────────────┐
        │  Enable Processing Controls         │
        │  - Offset correction                │
        │  - Baseline correction              │
        └──────────────────────────────────────┘
```

---

## 2. FULL AUTO ANALYSIS EXECUTION FLOW

```
┌────────────────────────────────────────────────┐
│      User Clicks #fullAutoAnalysisBtn          │
│            (Line 2438, 13725)                  │
└────────────────┬───────────────────────────────┘
                 │
    ┌────────────▼────────────┐
    │ performFullAutoAnalysis()
    │ (Lines 13396-13560)     │
    └────────────┬────────────┘
                 │
    ┌────────────▼────────────┐
    │  Validate Data Loaded   │
    │  check                  │
    │  AppState.spectraData   │
    └────────────┬────────────┘
                 │
         ┌───────▼───────┐
         │ No Data?      │
         └───┬───────┬───┘
             │       │
          YES│       │NO
             │       │
        [Show    [Continue]
         Error]    │
                   │
    ┌──────────────▼──────────────┐
    │ Button State: Disabled      │
    │ Change text to "Analyzing..."│
    │ Show spinner                │
    └──────────────┬──────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ Sync Method Selections                  │
    │ Full Auto → Individual Tabs             │
    │ fullAutoOxidationMethod → oxidationMeth │
    │ fullAutoCrystallinityMethod → cryst... │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ STEP 1: Offset Correction               │
    │         applyOffsetCorrection()         │
    │         (Line 13440)                    │
    │         ⏱️  ~800ms delay                 │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ STEP 2: Oxidation Analysis              │
    │         performAutoAnalysis('oxidation')│
    │         (Line 13450)                    │
    │         Method: OI (area) or KOI (hgt) │
    │         ⏱️  ~800ms delay                 │
    │         Results → AppState.analysisRe..│
    │                   .oxidation            │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ STEP 3: Crystallinity Analysis          │
    │         performAutoAnalysis('crystall..')│
    │         (Line 13459)                    │
    │         Method: CI Area or CI Height    │
    │         ⏱️  ~800ms delay                 │
    │         Results → AppState.analysisRe..│
    │                   .crystallinity        │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ STEP 4: TVI Analysis                    │
    │         performAutoAnalysis('tvi')      │
    │         (Line 13468)                    │
    │         ⏱️  ~800ms delay                 │
    │         Results → AppState.analysisRe..│
    │                   .tvi                  │
    │         Calc Irradiation Dose           │
    │         (4196.8 × TVI - 12.331)         │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ STEP 5: Generate Depth Profile          │
    │         updateDepthProfile()            │
    │         (Line 13483)                    │
    │         → plotDepthProfile()            │
    │         ⏱️  ~800ms delay                 │
    │         Visualize trends across depths  │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ Hide Loading Indicator                  │
    │ Call showAnalysisCompleteSummary()      │
    │ Display statistics in notification      │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ Button State: Enabled                   │
    │ Restore original button text            │
    │ Remove spinner                          │
    └──────────────┬──────────────────────────┘
                   │
    ┌──────────────▼──────────────────────────┐
    │ User can now:                           │
    │ - Export results (Excel/CSV/JSON/PDF)   │
    │ - View individual analysis tabs         │
    │ - Export complete report                │
    │ - Compare with manufacturer data (TVI)  │
    └──────────────────────────────────────────┘
```

---

## 3. DATA STORAGE STRUCTURE

```
window.AppState
│
├── rawData
│   └── Original loaded data from file
│
├── processedData
│   └── Data after offset/baseline correction
│
├── spectraData = {
│   '100μm': {
│       frequency: [4000, 3999.5, ...],
│       transmittance: [0.95, 0.94, ...],
│       originalTransmittance: [0.95, 0.94, ...],  ◄─ Backup
│       depth: 100,
│       color: '#FF6B6B',
│       sampleName: 'SAMPLE_A',
│       originalName: 'SAMPLE_A_01.dpt'
│   },
│   '200μm': { ... },
│   '300μm': { ... }
│   ...
├── maskedDepths = Set(['100μm', '250μm'])  ◄─ Excluded from analysis
│
├── analysisResults = {
│   oxidation: {
│       '100μm': {
│           index: 1.234,           ◄─ OI value
│           ratio: 0.00567,
│           method: 'area',
│           referenceArea: 0.456,
│           interestArea: 0.00257
│       },
│       '200μm': { ... }
│   },
│   crystallinity: {
│       '100μm': {
│           index: 5.678,           ◄─ CI value
│           ratio: 0.0234,
│           method: 'area',
│           ...
│       },
│       '200μm': { ... }
│   },
│   tvi: {
│       '100μm': {
│           index: 0.123,           ◄─ TVI value
│           ratio: 0.000567,
│           method: 'area',
│           ...
│       },
│       '200μm': { ... }
│   }
│   vitaminE: { ... },
│   dualIndex: { ... }
├── processingStatus = {
│   offsetCorrected: true,
│   baselineCorrected: true
│
├── currentFileName: 'SAMPLE_A_01.dpt'
└── loadedFiles: ['SAMPLE_A_01.dpt', 'SAMPLE_A_02.dpt', ...]
```

---

## 4. ANALYSIS FUNCTION CALL GRAPH

```
performFullAutoAnalysis()
  │
  ├─► applyOffsetCorrection()
  │
  ├─► performAutoAnalysis('oxidation')
  │   ├─► calculateOxidationIndices()
  │   ├─► plot3DOxidationReference()
  │   ├─► plot3DOxidationInterest()
  │   └─► displayOxidationResults()
  │       └─► window.AppState.analysisResults.oxidation[depth]
  │
  ├─► performAutoAnalysis('crystallinity')
  │   ├─► calculateCrystallinityIndices()
  │   ├─► plot3DCrystallinityReference()
  │   ├─► plot3DCrystallinityInterest()
  │   └─► displayCrystallinityResults()
  │       └─► window.AppState.analysisResults.crystallinity[depth]
  │
  ├─► performAutoAnalysis('tvi')
  │   ├─► calculateTVIIndices()
  │   ├─► plot3DTVIReference()
  │   ├─► plot3DTVIInterest()
  │   ├─► displayTVIResults()
  │   │   └─► window.AppState.analysisResults.tvi[depth]
  │   └─► updateIrradiationDoseDisplay(meanTVI)
  │       └─► Dose (kGy) = 4196.8 × TVI - 12.331
  │
  ├─► updateDepthProfile()
  │   └─► plotDepthProfile()
  │       └─► getAllAnalysisResults()  ◄─ Consolidates all results
  │           └─► Depth vs [OI, CI, TVI] chart
  │
  └─► showAnalysisCompleteSummary()
      └─► Display stats: count, average, dose, etc.
```

---

## 5. EXPORT DATA FLOW

```
┌────────────────────────────────────────┐
│  User clicks Export Button              │
│  (various export buttons in UI)         │
└────────────┬───────────────────────────┘
             │
    ┌────────▼────────┐
    │ exportReports() │
    │ (Line 12423)    │
    └────────┬────────┘
             │
    ┌────────▼────────────────┐
    │ createExportModal()     │
    │ Shows dialog with:      │
    │ - Format selection      │
    │ - Options checkboxes    │
    │ (Lines 12437-12523)     │
    └────────┬────────────────┘
             │
    ┌────────▼────────────────┐
    │ exportData(format)      │
    │ (Line 12524)            │
    └────┬─────────┬──────┬──┘
         │         │      │
    ┌────▼──┐ ┌───▼──┐ ┌─▼───┐
    │Excel  │ │ CSV  │ │JSON │
    │Export │ │Export│ │Exprt│
    └────┬──┘ └───┬──┘ └─┬───┘
         │        │      │
    ┌────▼────────▼──────▼──────────────┐
    │ getAllAnalysisResults()           │
    │ → { depths, oxidation,           │
    │     crystallinity, tvi }         │
    └────────┬───────────────────────────┘
             │
         ┌───▼───┐
         │EXCEL? │
         └───┬───┘
             │
    ┌────────▼─────────────────────────┐
    │ exportToExcel()                  │
    │ (Lines 12653-12787)              │
    │ ─────────────────────────────────│
    │ Creates 4 Sheets:               │
    │ 1. Analysis Results              │
    │    └─ Depth, OI, CI, TVI        │
    │ 2. Statistics (optional)         │
    │    └─ Mean, StdDev, Min, Max    │
    │    └─ Max values by depth        │
    │ 3. Parameters (optional)         │
    │    └─ Peak intervals             │
    │ 4. Raw Data (optional)           │
    │    └─ Frequency + all spectra   │
    └────────┬─────────────────────────┘
             │
    ┌────────▼─────────────────────────┐
    │ Create XLSX Workbook             │
    │ using XLSX.utils                 │
    └────────┬─────────────────────────┘
             │
    ┌────────▼─────────────────────────┐
    │ Generate & Download File         │
    │ filename: [name]_[timestamp].    │
    │           xlsx                   │
    └──────────────────────────────────┘
```

---

## 6. MULTI-FILE IMPORT PROCESSING

```
┌──────────────────────────────────────────────┐
│  User selects multiple .dpt files            │
│  #multiFileInput (multiple)                  │
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ handleMultiFileSelection()                   │
│ (Lines 4624-4667)                           │
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ Parse Each Filename                          │
│ Regex: /(\d+)\.dpt$/i                       │
│ Extract: Depth number (01=100μm, 02=200μm) │
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ Sort by Depth                                │
│ files.sort((a,b) => a.depth - b.depth)      │
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ Extract Sample Name from Filename            │
│ Regex: /^(.+?)_?(\d+)\.dpt$/i               │
│ Group sample names for display               │
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ displayMultiFilePreview()                    │
│ (Line 4669)                                  │
│ Show grouped file list with depth badges    │
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ User clicks Load Data                        │
│ loadSpectrumData()                          │
│ (Lines 4739-4806)                           │
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ loadMultipleDPTFiles()                       │
│ (Lines 4809-4900)                           │
│ Iterate through parsedFiles array           │
└──────────┬───────────────────────────────────┘
           │
     ┌─────▼──────────────────────────┐
     │ For each file:                 │
     │ 1. Load DPT data               │
     │ 2. Extract frequency & trans   │
     │ 3. Check frequency consistency │
     │    └─ Interpolate if needed    │
     │ 4. Store in spectraData        │
     └─────┬──────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ processMultiFileData()                       │
│ (Lines 5143-5177)                           │
│ Store consolidated spectrum data             │
│ Create rawData table (frequency × all depths)
└──────────┬───────────────────────────────────┘
           │
┌──────────▼───────────────────────────────────┐
│ Result: AppState.spectraData populated       │
│ {                                            │
│   '100μm': { frequency, transmittance, ...}, │
│   '200μm': { frequency, transmittance, ...}, │
│   '300μm': { frequency, transmittance, ...}  │
│   ...                                        │
│ }                                            │
└──────────────────────────────────────────────┘
```

---

## 7. GET ALL ANALYSIS RESULTS FUNCTION

```
getAllAnalysisResults()
(Lines 11721-11751)
│
├─ Input: None (reads from AppState)
│
├─ Initialize results object:
│  {depths: [], oxidation: [], crystallinity: [], tvi: []}
│
├─ Get masked depths:
│  maskedDepths = AppState.maskedDepths (Set)
│
├─ Iterate through all analysis types:
│  for each type in ['oxidation', 'crystallinity', 'tvi']
│    for each depth label in AppState.analysisResults[type]
│      if NOT in maskedDepths:
│        extract depth number
│        add to allDepths Set
│
├─ Sort depths numerically (ascending)
│  depths = [100, 200, 300, ...]
│
├─ For each depth:
│  results.oxidation[i] = analysisResults.oxidation[depthLabel]?.index
│  results.crystallinity[i] = analysisResults.crystallinity[depthLabel]?.index
│  results.tvi[i] = analysisResults.tvi[depthLabel]?.index
│  (use null if not available)
│
└─ Return:
  {
    depths: [100, 200, 300, ...],
    oxidation: [1.234, 2.345, ...],
    crystallinity: [5.678, 6.789, ...],
    tvi: [0.123, 0.234, ...]
  }
```

---

## 8. UI STATE TRANSITIONS

```
┌───────────────────────────────────────┐
│     APPLICATION INITIAL STATE          │
│  - No data loaded                      │
│  - All analysis buttons disabled       │
│  - Overview section hidden             │
│  - Processing controls hidden          │
│  - Analysis tabs hidden                │
└───────────────┬───────────────────────┘
                │
        [User selects & loads file]
                │
┌───────────────▼───────────────────────┐
│     DATA LOADED STATE                  │
│  ✓ Spectrum Overview visible           │
│  ✓ Processing Controls visible         │
│  ✓ Can apply offset/baseline           │
│  ✓ Analysis section visible            │
│  ✓ Full Auto button enabled            │
└───────────────┬───────────────────────┘
                │
        [User clicks Full Auto button]
                │
┌───────────────▼───────────────────────┐
│     PROCESSING STATE                   │
│  • Button disabled, shows spinner      │
│  • Loading indicator visible           │
│  • Sequential steps executing          │
│  • Steps: Offset → Ox → Cryst → TVI   │
│           → Depth Profile → Done       │
└───────────────┬───────────────────────┘
                │
┌───────────────▼───────────────────────┐
│     ANALYSIS COMPLETE STATE            │
│  ✓ Button re-enabled                   │
│  ✓ Results visible in each tab         │
│  ✓ Depth profile chart populated       │
│  ✓ Export buttons enabled              │
│  ✓ Summary notification shown          │
│  ✓ Irradiation dose calculated (TVI)   │
└───────────────────────────────────────┘
```

---

## 9. PEAK INTERVAL REFERENCE TABLE

```
╔═══════════════════════════════════════════════════════════════════╗
║              ANALYSIS PEAK INTERVALS (cm⁻¹)                       ║
╠═════════════════════════┬═════════════┬═════════════════════════════╣
║  Analysis Type          │  Interest   │  Reference                  ║
╠═════════════════════════╪═════════════╪═════════════════════════════╣
║  Oxidation (OI/KOI)     │ 1650-1760   │ 2800-3000 (CH₂ stretch)     ║
║  Carbonyl Region        │ Peak: 1720  │ Ref: 2930                   ║
╠═════════════════════════╪═════════════╪═════════════════════════════╣
║  Crystallinity (CI)     │  900-970    │ 2800-3000 (CH₂ stretch)     ║
║  Crystalline Region     │ Peak: 942   │ Ref: 2930                   ║
╠═════════════════════════╪═════════════╪═════════════════════════════╣
║  TVI (Thermal Index)    │ 1200-1300   │ 2800-3000 (CH₂ stretch)     ║
║  C-O Stretching         │ Peak: 1260  │ Ref: 2930                   ║
╚═════════════════════════╧═════════════╧═════════════════════════════╝
```

---

## 10. FILE FORMAT SUPPORT MATRIX

```
┌──────────┬────────────────┬────────────────┬──────────────┐
│ Format   │ Single File    │ Multi-File     │ Description  │
├──────────┼────────────────┼────────────────┼──────────────┤
│ .xlsx    │ ✓ Supported    │ ✗ Not used     │ Excel sheets │
│ .csv     │ ✓ Supported    │ ✗ Not used     │ CSV text     │
│ .txt     │ ✓ Supported    │ ✗ Not used     │ Text file    │
│ .dpt     │ ✓ Supported    │ ✓ Preferred    │ Origin binary│
│ .spc     │ ✓ Supported    │ ✗ Not used     │ Galactic SPC │
│ .0       │ ✓ Supported    │ ✗ Not used     │ JCAMP binary │
└──────────┴────────────────┴────────────────┴──────────────┘

Multi-file Mode:
- Only .dpt files accepted
- Filename convention: SAMPLE_NAME_##.dpt (## = 01, 02, 03...)
- Auto-parsed depth: ## × 100 = depth in μm
```
