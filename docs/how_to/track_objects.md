---
comments: true
---

# Track Objects

Leverage Supervision's advanced capabilities for enhancing your video analysis by
seamlessly [tracking](/latest/trackers/) objects recognized by
a multitude of object detection and segmentation models. This comprehensive guide will
take you through the steps to perform inference using the YOLOv8 model via either the
[Inference](https://github.com/roboflow/inference) or
[Ultralytics](https://github.com/ultralytics/ultralytics) packages. Following this,
you'll discover how to track these objects efficiently and annotate your video content
for a deeper analysis.

To make it easier for you to follow our tutorial download the video we will use as an
example. You can do this using
[`supervision[assets]`](/latest/assets/) extension.

```python
from supervision.assets import download_assets, VideoAssets

download_assets(VideoAssets.PEOPLE_WALKING)
```

<video controls>
    <source src="https://media.roboflow.com/supervision/video-examples/people-walking.mp4" type="video/mp4">
</video>

## Run Inference

First, you'll need to obtain predictions from your object detection or segmentation
model. In this tutorial, we are using the YOLOv8 model as an example. However,
Supervision is versatile and compatible with various models. Check this
[link](/latest/how_to/detect_and_annotate/#load-predictions-into-supervision)
for guidance on how to plug in other models.

We will define a `callback` function, which will process each frame of the video
by obtaining model predictions and then annotating the frame based on these predictions.
This `callback` function will be essential in the subsequent steps of the tutorial, as
it will be modified to include tracking, labeling, and trace annotations.

=== "Ultralytics"

    ```{ .py }
    import numpy as np
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    box_annotator = sv.BoundingBoxAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model(frame)[0]
        detections = sv.Detections.from_ultralytics(results)
        return box_annotator.annotate(frame.copy(), detections=detections)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

=== "Inference"

    ```{ .py }
    import numpy as np
    import supervision as sv
    from inference.models.utils import get_roboflow_model

    model = get_roboflow_model(model_id="yolov8n-640", api_key=<ROBOFLOW API KEY>)
    box_annotator = sv.BoundingBoxAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model.infer(frame)[0]
        detections = sv.Detections.from_inference(results)
        return box_annotator.annotate(frame.copy(), detections=detections)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

<video controls>
    <source src="https://media.roboflow.com/supervision/video-examples/how-to/track-objects/run-inference.mp4" type="video/mp4">
</video>

## Tracking

After running inference and obtaining predictions, the next step is to track the
detected objects throughout the video. Utilizing Supervision’s
[`sv.ByteTrack`](/latest/trackers/#supervision.tracker.byte_tracker.core.ByteTrack)
functionality, each detected object is assigned a unique tracker ID,
enabling the continuous following of the object's motion path across different frames.

=== "Ultralytics"

    ```{ .py hl_lines="6 12" }
    import numpy as np
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    tracker = sv.ByteTrack()
    box_annotator = sv.BoundingBoxAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model(frame)[0]
        detections = sv.Detections.from_ultralytics(results)
        detections = tracker.update_with_detections(detections)
        return box_annotator.annotate(frame.copy(), detections=detections)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

=== "Inference"

    ```{ .py hl_lines="6 12" }
    import numpy as np
    import supervision as sv
    from inference.models.utils import get_roboflow_model

    model = get_roboflow_model(model_id="yolov8n-640", api_key=<ROBOFLOW API KEY>)
    tracker = sv.ByteTrack()
    box_annotator = sv.BoundingBoxAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model.infer(frame)[0]
        detections = sv.Detections.from_inference(results)
        detections = tracker.update_with_detections(detections)
        return box_annotator.annotate(frame.copy(), detections=detections)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

## Annotate Video with Tracking IDs

Annotating the video with tracking IDs helps in distinguishing and following each object
distinctly. With the
[`sv.LabelAnnotator`](/latest/detection/annotators/#supervision.annotators.core.LabelAnnotator)
in Supervision, we can overlay the tracker IDs and class labels on the detected objects,
offering a clear visual representation of each object's class and unique identifier.

=== "Ultralytics"

    ```{ .py hl_lines="8 15-19 23-24" }
    import numpy as np
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    tracker = sv.ByteTrack()
    box_annotator = sv.BoundingBoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model(frame)[0]
        detections = sv.Detections.from_ultralytics(results)
        detections = tracker.update_with_detections(detections)

        labels = [
            f"#{tracker_id} {results.names[class_id]}"
            for class_id, tracker_id
            in zip(detections.class_id, detections.tracker_id)
        ]

        annotated_frame = box_annotator.annotate(
            frame.copy(), detections=detections)
        return label_annotator.annotate(
            annotated_frame, detections=detections, labels=labels)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

=== "Inference"

    ```{ .py hl_lines="8 15-19 23-24" }
    import numpy as np
    import supervision as sv
    from inference.models.utils import get_roboflow_model

    model = get_roboflow_model(model_id="yolov8n-640", api_key=<ROBOFLOW API KEY>)
    tracker = sv.ByteTrack()
    box_annotator = sv.BoundingBoxAnnotator()
    label_annotator = sv.LabelAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model.infer(frame)[0]
        detections = sv.Detections.from_inference(results)
        detections = tracker.update_with_detections(detections)

        labels = [
            f"#{tracker_id} {results.names[class_id]}"
            for class_id, tracker_id
            in zip(detections.class_id, detections.tracker_id)
        ]

        annotated_frame = box_annotator.annotate(
            frame.copy(), detections=detections)
        return label_annotator.annotate(
            annotated_frame, detections=detections, labels=labels)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

<video controls>
    <source src="https://media.roboflow.com/supervision/video-examples/how-to/track-objects/annotate-video-with-tracking-ids.mp4" type="video/mp4">
</video>

## Annotate Video with Traces

Adding traces to the video involves overlaying the historical paths of the detected
objects. This feature, powered by the
[`sv.TraceAnnotator`](/latest/detection/annotators/#supervision.annotators.core.TraceAnnotator),
allows for visualizing the trajectories of objects, helping in understanding the
movement patterns and interactions between objects in the video.

=== "Ultralytics"

    ```{ .py hl_lines="9 26-27" }
    import numpy as np
    import supervision as sv
    from ultralytics import YOLO

    model = YOLO("yolov8n.pt")
    tracker = sv.ByteTrack()
    box_annotator = sv.BoundingBoxAnnotator()
    label_annotator = sv.LabelAnnotator()
    trace_annotator = sv.TraceAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model(frame)[0]
        detections = sv.Detections.from_ultralytics(results)
        detections = tracker.update_with_detections(detections)

        labels = [
            f"#{tracker_id} {results.names[class_id]}"
            for class_id, tracker_id
            in zip(detections.class_id, detections.tracker_id)
        ]

        annotated_frame = box_annotator.annotate(
            frame.copy(), detections=detections)
        annotated_frame = label_annotator.annotate(
            annotated_frame, detections=detections, labels=labels)
        return trace_annotator.annotate(
            annotated_frame, detections=detections)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

=== "Inference"

    ```{ .py hl_lines="9 26-27" }
    import numpy as np
    import supervision as sv
    from inference.models.utils import get_roboflow_model

    model = get_roboflow_model(model_id="yolov8n-640", api_key=<ROBOFLOW API KEY>)
    tracker = sv.ByteTrack()
    box_annotator = sv.BoundingBoxAnnotator()
    label_annotator = sv.LabelAnnotator()
    trace_annotator = sv.TraceAnnotator()

    def callback(frame: np.ndarray, _: int) -> np.ndarray:
        results = model.infer(frame)[0]
        detections = sv.Detections.from_inference(results)
        detections = tracker.update_with_detections(detections)

        labels = [
            f"#{tracker_id} {results.names[class_id]}"
            for class_id, tracker_id
            in zip(detections.class_id, detections.tracker_id)
        ]

        annotated_frame = box_annotator.annotate(
            frame.copy(), detections=detections)
        annotated_frame = label_annotator.annotate(
            annotated_frame, detections=detections, labels=labels)
        return trace_annotator.annotate(
            annotated_frame, detections=detections)

    sv.process_video(
        source_path="people-walking.mp4",
        target_path="result.mp4",
        callback=callback
    )
    ```

<video controls>
    <source src="https://media.roboflow.com/supervision/video-examples/how-to/track-objects/annotate-video-with-traces.mp4" type="video/mp4">
</video>

This structured walkthrough should give a detailed pathway to annotate videos
effectively using Supervision’s various functionalities, including object tracking and
trace annotations.
