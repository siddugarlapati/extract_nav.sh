# extract_nav.sh
#!/bin/bash


# Exit the script if any command fails
set -e

# Step 1: Define URLs and file names
NAV_URL="https://www.amfiindia.com/spages/NAVAll.txt"  # URL to download the data
RAW_FILE="NAVAll.txt"                                # File to store downloaded data
OUTPUT_FILE="nav_data.tsv"                           # File to store processed data

# Step 2: Download the NAV data
echo "Downloading NAV data from AMFI India..."
curl -s "$NAV_URL" -o "$RAW_FILE"

# Step 3: Check if the download was successful and the file is not empty
if [ ! -s "$RAW_FILE" ]; then
    echo "Error: The downloaded file is empty or missing." >&2
    exit 1
fi

# Step 4: Process the data
# Use awk to extract "Scheme Name" and "Asset Value" from the downloaded file
echo "Processing the data and creating a TSV file..."
awk -F';' '                                       # Use semicolon as the field separator
    BEGIN {                                       # Run this at the start
        OFS="\t"                                  # Output field separator is a tab
        print "Scheme_Name\tAsset_Value"         # Print header for the TSV file
    }
    NF >= 6 && $4 != "" && $6 != "" {            # Process rows with at least 6 fields
        gsub(/^[ \t]+|[ \t]+$/, "", $4)          # Remove extra spaces from field 4
        gsub(/^[ \t]+|[ \t]+$/, "", $6)          # Remove extra spaces from field 6
        print $4, $6                             # Print the scheme name and value
    }' "$RAW_FILE" > "$OUTPUT_FILE"

# Step 5: Check if the output file was created successfully
if [ ! -s "$OUTPUT_FILE" ]; then
    echo "Error: No valid data found to create the output file." >&2
    exit 1
fi

# Step 6: Clean up temporary files
echo "Cleaning up temporary files..."
rm -f "$RAW_FILE"

# Step 7: Let the user know the script finished successfully
echo "TSV file created successfully: $OUTPUT_FILE"
