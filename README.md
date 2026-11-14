# Words to Worlds — Recognition

Private intake repository for the Python letter, block, and word recognition code used by Words to Worlds.

## Upload your code

Open [recognition/](recognition/) and choose **Add file → Upload files**, or use the [direct upload page](https://github.com/hyuntori2000/words-to-worlds-recognition/upload/main/recognition).

Upload your existing Python modules or package, dependency file, and short run instructions. Keep your current structure; you do not need to rewrite the algorithm before uploading. Describe its current image input and returned results in your README.

Do not upload API keys, environment files, or private camera/voice recordings. Browser uploads also need this check; `.gitignore` is not an upload security filter.

## Current implementation status

This repository initially contains documentation only. It does not yet contain a working recognizer or Quest camera receiver.

The separate Unity demo already has a Python voice/story backend and a TCP bridge for recognized words with board-local 3D coordinates. Its word recognition remains mocked. Camera capture, image transport, the uploaded recognition algorithm, and pixel-to-3D integration still need to be connected.

## Proposed integration

```text
Quest Unity camera capture
  → image bytes + frame reference
Mac Python image receiver
  → decoded image array
  → your recognition function
  → letters, grouped words, image coordinates + same frame reference
Quest Unity
  → capture-time geometry maps pixels to a physical position
  → confirm and freeze the word position
  → existing spatial word sound, question, microphone, and story flow
```

The Python algorithm runs on the Mac, not inside the Quest camera driver. Start with one snapshot per Complete action. If the algorithm tracks blocks across time, use a persistent recognizer and an ordered, rate-limited sequence of frames. A readable word is a candidate; **Done/Complete** confirms the child's intent to finish it.

See [the proposed camera contract](docs/CAMERA_FRAME_CONTRACT.md) for data formats, coordinate handling, and integration gaps. This is a design proposal, not an endpoint that is already running.
