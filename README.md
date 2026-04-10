# Daily cloud recording: local video + all audio (Tavus)

**Repo:** [github.com/andy-tavus/daily-recording-single-participant-video-all-audio](https://github.com/andy-tavus/daily-recording-single-participant-video-all-audio)

Minimal **`daily-js`** demo: **one cloud recording** with **only the human’s camera** in the composite and **everyone’s audio** (human + Tavus replica) in the same file.

## Run the demo

Open `index.html` in a browser (or serve the folder with any static server). Enter a Daily room URL where **cloud recording** is enabled, then **Join & Start Recording**.

## Who this is for

Same integration shape as **Tavus** conversations:

- One **human** in the browser (**`participants().local`**).
- One **Tavus replica** remote participant (`user_id` contains `tavus-replica`).

## Implementation (match this exactly)

Use **`Daily.createFrame`** (or equivalent) and **`join({ url })`** like this repo.

### Before `startRecording`

1. Wait until **`call.participants().local.session_id`** exists (poll ~50ms; timeout e.g. 20s).
2. If **`local.user_id`** matches the replica pattern (substring **`tavus-replica`**), **do not** record — the wrong client is “local.”

### `startRecording` payload

**Important:** use **flat dotted keys** in **`composition_params`** (e.g. `'videoSettings.preferredParticipantIds'`), **not** nested `videoSettings: { … }`. That matches what Daily’s compositor expects.

```js
await call.startRecording({
  type: 'cloud',
  layout: {
    preset: 'custom',
    composition_id: 'daily:baseline',
    composition_params: {
      mode: 'single',
      'videoSettings.preferredParticipantIds':
        local.user_id && local.user_id !== local.session_id
          ? `${local.session_id},${local.user_id}`
          : String(local.session_id),
      'videoSettings.preferScreenshare': false,
    },
    session_assets: {},
  },
  participants: {
    video: [local.session_id],
    audio: /* every participant session_id from call.participants(), deduped — or ['*'] if empty */,
  },
});
```

- **`participants.video`**: **only** **`[local.session_id]`** — never the replica’s session id.
- **`participants.audio`**: all session ids in the call (human + replica), or **`['*']`** if you have no ids yet.
- With typical Tavus/human tokens, **`local.user_id === local.session_id`**, so **`preferredParticipantIds`** is a **single** id string.

### Do **not** use

- **`layout.preset: 'single-participant'`** — it pins **both** video **and** audio to one participant, so you lose “all audio.”

### Debug

The page logs **`[Daily] startRecording JSON`** — copy that object when talking to Daily support.

## Prerequisites

- Daily **cloud recording** enabled for the room or domain.
- **`@daily-co/daily-js`** (this demo loads it from `unpkg` in `index.html`).

## License

Use freely for integration reference.
