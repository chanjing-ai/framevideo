# Voiceover Workflow

## Goal

Turn project subtitles or AI analysis into a narration script, generate audio, then feed the result back into transcript and caption assets.

## Recommended flow

1. Locate the text source.
2. Normalize it into a narration draft.
3. Add SSML-style markup only where it improves delivery.
4. Decide whether the current TTS provider can preserve the markup.
5. Generate audio.
6. Save the marked script, the fallback script if needed, and the final audio.
7. Regenerate transcript/captions from the audio and keep them aligned with the project content.

## Source priority

- Project subtitles and transcript are the primary source.
- AI analysis copy is the next fallback when subtitles are too thin.
- Manual narration is the last resort.

## Fallback rule

When SSML is not supported by the provider:

- remove markup
- keep the spoken text natural
- preserve the original marked version for future regeneration

## Writeback rule

Write generated assets back into the same project structure used by the existing voice workflow so later steps can find them without extra wiring.

