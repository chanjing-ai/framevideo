# SSML Notes

## Supported tags

### `phoneme`

Use for pronunciation correction.

Example:

```xml
<phoneme alphabet="py" ph="xi1">茜</phoneme>
```

### `break`

Use for deliberate pauses.

Example:

```xml
<break time="0.5s"/>
```

### `ttnumber`

Use for custom number reading.

Example:

```xml
<ttnumber pronounce="一九五五">1955</ttnumber>
```

## Rules

- Keep the child text plain and short.
- Use markup sparingly.
- Preserve the source script alongside the generated fallback text.
- If the provider cannot read these tags, strip them before synthesis.

