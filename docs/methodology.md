# Methodology

This project extracts data from graph images using a rule-based computer vision pipeline. The same high-level flow is used for bar graphs, line graphs, and scatter plots, with graph-specific logic for detecting visual marks.

## High-Level Pipeline

```text
Input graph image
        |
        v
Crop axis text regions
        |
        v
Detect text using Keras OCR
        |
        v
Identify axis labels and titles
        |
        v
Detect x-axis and y-axis lines
        |
        v
Calculate Value Per Pixel
        |
        v
Detect graph-specific marks
        |
        v
Convert pixel positions to data values
        |
        v
Save extracted values and compare with ground truth
```

## Data Generation

Synthetic datasets were generated to validate the extraction pipeline.

For each graph type, random values are generated and plotted using Matplotlib. The generated values are saved into CSV files so that extracted values can later be compared with the original known values.

General approach:

- Choose a random number of elements for the graph.
- Generate random numerical values around a base value.
- Plot the graph using Matplotlib.
- Save the graph image.
- Save the generated values into CSV files.

The synthetic data files are available in:

```text
data/synthetic/
```

## Text Detection

Graphs contain both visual marks and textual information. The textual information is important because axis labels and tick values define how pixels map to actual numerical values.

The image is cropped into two important regions:

- Bottom strip for x-axis labels and x-axis title
- Left strip for y-axis labels and y-axis title

Keras OCR is applied to these cropped regions. For each detected text box, the pipeline stores:

- Detected text
- Bounding box
- x-center
- y-center
- box width
- box height

These values are used to group axis labels and detect axis titles.

## Axis Label and Title Detection

For the x-axis:

- OCR is applied to the bottom strip.
- Text boxes with similar y-coordinates are grouped as x-axis labels.
- Text below the label row is treated as the x-axis title.

For the y-axis:

- OCR is applied to the left strip.
- Text boxes with similar x-coordinates are grouped as y-axis labels.
- The vertical strip is also rotated to improve detection of the y-axis title.

## Axis Line Detection

Axis lines are detected using:

- Grayscale conversion
- Prewitt edge detection
- Probabilistic Hough line transform

The x-axis is selected from prominent horizontal lines near the bottom of the graph region.

The y-axis is selected from prominent vertical lines near the left side of the graph region.

These axes define the graph plotting area and provide reference coordinates for value extraction.

## Value Per Pixel

Value Per Pixel, or VPP, converts a pixel distance into a real numerical value.

For two consecutive axis labels:

```text
pixel_difference = absolute difference between label pixel positions
value_difference = absolute difference between label numeric values
VPP = value_difference / pixel_difference
```

The final VPP is calculated by averaging the VPP values across valid consecutive label pairs.

Bar and line graphs primarily need y-axis VPP.

Scatter plots need both:

- x-axis VPP
- y-axis VPP

## Bar Graph Extraction

Bar graph extraction focuses on detecting the top edge of each bar.

Steps:

1. Detect x-axis and y-axis.
2. Detect x-axis labels and y-axis numerical labels.
3. Calculate y-axis VPP.
4. Use K-means color clustering to distinguish bars from the background.
5. Use Prewitt edge detection to find horizontal edges.
6. Use Hough line detection to identify potential top edges of bars.
7. Confirm top edges by checking color clusters above and below each detected line.
8. Map each bar top to the nearest x-axis label.
9. Convert the vertical pixel distance into a numerical value.
10. Save the extracted values to CSV.

Formula:

```text
extracted_value = abs(bar_top_y - reference_y) * y_vpp + offset
```

## Line Graph Extraction

Line graph extraction focuses on detecting the plotted line and its data points.

Steps:

1. Detect axes and OCR labels.
2. Crop the graph plotting region.
3. Use K-means color clustering to isolate the line color.
4. Binarize the cropped image.
5. Use edge and line detection to identify line segments.
6. Merge nearby line segments.
7. Find intersections between consecutive line segments.
8. Handle missing line segments by preserving endpoints.
9. Map detected points to nearest x-axis labels.
10. Convert point y-coordinates into numerical values.
11. Save extracted values to CSV.

The line graph method includes edge-case handling for missing data points and identical slopes between consecutive segments.

## Scatter Plot Extraction

Scatter plot extraction focuses on detecting individual point centers.

Steps:

1. Detect axes and OCR labels.
2. Calculate both x-axis and y-axis VPP.
3. Crop the plotting region.
4. Use K-means color clustering to isolate scatter points.
5. Binarize the image.
6. Detect blobs using OpenCV blob detection.
7. Convert each blob center into graph coordinates.
8. Save extracted x-y values to CSV.

Formula:

```text
x_value = abs(point_x - reference_x) * x_vpp + x_offset
y_value = abs(point_y - reference_y) * y_vpp + y_offset
```

## Validation

The extracted values are compared against the generated ground-truth CSV files.

Metrics used:

- Root Mean Square Error
- Accuracy after rounding to nearest integer
- Correct predictions versus total values

Validation notebooks store extracted values and calculate performance scores.

## Main Assumptions

The project assumes:

- Graphs are 2D.
- Graphs use solid colors.
- There is one main plotted series.
- Scatter points do not overlap.
- Axis labels are visible and readable.
- Legends are absent or not interfering with the plotting area.
- The graph image is clean enough for OCR and edge detection.

## Important Edge Cases

The pipeline includes logic for:

- Incorrect OCR values
- Missing first y-axis labels
- Unequal pixel spacing near the axis origin
- Missing line segments
- Similar slopes in line graphs where a middle point may otherwise be skipped
- Noise causing extra detected bar-top lines

Detailed explanations of these cases are available in:

```text
docs/challenges_and_edge_cases.md
```
