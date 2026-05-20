# Audio — Reference

The demo can use either a voiceover (single MP3 synced to the timeline) or discrete sound cues (one-shot SFX fired at specific moments). Don't do both — they compete.

## Decision matrix

| Context | Recommended approach |
|---|---|
| Homepage hero video (autoplay muted) | Sound cues + captions |
| App store preview | Sound cues |
| Sales deck (sound on) | Voiceover |
| Investor pitch (sound on) | Voiceover |
| Social autoplay (Twitter, LinkedIn) | Sound cues + captions |

Default: **sound cues**. Most viewers see autoplay-muted; cues degrade gracefully to silence. VO requires the viewer to unmute.

## Voiceover component

```jsx
function VoiceOver({ src }) {
  const { time, playing, recording } = useTimeline();
  const audioRef = React.useRef(null);
  const [unlocked, setUnlocked] = React.useState(false);
  const [loaded, setLoaded] = React.useState(false);
  const [missing, setMissing] = React.useState(false);
  const [muted, setMuted] = React.useState(false);

  // Unlock on first user gesture
  React.useEffect(() => {
    const unlock = () => setUnlocked(true);
    const opts = { once: true, passive: true };
    window.addEventListener('click', unlock, opts);
    window.addEventListener('keydown', unlock, opts);
    return () => { /* cleanup */ };
  }, []);

  // Play/pause follows timeline
  React.useEffect(() => {
    const a = audioRef.current;
    if (!a || !unlocked || !loaded) return;
    if (playing && !muted) a.play().catch(() => {});
    else a.pause();
  }, [playing, muted, unlocked, loaded]);

  // Critical sync formula: expected = time / playbackSpeed
  // Audio plays at natural 1.0x rate; timeline may be at any speed.
  React.useEffect(() => {
    const a = audioRef.current;
    if (!a || !loaded) return;
    const expected = time / playbackSpeed; // playbackSpeed from context if used
    const drift = Math.abs(a.currentTime - expected);
    if (drift > 0.35) {
      try { a.currentTime = Math.max(0, expected); } catch {}
      if (playing && !muted && unlocked && a.paused) a.play().catch(() => {});
    }
  }, [Math.floor(time * 4), loaded]);

  // Expose audio element for the recorder fallback
  React.useEffect(() => {
    if (audioRef.current) window.__voiceoverAudio = audioRef.current;
    return () => { delete window.__voiceoverAudio; };
  }, [loaded]);

  return (
    <>
      <audio
        ref={audioRef}
        src={src}
        preload="auto"
        onCanPlayThrough={() => { setLoaded(true); setMissing(false); }}
        onError={() => setMissing(true)}
      />
      {/* DO NOT set crossOrigin on <audio> — breaks file:// loading */}
      {loaded && !unlocked && !recording && (
        <div className="vo-unlock-pill">Click anywhere to start voice-over</div>
      )}
      {!recording && (
        <button onClick={() => setMuted(m => !m)}>{muted ? 'Unmute' : 'Mute'}</button>
      )}
    </>
  );
}
```

**Critical:** If using VO, set `playbackSpeed={1.0}` in the Stage. Audio plays at natural rate; if the timeline runs at 1.3×, they desync immediately.

## Sound cue component

```jsx
function SoundCue({ src, at, volume = 1.0 }) {
  const { time } = useTimeline();
  const audioRef = React.useRef(null);
  const playedRef = React.useRef(false);

  // Reset trigger when timeline scrubs back past the fire point
  React.useEffect(() => {
    if (time < at - 0.1) playedRef.current = false;
  }, [Math.floor(time * 4), at]);

  React.useEffect(() => {
    if (time >= at && !playedRef.current && audioRef.current) {
      playedRef.current = true;
      try {
        audioRef.current.currentTime = 0;
        audioRef.current.volume = volume;
        audioRef.current.play().catch(() => {});
      } catch (e) {}
    }
  }, [time >= at, at, volume]);

  return <audio ref={audioRef} src={src} preload="auto" style={{ display: 'none' }}/>;
}
```

## Standard sound cue palette

Drop 6–10 of these in `sounds/`:

| File | Use | Duration |
|---|---|---|
| `tap.mp3` | UI button/icon clicks | ~0.1s |
| `whoosh.mp3` | Scene transitions, motion | ~0.4s |
| `thud.mp3` | Item landing / file appearing | ~0.2s |
| `chime.mp3` | Success state | ~0.5s |
| `dissolve.mp3` | Particle scatter / vanishing | ~1.0s |
| `stinger.mp3` | Logo reveal at close | ~1.0s |
| `msg-send.mp3` | Outgoing chat message | ~0.3s |
| `msg-receive.mp3` | Incoming chat message | ~0.3s |

Source recommendations: Freesound.org, Zapsplat. Avoid attribution-required licensing for production use.

## Composition in the demo

```jsx
function SoundTrack() {
  return (
    <>
      <SoundCue src="sounds/tap.mp3" at={0.5}/>
      <SoundCue src="sounds/whoosh.mp3" at={6.0}/>
      <SoundCue src="sounds/thud.mp3" at={8.5}/>
      <SoundCue src="sounds/chime.mp3" at={28.0}/>
      <SoundCue src="sounds/dissolve.mp3" at={45.0}/>
      <SoundCue src="sounds/stinger.mp3" at={52.0}/>
    </>
  );
}
```

Place `<SoundTrack/>` inside `<Stage>` alongside the scenes.
