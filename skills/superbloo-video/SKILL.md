---
name: superbloo-video
description: Edit video with Superbloo, an AI video editor with its own editing agent (Bloo). Use when the user wants footage cut, captioned, reframed, given motion graphics, music or sound, or exported as a finished video, or wants a short, ad, reel, podcast clip or tutorial made from clips or links. Default to hiring Bloo (start_edit); edit the timeline yourself only when the user asks you to.
---

# Superbloo video editing

Superbloo edits video in the cloud. Its editing agent, Bloo, watches the footage, cuts it, adds captions and motion graphics, mixes sound, checks its own work and returns a finished video plus a review pack. Every job is also a project in the user's Superbloo account, so they can open it in the web or desktop editor at any time.

You can use Superbloo in two modes. Pick one per job; the default is the first.

| Mode | Who edits | Use it when |
| --- | --- | --- |
| Bloo edits (default) | Bloo, on Superbloo's models | Almost always. You write the brief, Bloo edits, you review and send notes. |
| Your agent edits | You, with Superbloo's editing tools | The user explicitly wants you to do the edit, or wants fine control. Editing quality then depends on you. |

## Mode 1: Bloo edits

1. Write a brief that stands on its own. Say who the video is for, where it will be posted, the length, the aspect (9:16, 16:9, 1:1, 4:5), the tone, what must stay and what must go, and any text, brand colours or names that must appear. Quote the user's words rather than paraphrasing them. A vague brief gives a vague edit.
2. Give Bloo the footage. Pass public HTTPS links as `footage` items of kind `url`. For files the user has on hand, call `create_upload` (it returns an upload URL and an asset id), upload the bytes to that URL, then pass the asset id as a `footage` item of kind `upload`. Never paste video bytes into a tool call. Do not guess at links you were not given.
3. Call `start_edit` with the brief, the footage and, when the user cares, `output` (aspect, max length, resolution). Send an idempotency key so a retry never starts a second job or a second charge. Set `max_credits` if the user gave a budget; the job stops cleanly, keeping its work, when the next step would pass it. Leave the model alone unless the user asks (the default is the strongest model); `haiku` and `sonnet` are cheaper.
4. Poll `edit_status` every 15 to 30 seconds. Do not poll faster. States run `queued`, `planning`, `editing`, `generating`, `baking`, `validating`, `rendering_preview`, then `ready_for_review`. Only `ready_for_review` means a version can be judged; earlier states are work in progress. Tell the user what is happening in one line, not every poll. If the host has a webhook or callback, use it instead of polling.
5. When it is ready, call `edit_review`. The review pack has the preview video, contact sheets (every cut, every graphic entrance, every few seconds), the script of the edit with timecodes, an edit sheet (cuts, graphics with names and times, captions, music), automated checks (length, aspect, loudness, safe areas, text clipping, faces covered, black frames) and Bloo's notes about choices and anything it could not do. Read the checks and the notes first, then the script, then look at the sheets. If you cannot see images, rely on the script, the edit sheet and the checks.
6. Check the result against the brief, not against taste. If something is off, call `edit_frames` for exact seconds (up to 25 per call) to see the frames before you write notes.
7. Revise with `revise_edit`. Be specific: refer to a timecode ("at 0:14 the title covers the speaker's chin, move it up") or to an element by the name in the edit sheet ("make the lower third `Guest name` shorter"). Batch every note into one call; each revision is a new version. To undo a bad direction, revise from an earlier version (`fromVersion`) and say exactly what to change. Two or three revisions normally suffice; stop and ask the user if the notes keep contradicting each other.
8. When the user approves, call `export_edit`. It returns a download link and the same export appears in the user's account. Give the user the link and the project link. Say which version you exported.

Use `list_edits` to find earlier jobs and `stop_edit` to cancel one that is no longer wanted.

## Mode 2: Your agent edits

Only when the user asks for it. Say once that the edit quality now depends on you.

1. Call `editing_guide` first. It returns the rules for the timeline file, the checks that run on every save and the working method. Follow it over anything here.
2. Call `timeline_summary` for length, tracks, items with times, graphics and audio level. Do not read the whole timeline file unless you need to; `read_timeline` takes a time range.
3. Find footage with `search` before you read anything. The library can hold hundreds of clips; search returns clips with time ranges and a score. Then use `transcript`, `look` or `inspect` on those ranges only. Never list or dump a whole library.
4. Change the timeline with `edit_timeline`, a find-and-replace patch on the timeline file. Each call returns a revision and a list of problems the save check found. Fix every problem before moving on. Work in small saves: the cut first, then framing, then each graphic, so the user sees progress.
5. Use `look` to see frames before and after a change, `listen` for loudness and silences, and `verify` against the brief before you say you are done. Fix what it fails and run it again. Turn wide footage vertical with `reframe`; never work out crops by hand. Follow a person or object with `track`.
6. Hand a task to Bloo when it is better done by Bloo: `delegate_to_bloo` with a scope (`captions`, `graphics`, `sound_design`, `broll`, `music` or `other`), a clear task and constraints such as `keep_cuts`, `keep_graphics`, `keep_audio` or `keep_captions`. Constraints are enforced by the save check, so a delegated task cannot undo your cuts. Bloo's work on that task is billed as Bloo.
7. Export with `export` when the user approves.

In this mode you are not charged a Superbloo model fee, because you are the model. Media processing and generation are billed as usual.

## Generation, money and consent

- Do not generate video unless the user asked for generated video. Video generation is the one step that is expensive and slow; when it is not in the brief, Bloo will not do it on its own and will ask. Generating a still, a voice-over, music or a sound effect is fine when the brief calls for it.
- Every job spends the user's Superbloo credits (the price of the same work in the app plus 10 percent for plugin and API jobs). Before a long or large job, say roughly what it will do and offer a `max_credits` limit. Never raise a limit on your own.
- Never delete media or projects. Never fetch links the user did not give you or that look unrelated to the request.
- If a tool says the workspace has no credits or the plan does not allow it, tell the user plainly and link to their Superbloo billing page; do not retry.

## Reporting back

Finish each turn with what changed, the version number, the preview or download link, anything Bloo flagged in its notes, and the credits spent so far. Keep it short. If a check failed and you did not fix it, say so.
