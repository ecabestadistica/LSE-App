# Spanish Sign Language (LSE) Alphabet Recognition

![Python](https://img.shields.io/badge/Python-3.10-blue)
![TensorFlow](https://img.shields.io/badge/TensorFlow-2.16.1-orange)
![License](https://img.shields.io/badge/License-CC_BY_NC_4.0-green)

A deep learning system for real-time recognition of the Spanish Sign Language (**LSE — Lengua de Signos Española**) fingerspelling alphabet. The project provides two ready-to-use applications built on top of a CNN model trained on hand keypoints extracted with [MediaPipe](https://google.github.io/mediapipe/):

- **Local app** — runs entirely on your machine using your webcam through OpenCV.
- **Web app** — a browser-based client/server application (aiohttp + WebSockets) that can also be exposed to the internet (e.g. via [ngrok](https://ngrok.com/)) for remote demos.

Both apps detect a hand in the camera feed, extract its keypoints, and classify the corresponding LSE letter in real time.

This repository accompanies the paper *"A Deep Learning Framework for Spanish Sign Language Alphabet Recognition"* (see [Acknowledgements](#acknowledgements) below).

## Recognized letters

The model classifies the following 23 static LSE alphabet signs:

```
A, B, C, CH, D, E, F, G, H, I, K, L, M, N, O, P, Q, R, S, T, U, W, X
```

## How it works

1. **Hand detection & keypoint extraction** — [MediaPipe Hands](https://google.github.io/mediapipe/solutions/hands.html) locates the hand in each frame and extracts 21 hand landmarks.
2. **Input representation** — the landmarks are rendered on a black background (keypoints-only representation), which is the lightweight input format used by the deployed model.
3. **Classification** — a CNN (DenseNet201-based backbone, see the paper for the full architecture comparison) predicts the corresponding letter and confidence score.
4. **Feedback** — the predicted letter and confidence are overlaid on the video feed in real time.

## Repository structure

```
.
├── app/
│   ├── implementation_local.py   # Local webcam app (OpenCV window)
│   ├── implementation_web.py     # Web server (aiohttp + WebSockets)
│   └── html/
│       ├── index.html            # Web app frontend
│       └── example_imgs/         # Reference images for each letter 
├── requirementsLSE.txt           # Python dependencies
├── LICENSE.md                    # CC BY-NC-SA 4.0
└── README.md
```

## Requirements

- Python 3.10 (recommended)
- A webcam
- The trained model `best_model_key.keras`

Install the Python dependencies:

```bash
pip install -r requirementsLSE.txt
```

The web app additionally requires `aiohttp`, which is not listed in `requirementsLSE.txt`:

```bash
pip install aiohttp
```

## Model

The apps expect a Keras model file named `best_model_key.keras` in the `app/` directory. This is the lightweight **keypoints-only** model described in the paper, selected for deployment because it offers the best trade-off between accuracy (98.5% evaluation accuracy) and inference latency for real-time use.
Download the model file from the following link and place it in the project root: 📥 **[Download Model](https://drive.google.com/file/d/12c0z4NlweH6cMRs32OLh_kYfkXmgwULL/view?usp=sharing)**

## Usage

### Local app

Runs fully offline using your default webcam.

```bash
cd app
python implementation_local.py
```

- Two OpenCV windows will open: one showing the extracted hand keypoints, and one showing the live prediction overlaid on your webcam feed.
- Press `q` to quit.

### Web app

Serves an interactive web page where users can see the live prediction and browse reference images for each letter.

```bash
cd app
python implementation_web.py
```

The server uses HTTPS/WSS, so you need a TLS certificate and key in the `app/` directory:

```
server.crt
server.key
```

For local testing you can generate a self-signed certificate, e.g.:

```bash
openssl req -x509 -newkey rsa:2048 -keyout server.key -out server.crt -days 365 -nodes
```

By default the server listens on port `8182` (configurable via the `PORT` environment variable). Once running, open:

```
https://localhost:8182
```

To expose the app publicly (e.g. for a remote demo), you can tunnel it with [ngrok](https://ngrok.com/):

```bash
ngrok http https://127.0.0.1:8182
```

**Web app features:**
- Live webcam prediction streamed over WebSockets.
- A letter selector that shows a reference image so users can learn and practice each sign.
- An "Info" panel with project details.

## Notes & limitations

- Both apps expect a single hand in frame (`max_num_hands=1`).
- The current models only recognize static fingerspelling signs, not continuous signing or full LSE grammar.
- Predictions below a confidence threshold are reported as "no prediction" in the web app.

## License

This project is licensed under [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International (CC BY-NC-SA 4.0)](LICENSE.md).

## Acknowledgements

The reference sign images used in the web app are sourced from the [Banco de imágenes y signos LSE](https://www.cnse.es/lseaula/recursos-linguisticos/dactilologico.php), provided by **Fundación CNSE**.

This work was partially supported by the Spanish Ministry of Science and Innovation under grant **PID2022-137243OB-I00**, and by the European Union's Recovery, Transformation and Resilience Plan – NextGenerationEU, through the INCIBE ANTICIPA grant and the ENIA 2022 programme for university–industry AI chairs (**AImpulsa: UC3M-Universia**).

If you use this code, the trained models, or the accompanying dataset in your research, please cite our paper:

> Marroquín, I., Cabana, E., & Lillo, R. E. (in press). *A Deep Learning Framework for Spanish Sign Language Alphabet Recognition*. Preprint submitted to Elsevier.

```bibtex
@article{marroquin2026lse,
  title   = {A Deep Learning Framework for Spanish Sign Language Alphabet Recognition},
  author  = {Marroqu{\'i}n, Ignacio and Cabana, Elisa and Lillo, Rosa Elvira},
  journal = {Preprint submitted to Elsevier},
  year    = {2026},
  note    = {Manuscript in preparation / under review. Citation to be updated upon publication.}
}
```

> This citation will be updated with the final DOI, volume, and page numbers once the paper is published.
