---
sidebar_position: 3
sidebar_label: '🚀 API Reference'
---

# 🚀 API Reference

The core of this project is a serverless API endpoint that allows you to get a view count prediction based on video metadata.

## Predict Endpoint

*   **HTTP Method:** `POST`
*   **Endpoint URL:** `https://3bd5z76vu0.execute-api.us-east-1.amazonaws.com/v1/predict`
*   **Description:** Sends video feature data to the model and returns a predicted view count.

---

## Request Body (JSON)

The `POST` request must include a JSON body with the following structure and data types.

| Parameter             | Type      | Description                                                 | Example      |
| :-------------------- | :-------- | :---------------------------------------------------------- | :----------- |
| `like_count`          | `integer` | The total number of likes on the video.                     | `5000`       |
| `comment_count`       | `integer` | The total number of comments on the video.                  | `200`        |
| `duration_seconds`    | `integer` | The length of the video in seconds.                         | `300`        |
| `tag_count`           | `integer` | The number of tags associated with the video.               | `15`         |
| `category_id`         | `integer` | The numerical ID for the YouTube category (e.g., 24 is Entertainment). | `24`         |
| `publish_hour`        | `integer` | The hour of the day the video was published (0-23).         | `14`         |
| `publish_day_of_week` | `integer` | The day of the week (0=Monday, 6=Sunday).                   | `3`          |
| `channel_title`       | `string`  | The name of the YouTube channel.                            | `"Some Channel"` |

### Example Request Body

```json
{
  "like_count": 5000,
  "comment_count": 200,
  "duration_seconds": 300,
  "tag_count": 15,
  "category_id": 24,
  "publish_hour": 14,
  "publish_day_of_week": 3,
  "channel_title": "Some Channel"
}
```
### Responses
#### Successful Response (200 OK)

If the request is successful, the API will return a JSON object with the predicted view count.
```json
{
  "predicted_views": 1234567.89
}
```

#### Error Response (400 Bad Request)
If the input data is missing or invalid, the API may return an error.
```json
{
  "error": "Invalid input provided. Please check your JSON payload."
}
```




