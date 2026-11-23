# Run log

Technical notes only. No video, no song, no voice files, no weights. Times are local. Host addresses and account paths are omitted.

## Order of work

1. Install ComfyUI 0.46.0 and the pi-H3 nodes, and load the public MiniMax H3 INT8 pack plus the Lakonik 4-step adapter. A short smoke render returned a file. The history entry stores that mp4 under `outputs.images` with `metadata.kind == video`, not under a `videos` key. Treating a missing `videos` key as failure throws away a finished render. That happened on the first scene of the night clip. The file was already on disk.
2. Night clip, neon-water-001. 22 scenes, 1696x768, 24 fps, 4 steps, DWD on. Scene 1 was rendered before the batch log. Scenes 2–22 ran as one queue from 22:17:57 on 2026-11-22 to 03:23:36 on 2026-11-23. The first assembly waited from 22:18:48 and mixed the song at 03:23:49–03:24:13. The second assembly, which cuts reference jumps and stretches each scene back, finished at 03:25:17. Original scene files were not deleted.
3. While later scenes were still rendering, CPU previews of the second-assembly cut were built from the finished prefix: 5 scenes, then 8, then 12, then 13. Each preview mixed the song from its start for exactly that prefix. The renderer was not stopped for those previews.
4. Reporting on the 22-scene clip was stopped after all 22 files and both assemblies existed. Raw scene files, both assemblies, and the logs were kept.
5. On 2026-11-23 at 07:38 the owner gave a new brief to another agent. That agent wrote the prompt from the owner's public post and from the owner's words. Writing the prompt took 2 to 3 minutes. The owner then handed the whole job to this agent and said not to ask again. That job is hybrid-ai-001. The picture pass started at 07:46:03, after the night clip had already released the card. It finished at 08:20:57.

## neon-water-001, scene wall times

Scene 1 is not in the batch log. The file was written at 22:04:26 and served at 22:12:04. Sampling for that one piece was about 22 minutes.

Scenes 2–22, from the batch log. `wall_min` is that piece only, not the whole clip. Sum of those 21 pieces: 305.5 minutes.

| Scene | Done (local) | Minutes | Frames at start |
| --- | --- | --- | --- |
| 02 | 2026-11-22 22:37:23 | 19.4 | 328 |
| 03 | 2026-11-22 22:58:48 | 21.4 | 345 |
| 04 | 2026-11-22 23:14:25 | 15.6 | 294 |
| 05 | 2026-11-22 23:38:58 | 24.6 | 362 |
| 06 | 2026-11-23 00:02:33 | 23.6 | 362 |
| 07 | 2026-11-23 00:24:54 | 22.3 | 345 |
| 08 | 2026-11-23 00:40:33 | 15.6 | 294 |
| 09 | 2026-11-23 00:50:09 | 9.6 | 226 |
| 10 | 2026-11-23 01:00:25 | 10.3 | 226 |
| 11 | 2026-11-23 01:22:45 | 22.3 | 345 |
| 12 | 2026-11-23 01:33:41 | 10.9 | 243 |
| 13 | 2026-11-23 01:45:18 | 11.6 | 243 |
| 14 | 2026-11-23 01:54:54 | 9.6 | 226 |
| 15 | 2026-11-23 02:04:30 | 9.6 | 226 |
| 16 | 2026-11-23 02:28:54 | 24.4 | 362 |
| 17 | 2026-11-23 02:38:30 | 9.6 | 226 |
| 18 | 2026-11-23 02:46:14 | 7.7 | 192 |
| 19 | 2026-11-23 02:56:31 | 10.3 | 226 |
| 20 | 2026-11-23 03:04:15 | 7.7 | 192 |
| 21 | 2026-11-23 03:14:31 | 10.3 | 226 |
| 22 | 2026-11-23 03:23:36 | 9.1 | 226 |

First assembly: wait started 22:18:48, mix started 03:23:49, done 03:24:13, 89092656 bytes. Song mixed with `-shortest`. The song is not in this repo.

Second assembly: started 03:24:31, done 03:25:17, 84173327 bytes. A cut fires only when a held picture then jumps. The largest one-frame change must be at least 12, and the two sides of that change must differ by at least 18. A smooth glide is not a cut. Neither end loses more than 2.0 seconds. The clean middle is stretched back to the original duration at 24 fps. An output `-t` paired with `setpts` was tried first and discarded: it capped the file at the unstretched length. The working path is a `trim` filter, then `setpts=(PTS-STARTPTS)*factor`, then `fps=24`, with `-frames:v` pinned to the original duration times 24.

| Scene | Start cut, s | Start frames | End cut, s | End frames | Stretch | Original, s |
| --- | --- | --- | --- | --- | --- | --- |
| 01 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 15.083 |
| 02 | 1.792 | 43 | 0.000 | 0 | 1.1509 | 13.667 |
| 03 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 14.375 |
| 04 | 0.417 | 10 | 0.000 | 0 | 1.0352 | 12.250 |
| 05 | 0.000 | 0 | 1.442 | 35 | 1.1057 | 15.083 |
| 06 | 0.500 | 12 | 0.000 | 0 | 1.0343 | 15.083 |
| 07 | 0.000 | 0 | 1.275 | 31 | 1.0973 | 14.375 |
| 08 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 12.250 |
| 09 | 1.333 | 32 | 0.000 | 0 | 1.1649 | 9.417 |
| 10 | 0.000 | 0 | 0.442 | 11 | 1.0492 | 9.417 |
| 11 | 0.083 | 2 | 0.000 | 0 | 1.0058 | 14.375 |
| 12 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 10.125 |
| 13 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 10.125 |
| 14 | 0.083 | 2 | 0.000 | 0 | 1.0089 | 9.417 |
| 15 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 9.417 |
| 16 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 15.083 |
| 17 | 0.875 | 21 | 0.000 | 0 | 1.1024 | 9.417 |
| 18 | 0.000 | 0 | 1.025 | 25 | 1.1470 | 8.000 |
| 19 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 9.417 |
| 20 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 8.000 |
| 21 | 0.000 | 0 | 0.442 | 11 | 1.0492 | 9.417 |
| 22 | 0.000 | 0 | 0.000 | 0 | 1.0000 | 9.417 |

CPU previews of that second cut, song from the start, `-shortest`:

- 5 scenes, 70.458 s
- 8 scenes, 112.167 s
- 12 scenes, 155.500 s
- 13 scenes, 165.625 s

## hybrid-ai-001, picture pass

Six new people, no reference stills from the night clip. Same stack: MiniMax H3 INT8 ConvRot, pi-H3 4-step adapter, 1696x768, 24 fps, 4 steps. Length is the spoken line plus about 2 seconds, snapped up onto the `17k+5` grid.

Voices, Microsoft neural voices through edge-tts, not a local speech model:

- Scene 1, en-US-AriaNeural, 4752 ms, 175 frames, done 07:51:46, 5.7 min, 3109320 bytes
- Scene 2, en-US-AndrewNeural, 6144 ms, 209 frames, done 07:59:34, 7.8 min, 3655331 bytes
- Scene 3, en-US-EmmaNeural, 4200 ms, 158 frames, done 08:04:18, 4.7 min, 5130047 bytes
- Scene 4, en-US-AndrewNeural at +12% rate, 2952 ms, 124 frames, done 08:07:26, 3.1 min, 6657534 bytes
- Scene 5, en-US-AriaNeural, 6168 ms, 209 frames, done 08:15:14, 7.8 min, 5410095 bytes
- Scene 6, en-US-BrianNeural, 4608 ms, 175 frames, done 08:20:57, 5.7 min, 3862822 bytes

Sum of the six piece times: 34.8 minutes. Batch clock: 07:46:03 to 08:20:57.

A raw voiced roll, hard-cut, each scene shortened to its line with `-shortest`, was written at 08:21:15. 7458226 bytes. That file has the six pictures and the six spoken lines. It does not yet have lip-sync, the corner cards, or the ducked music bed. Those three steps are not in this result.

Lip-sync was planned with LatentSync 1.6, ByteDance. The checkpoint download did not finish. `whisper/tiny.pt` arrived (75572083 bytes). The request for `vae/config.json` at the published repo root returned not found, and `latentsync_unet.pt` was not on disk when this note was written. The card was free after 08:20:57. The lip-sync pass had not started.

## What is not in this repo

- The song used on the night clip
- Any rendered scene or finished clip
- The spoken voice files from the day roll
- Model weights
- Host addresses, account paths, and keys

## hybrid-lipsync-001, order of work

1. The owner rejected the day roll: the lips did not match the words. Cause: the picture model never saw the audio; the voice was muxed on top afterwards.
2. Rewrote the brief for this line: seven short lines, one speaker per scene, face straight to camera, mouth closed in the generated picture, calm face, no head turns. The picture model only supplies the body; LatentSync 1.6 moves the mouth from the speech audio.
3. Generated the seven lines at 09:12 to 09:16 with edge-tts. Downloaded the LatentSync 1.6 checkpoints (unet 5 072 222 488 bytes, whisper tiny) by 09:14.
4. Rendered seven picture pieces 09:15:00 to 09:45:58, 31.0 minutes total. Frame counts 192, 158, 141, 141, 141, 124, 158 at 24 fps.
5. First sync attempt failed at 09:47 with CUDA out of memory. The cleanup step used a process-match string that did not appear in the picture server's command line, so the server was never stopped and kept 16.2 GiB of VRAM. The sync model peaked at 7.2 GiB and failed.
6. Fixed the match string, added a VRAM drain loop that waits for an empty compute list, and added the expandable-segments allocator option. Restarted at 10:17:39. Scene 1 sync started 10:17:53.
