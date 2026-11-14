# Upload the recognition code here

Please keep the existing package layout and include:

- Python source files and the current entry point.
- `requirements.txt` or `pyproject.toml`, plus the Python version used.
- Run instructions, model download instructions if needed, and expected input/output examples.
- Whether the code reads one image independently or tracks blocks across multiple frames.
- Expected color format (BGR/RGB), image dimensions, and how it handles rotated blocks, repeated letters, grouping, and reading direction.

If the current application reads `cv2.VideoCapture(0)`, include it as-is. We can separate camera acquisition from recognition and replace the input source with Quest frames. A USB-connected Quest does not automatically become camera 0 on the Mac.

An eventual adapter might expose `recognize_frame(frame_bgr)`, but your current function does not need to use that name. The actual signature will be agreed after inspecting your code.

Keep API keys and private camera/voice recordings out of this repository. Share only sample data you have permission to share. No recognizer is implemented in this folder yet.
