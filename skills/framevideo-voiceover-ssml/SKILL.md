---
name: framevideo-voiceover-ssml
description: Create narration from project subtitles or AI analysis output, then author SSML-style voice scripts with phoneme, break, and ttnumber markup for FrameVideo voiceovers. Use when the user wants subtitles linked to voice generation, automatic narration from transcript/content, pronunciation fixes, pause insertion, or text normalization before TTS.
---

# FrameVideo Voiceover SSML

Generate narration scripts from existing project content (subtitles, transcripts, AI analysis) and enhance them with SSML-style markup for pronunciation fixes, pauses, and number reading control.

## When To Use

- User wants narration generated from project subtitles or transcript
- Need to fix pronunciation of brand names, technical terms, or foreign words
- Need to control pause timing for dramatic effect or pacing
- Need to normalize how numbers, dates, or prices are spoken
- Expanding terse subtitles into natural narration
- Creating voiceover that stays aligned with visible text

## Do NOT Use

- For simple text-to-speech without markup → use `framevideo-media` (tts) directly
- For digital human video synthesis → use `chanjing-digital-human`
- For choosing TTS voices or providers → use `framevideo-media`
- When the user provides a complete narration script ready for TTS

---

## Quick Start

**Minimal workflow:**

```bash
# 1. Find source text (subtitle file, transcript, or analysis)
cat subtitles.txt
# Output: "Visit chanjing.ai for more info"

# 2. Add SSML markup for pronunciation
echo '<phoneme alphabet="ipa" ph="tʃæn.dʒɪŋ">Chanjing</phoneme> <break time="0.3s"/> Visit chanjing dot AI for more info.' > script-marked.txt

# 3. Generate TTS with markup (if provider supports)
npx framevideo tts script-marked.txt --voice af_heart --output narration.wav

# 4. Transcribe back to get timestamps
npx framevideo transcribe narration.wav --output transcript.json

# 5. Reference in composition
# <audio src="narration.wav" data-start="0" data-duration="5.2"></audio>
```

---

## Core Concepts

### Source Text Priority

Find the source text in this order:

1. **Project subtitles/transcript** — the primary source, already aligned with visible content
2. **AI analysis text** — scene descriptions or product copy extracted during storyboard
3. **Existing narration draft** — if the user provided one
4. **Manual authoring** — last resort when no source exists

**Why subtitles first?** They're already timed to the visual beats and match what viewers see on screen.

### SSML-Style Markup

Three tags for delivery control:

| Tag | Purpose | Example |
|-----|---------|---------|
| `<phoneme>` | Fix pronunciation | `<phoneme alphabet="ipa" ph="tʃæn.dʒɪŋ">Chanjing</phoneme>` |
| `<break>` | Insert pause | `<break time="0.5s"/>` |
| `<ttnumber>` | Control number reading | `<ttnumber pronounce="twenty twenty-six">2026</ttnumber>` |

**Markup is authoring-layer only.** Keep it in your script file, but strip it before sending to TTS providers that don't support SSML.

### Fallback Strategy

Not all TTS providers support SSML tags:

- **Kokoro (local):** Does NOT support SSML — strip tags before generation
- **Chanjing TTS:** Supports `phoneme`, `break`, `ttnumber` — pass through unchanged
- **ElevenLabs:** Supports subset of SSML — check their docs

**Always save both versions:**
- `script-marked.txt` — original with SSML tags
- `script-fallback.txt` — clean text for non-SSML providers

---

## Workflow

### Step 1: Locate Source Text

```bash
# Check for existing subtitles
ls subtitles.txt captions.json transcript.json

# Check AI analysis output (if using website-to-framevideo)
cat STORYBOARD.md SCRIPT.md

# Check existing narration drafts
ls narration-draft.txt script.txt
```

### Step 2: Normalize Into Speakable Script

**Expand terse subtitles:**

```
Source: "New feature: AI search"
Narration: "Introducing our new feature: AI-powered search that understands your intent."
```

**Keep alignment with visible text:**
- If subtitle says "50% faster", narration should say "fifty percent faster" at the same timing
- If product name appears on screen as "FrameVideo", say "FrameVideo" not "frame video"

**Text normalization rules:**
- Convert `&` to "and"
- Spell out acronyms on first use: "TTS, or text to speech"
- Convert URLs: "chanjing.ai" → "chanjing dot AI"
- Convert symbols: `$99` → "ninety-nine dollars"

### Step 3: Add SSML Markup

**Use sparingly.** Only add markup where TTS misreads or pacing is critical.

#### Phoneme Tag — Pronunciation Fixes

**Chinese names and brands:**
```xml
<phoneme alphabet="py" ph="chan1 jing4">蝉镜</phoneme>
<phoneme alphabet="py" ph="xi1">茜</phoneme> (name: "Xi", not "qi")
```

**English technical terms:**
```xml
<phoneme alphabet="ipa" ph="dʒiː.sæp">GSAP</phoneme> (not "guh-sap")
<phoneme alphabet="ipa" ph="weɪ.ˈeɪ.ˈeɪ.ˈpiː">WAAPI</phoneme> (spell it out)
```

**Product names:**
```xml
<phoneme alphabet="ipa" ph="freɪm.ˈvɪd.i.oʊ">FrameVideo</phoneme>
```

**When to use:**
- Brand names mispronounced by TTS
- Technical jargon with non-standard pronunciation
- Foreign words in English narration
- Acronyms that should be spelled vs pronounced

**Alphabets:**
- `ipa` — International Phonetic Alphabet (universal)
- `py` — Pinyin for Mandarin Chinese
- `x-sampa` — Extended SAMPA (ASCII-safe IPA)

#### Break Tag — Pause Control

**Dramatic pauses:**
```xml
Introducing our new product. <break time="0.8s"/> Built for creators.
```

**List separation:**
```xml
Three reasons: speed<break time="0.4s"/>, quality<break time="0.4s"/>, and ease of use.
```

**Transition markers:**
```xml
In the past<break time="0.3s"/>, this took hours. Now? Seconds.
```

**Timing guidelines:**
- `0.2s` — natural comma pause
- `0.3-0.5s` — sentence boundary, breath
- `0.6-0.8s` — dramatic pause, emphasis
- `1.0s+` — scene transition, major shift

**When to use:**
- Default TTS pacing feels rushed
- Need emphasis before key message
- Narration must sync with on-screen timing

**When NOT to use:**
- Every comma (let TTS handle natural pauses)
- Replacing proper sentence structure

#### Ttnumber Tag — Number Reading

**Years:**
```xml
<ttnumber pronounce="twenty twenty-six">2026</ttnumber>
<ttnumber pronounce="nineteen fifty-five">1955</ttnumber>
```

**Product codes:**
```xml
Model <ttnumber pronounce="A X dash three hundred">AX-300</ttnumber>
```

**Prices:**
```xml
Only <ttnumber pronounce="ninety-nine dollars">$99</ttnumber>
```

**Dates:**
```xml
Launch date: <ttnumber pronounce="July seventeenth">7/17</ttnumber>
```

**When to use:**
- Default TTS reads numbers awkwardly ("two thousand twenty-six" vs "twenty twenty-six")
- Product codes should be spelled ("A-X-three-hundred" not "ax three hundred")
- Currency needs natural phrasing
- Dates need cultural reading convention

### Step 4: Choose TTS Provider

**Decision tree:**

```
Does user require Chanjing platform voices?
├─ YES → Use chanjing-digital-human skill (supports SSML)
└─ NO
   ├─ Need offline/free → Kokoro via framevideo-media (strip SSML)
   └─ Need high quality → ElevenLabs (check SSML support)
```

**Provider SSML support:**

| Provider | Phoneme | Break | Ttnumber | Invoke via |
|----------|---------|-------|----------|------------|
| Kokoro | ❌ | ❌ | ❌ | `framevideo-media` |
| Chanjing TTS | ✅ | ✅ | ✅ | `chanjing-digital-human` |
| ElevenLabs | ✅ | ✅ | ⚠️ | External API |

### Step 5: Generate Audio

**With SSML support (Chanjing):**
```bash
# Pass marked script directly
npx framevideo chanjing tts script-marked.txt \
  --voice-id <voice-id> \
  --output narration.wav
```

**Without SSML support (Kokoro):**
```bash
# Strip tags first
sed 's/<[^>]*>//g' script-marked.txt > script-fallback.txt

# Generate clean
npx framevideo tts script-fallback.txt \
  --voice af_heart \
  --output narration.wav
```

**Stripping SSML tags:**
```bash
# Simple regex (removes all XML-like tags)
sed 's/<[^>]*>//g' input.txt > output.txt

# Or Python for precise control
python3 << 'EOF'
import re
with open('script-marked.txt') as f:
    text = f.read()
# Remove SSML tags but keep inner text
text = re.sub(r'<phoneme[^>]*>(.*?)</phoneme>', r'\1', text)
text = re.sub(r'<break[^>]*/?>', ' ', text)
text = re.sub(r'<ttnumber[^>]*>(.*?)</ttnumber>', r'\1', text)
with open('script-fallback.txt', 'w') as f:
    f.write(text)
EOF
```

### Step 6: Save Both Versions

**File structure:**
```
project/
├── script-marked.txt        # Original with SSML
├── script-fallback.txt      # Stripped for non-SSML TTS
├── narration.wav            # Generated audio
└── transcript.json          # Re-transcribed with timestamps
```

**Why save both?**
- Marked version preserves authoring intent
- Fallback version for re-generation with different TTS
- Future TTS providers may add SSML support

### Step 7: Writeback to Project

**Generate transcript from audio:**
```bash
npx framevideo transcribe narration.wav --output transcript.json
```

**Use in composition:**
```html
<audio 
  src="narration.wav" 
  data-start="0" 
  data-duration="12.4"
  data-track-index="10"
  data-volume="1.0">
</audio>
```

**Sync captions (see framevideo skill for caption patterns):**
```javascript
// transcript.json → caption timing
const WORDS = [
  { text: "Introducing", start: 0.2, end: 0.8 },
  { text: "FrameVideo", start: 0.9, end: 1.4 },
  // ...
];
```

---

## Patterns

### Pattern 1: Subtitle Expansion

**Input subtitles.txt:**
```
🚀 New: AI Search
50% faster
Try it now
```

**Output script-marked.txt:**
```
Introducing our newest feature:<break time="0.4s"/> AI-powered search.
<break time="0.3s"/>
It's fifty percent faster than traditional search.
<break time="0.5s"/>
Try it now at <phoneme alphabet="ipa" ph="tʃæn.dʒɪŋ">chanjing</phoneme> dot AI.
```

### Pattern 2: Brand Name Consistency

**DESIGN.md snippet:**
```markdown
## Brand Voice
- Company: "Chanjing" (IPA: tʃæn.dʒɪŋ, NOT "chan-jing")
- Product: "FrameVideo" (one word, capital F, capital V)
```

**Apply in script:**
```xml
Welcome to <phoneme alphabet="ipa" ph="tʃæn.dʒɪŋ">Chanjing</phoneme>.
<phoneme alphabet="ipa" ph="freɪm.ˈvɪd.i.oʊ">FrameVideo</phoneme> makes video simple.
```

### Pattern 3: Technical Demo Narration

**For product walkthroughs:**
```xml
First, click the Timeline button.<break time="0.6s"/>
Then drag the <phoneme alphabet="ipa" ph="dʒiː.sæp">GSAP</phoneme> animation track.<break time="0.5s"/>
Notice the seek time updates in real-time.
```

### Pattern 4: Multilingual Product Names

**English narration with Chinese brand:**
```xml
This is <phoneme alphabet="py" ph="chan1 jing4">蝉镜</phoneme><break time="0.3s"/>, the AI video platform from China.
```

### Pattern 5: Number-Heavy Content

**Pricing and metrics:**
```xml
Available in three tiers:<break time="0.4s"/>
Starter at <ttnumber pronounce="forty-nine dollars">$49</ttnumber>,<break time="0.3s"/>
Pro at <ttnumber pronounce="ninety-nine dollars">$99</ttnumber>,<break time="0.3s"/>
and Enterprise at <ttnumber pronounce="four ninety-nine">$499</ttnumber>.
```

---

## Rules (Non-Negotiable)

1. **Always save the marked script** — even if TTS doesn't support SSML today
2. **Fallback text must be natural** — stripping tags shouldn't break grammar
3. **Transcribe generated audio** — don't assume timing matches script
4. **Source text takes priority** — expand subtitles, don't replace them
5. **Markup is sparse** — only where TTS fails or timing is critical

---

## Avoid

- ❌ Adding SSML to every sentence — only fix actual problems
- ❌ Using `<break>` instead of rewriting awkward phrasing
- ❌ Assuming Kokoro supports SSML — it doesn't, strip tags first
- ❌ Hard-coding phonemes without testing TTS output — verify it helps
- ❌ Ignoring provider limitations — check what tags are supported
- ❌ Discarding the marked script after generation — keep for iteration

---

## Integration

### With framevideo-media

```bash
# This skill: create marked script
echo '<phoneme>...</phoneme> text' > script-marked.txt

# framevideo-media: generate TTS
npx framevideo tts script-fallback.txt --voice af_heart --output narration.wav

# framevideo-media: transcribe
npx framevideo transcribe narration.wav --output transcript.json

# framevideo skill: use in composition
```

### With chanjing-digital-human

```bash
# This skill: create marked script with SSML
cat script-marked.txt

# chanjing-digital-human: generate with SSML support
npx framevideo chanjing tts script-marked.txt --voice-id <id> --output narration.wav
```

### With website-to-framevideo

```
Step 3 (Storyboard) → produces SCRIPT.md
↓
This skill → adds SSML markup to SCRIPT.md content
↓
Step 4 (VO) → generates audio via framevideo-media or chanjing-digital-human
↓
Step 5 (Build) → uses transcript.json for caption timing
```

---

## Validation

**1. Check SSML syntax:**
```bash
# No unclosed tags
grep -o '<[^/>]*>' script-marked.txt | sort | uniq -c

# Phoneme has alphabet attribute
grep '<phoneme' script-marked.txt | grep -v 'alphabet='
# (should return nothing)
```

**2. Test fallback stripping:**
```bash
sed 's/<[^>]*>//g' script-marked.txt > test-fallback.txt
cat test-fallback.txt
# Verify: readable, no XML artifacts, natural grammar
```

**3. A/B test TTS output:**
```bash
# Generate without markup
npx framevideo tts script-fallback.txt --voice af_heart -o no-markup.wav

# Generate with markup (if supported)
npx framevideo chanjing tts script-marked.txt --voice-id <id> -o with-markup.wav

# Compare audio quality and pronunciation
```

**4. Verify transcript alignment:**
```bash
npx framevideo transcribe narration.wav --output transcript.json
# Check: word timing matches intended beats
```

---

## References

- [references/workflow.md](references/workflow.md) — step-by-step workflow details
- [references/ssml.md](references/ssml.md) — SSML tag specifications
- `framevideo-media` skill — TTS generation and transcription
- `chanjing-digital-human` skill — SSML-aware TTS via Chanjing platform
- `framevideo` skill — audio element conventions and caption timing
- SSML W3C spec: https://www.w3.org/TR/speech-synthesis/
- IPA chart: https://www.internationalphoneticalphabet.org/ipa-charts/
