# Output, Results, and Metrics

This document describes the API output format for `POST /predict`, the prediction result structure, and the model metrics returned by `GET /metrics`.

## 1. Prediction Output (`POST /predict`)

The `POST /predict` endpoint accepts a `.npz` scan upload and returns AI segmentation results, centroid data, curve fitting output, and a diagnosis summary.

### Response Fields

- `points`: Normalized 3D points sampled from the input scan. These are returned at a reduced rate (`[::5]`) for frontend performance.
- `labels`: Predicted tooth class IDs for each sampled point (`0` = background/gum, `1-49` = tooth IDs).
- `ground_truth`: Optional labels from the uploaded `.npz` file, if present, also sampled for comparison.
- `centroids`: Display-space centroids of each predicted tooth cluster.
- `aligned_centroids`: The closest points on the fitted arch curve for each centroid.
- `centroid_ids`: The tooth IDs present in the prediction.
- `curve`: The fitted arch curve expressed as arrays for `x`, `y`, and `z` coordinates.
- `diagnosis`: A rule-based diagnosis object containing a title and detail list.

### Example Response

```json
{
  "points": [[x,y,z], ...],
  "labels": [tooth_id, ...],
  "ground_truth": [tooth_id, ...],
  "centroids": [[x,y,z], ...],
  "aligned_centroids": [[x,y,z], ...],
  "centroid_ids": [1, 2, ...],
  "curve": { "x": [...], "y": [...], "z": [...] },
  "diagnosis": { "title": "Mild Crowding", "details": ["Overlap: Tooth 3 & 4"] }
}
```

### Notes

- The endpoint is protected with JWT auth; a valid bearer token is required.
- The model output uses normalized coordinates for visualization, while centroids are computed from raw scan coordinates internally.
- Diagnosis categories include: `Normal Alignment`, `Mild Crowding`, `Severe Crowding`, `Generalized Spacing`, and `Arch Misalignment`.

## 2. Results Display in Frontend

The frontend uses the `predict` response to render:

- 3D segmentation scatter plots for AI predictions and optional ground truth.
- A comparison of actual tooth centroids versus aligned arch curve points.
- A downloadable diagnosis report containing the diagnosis title and details.

The UI also exposes a `metrics` tab that fetches performance values from the backend.

## 3. Model Metrics (`GET /metrics`)

The metrics endpoint returns model evaluation numbers for the trained PointNet segmentation model.

### Response Fields

- `accuracy`: Overall point-level classification accuracy.
- `precision`: Weighted precision across classes.
- `recall`: Weighted recall across classes.
- `f1_score`: Weighted F1 score.
- `total_points`: The number of points evaluated in the test or validation dataset.

### Example Response

```json
{
  "accuracy": 80.32,
  "precision": 82.49,
  "recall": 80.32,
  "f1_score": 80.72,
  "total_points": "163,840"
}
```

### Frontend Metrics View

The app displays:

- Overall Accuracy
- Weighted Precision
- Weighted Recall
- F1 Score
- Total points evaluated
- Output classes (`50` tooth IDs + background)

## 4. Usage Summary

- Use `POST /predict` to upload a `.npz` point cloud and receive segmentation results plus an orthodontic diagnosis.
- Use `GET /metrics` to retrieve the current model evaluation metrics displayed in the frontend.
- Both endpoints require authentication via JWT.
