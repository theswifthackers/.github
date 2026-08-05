---
name: write-ios-app-readme
description: Create or refresh a polished root README.md for a single iOS, iPadOS, or multi-platform Apple app repository using the bundled Swift-for-Good-style template. Use when Codex is asked to document an iOS app GitHub repository, turn project source and metadata into an app profile, replace a generic README, add badges/screenshots/setup instructions, or standardize an existing app README without inventing project facts.
---

# Write an iOS App README

Create a repository-specific `README.md` from verified project facts. Keep the supplied template's visual character while making every section accurate, useful, and ready for GitHub.

## Workflow

### 1. Inspect the repository

Work from the repository root. Read existing documentation and inspect enough source and configuration to understand the app before writing.

Check, when present:

- Existing `README.md`, `LICENSE*`, `CONTRIBUTING.md`, and issue or pull-request guidance.
- `*.xcodeproj/project.pbxproj`, `*.xcworkspace`, `Package.swift`, build configurations, and deployment targets.
- App entry points, main screens, models, services, entitlements, privacy manifests, tests, and package dependencies.
- `Info.plist`, localized app metadata, asset catalogs, screenshots, demo media, and app icons.
- Git remote URL and repository name.
- CI workflows and the commands they use to build or test.

Use fast, read-only searches first. Do not infer marketing claims merely from filenames. Treat source code, project settings, licenses, and distribution metadata as stronger evidence than an old README.

### 2. Build a fact sheet

Determine:

- App name and a one-sentence value proposition.
- Problem, solution, audience, and two to five user-visible features.
- Supported platforms and minimum OS versions.
- Swift version, Xcode requirement, frameworks, packages, and services actually used.
- Exact workspace or project, scheme, setup, build, and test instructions.
- Repository URL, license, release links, contact details, and acknowledgments.
- Existing image paths suitable for the logo, hero, feature previews, or screenshots.

Record whether each fact is verified, absent, or ambiguous. If multiple app targets exist, document the main user-facing target unless the user specifies another.

### 3. Resolve missing information

Never fabricate app behavior, impact metrics, compatibility, versions, release links, commands, people, or contact details.

- For a finished README, omit optional sections or entries whose facts are unavailable.
- For a requested draft, use visible `[TODO: ...]` markers only for information the repository cannot provide.
- Ask the user only when a missing choice materially changes the result and cannot be resolved from the repository.
- Do not leave template values such as `AppName`, `yourclub`, `XX.XX`, `yourname`, or example food-waste copy in the output.

### 4. Draft from the template

Use [assets/README.template.md](assets/README.template.md) as the structural starting point. Adapt it rather than copying it blindly.

Apply these rules:

- Put the concise value proposition before implementation detail.
- Explain features in user language; mention a technology only when it clarifies a capability.
- Include badges only when their values are verified.
- Use `Compatibility`, `Built With`, and `Compile`; keep spelling and capitalization consistent.
- Keep the table of contents and hero navigation synchronized with actual headings.
- Make all relative image paths and document links valid from the root `README.md`.
- Prefer real screenshots from the repository. Do not claim a placeholder image exists.
- Use one representative image per feature or a compact screenshot row; add meaningful `alt` text.
- Omit TestFlight or App Store entries when no public URL exists.
- Match the license text and filename exactly.
- Link contributors dynamically when appropriate, for example with GitHub's contributors graph, instead of inventing profile avatars.
- Preserve useful, accurate material from an existing README, including special setup steps and credits.
- Keep the tone warm and confident, not inflated. State measured impact only when evidence exists.

### 5. Write safely

Create or update the repository-root `README.md`. Preserve unrelated user changes and avoid changing app code, project settings, assets, licenses, or remote state unless explicitly requested.

If the repository already has a substantial README, integrate the new structure deliberately instead of deleting unique instructions or attribution.

### 6. Verify

Before finishing:

- Search for unresolved template tokens: `{{`, `TODO`, `AppName`, `yourclub`, `yourname`, `XX.XX`, and `example`.
- Confirm every table-of-contents and hero link matches a rendered GitHub heading anchor.
- Confirm local links and image paths exist, with case-sensitive spelling.
- Compare versions, platforms, dependencies, build commands, license, and URLs against repository evidence.
- Review the rendered hierarchy for a clear story: value, mission, access, compatibility, features, stack, setup, contribution, license, contact, and credits.
- Report any intentionally omitted or unresolved information in the handoff.

## Section policy

Use these sections when supported:

1. Hero and badges
2. The Mission
3. Try It Now
4. Compatibility
5. Features
6. Built With
7. Getting Started
8. Contributing
9. License
10. Contact
11. Acknowledgments

The Mission, Features, Built With, and Getting Started are normally essential. Make distribution, contact, contribution, license, and acknowledgment sections conditional on available facts and project needs.
