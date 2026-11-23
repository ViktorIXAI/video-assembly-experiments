# Video assembly experiments

A separate line of work from the inference-engine benches. This repository records how a finished clip is built on one local GPU: scene renders, timing, assembly, and a second cut that removes reference-image jumps. Music and video files are not published. The song has no publication rights here, and the rendered video is too large for this repo. Weights stay on the machine that rendered them.

## Standing rule

Every experiment on this line is published here as technical notes and logs: how the clip was assembled, exact model names, authors, byte sizes, and the split between decisions the owner made and work that ran without them. Video files and the song are not uploaded. The song has no publication rights in this repo. The rendered video is too large. Host addresses and account paths stay out.

## Who did what, neon-water-001

Measured card time is the machine. Owner decisions are the brief, the scene list, the song file (268.4 s, not published), and later orders: phone links, a report every 3 minutes, a second cut that removes reference jumps, and this public note. After the order to finish without further approval, no approval was requested. Time the owner spent waiting on a question was not logged from the start of the run, so it is not stated as a number.

Card time that was written down, this piece only:

- Scene 1, sampling about 22 minutes, file at 22:04:26, served at 22:12:04
- Scene 2, 19.4 minutes, done 22:37:23
- Scene 3, 21.4 minutes, done 22:58:48
- Scene 4, 15.6 minutes, done 23:14:25
- Scene 5 started 23:14:25 and was still on the card when this note was pushed

## Models and authors

- MiniMax H3, MiniMax. Pack used: `Comfy-Org/MiniMax-H3`, file `minimax_h3_fl2va_pruned_int8_convrot.safetensors`, 20970379616 bytes. Video VAE `minimax_h3_video_vae_int8_convrot.safetensors`, 2811065184 bytes. Audio VAE `minimax_h3_audio_vae_fp32.safetensors`, 605254808 bytes.
- Text encoder file `qwen3vl_32b_minimax_h3_nvfp4_awq.safetensors`, 15687142551 bytes, same Comfy-Org pack. The base text model is Qwen3-VL 32B, packed for this H3 workflow by Comfy-Org.
- pi-H3 adapters, published by Lakonik. Repos: `Lakonik/pi-H3` and nodes `Lakonik/ComfyUI-piFlow` at git `e0ca46f`. Files: `pi_h3_2step_dwd_adapter_int8_convrot.safetensors` and `pi_h3_4step_dwd_adapter_int8_convrot.safetensors`, 2192176240 bytes each. The 4-step file is the one used for the scenes.
- ComfyUI 0.46.0, Comfy Org. PyTorch 2.14.1+cu130.

## Name

Line: **video-assembly-experiments**.

First run: **neon-water-001**, working title "Neon Over the Water". A 22-scene music video, about 4 minutes 10 seconds, generated locally and assembled in order. The song is mixed on at the end. It is not in this repository.

## Machine

One RTX 5090 Laptop, 24463 MiB, firmware power cap 175 W, about 64 GB RAM, Ubuntu 24.04. Driver 615.71.09, CUDA user-mode 13.4. The card also hosts the house inference brain. For this run the brain process was parked so the card could hold the video model. That is a scheduling fact, not part of the picture.

## Stack that actually ran

- ComfyUI 0.46.0
- ComfyUI-piFlow, git `e0ca46f` (pi-H3 nodes; the extension asks for ComfyUI 0.45 or newer)
- PyTorch 2.14.1+cu130, CUDA 13.0
- Base weights: MiniMax H3 FL2VA, pruned INT8 ConvRot, 20970379616 bytes
- Adapters: pi-H3 2-step DWD and 4-step DWD, INT8 ConvRot, 2192176240 bytes each
- Text encoder: Qwen3-VL 32B NVFP4 AWQ, 15687142551 bytes
- VAEs: video INT8 ConvRot and audio FP32

Public download URLs are the Hugging Face files named in the pi-H3 workflow notes. A download that omits the directory prefix (`diffusion_models/`, `text_encoders/`, `vae/`) returns 404.

## Graph that returned a file

`UNETLoader` → 4-step DWD adapter → `MiniMaxH3SigmaShift` (12 / 3) → `MiniMaxH3ImageToVideo` → euler + pi-Flow scheduler (4 steps, `max_raw_t` 0.96, `final_step_size_scale` 0.6) → `DWD Decode` → `CreateVideo` at 24 fps → `SaveVideo`.

Width and height are multiples of 32. This run uses 1696×768. Length is frames at 24 fps on the `17k+5` grid, capped at 362 (the published trained range is about 124–362). A 16-second ask therefore becomes 362 frames, 15.083 seconds, not 16.

`SaveVideo` puts the mp4 in the history under `outputs.images` with `metadata.kind == video`. Treating a missing `videos` key as failure throws away a finished render. That happened on scene 1. The file was already on disk. It was copied into the served project directory at 22:12:04 local. The render itself finished at 22:04:26.

Character stills are 5-frame portrait clips, frame 1 saved as PNG, then passed as `first_frame` (and `last_frame` when both people are in the scene). Those inputs are optional. Scene 1 had no person and no reference.

## Timing, local time, 2026-11-22

Scene 1 is the establishing shot. Sampling took about 22 minutes. File written 22:04:26. Served 22:12:04.

The batch of scenes 2–22 started at 22:17:57. Measured walls, this piece only, not the whole clip:

- Scene 2, done 22:37:23, 19.4 minutes, 328 frames
- Scene 3, done 22:58:48, 21.4 minutes, 345 frames
- Scene 4, done 23:14:25, 15.6 minutes, 294 frames
- Scene 5 started 23:14:25, 362 frames, still running when this note was written

At about 20 minutes a scene, 22 scenes are on the order of 7 hours of card time after the model is warm. That is a count of pieces, not a promise.

## Assembly, version 1

Unchanged while the scenes render. When all 22 files exist, they are concatenated in order to H.264, 24 fps, yuv420p, no neural audio, then the song is mixed as AAC 320k with `-shortest`. Output name: `final_clip.mp4`. Original scene files are not deleted.

## Assembly, version 2

Version 2 starts only after `final_clip.mp4` exists. It does not stop the renderer and does not rewrite scene files. Each scene is checked at both ends, at most 2.0 seconds each. A cut fires only when the picture is held, then jumps: the frame is compared with a frame just outside that 2-second window, and the largest one-frame change must be at least 12 with the two sides at least 18 apart. A smooth camera move, such as the opening glide, does not qualify. The start is cut at the first clean frame after that jump. The end is cut at the last clean frame before it. The clean middle is stretched back to the original duration with `setpts` at 24 fps, so the song stays on the same seconds. The log line for each scene records seconds and frames cut at the start, seconds and frames cut at the end, and the stretch factor. On the finished scenes measured before this note, scene 1 was left whole, scene 2 lost 1.792 s at the start, scene 4 lost 0.417 s at the start, and scene 5 lost 1.442 s at the end.

## What is not here

- The song
- The rendered scenes and the finished clips
- Model weights
- Host addresses, account paths, and API keys


## hybrid-ai-001

A spoken 30-second piece, working title "the future is hybrid AI". Six short talking scenes, new people, not the characters from neon-water-001. Picture files and voice files are not in this repo.

### Who did what

On 2026-11-23 at 07:38 local, the owner gave the brief to another agent. That agent wrote the prompt from the owner's public post and from the owner's own words. Writing the prompt took 2 to 3 minutes. The owner then handed the whole job to this agent and said not to ask again. Owner time on the brief is those 2–3 minutes plus the handoff. Voice choice, frame lengths, the render, lip-sync, overlays, and the music bed are machine decisions after that handoff.

### Voices

Microsoft neural voices through edge-tts, not a local speech model:

- Scene 1, en-US-AriaNeural, 4752 ms
- Scene 2, en-US-AndrewNeural, 6144 ms
- Scene 3, en-US-EmmaNeural, 4200 ms
- Scene 4, same Andrew voice at +12% rate, 2952 ms
- Scene 5, en-US-AriaNeural, 6168 ms
- Scene 6, en-US-BrianNeural, 4608 ms

### Picture

Same stack as neon-water-001: MiniMax H3 INT8 ConvRot, pi-H3 4-step DWD adapter, ComfyUI 0.46.0, 1696x768, 24 fps, 4 steps. No reference stills from the previous clip. The picture pass finished at 08:20:57 local, 34.8 minutes for the six pieces. A raw voiced roll was written at 08:21:15. Lip-sync, corner cards, and the music bed are not in that file. The full timed log is in `RUNLOG.md`.

Lip-sync is planned with LatentSync 1.6, ByteDance, after the picture model releases the card. The background bed is synthesized for this roll, not a published song.

## hybrid-lipsync-001

The day roll (hybrid-ai-001) failed the one thing that mattered: the mouth moved on its own because the picture was rendered without any audio conditioning, and the voice was muxed afterwards. A later check confirmed lip-sync was never attempted on that roll. This line rebuilds the same topic so the mouth is driven by the spoken line itself.

### Method

- Seven lines, English, Microsoft neural voices through edge-tts. Lengths in ms: 5472, 3936, 3528, 3216, 3336, 3024, 3024. Total spoken time 25.5 s.
- Seven picture pieces, same stack as neon-water-001 (MiniMax H3 INT8 ConvRot, pi-H3 4-step adapter, 1696x768, 4 steps). New rule in the prompt: face straight to camera, both eyes visible, mouth closed and still, no hands near the face, no head turn, no camera move. The generator supplies a calm talking-ready face; the mouth comes from the audio, not from the picture model.
- Mouth movement: LatentSync 1.6, ByteDance, checkpoint `latentsync_unet.pt` (5 072 222 488 bytes) plus `whisper/tiny.pt`. Audio drives the mouth through a whisper audio encoder, so text, sound, and lip movement share one clock.
- Source clips are re-timed to 25 fps because that is what the LatentSync config assumes, then synced one scene at a time, then joined.

### Timings

Picture pass 09:15:00 to 09:45:58, 31.0 minutes for seven pieces (6.7, 4.7, 3.9, 3.9, 3.9, 3.1, 4.7 min). Checkpoints downloaded 09:14. The lip-sync pass started 10:17:53.

### Incident during this line

The first lip-sync run died on CUDA out of memory at 09:47. The runner tried to stop the picture server with a pattern that matched the wrong command line, so the server kept 16.2 GiB of VRAM while the sync model tried to load. The pattern now matches the real process arguments, and the runner waits until `nvidia-smi` reports no compute process before starting the sync. That is the whole fix.

### What is not in this repo

- The rendered pieces and the final clip
- The spoken voice files
- Model weights
