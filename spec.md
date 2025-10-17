# ValidityClassifier Specification

## Export Functionality

### Overview
The ValidityClassifier application includes an export feature that allows users to download the current dataframe with all labeling information as a CSV file.

### User Interface
- An "Export labeled data" button is displayed in the UI next to the "Reset demo data" button
- Clicking this button triggers a download of the current dataset as a CSV file
- The exported filename includes a timestamp for easy identification (format: `labeled_data_YYYYMMDD_HHMMSS.csv`)

### Exported Data Format
The exported CSV file contains all original data columns plus the following additional fields:

1. **valid_sale**
   - Contains the current label for each row
   - Possible values: "valid", "invalid", or "unknown"

2. **valid_sale_source**
   - Indicates the source of the label
   - Possible values:
     - "user": Label was manually assigned by a user
     - "machine": Label was predicted by the CatBoost model

3. **valid_sale_pct**
   - Represents the confidence/probability value for the label
   - Values:
     - 0.0: For user-labeled "invalid" rows
     - 1.0: For user-labeled "valid" rows
     - Floating point number between 0 and 1: The prediction probability for machine-labeled rows

### Implementation Details
- The export functionality is implemented as a POST endpoint (/export)
- The server creates a temporary CSV file containing the enhanced dataframe
- The file is sent to the client with appropriate headers for download
- No data is permanently stored on the server during export

## Drag and Drop File Upload

### Overview
The ValidityClassifier application allows users to upload their own CSV data files via drag and drop functionality, replacing the demo data with their own dataset for labeling and prediction.

### User Interface
- A drag and drop area is displayed prominently in the UI when no data is loaded
- Users can either drag a CSV file onto this area or click a "Browse files" button to select a file
- Visual feedback is provided during the drag operation (highlighting the drop area)
- After successful upload, the UI refreshes to display the uploaded data
- The export button is disabled when no data is loaded

### File Requirements and Validation
1. **File Type**
   - Only CSV files are accepted
   - Files with other extensions will trigger an error message

2. **Required Fields**
   - The CSV must contain a `key_sale` column (required)
   - This column is used as a unique identifier for each row

3. **Size Limitations**
   - Files larger than 5,000 rows are rejected
   - An error message is displayed for oversized files

4. **Handling Existing Labels**
   - If the uploaded CSV contains a `valid_sale` column:
     - Values of "valid" or "invalid" are preserved as user labels
     - Any other values are reset to "unknown"
   - If no `valid_sale` column exists, one is created with all values set to "unknown"

### Error Handling
- Clear error messages are displayed for various validation failures:
  - Missing required columns
  - File too large
  - Invalid file format
  - General processing errors
- Error messages are shown in a dismissable alert box

### State Management
- When no data is loaded:
  - The export button is disabled
  - The drag and drop area is prominently displayed
  - The table area shows a message prompting the user to upload a file
- After successful upload:
  - The export button is enabled
  - The table displays the uploaded data
  - Any existing model is cleared, and prediction starts fresh with the new data

### Implementation Details
- The upload functionality is implemented as a POST endpoint (/upload)
- File validation occurs server-side
- The server processes the CSV file, sanitizes the data, and stores it in the session
- Label sources are initialized appropriately based on any existing valid_sale values
- The model registry is cleared for the session to ensure predictions start fresh

## Data Management Buttons

### Overview
The ValidityClassifier application provides two buttons for managing data state: "Clear Labels" and "Upload New File". These buttons replace the previous single "Reset Demo Data" button to provide more granular control over the application's state.

### User Interface
- Two buttons are displayed side by side in the UI:
  - "Clear Labels" button (left): Resets all labels to their initial state
  - "Upload New File" button (right): Clears the current dataframe and displays the upload area
- The "Export labeled data" button is positioned to the right of these buttons

### Button Behaviors

1. **Clear Labels**
   - Resets to demo data with all labels set to "unknown"
   - Preserves the same dataset structure but removes all user and machine labels
   - Clears any trained model from the session
   - Resets the labels version counter
   - Equivalent to the previous "Reset Demo Data" functionality

2. **Upload New File**
   - Completely clears the existing dataframe from the session
   - Removes all related session data (label sources, probabilities, etc.)
   - Clears any trained model from the session
   - Displays the drag and drop upload area
   - Allows users to upload a new file without having to manually clear the session

### State Management
- After clicking "Clear Labels":
  - All rows show "unknown" in the valid_sale column
  - The prediction button is disabled until sufficient labels are provided
  - The export button remains enabled
  - The table continues to display the (now unlabeled) data

- After clicking "Upload New File":
  - The table area is replaced with the drag and drop upload area
  - The export button is disabled until new data is loaded
  - Any in-flight model training is canceled

### Implementation Details
- The "Clear Labels" functionality is implemented as a POST endpoint (/reset)
- The "Upload New File" functionality is implemented as a POST endpoint (/clear_data)
- Both endpoints handle proper cleanup of session data and model registry entries
- The "Upload New File" endpoint additionally cancels any in-flight model training futures
