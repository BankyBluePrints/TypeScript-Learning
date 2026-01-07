# TypeScript Learning — Complete Feature Documentation

## Purpose

This repository is a **complete, structured, feature-by-feature documentation reference** for TypeScript. It covers fundamental concepts and version-specific features, with each feature documented in a single, self-contained markdown file.

## Philosophy: Feature-Per-File

**Core Principle:** 1 TypeScript feature = 1 Markdown file

Every feature file contains:
- Clear explanation of what the feature is
- Code examples showing usage before and after the feature
- Benefits and use cases
- Edge cases, gotchas, and compiler settings
- Quick practice exercises

No feature is split across multiple files. No generic "overview" or "new features" documents exist.

## Repository Structure

```
/00_Basics/
  0001_why-typescript.md
  0002_installation-and-tsconfig.md
  0003_basic-types.md
  ... (fundamental TypeScript concepts)

/Versions/
  /TS_1.x/
    /Features/
      ts1_0001_<feature-slug>.md
      ts1_0002_<feature-slug>.md
      ...
  /TS_2.x/
    /Features/
      ts2_0001_<feature-slug>.md
      ...
  /TS_3.x/
    /Features/
      ts3_0001_<feature-slug>.md
      ...
  /TS_4.x/
    /Features/
      ts4_0001_<feature-slug>.md
      ...
  /TS_5.x/
    /Features/
      ts5_0001_<feature-slug>.md
      ...
```

## How to Use This Repository

### For Learning TypeScript from Scratch
1. Start with `/00_Basics/` files in sequential order (0001, 0002, 0003...)
2. Each file builds on previous concepts
3. Complete the practice exercises in each file

### For Learning Version-Specific Features
1. Navigate to `/Versions/TS_X.x/Features/`
2. Each feature is self-contained and can be read independently
3. Features are numbered sequentially but can be explored in any order

### For Reference
- Use file names to quickly locate features (e.g., `ts4_0012_variadic-tuple-types.md`)
- Each file contains complete information about one feature
- No need to jump between files for related information

## Naming Conventions

- **Basics:** `0001_*.md`, `0002_*.md` (sequential, zero-padded)
- **Features:** `ts<version>_<index>_<feature-slug>.md`
  - Example: `ts4_0012_variadic-tuple-types.md`
- All lowercase, hyphen-separated slugs
- No spaces, descriptive names

## Contributing

When adding new features:
- Follow the feature-per-file structure
- Use the standard template (see any existing feature file)
- Include all sections: What it is, Before/After, Why better, Notes, Practice
- Ensure the file is self-contained and complete