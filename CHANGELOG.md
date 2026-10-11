# Azure Basic Workshop Changelog

## Unreleased

### Documentation

- Added workshop duration estimates (240-minute instructor-led agenda: 40-minute theory session + 200-minute hands-on labs) based on the author-provided official agenda (L200, 4 hours). Updated `level` from `beginner` to `intermediate` to match the L200 classification, and added a `private-endpoint` tag.
- Standardized the three learning directories and all 56 image paths; image content is unchanged.
- Added portal workshop metadata, consistent lab sections, and previous/next navigation.
- Preserved existing lab commands and cleanup instructions. `validated_on` remains blank; durations are planning estimates, not measured values.
- Added agent validation instructions and replaced unused governance template placeholders with project-specific guidance and the official Microsoft security policy.
- Kept the existing MIT license. Document licensing, support scope, portal execution, and screenshot review remain pending.

### Path changes

- `0. prerequisite` is now `00-prerequisites`.
- `1. basic` is now `01-basic`.
- `2. advanced` is now `02-advanced`.
- Each lab's images are now under its `assets` directory. External links to the old paths need updating.
