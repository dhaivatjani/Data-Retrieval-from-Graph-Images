# Challenges and Edge Cases

This document explains the important implementation challenges solved during the project. These details are useful because the extraction pipeline is not only about detecting graph marks, but also about handling messy conditions caused by OCR errors, missing labels, irregular spacing, and incomplete line detection.

## 1. Vertical Text Detection

One of the early challenges was detecting y-axis titles and labels correctly. Many OCR tools struggle when text is rotated vertically, which is common for y-axis titles.

### Problem

The y-axis text can be vertical or rotated. Direct OCR on the original left-side crop may fail or return incomplete text.

### Solution

The left vertical strip of the image is processed in two ways:

- Direct OCR is used to detect y-axis numerical labels.
- The strip is rotated by 90 degrees so that vertical title text becomes horizontal.

This improved the OCR result for y-axis titles and reduced missed or incorrect detections.

## 2. Incorrect OCR Values

OCR can misread axis values. For example, a tick label may be detected as the wrong number, or non-label text may be interpreted as a label.

### Problem

Incorrect OCR labels affect the Value Per Pixel calculation. Since VPP is used to convert pixel distances into real values, one wrong label can shift the extracted values for the whole graph.

### Solution

The detected labels are cleaned using consistency checks:

- Labels are sorted by their axis position.
- Consecutive value differences are checked.
- Pixel differences between labels are checked.
- Outliers that do not match the common difference pattern are removed or ignored.

This makes the VPP calculation more stable.

## 3. Missing or Irregular First Y-Axis Label

This was one of the most important edge cases. The first y-axis label after the x-axis is not always detected correctly, and the pixel distance between the x-axis and the first detected label may not match the regular spacing between other labels.

To handle this, the project defines eight cases using:

- `corVal`: the first detected y-axis label value near the x-axis
- `corFir`: the pixel coordinate of that detected label
- `most_common_diff`: the most common numerical difference between y-axis labels
- `most_common_pdiff`: the most common pixel difference between y-axis labels
- `betweenFirst`: pixel distance between the x-axis and first detected y-axis label
- `toAdd`: value offset used during final value conversion
- `fromCal`: reference pixel coordinate used during final value conversion

The final extraction formula depends on these corrected reference values:

```text
extracted_value = abs(point_y - fromCal) * VPP + toAdd
```

## The Eight Cases

### Case 1: First Label Detected and Pixel Spacing Is Normal

Situation:

- The first detected y-axis label matches the expected first label.
- The pixel gap from the x-axis to this label is similar to the common label spacing.

Action:

```text
toAdd = 0
fromCal = x-axis coordinate
```

Reason:

The x-axis can safely be treated as the zero/base reference.

### Case 2: First Label Missed but Pixel Spacing Is Normal

Situation:

- The first actual label is missed.
- The spacing still indicates that the axis starts cleanly from the x-axis.
- After calculation, the missing label would match the common label difference.

Action:

```text
toAdd = 0
fromCal = x-axis coordinate
```

Reason:

Even though OCR missed the label, the pixel spacing confirms that the x-axis is still the correct reference.

### Case 3: First Label Detected, Pixel Spacing Normal, but Label Value Is Not the Common Difference

Situation:

- The first detected label exists.
- The pixel spacing from the x-axis is normal.
- The label value does not equal the common difference.

Action:

```text
toAdd = detected_label_value - most_common_diff
fromCal = x-axis coordinate
```

Reason:

The graph likely does not start at zero, so an offset is needed.

### Case 4: First Label Missed, Pixel Spacing Normal, and Label Value Does Not Match Expected Difference

Situation:

- The first label is missed.
- Pixel spacing is still normal.
- The inferred value sequence shows that the graph does not start from zero.

Action:

```text
toAdd = corrected offset based on detected label and common difference
fromCal = x-axis coordinate
```

Reason:

The x-axis remains the pixel reference, but the numerical base must be shifted.

### Case 5: First Label Detected but Pixel Spacing Is Irregular

Situation:

- The first label is detected.
- The pixel distance between the x-axis and first detected label is not equal to the common spacing.
- The first detected label matches the common difference.

Action:

```text
toAdd = detected_label_value
fromCal = first detected label coordinate
```

Reason:

The first label itself becomes a safer reference point than the x-axis.

### Case 6: First Label Missed and Pixel Spacing Is Irregular

Situation:

- The first label is missed.
- Pixel spacing from the x-axis does not match the common label spacing.
- The missing label would have matched the common difference.

Action:

```text
toAdd = inferred label value
fromCal = adjusted coordinate near the x-axis
```

Reason:

Both the value offset and pixel reference must be corrected because the first bin is irregular.

### Case 7: First Label Detected but It Appears Too Close to the X-Axis

Situation:

- The first detected label appears closer to the x-axis than expected.
- The pixel distance is smaller than the common spacing.

Action:

```text
toAdd = detected_label_value
fromCal = first detected label coordinate
```

Reason:

The detected label is used as the reference because the x-axis-to-label gap does not represent a normal interval.

### Case 8: First Label Missed and Pixel Spacing Is Irregular with Non-Zero Offset

Situation:

- OCR misses the first expected label.
- Pixel spacing is irregular.
- The numerical sequence indicates that the graph has a non-zero base.

Action:

```text
toAdd = inferred corrected value
fromCal = inferred corrected coordinate
```

Reason:

The algorithm reconstructs both the missing value and the correct reference coordinate before calculating plotted values.

## 4. Extra Top-Bar Detections in Bar Graphs

Bar graph extraction can detect more horizontal lines than the actual number of bars because axis lines, grid lines, text edges, or noise can also appear as horizontal edges.

### Problem

If every detected horizontal line is treated as a bar top, the output contains false values.

### Solution

A detected line is accepted as a bar top only when:

- It lies inside the plotting region.
- It is horizontal.
- The pixel above the line belongs to the background cluster.
- The pixel below the line belongs to the bar-color cluster.

After that, each valid top edge is mapped to the nearest x-axis label.

## 5. Missing Middle Points in Line Graphs with Same Slope

Line graphs can have consecutive segments with the same slope. If the line appears visually continuous, the middle point may not create a clear corner or intersection.

### Problem

When points lie on the same straight path, the algorithm may detect one long line instead of two connected segments. This can cause a middle data point to be skipped.

Example:

```text
January -> February -> March
```

If January-to-February and February-to-March have the same slope, February may not be detected as a separate point.

### Solution

The algorithm uses expected x-axis label spacing to infer where points should exist. It compares detected line segment positions with the average distance between x-axis labels. If a point is expected based on label spacing but no obvious intersection appears, the algorithm can still account for the point position instead of dropping it entirely.

This helps preserve all data points even when the plotted line is visually continuous.

## 6. Missing Line Segments in Line Graphs

Another line graph challenge occurs when a line segment is not detected. This can happen because of weak color contrast, broken edges, or preprocessing noise.

### Problem

If one segment is missing:

- The intersection between adjacent segments cannot be calculated.
- Two real points may be lost.
- A false point may be produced from unrelated detected geometry.

### Solution

The algorithm checks consecutive line segments and measures the horizontal gap between the end of one segment and the start of the next.

If the gap is small:

```text
calculate intersection point
```

If the gap is large:

```text
do not force an intersection
store the endpoint of the previous segment
store the starting point of the next segment
```

Finally, the endpoint of the last segment is also included.

This keeps the extracted line graph data continuous and reduces false intersection points.

## 7. Scatter Point Detection Sensitivity

Scatter plots are sensitive because both x and y values must be extracted from every detected point.

### Problem

Small errors in blob center detection, OCR tick values, or axis detection affect coordinate extraction. X-coordinate extraction was especially more error-prone than y-coordinate extraction in the reported results.

### Solution

The pipeline:

- Calculates VPP separately for x and y axes.
- Uses K-means clustering to isolate point color.
- Uses blob detection to find point centers.
- Converts every point center into x-y values using separate offsets and references.

This made scatter plot extraction possible without training a custom neural network.

## Summary

The most important challenges solved were:

- Reading rotated y-axis text
- Cleaning incorrect OCR labels
- Handling missing or irregular first y-axis labels through eight cases
- Filtering false top-bar detections
- Recovering line graph points when slopes are identical
- Handling missing line segments without creating false intersections
- Extracting scatter point coordinates using both x-axis and y-axis calibration

These edge-case solutions are a major part of why the project works beyond only simple ideal graphs.
