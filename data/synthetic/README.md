# Synthetic Ground Truth

This folder contains generated ground-truth values used to validate extraction accuracy.

## Bar

```text
bar/testData.csv
```

Contains generated bar values for synthetic bar graph images.

## Line

```text
line/testData.csv
```

Contains generated y-values for synthetic line graph images.

## Scatter

```text
scatter/testDataX.csv
scatter/testDataY.csv
```

Scatter plots have separate x and y value files because each point has two coordinates.

## Usage

The extraction notebooks compare predicted values against these CSV files to calculate RMSE and accuracy.
