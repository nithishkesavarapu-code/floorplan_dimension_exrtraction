# **Floorplan Dimension Extractor Report: Pipeline Summary**

This document summarizes the technical approach, library choices, and problem-solving steps undertaken to create a Python pipeline for extracting and standardizing dimensions and appliance codes from floorplan PDFs.

## **1\. Technical Approach and Tooling**

The pipeline was built to satisfy all core assignment requirements, focusing on accuracy in extraction, unit conversion, and bounding box determination.

| Component | Library/Technique | Rationale |
| :---- | :---- | :---- |
| **PDF Text & Position Data** | pdfplumber | Chosen specifically over simpler text extractors (like PyMuPDF's basic text function) because it provides **word-level bounding boxes (BBox)**. This was crucial for calculating the precise location of each extracted dimension/code in the final JSON output. |
| **Pattern Matching** | regex (Python library) | Utilized the advanced features of the regex library (instead of the standard re) to create highly readable and robust patterns, particularly by leveraging **named capture groups**. |
| **Unit Conversion** | Custom Python Logic | Implemented a dedicated convert\_to\_inches function capable of parsing mixed-unit strings (feet, inches, and fractions) and accurately converting them to a single float value in inches. |

## **2\. Extraction Strategy (Regex)**

A comprehensive regular expression (DIMENSION\_REGEX) was developed to capture all known dimension formats found on floorplans.

### **Dimension Patterns Covered:**

1. **Feet, Inches, and Fractions:** Handles formats like 2′61/2′′ by isolating feet, whole inches, numerator, and denominator via named groups.  
2. **Inches Only:** Captures formats like 341/2′′ where a unit symbol is present, but no feet component is.  
3. **Room Dimensions (e.g.,** 14′×8′**):** A specific pattern (room\_dim) was integrated to handle dimensions separated by an 'x', which are common for indicating room size.  
4. **Cabinet/Appliance Codes:** A separate pattern (CODE\_REGEX) was used to target standard codes (e.g., DB24, SB42FH) consisting of two or more letters followed by two or more digits.

## **3\. Challenges and Solutions**

| Challenge | Detail | Solution Implemented |
| :---- | :---- | :---- |
| **Inaccurate BBox Spans** | Regex matches a string, which often includes spaces or spans multiple discrete pdfplumber word objects (e.g., "2′" and "6′′" are separate words). Simply using the character span index was insufficient. | A custom mapping structure (index\_to\_word\_map) was implemented to reliably link the regex character indices back to the corresponding word objects. The BBox is then calculated by taking the combined perimeter from the first matched word's top-left to the last matched word's bottom-right corner. |
| **JSON Schema Ambiguity for Room Dimensions** | Room dimensions (e.g., 14′×8′) provide two values, but the required JSON schema entry for dimensions only contains a single inches field. | The convert\_to\_inches function was pragmatically designed to parse and convert **only the first dimension** (e.g., the 14′) for all room\_dim matches. This aligns the output with the single-value JSON field while acknowledging the input's complexity. |
| **Variable Unit Symbols** | Floorplans use various characters for feet/inches, including standard quotes (', ") and unicode prime symbols (2˘032, 2˘033). | The regex pattern was updated to explicitly account for and capture both the ASCII and Unicode representations of the unit symbols, ensuring consistency across different PDF outputs. |

The resulting pipeline successfully performs the necessary extraction and conversion, delivering structured, positional data in the required JSON format.
