# Proposed Quest camera → Python contract

Status: design proposal. The demo does not yet implement this camera capture, HTTP receiver, or pixel-result adapter.

## Capture source

Use Quest 3/3S forward-facing RGB camera access in the Unity app through Meta MR Utility Kit's `PassthroughCameraAccess`, with camera permission and passthrough configured. A screen cast or Unity scene screenshot includes a different view and is not a substitute for the calibrated sensor image.

Use the actual supported image size and selected left/right camera. Keep camera identity, capture timestamp, image orientation, intrinsics, and the actual sensor pose matched to each frame. Do not substitute the latest headset-center pose when a delayed result arrives.

Official references:

- [Passthrough Camera API overview](https://developers.meta.com/vr/documentation/unity/unity-pca-overview/)
- [PassthroughCameraAccess API and migration](https://developers.meta.com/vr/documentation/unity/unity-pca-migration-from-webcamtexture/)

## Snapshot versus continuous recognition

For a stateless OCR function, first test one snapshot after a Complete action. This avoids a streaming implementation while exercising the complete recognition and positioning path.

For live preview, send a limited-rate sequence of frames, not necessarily every camera frame. Bound the queue and avoid processing an ever-growing backlog. A stateful tracker must stay alive across requests, process frames in order, and reset on session changes; agree whether dropped frames are supported. Do not create a new tracker for each request.

Seeing a valid word does not establish that the child is finished spelling. A stable candidate can be shown internally, but explicit Done/Complete confirms it for the first version. Do not commit a partial word such as CAT while the child is still building CATS. The final-word action also asks its question and records the answer before generating the story.

## Request to Python

Suggested new endpoint: `POST /recognize`, multipart image bytes plus JSON metadata. This endpoint does not currently exist.

Transport can use a high-quality JPEG for efficiency or PNG to avoid compression artifacts. Decode it into an image array before calling the recognizer. Keep the existing small-message TCP channel separate from images.

Example metadata (proposed):

```json
{
  "schemaVersion": 1,
  "sessionId": "session-001",
  "frameRevision": 2,
  "frameId": "capture-000042",
  "cameraEye": "left",
  "imageWidth": 1280,
  "imageHeight": 960,
  "pixelOrigin": "top-left"
}
```

The dimensions above are illustrative; send the actual decoded image dimensions. `frameRevision` identifies the board coordinate revision used by the existing demo, not the video frame number. `frameId` identifies one specific image. Unity caches the matching sensor timestamp and clock domain, sensor pose, intrinsics, image transforms, and board geometry under this reference. If Python needs those values for its own geometry algorithm, extend the request explicitly with units and coordinate conventions.

Suggested function input: NumPy `uint8`, shape `H × W × 3`, BGR channels for OpenCV. Convert to RGB if the model expects RGB. Illustrative adapter only:

```python
frame_bgr = cv2.imdecode(np.frombuffer(image_bytes, dtype=np.uint8), cv2.IMREAD_COLOR)
if frame_bgr is None:
    raise ValueError("Invalid image")
result = recognizer.recognize_frame(frame_bgr)
```

The real function name and result shape will follow the uploaded code. For a persistent tracker, `recognizer` is retained between frames.

## Result to Unity

Return the same session, board revision, and frame ID with letters, bounding boxes or centers in original-image pixels, grouped words, and confidence where available. If preprocessing crops, resizes, rotates, or mirrors the image, map results back to original-image coordinates or provide the exact inverse transform.

Keep component IDs stable only if the algorithm actually tracks physical bricks. A letter label is not a unique brick ID: a word can contain two As. An OCR group is a candidate, not automatically a completed word. If an end marker is detected, report it separately from the word text.

## Pixels → a location for sound

An RGB image and OCR bounding box do not provide depth or a block's 3D position by themselves. Recommended first setup: a calibrated fixed board and known-height blocks. Unity projects the returned image point using the matching capture-time camera geometry and intersects the ray with the relevant block surface plane. Account for the printed face height rather than assuming the baseplate surface. Alternative geometry can come from calibrated markers or a validated depth method.

Convert the resulting point to the demo's board-local meters: +X right, +Y up, +Z forward relative to `DetectionOrigin`. Reject results from old sessions or a changed coordinate revision. Do not attach a late result to the current frame merely because it arrived now.

Once a word is confirmed, register it once and freeze its story position. Later block movement or recognition updates must not move its sound. Head tracking continues locally in Quest. Release the story positions after narration according to the agreed experience flow. These freeze/reset behaviors still need implementation in the current demo.

## Existing bridge limitations

The existing TCP bridge on port 8765 accepts completed words and already-calculated board-local XYZ positions. It does not accept images or raw pixel detections; its messages are limited to 16 KB. `capture_reference()` returns a session/board revision reference, not a captured camera image.

The new HTTP result path needs a dedicated Unity adapter or a transport-neutral validated input method. The current local input method rejects Python mode, and Python-mode readiness is tied to the TCP bridge. Do not route HTTP pixels straight into that method or send them as XYZ. Keep capture-time validation and duplicate-completion prevention when connecting the new path to the existing audio/conversation flow.

## Integration checks after code upload

1. Run the uploaded algorithm on an allowed static sample and verify its dependency setup and output coordinates.
2. Capture one actual Quest image and check camera identity, orientation, dimensions, and permission behavior.
3. Send that image to Python and confirm returned pixel centers on the same image.
4. Verify physical alignment on known board points while the headset moves; measure error before relying on sound placement.
5. Confirm one completion triggers one question, then moving the physical blocks leaves the confirmed sound location unchanged.
6. Add continuous tracking only if the recognizer or interaction needs it; test latency, frame ordering, drop behavior, and session resets.

This repository setup does not install an APK or change the connected Quest.
