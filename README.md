# Camera-handoff AI video: Blender blocking + your phone + Seedance

How to make a 15-second, one-take AI video in which a rider grabs the camera from the roadside without stopping and turns it into a selfie, and how to present it as a vertical reel that shows how it was made. You block the shot in Blender with simple stand-ins, perform the camera move live with your phone, and let a reference-to-video model turn the result into real people in a real place.

The idea and the method come from [mrksh.ai](https://t.me/code_viberr/120) ([original guide](https://mrkshh.notion.site/3eb81bc96083813d8894f0722710521e)). This write-up documents a full recreation with two riders (a parent on a longboard and a child on inline skates). It covers the settings, prompts, costs and fixes that worked, and the things that didn't.

<!-- Add a preview of your reel here, e.g. ![Result](media/preview.gif) -->

## Contents

1. [The result](#the-result)
2. [How it works](#how-it-works)
3. [What you need](#what-you-need)
4. [Step 1: Plan the shot](#step-1-plan-the-shot)
5. [Step 2: Block it in Blender](#step-2-block-it-in-blender)
6. [Step 3: Perform the camera with your phone](#step-3-perform-the-camera-with-your-phone)
7. [Step 4: Prepare the references](#step-4-prepare-the-references)
8. [Step 5: Write the prompt](#step-5-write-the-prompt)
9. [Step 6: Test on a 5-second slice](#step-6-test-on-a-5-second-slice)
10. [Step 7: Generate with Seedance 2.5](#step-7-generate-with-seedance-25)
11. [Step 8: Assemble the reel](#step-8-assemble-the-reel)
12. [Step 9: Put the music drop on the grab](#step-9-put-the-music-drop-on-the-grab)
13. [Costs and timings](#costs-and-timings)
14. [Lessons learned](#lessons-learned)
15. [Privacy, consent and safety](#privacy-consent-and-safety)
16. [Credits](#credits)

## The result

- **The shot:** one continuous 15-second take, 16:9, 24 fps.
  - A camera stands low at the edge of an empty road while two riders approach.
  - At 5 seconds the lead rider grabs it in passing and films a selfie at arm's length, with the second rider skating behind.
  - At the end the second rider pulls up alongside.
- **The reel:** 1080×1920, three 16:9 panels stacked like the original post.
  - Top: the AI shot.
  - Middle: the Blender camera view.
  - Bottom: you moving the phone in front of your laptop.
  - Music, with its drop landing exactly on the grab.

## How it works

1. **Block the shot in Blender.** Low-poly road, pillar stand-ins for the people, and one arm for the rider who grabs the camera.
2. **Perform the camera with your phone.** The phone streams its motion into the Blender camera while the timeline plays, and you record it. This hand-held motion is what makes the result feel real.
3. **Render the camera view.** That render is the reference video.
4. **Generate with a reference-to-video model.** Seedance 2.5 takes the reference video for the camera and timing, plus images of the place and the people and a prompt.
5. **Assemble the reel with ffmpeg.** Sync your side-view footage to the render, stack the three panels, and line the music's drop up with the grab.

## What you need

| What | Used for | Cost (Nov 2026) |
|---|---|---|
| [Blender](https://www.blender.org) 5.x | Blocking, camera, Workbench render | Free |
| Omniscient (iPhone app with a Blender link) | Streaming and recording the phone's motion on the Blender camera (you pair it by scanning a QR code that Blender shows) | Check the app |
| A second phone or camera | Filming yourself operating the phone (the bottom panel) | — |
| Seedance 2.5 reference-to-video on the [Yotta AI Gateway](https://www.yottalabs.ai/ai-gateway) | Generating the video | $0.08/s at 480p, $0.15/s at 720p, billed on reference + output seconds |
| Temporary public file hosting (for example Cloudflare Pages) | The API only accepts links to the reference files | Free |
| ffmpeg, Python 3 with numpy, scipy and soundfile | Reel assembly, sync and music analysis | Free |

Our whole run cost **$6.82**: three paid Seedance jobs, including two tests.

## Step 1: Plan the shot

| Time | Camera | Action |
|---|---|---|
| 0–5 s | Static, low at the left edge of the road; quick zoom in at about 1 s and back out at about 3 s | Both riders skate toward the camera |
| ~4.7 s | — | The lead rider reaches toward the lens with the right hand |
| 5.0 s | Grabbed | Picked up on the go; the view lurches up and swings to the rider's face |
| 5–15 s | Selfie at arm's length, about neck height | Face, then the road ahead (about 7.5–9 s), then the face with the second rider behind, then the second rider alongside as the camera dips at the end |

Work in 1280×720 at 24 fps: 15 s is 360 frames.

## Step 2: Block it in Blender

- **Set:** a low-poly road with lane markings, fields and mountains. It only has to read as a layout.
- **People:** plain pillars with a ball head, standing on a board.
  - Give each person a colour: mustard for the rider who grabs the camera, teal for the other.
  - Use real heights and a believable speed (about 18 km/h).
  - The model uses the pillars only for position and scale, and the prompt says so.
- **The arm (important):** give the lead rider one right arm, a thin box with a ball hand.
  - It reaches for the camera at about 4.65 s, closes on it at 5 s, then holds it out.
  - In selfie frames it shows as a wedge in the bottom-left corner. The model turns that wedge into a real outstretched arm, so it understands who is holding the camera. The original reel did the same.
- **Arm rig:**
  - Drive the hand with a Copy Location constraint to a grip point on the camera, and the arm with Stretch To from the shoulder.
  - Compute the reach and lift as arcs in the rider's own frame, so the arm keeps a sane length (about 0.5–0.65 m) instead of stretching or folding.
  - Keep it thin (about 9 cm) with a small hand (about 6.5 cm) so it doesn't cover the lens.
- **Camera:** use a carrier object that sits at the roadside "tripod" spot until the grab and then rides along with the lead rider at the selfie hold. The camera copies the carrier's location, and your phone supplies the rotation.
- **Zoom:** keep the lens zoom keyed in Blender. In our setup, live lens changes from the phone were not recorded.
  - In Blender 5, the recorded rotation keys and the lens keys can end up in the same action, so check that both survive.

## Step 3: Perform the camera with your phone

1. Pair the phone with Blender (the Omniscient link: scan the QR code shown in Blender).
2. Hold the phone level in landscape and don't pinch-zoom.
3. **Record a 2-second test** and check it: every frame keyed, no frozen samples, a plausible rotation sweep.
4. **Record the full 15 seconds** while the timeline plays, and act it out.
   - Hold the phone low and still for 5 s, as if it stood on a tripod.
   - "Grab" it at 5 s.
   - Then film the selfie: face, road, companion, and a dip at the end.
5. **Keep the take as performed.** We tried a tidied version with a locked tripod opening and a held ending, and it looked fake. The raw take, with its slightly hand-held opening and natural dip at the end, read as real. Only fill in dropped samples, the few frames where the phone stream froze (18 of 360 in our take).
6. **Film yourself doing it** from the side with a second phone, with the laptop screen in view. That footage becomes the bottom panel.
7. **Render the camera view:** Workbench, 1280×720, 24 fps, about 1.2 s per frame. This is `reference.mp4`.

## Step 4: Prepare the references

- **Location:** one image of the road and landscape you want (`@Image1`).
- **People:** one sheet per person: front, back and a face close-up, with the outfit and the board or skates (`@Image2`, `@Image3`).
- **Expression photos (don't skip these):** 2–3 real photos of the lead rider smiling and laughing, cropped to the face (`@Image4` onward).
  - Without them, the model invented an exaggerated, eyes-squeezed laugh.
  - With them, the smile looked like the real person.
  - Use real photos, not AI-edited ones: an edited laugh is a guess that the model copies, and edits tend to shift the likeness.
- **Remove location and other metadata** from every photo before it goes online, for example `exiftool -all= photo.jpg`, or re-save the image without EXIF.

## Step 5: Write the prompt

The rules that made the difference:

- **Reference the uploads by type and order:** `@Video1` for the clip, `@Image1…` for the images in upload order.
- **Explain the stand-ins:** the pillars only mark positions, and the mustard box with a ball is the arm.
- **Never mention equipment in the action.**
  - When the prompt said a phone stood on a tripod, Kling drew a tripod by the road, and later a phone on a grip in the rider's hand.
  - Describe the camera itself instead ("the camera stands still, low at the left edge of the road… he picks up the camera"), and add one line saying no filming equipment is ever visible.
- **Ask for small, natural expressions.**
  - "His expression keeps changing, never frozen" cured a frozen face in Kling but made Seedance overact.
  - "He laughs… squints in the wind" produced a grimace.
  - What worked: a relaxed smile with open eyes, short quiet laughs, plus the real smile photos.
- **Sound:** ask for the scene's own sound only (wind, wheels, short laughs) and no music or words. The music goes in later.

Full 15-second template. Replace the bracketed parts; our version was about 2,550 characters:

```text
Use @Video1 as the strict reference for the camera: its position, every movement, rotation and zoom, the timing, the moment the camera is picked up, and where each person is in the frame. The coloured pillars and spheres in @Video1 only mark people's positions; do not copy their shapes or stiff poses. The mustard box with a ball at its end is Dad's right arm and hand.

Photorealistic shot filmed on a phone, on the empty [country] road from @Image1: [road surface, lane markings, landscape, mountains, weather, light].

Replace the two pillars with this father and daughter:
- Mustard pillar: Dad, the man from @Image2: [hair, top, layer underneath, trousers, shoes, board]. @Image4 and @Image5 are real photos of him smiling and laughing: use them only for his face and expressions.
- Short teal pillar: his daughter from @Image3: [hair, top, trousers, skates].

0-5 s: the camera stands still, low at the left edge of the road, filming Dad and his daughter as they skate toward it; it moves only slightly, as in @Video1. It zooms in on them at about 1 s and back out at about 3 s.
At about 4.7 s Dad reaches straight toward the lens with his right hand, and at 5 s he picks up the camera as he rides past, without stopping. The view lurches up and swings to his face.
5-15 s: he films himself, holding the camera at arm's length at neck height. While the camera faces him, his outstretched right arm in the [colour] sleeve shows at the bottom-left edge. Follow @Video1: his face; at about 7.5-9 s the road ahead, only the empty road; his face with his daughter skating behind him; his face; then his daughter alongside, as the camera dips at the end.

His face stays relaxed and natural, as in a casual selfie: a warm, easy smile with his eyes open, short, quiet laughs that fade back into the smile, calm glances at the road and at his daughter; small, true-to-life expressions, never an exaggerated or held grin. His smile looks like his own in @Image4 and @Image5. His daughter smiles and keeps up with him.

One continuous shot, no cuts. No filming equipment (camera, phone, tripod or stand) is ever visible in the frame. Wind in hair and clothes, spinning wheels, realistic handheld movement. Faces and clothes stay the same. Exactly two people. Sound: wind, wheels on asphalt and short, soft laughs; no music, no words, no captions.
```

Adapt the people, pronouns and colours to your cast.

## Step 6: Test on a 5-second slice

The hardest moment is the reach, the grab and the swing to the selfie. Cut 3.5–8.5 s from the reference and test that before paying for 15 seconds:

```sh
ffmpeg -ss 3.5 -t 5 -i reference.mp4 -an -c:v libx264 -crf 16 -r 24 -pix_fmt yuv420p probe.mp4
```

Shift the prompt's times to the slice: static until 1.2 s, reach at 1.2 s, grab at 1.5 s, then the selfie and the road. Our sequence:

1. **Kling 4.0, Omni reference mode, free daily credits.** The app's default "Keyframes" mode ignores a reference video. Kling got the likeness, the grab and the selfie arm right, but the face stayed static and it hallucinated a tripod and a phone grip.
2. **Seedance 2.5, 480p, $0.80.** Far more realistic, with no equipment anywhere. The laugh was exaggerated.
3. **Seedance 2.5, 720p, calmer prompt and 3 real smile photos, $1.51.** Approved. Then the full 15 s at 720p, $4.51.

## Step 7: Generate with Seedance 2.5

- **Why this route:** in November 2026 the standard Seedance 2.5 routes rejected real faces in references. Yotta announced that its [reference-to-video endpoint accepts real people](https://www.yottalabs.ai/post/seedance-2-5-api-reference-to-video-real-people-pricing), with permission for every likeness. Check the current terms before you start.
- **The API only takes links.** Inline or base64 uploads are refused, so put the files on a temporary public site under random names, and delete it when you're done. With Cloudflare Pages:

```sh
npx wrangler pages project create tmp-$RANDOM --production-branch main   # note the name it prints
npx wrangler pages deploy ./upload --project-name <that-name>
# ...after the runs:
npx wrangler pages project delete <that-name>
```

**Submit, poll and download:**

```sh
curl -X POST https://gateway.yottalabs.ai/api/maas/reference-to-video/generations \
  -H "X-API-KEY: $YOTTA_API_KEY" -H "Content-Type: application/json" \
  -d '{
    "model": "seedance-2.5-r2v",
    "parameters": {
      "prompt": "…your prompt…",
      "reference_items": [
        {"url": "https://<site>/v-1a2b3c.mp4", "type": "video"},
        {"url": "https://<site>/i-4d5e6f.jpg", "type": "image"},
        {"url": "https://<site>/i-7a8b9c.png", "type": "image"},
        {"url": "https://<site>/i-0d1e2f.png", "type": "image"},
        {"url": "https://<site>/i-3a4b5c.jpg", "type": "image"},
        {"url": "https://<site>/i-6d7e8f.jpg", "type": "image"}
      ],
      "duration": 15, "resolution": "720p", "aspect_ratio": "16:9", "generate_audio": true
    }
  }'
# -> {"code": 10000, "data": {"request_id": "…", "status": "queued"}}

curl -s -H "X-API-KEY: $YOTTA_API_KEY" \
  https://gateway.yottalabs.ai/api/maas/reference-to-video/generations/<request_id>
# poll until data.status == "completed", then download data.output_url (the link expires after about a day)
```

- **Billing:** you pay for the reference clip's seconds as well as the output's. For example, 5 s of reference plus 5.06 s of output at 480p came to $0.804, and 15 s plus 15 s at 720p came to $4.51.
- **Balance:** a 720p request was refused with $5 on the account (code `16001`, "Don't have enough balance") even though it would have cost about $1.50. 480p went through. Keep about $10 free for 720p jobs. Refused requests aren't charged.
- **Resolution:** 720p is enough, because each panel of the reel is only 1080×608.
- **Time:** 2.5–4.5 minutes per job.
- Keep the API key in an environment variable, never in the repository.

## Step 8: Assemble the reel

**Layout, measured on the original:**

| Part | Size |
|---|---|
| Top bar | 51 px |
| Top panel | 608 px |
| Middle panel | 609 px |
| Gap | 8 px in the original; we removed it |
| Bottom panel | 608 px |
| Bottom bar | Whatever is left (44 px without the gap) |

**Sync your side-view footage.** The laptop in that footage shows the live Blender viewport, so you can find where the take starts by matching the screen against the render. This script compares the share of yellow pixels (the lead rider's colour) frame by frame and picks the best offset:

```python
# sync_offset.py: python3 sync_offset.py reference.mp4 operator.mov <w:h:x:y of the laptop screen>
import subprocess, sys
import numpy as np

def frames(path, vf, w, h):
    raw = subprocess.run(['ffmpeg', '-v', 'error', '-i', path, '-vf', f'{vf},fps=24,scale={w}:{h},format=rgb24',
                          '-f', 'rawvideo', '-'], capture_output=True, check=True).stdout
    return np.frombuffer(raw, np.uint8).reshape(-1, h, w, 3).astype(float) / 255

def yellow(rgb):
    r, g, b = rgb[..., 0], rgb[..., 1], rgb[..., 2]
    return ((r > 0.45) & (g > 0.35) & (b < 0.6 * g) & (r - b > 0.25)).mean(axis=(1, 2))

render, operator, crop = sys.argv[1:4]
a = yellow(frames(render, 'null', 160, 90))
b = yellow(frames(operator, f'crop={crop}', 160, 120))
r, k = max((np.corrcoef(a, b[k:k + len(a)])[0, 1], k) for k in range(len(b) - len(a) + 1))
print(f'take starts {k / 24:.3f} s into the operator video (correlation {r:.2f})')
```

Ours found 0.542 s with a correlation of 0.84.

**Stack the panels and mix the sound.** Composite in 4:4:4 (`yuv444p`) and convert to 4:2:0 once at the end. If you overlay in 4:2:0, the odd panel edges (51, 659) get rounded and you see thin black lines between the panels.

```sh
ffmpeg -i ai.mp4 -i reference.mp4 -ss 0.542 -i operator.mov -ss 105.68 -i song.mp3 -filter_complex "
[0:v]fps=24,scale=1080:608:force_original_aspect_ratio=increase,crop=1080:608,setsar=1,format=yuv444p[top];
[1:v]fps=24,scale=1080:609:force_original_aspect_ratio=increase,crop=1080:609,setsar=1,format=yuv444p[mid];
[2:v]fps=24,scale=1080:608:force_original_aspect_ratio=increase,crop=1080:608,setsar=1,format=yuv444p[bot];
color=black:s=1080x1920:r=24:d=15,format=yuv444p[bg];
[bg][top]overlay=0:51:format=yuv444[l1];
[l1][mid]overlay=0:659:format=yuv444[l2];
[l2][bot]overlay=0:1268:format=yuv444,format=yuv420p[v];
[3:a]aresample=48000,volume=-8.3dB,afade=t=in:d=0.4[music];
[0:a]aresample=48000,volume=3dB[sfx];
[music][sfx]amix=inputs=2:duration=first:normalize=0,alimiter=limit=0.891:level=false,afade=t=out:st=14.2:d=0.8[audio]" \
  -map "[v]" -map "[audio]" -t 15 -r 24 -c:v libx264 -preset veryfast -crf 18 -pix_fmt yuv420p \
  -c:a aac -b:a 192k -movflags +faststart reel.mp4
```

What the options do:

| Option | What it does |
|---|---|
| `-ss 0.542` before the operator video | Starts that footage where the take starts |
| `-ss 105.68` before the song | Picks the part of the song (see Step 9) |
| `volume=-8.3dB` | Brings that song to about −15 LUFS. Measure yours with `ffmpeg -ss <start> -t 15 -i song.mp3 -af ebur128 -f null -` and adjust |
| `volume=3dB` on `[0:a]` | Keeps the model's wind, wheels and laughs audible under the music |
| The limiter | Keeps peaks under −1 dBFS |

## Step 9: Put the music drop on the grab

The grab is the culmination: the frame where the hand closes over the lens. In our clip it is at 5.00 s. The song's drop should land exactly there, with its build filling the approach before it.

1. **Look at the whole song.** The drop is where the bright low band (kick and bass) comes back after a darker stretch (the build), leading into the loudest part.

   ```sh
   ffmpeg -i song.mp3 -lavfi "showspectrumpic=s=1600x420:scale=log:fscale=log" spectrum.png
   ```

   Zoom in on a candidate with `-ss <start> -t 20` before `-i`.

2. **Find the exact hit**, to a few milliseconds:

   ```python
   # refine_drop.py: python3 refine_drop.py song.wav <approximate drop time in s>
   import sys
   import numpy as np, soundfile as sf
   from scipy.signal import butter, sosfilt
   path, approx = sys.argv[1], float(sys.argv[2])
   y, sr = sf.read(path, always_2d=True)
   low = sosfilt(butter(4, 120, 'lowpass', fs=sr, output='sos'), y.mean(axis=1))
   seg = low[int((approx - 0.6) * sr):int((approx + 0.6) * sr)]
   n = int(0.003 * sr)
   env = 20 * np.log10(np.sqrt(np.convolve(seg ** 2, np.ones(n) / n, 'same')) + 1e-9)
   base = np.median(env[:int(0.3 * sr)])
   print(f'drop at {approx - 0.6 + np.argmax(env > base + 15) / sr:.3f} s')
   ```

   Convert the song to WAV first: `ffmpeg -i song.mp3 song.wav`.

3. **Start the song at drop time minus grab time.** Then add a short fade-in (0.4 s) and a fade-out (0.8 s), because you are cutting into the middle of the song.

Our two picks:

| Song | Drop chosen | Song starts at | Why |
|---|---|---|---|
| The Flaming Lips, *The Yeah Yeah Yeah Song* | Bass hit at 2:55.9, after the quiet breakdown and a 2 s build | 2:50.9 | The song's biggest moment, and the final chorus plays under the selfie |
| Ty Dolla $ign, *Blasé* (Louis the Child Remix) | Second drop, bass hit at 1:50.68 | 1:45.68 | The remix has two identical drops; the second has a 3.7 s build against 1.9 s, which fills the approach |

Songs are not included here. For anything you post publicly, use your platform's licensed music library or music you have the rights to.

## Costs and timings

| Job | Settings | Cost | Time |
|---|---|---:|---:|
| Kling 4.0 Omni reference, 5 s test | 720p | free daily credits | — |
| Seedance 2.5 R2V, 5 s test | 480p | $0.80 | 151 s |
| Seedance 2.5 R2V, 5 s test (calmer prompt, smile photos) | 720p | $1.51 | 263 s |
| Seedance 2.5 R2V, final 15 s | 720p | $4.51 | 224 s |
| Refused requests (format errors, balance) | — | $0.00 | — |
| **Total** | | **$6.82** | |

## Lessons learned

1. **The phone take is what makes it work.** Perform it like real camerawork and keep the imperfections.
2. **Give the grabbing rider an arm in the blocking.** The model turns the wedge in the corner into a real arm holding the camera.
3. **Never write "tripod" or "phone" into the action.** Describe the camera instead, and add one "no filming equipment is visible" line.
4. **For faces:** use a calm expression prompt, real smile photos and 720p. One bad word ("squints") produced a grimace.
5. **Test the hardest 5 seconds first,** at 480p (under $1), before paying for 15 seconds.
6. **On Yotta:** the API takes links only, you pay for the reference seconds too, and keep about $10 free for 720p.
7. **720p is plenty** for a three-panel vertical reel.
8. **For the reel:** composite in 4:4:4, sync the side-view footage by matching the laptop screen, and land the music drop on the grab with a build that fills the approach.

## Privacy, consent and safety

- Get consent from everyone who appears in the video. For children, that means their parents or guardians.
- Don't edit, stylise or disguise faces to get past a provider's real-person checks. If a reference is refused, stop and choose another route.
- Remove location data from photos, keep the reference files online only while the jobs run, and delete the temporary site afterwards.
- Keep API keys out of code, prompts and repositories.
- Use only footage and music you have the rights to.

## Credits

- **Idea and method:** [mrksh.ai](https://t.me/code_viberr/120) and the [original guide](https://mrkshh.notion.site/3eb81bc96083813d8894f0722710521e).
- **Tools:**
  - [Blender](https://www.blender.org)
  - Omniscient (phone camera link)
  - Seedance 2.5 by ByteDance, via the [Yotta AI Gateway](https://www.yottalabs.ai/ai-gateway)
  - Kling (tested)
  - [ffmpeg](https://ffmpeg.org)
  - numpy, scipy, soundfile
