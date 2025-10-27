# Undercaption — Subtitle Recovery & Translation

Recover subtitles from long/old films with a robust, chunked ASR pipeline (Whisper/Faster-Whisper), VAD-based splitting, optional diarization, QC rules, and multi-language SRT export.

## Key features
- Large-file safe: chunked processing (VAD + overlaps), resume-safe manifests.
- Clean audio path: ffmpeg extract, loudness norm; optional RNNoise.
- Accurate ASR: Whisper (word/segment timestamps), lexicon boosts.
- Readable SRT: CPS limits, min/max durations, smart merge/split.
- Diarization (opt): speaker tags; SDH cues ([MUSIC], [APPLAUSE]).
- Translation: preserve timestamps across EN/JA/FR/IT/ES/SV (pluggable MT).
- Artifacts: `.srt` + optional `.vtt` and word-level `json`.

## Quickstart (≤10 min, English)
```bash
# 1) Setup (Python 3.11+)
uv venv && uv pip install -U pip wheel && uv pip install -e .
# or: poetry install

# 2) Minimal run
undercaption process input.mp4 \
  --out out/ \
  --lang en \
  --vad silero \
  --model faster-whisper:medium

# 3) Translate the produced SRT
undercaption translate out/input.en.srt --to es fr it --out out/
CLI overview

undercaption process <input_video> \
  [--audio-rate 16000 --mono] \
  [--chunk-min 1.5 --chunk-max 15.0 --overlap 0.3] \
  [--lang auto|en|es|fr|it|ja|sv] \
  [--model faster-whisper:{small|medium|large-v3}] \
  [--denoise rnnoise|off] \
  [--diarization on|off] \
  [--srt-max-dur 7.0 --srt-min-dur 1.0 --cps-max 20]
Docker

docker build -t undercaption .
docker run --rm -v $PWD:/work undercaption \
  undercaption process /work/input.mkv --out /work/out --lang auto
Pipeline
Probe & extract: ffprobe → ffmpeg (mono, 16 kHz PCM/FLAC).

Chunking: VAD (WebRTC/Silero) with overlaps + manifest for resume.

ASR: Faster-Whisper (timestamps, confidences); optional phrase list.

(Opt) Diarize: pyannote speakers; SDH cues for common events.

Shape SRT: enforce CPS, min/max, merge/split, punctuation restore.

QC: no overlaps, bounded gaps, monotonic times; flag low-confidence.

Translate: NLLB/Marian (default) or plugin providers; keep timings.

Export: .srt/.vtt + job.json (segments, words, confidences).

Config examples

# Higher accuracy, GPU, diarization on
undercaption process film.mkv --model faster-whisper:large-v3 --diarization on --gpu 0

# Long film (4h): safer chunking
undercaption process film.mkv --chunk-max 10.0 --overlap 0.4 --resume
Output quality rules
≤42 chars/line, 1–2 lines; 1.0–7.0 s per cue; ~12–20 cps target.

Unicode NFC; monotonic timestamps; speaker tags consistent.

Non-speech labels preserved; not translated.

Example SRT

1
00:00:01,000 --> 00:00:03,400
[sirens wailing in the distance]

2
00:00:03,600 --> 00:00:07,200
I told you this street was quieter at night.
Project structure

undercaption/
  cli.py              # Typer CLI
  audio.py            # ffmpeg I/O, loudness, denoise
  vad.py              # WebRTC/Silero chunking
  asr.py              # Faster-Whisper wrapper
  diarization.py      # pyannote integration (optional)
  srt_format.py       # shaping, CPS checks
  translate.py        # NLLB/Marian + providers
  qc.py               # alignment & sanity checks
  report.py           # HTML job report
Performance & limits
Handles 10 GB / 4 h via chunking; memory bounded by window size.

Accuracy varies with audio quality/dialect; diarization on noisy sources may drift.

OCR for burned-in subs is experimental.

Roadmap
Forced alignment fallback (aeneas/Gentle-style).

Scene-cut aware segmentation.

Web dashboard with waveform + CPS heatmap edits.

Subtitle repository discovery (ToS-compliant, opt-in).

Legal
Process only material you have the rights to. Respect platform ToS when sourcing audio/subtitles.

Contributing
Issues and PRs welcome. Please attach a <10 min test clip and redacted logs.

License
MIT (see LICENSE).

---

# HTML

<h1>Undercaption — Subtitle Recovery &amp; Translation</h1>
<p>Recover subtitles from long/old films with a robust, chunked ASR pipeline (Whisper/Faster-Whisper), VAD-based splitting, optional diarization, QC rules, and multi-language SRT export.</p>

<h2>Key features</h2>
<ul>
  <li>Large-file safe: chunked processing (VAD + overlaps), resume-safe manifests.</li>
  <li>Clean audio path: ffmpeg extract, loudness norm; optional RNNoise.</li>
  <li>Accurate ASR: Whisper (word/segment timestamps), lexicon boosts.</li>
  <li>Readable SRT: CPS limits, min/max durations, smart merge/split.</li>
  <li>Diarization (opt): speaker tags; SDH cues ([MUSIC], [APPLAUSE]).</li>
  <li>Translation: preserve timestamps across EN/JA/FR/IT/ES/SV (pluggable MT).</li>
  <li>Artifacts: <code>.srt</code> + optional <code>.vtt</code> and word-level <code>json</code>.</li>
</ul>

<h2>Quickstart (≤10 min, English)</h2>
<pre><code># Setup (Python 3.11+)
uv venv &amp;&amp; uv pip install -U pip wheel &amp;&amp; uv pip install -e .
# or: poetry install

# Minimal run
undercaption process input.mp4 \
  --out out/ \
  --lang en \
  --vad silero \
  --model faster-whisper:medium

# Translate the produced SRT
undercaption translate out/input.en.srt --to es fr it --out out/
</code></pre>

<h2>CLI overview</h2>
<pre><code>undercaption process &lt;input_video&gt; \
  [--audio-rate 16000 --mono] \
  [--chunk-min 1.5 --chunk-max 15.0 --overlap 0.3] \
  [--lang auto|en|es|fr|it|ja|sv] \
  [--model faster-whisper:{small|medium|large-v3}] \
  [--denoise rnnoise|off] \
  [--diarization on|off] \
  [--srt-max-dur 7.0 --srt-min-dur 1.0 --cps-max 20]
</code></pre>

<h2>Docker</h2>
<pre><code>docker build -t undercaption .
docker run --rm -v $PWD:/work undercaption \
  undercaption process /work/input.mkv --out /work/out --lang auto
</code></pre>

<h2>Pipeline</h2>
<ol>
  <li><strong>Probe &amp; extract</strong>: ffprobe → ffmpeg (mono, 16 kHz PCM/FLAC).</li>
  <li><strong>Chunking</strong>: VAD (WebRTC/Silero) with overlaps + manifest for resume.</li>
  <li><strong>ASR</strong>: Faster-Whisper (timestamps, confidences); optional phrase list.</li>
  <li><strong>(Opt) Diarize</strong>: pyannote speakers; SDH cues for common events.</li>
  <li><strong>Shape SRT</strong>: enforce CPS, min/max, merge/split, punctuation restore.</li>
  <li><strong>QC</strong>: no overlaps, bounded gaps, monotonic times; flag low-confidence.</li>
  <li><strong>Translate</strong>: NLLB/Marian (default) or plugin providers; keep timings.</li>
  <li><strong>Export</strong>: <code>.srt</code>/<code>.vtt</code> + <code>job.json</code> (segments, words, confidences).</li>
</ol>

<h2>Config examples</h2>
<pre><code># Higher accuracy, GPU, diarization on
undercaption process film.mkv --model faster-whisper:large-v3 --diarization on --gpu 0

# Long film (4h): safer chunking
undercaption process film.mkv --chunk-max 10.0 --overlap 0.4 --resume
</code></pre>

<h2>Output quality rules</h2>
<ul>
  <li>&le; 42 chars/line, 1–2 lines; 1.0–7.0 s per cue; ~12–20 cps target.</li>
  <li>Unicode NFC; monotonic timestamps; speaker tags consistent.</li>
  <li>Non-speech labels preserved; not translated.</li>
</ul>

<h2>Example SRT</h2>
<pre><code>1
00:00:01,000 --&gt; 00:00:03,400
[sirens wailing in the distance]

2
00:00:03,600 --&gt; 00:00:07,200
I told you this street was quieter at night.
</code></pre>

<h2>Project structure</h2>
<pre><code>undercaption/
  cli.py              # Typer CLI
  audio.py            # ffmpeg I/O, loudness, denoise
  vad.py              # WebRTC/Silero chunking
  asr.py              # Faster-Whisper wrapper
  diarization.py      # pyannote integration (optional)
  srt_format.py       # shaping, CPS checks
  translate.py        # NLLB/Marian + providers
  qc.py               # alignment &amp; sanity checks
  report.py           # HTML job report
</code></pre>

<h2>Performance &amp; limits</h2>
<ul>
  <li>Handles 10 GB / 4 h via chunking; memory bounded by window size.</li>
  <li>Accuracy varies with audio quality/dialect; diarization on noisy sources may drift.</li>
  <li>OCR for burned-in subs is experimental.</li>
</ul>

<h2>Roadmap</h2>
<ul>
  <li>Forced alignment fallback (aeneas/Gentle-style).</li>
  <li>Scene-cut aware segmentation.</li>
  <li>Web dashboard with waveform + CPS heatmap edits.</li>
  <li>Subtitle repository discovery (ToS-compliant, opt-in).</li>
</ul>

<h2>Legal</h2>
<p>Process only material you have the rights to. Respect platform ToS when sourcing audio/subtitles.</p>

<h2>Contributing</h2>
<p>Issues and PRs welcome. Please attach a &lt;10 min test clip and redacted logs.</p>

<h2>License</h2>
<p>MIT (see <code>LICENSE</code>).</p>