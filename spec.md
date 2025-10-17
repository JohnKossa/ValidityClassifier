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