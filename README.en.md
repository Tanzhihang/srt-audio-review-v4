# SRT Audio Review V4

**Chinese subtitle proofreading across text, semantics and audio timing.**

Created and open-sourced by **[谭智航 / Tanzhihang](https://github.com/Tanzhihang)**. [中文说明](README.md)

This Agent Skill combines local Whisper transcription, terminology checks, exhaustive adjacent-boundary review, atomic group edits and validation evidence. An agent performs semantic review; Python tools collect candidates and enforce structural constraints. Running a detector alone is not a complete proofreading job.

## Features

- Full-audio local Whisper transcription with word timestamps; no transcription API or source-audio upload.
- Review every adjacent subtitle boundary for split words, broken terminology, repetition and misplaced syntax.
- Merge connected problems into disjoint, consecutive B groups; apply a whole group together.
- Locate new internal timing boundaries using audio evidence while preserving group outer bounds.
- Validate structure, residual candidates/fillers, character conservation, timing evidence and boundary ledgers.
- Topic-specific glossaries for general, consumer electronics and automotive content.

## Quick start

Download the complete source ZIP `srt-audio-review-v4-v0.1.0.zip` from the repository first. The archive contains all scripts, glossaries, references and tests.

Requires Python 3.10+, ffmpeg/ffprobe and a skill-capable agent with local execution.

```bash
unzip srt-audio-review-v4-v0.1.0.zip
cd srt-audio-review-v4
mkdir -p ~/.agents/skills
cp -R skills/srt-audio-review-v4 ~/.agents/skills/
python3 skills/srt-audio-review-v4/scripts/setup_local_whisper.py
```

The setup command downloads dependencies and the `small` model. Ask for network authorization if the environment requires it. Use the same Python environment for setup and transcription. Override the shared cache with `SRT_V4_CACHE` if needed. See the [official local skill locations](https://learn.chatgpt.com/docs/build-skills).

Provide the original SRT and corresponding original audio, then ask:

```text
Use $srt-audio-review-v4 to proofread this Chinese SRT against its original audio.
Review all adjacent boundaries, locate every resegmented group's timing from the audio,
and deliver a new SRT with change records and all validation evidence.
```

The default workflow delivers a validated copy directly. Material unresolved ambiguities require clarification; explicitly request a confirmation table for a review-first workflow.

## Boundaries

Original inputs are read-only. Preserve block count, numbering, internal line count and newline structure. After freezing the audio-calibrated text, resegmentation must conserve the exact non-whitespace character sequence. Only internal timing within reviewed B groups may change.

Audio transcription is local; cloud agent review can still send subtitle/transcript text to the agent platform. First setup requires network downloads. Detectors and validators do not prove semantic correctness or zero errors. Glossaries are editable preferences, not a live product database.

## Development

```bash
python3 -m unittest discover -s skills/srt-audio-review-v4/tests -v
```

See [source archive with workflow commands](srt-audio-review-v4-v0.1.0.zip), [contributing](CONTRIBUTING.md) and [changelog](CHANGELOG.md). The demo subtitle is synthetic and has no paired original audio.

## License

[MIT](LICENSE), copyright 谭智航 (Tanzhihang). [OpenAI Whisper](https://github.com/openai/whisper) and its dependencies retain their own licenses.
