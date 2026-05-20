# Recording — Reference

Capture the tab + audio and download as `.webm`. Used by `feature-demo.html` and `vision-film.html`.

## Implementation

```js
const startRecording = React.useCallback(async () => {
  if (recording) return;
  if (!navigator.mediaDevices || !navigator.mediaDevices.getDisplayMedia) return;

  let videoStream;
  try {
    // CRITICAL: `audio: true` as a literal boolean is what triggers the
    // "Share tab audio" toggle in Chrome's share dialog. Passing an audio-
    // constraints object (e.g., { echoCancellation: false }) SUPPRESSES the
    // toggle on many Chrome versions.
    videoStream = await navigator.mediaDevices.getDisplayMedia({
      video: { frameRate: 30, displaySurface: 'browser' },
      audio: true,
      selfBrowserSurface: 'include',
      surfaceSwitching: 'exclude',
    });
  } catch (e) { return; }

  // Use shared tab audio if user ticked the box;
  // fall back to a captureStream from the voiceover <audio> element.
  const tracks = [...videoStream.getVideoTracks()];
  const sharedAudio = videoStream.getAudioTracks();
  if (sharedAudio.length > 0) {
    tracks.push(...sharedAudio);
  } else {
    const voAudioEl = window.__voiceoverAudio;
    if (voAudioEl && typeof voAudioEl.captureStream === 'function') {
      try {
        const voStream = voAudioEl.captureStream();
        tracks.push(...voStream.getAudioTracks());
      } catch (e) { /* silent fallback */ }
    }
  }
  const stream = new MediaStream(tracks);

  const MIME_OPTIONS = [
    'video/webm;codecs=vp9,opus',
    'video/webm;codecs=vp8,opus',
    'video/webm;codecs=vp9',
    'video/webm',
  ];
  const mimeType = MIME_OPTIONS.find(t => MediaRecorder.isTypeSupported(t)) || 'video/webm';
  const recorder = new MediaRecorder(stream, {
    mimeType,
    videoBitsPerSecond: 8_000_000,
    audioBitsPerSecond: 128_000,
  });

  const chunks = [];
  recorder.ondataavailable = (e) => { if (e.data && e.data.size > 0) chunks.push(e.data); };

  recorder.onstop = () => {
    stream.getTracks().forEach(t => t.stop());
    videoStream.getTracks().forEach(t => t.stop());
    const blob = new Blob(chunks, { type: mimeType });
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = `[product-name]-${new Date().toISOString().slice(0,16).replace(/[:T]/g,'')}.webm`;
    document.body.appendChild(a); a.click(); a.remove();
    URL.revokeObjectURL(url);
    setRecording(false);
  };

  // User cancelled the share via the browser bar
  stream.getVideoTracks()[0].onended = () => {
    if (recorder.state !== 'inactive') recorder.stop();
  };

  setRecording(true);
  // Let the layout settle (UI chrome unmounts before capture starts)
  await new Promise(r => requestAnimationFrame(() => requestAnimationFrame(r)));
  setTime(0);
  setPlaying(true);
  recorder.start();

  // Auto-stop: timeline duration / playbackSpeed + 0.2s safety tail
  const realRuntimeMs = (duration / playbackSpeed + 0.2) * 1000;
  setTimeout(() => {
    if (recorder.state !== 'inactive') recorder.stop();
  }, realRuntimeMs);
}, [recording, duration, playbackSpeed]);
```

## Critical gotchas

1. **Hide UI chrome during recording.** Wrap the playback bar, mute button, tap-to-start pill, etc., with `{!recording && (...)}`. Otherwise they capture into the .webm.

2. **Stop the timeline from looping.** In the animation loop:
   ```js
   if (loop && !recording) next = next % duration;
   else { next = duration; setPlaying(false); }
   ```
   If the timeline loops during recording, the .webm tail captures Scene 1 again.

3. **Audio attribute on `<audio>` tag — no `crossOrigin`.** Setting `crossOrigin="anonymous"` makes the browser do a CORS fetch that fails on `file://`. Just omit it.

4. **`playbackSpeed` affects the auto-stop timer.** Real-time runtime = `duration / playbackSpeed`. At 1.3× speed, a 60s timeline plays in ~46 real seconds. Without dividing, recorder stops before the timeline finishes.

5. **The share dialog doesn't always show "Share tab audio" by default.** Tell the user explicitly: in the share picker, pick the **Chrome Tab** panel, then tick **"Also share tab audio"** before hitting Share.

## Expected flow when user clicks record

```
Click record
   ↓
Browser shows share dialog
   ↓
User picks tab + ticks "Also share tab audio"
   ↓
setRecording(true) → UI chrome hides
   ↓
Timeline resets to 0 and plays
   ↓
After (duration / playbackSpeed + 0.2) seconds → recorder.stop()
   ↓
.webm downloads
   ↓
setRecording(false) → UI chrome returns
```
