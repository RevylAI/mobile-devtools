# Mobile DevTools

**Open-source tools for mobile engineering teams.** Each tool uses [Revyl CLI](https://github.com/RevylAI/revyl-cli) to boot cloud devices, interact with apps using natural language, and capture evidence — combined with [Claude Code](https://claude.ai/code) for AI-powered automation.

These are real tools you can fork and run today. They're also blueprints — designed to show what's possible so your team can build internal tools tailored to your own workflows.

## Start Here

If you are trying Revyl locally, start with the browser-based login. You do not need to create and copy an API key for the first run.

```bash
# Install the Revyl CLI
brew install RevylAI/tap/revyl    # macOS
pip install revyl                 # or via pip

# Sign in and verify your local setup
revyl auth login
revyl doctor
```

Then pick a tool below and follow its README. Tools that boot cloud devices will ask you to upload an app build to Revyl. API keys are still supported, but they are mainly for CI, automation, and other non-interactive environments.

Want to try something with zero Revyl setup? [Greenlight](https://github.com/RevylAI/greenlight) runs entirely offline and does not require an account.

| Tool | What it does |
|---|---|
| [**Security Scanner**](https://github.com/RevylAI/mobile-pentest-agent) | Static + dynamic penetration testing for Android and iOS apps |
| [**PR Review Bot**](https://github.com/RevylAI/mobile-pr-reviewer) | Boots a device on every PR, validates the change, posts screenshots |
| [**Visual Regression**](https://github.com/RevylAI/visual-regression) | Pixel-diffs screenshots between two builds to catch UI changes |
| [**Figma Design Checker**](https://github.com/RevylAI/figma-design-checker) | Compares Figma frames against the real app to verify design fidelity |
| [**Greenlight**](https://github.com/RevylAI/greenlight) | Pre-submission App Store compliance scanner — catches rejection risks offline |
| [**App Explorer**](https://github.com/RevylAI/app-explorer) | Maps every screen and user path in a mobile app — generates interactive navigation maps |

---

## The Tools

### [Security Scanner](https://github.com/RevylAI/mobile-pentest-agent)

Automated mobile app penetration testing. Runs static analysis (semgrep + pattern matching) on your source code, then boots a cloud device to dynamically exploit the findings — testing insecure storage, network security, authentication bypasses, and more.

- Static analysis for Android (Java/Kotlin) and iOS (Swift/ObjC)
- Dynamic exploitation on live cloud devices
- Generates a full pentest report with screenshots as evidence
- 34+ detection patterns across 10 vulnerability categories

```bash
pentest scan ./my-app --platform android
```

| What it catches | How |
|---|---|
| Hardcoded secrets, API keys | Static grep + semgrep rules |
| Insecure data storage | Reads SharedPreferences/UserDefaults on device |
| Certificate pinning bypasses | Tests network requests on device |
| Debug logging in production | Static pattern matching |

---

### [PR Review Bot](https://github.com/RevylAI/mobile-pr-reviewer)

When a developer opens a PR, Claude reads the diff, figures out what changed, boots a cloud device with the new build, navigates to the affected screen, and posts screenshot evidence directly in the PR.

- Plugs into GitHub Actions via [Claude Code Action](https://github.com/anthropics/claude-code-action)
- Understands code diffs to determine what screens to test
- Uses natural language to navigate the app (no element IDs needed)
- Posts before/after screenshots as PR comments

```
Developer pushes a button color change
  -> Claude reads the diff
  -> Boots a cloud device
  -> Taps through to the changed screen
  -> Screenshots the result
  -> Posts it in the PR
```

| PR diff | Device screenshot |
|---|---|
| `CartContext.tsx` — fixed product substitution bug | Claude boots device, adds Orchid Mantis to cart, catches the bug |

---

### [Visual Regression](https://github.com/RevylAI/visual-regression)

Compare screenshots between two builds to catch unintended UI changes. Captures every key screen on both the baseline and current build, pixel-diffs them, and generates a visual report with highlighted change regions.

- Configurable screen list via `screens.yaml`
- Pixel-diff engine with red/magenta overlay on changed regions
- HTML report with side-by-side comparisons + stats
- CI integration: gates PRs on a configurable diff threshold (default 0.1%)

```
Baseline Build          Current Build
     |                       |
  [Device A]             [Device B]
     |                       |
  screenshots/           screenshots/
  baseline/              current/
     |                       |
     +----------+------------+
                |
           pixel diff
                |
         report.html
```

```
  [PASS] shop_home           0.00% diff
  [PASS] product_detail      0.03% diff
  [FAIL] cart                 2.45% diff    <- something changed
  [PASS] search               0.00% diff
```

---

### [Figma Design Checker](https://github.com/RevylAI/figma-design-checker)

Pull design frames from Figma, boot the real app on a cloud device, screenshot matching screens, and pixel-diff them to verify developers built what was designed.

- Fetches frames directly from the Figma API
- Maps Figma frame names to app screens via config
- Blended fidelity score (70% pixel + 30% structural similarity)
- Compliance grading: A (95%+) through F (<70%)
- Dark-themed HTML report with embedded images

```
Figma File              Cloud Device               Report
   |                         |                        |
   | Export frames           | Boot app               |
   | as PNGs          -->    | Navigate screens  -->  | Pixel diff
   |                         | Screenshot each        | Grade each screen
   v                         v                        v
figma_frames/           app_screenshots/          report.html
```

```
  Shop Home        96.8% [A]
  Product Detail   92.4% [B]
  Search           90.1% [B]
  Cart             86.7% [C]  <- needs review
  Profile          94.7% [B]

  Overall: 93.0% — Grade B
```

---

### [Greenlight](https://github.com/RevylAI/greenlight)

Pre-submission compliance scanner for the Apple App Store. Scans your source code, privacy manifests, IPA binaries, and App Store Connect metadata against Apple's Review Guidelines — catching rejection risks before Apple does.

- Entirely offline, no account needed, runs in under a second
- 30+ code pattern checks: private APIs, hardcoded secrets, payment violations, missing ATT
- Privacy manifest validator: Required Reason APIs, tracking SDKs, PrivacyInfo.xcprivacy completeness
- IPA binary inspector: Info.plist keys, app icons, framework privacy manifests
- Supports Swift, Objective-C, React Native, and Expo projects

```bash
# One command — scans everything
greenlight preflight /path/to/your/project

# Include IPA for binary analysis
greenlight preflight . --ipa build.ipa
```

| Scanner | What it checks |
|---|---|
| **metadata** | App name, version, bundle ID, icon, privacy policy URL, purpose strings |
| **codescan** | Private APIs, secrets, payment violations, missing ATT, social login, placeholders |
| **privacy** | PrivacyInfo.xcprivacy completeness, Required Reason APIs, tracking SDKs vs ATT |
| **ipa** | Binary Info.plist, launch storyboard, app icons, app size, framework manifests |

### [App Explorer](https://github.com/RevylAI/app-explorer)

Automated app screen mapping. Boots a cloud device, systematically explores every screen using AI-driven BFS navigation, and generates an interactive navigation map with screenshots of every screen and all user paths.

- AI-powered exploration — Claude Code navigates the app, identifies screens, and maps transitions
- Interactive viewer with React Flow graph, journey navigator, and screen details panel
- Step through user paths with highlighting and prev/next navigation
- Works on any Android or iOS app — no accessibility IDs or XPaths needed
- Generates markdown reports with Mermaid navigation graphs

```bash
# Install the CLI
pip install -e .

# Open Claude Code and explore
# > Explore the app at ./my-app.apk and map all screens
```

| Feature | How |
|---|---|
| Screen discovery | AI navigates and screenshots every reachable screen |
| Transition mapping | Records which button/tap leads to which screen |
| User path enumeration | DFS to find all distinct navigation paths |
| Interactive viewer | React Flow graph with journey stepping and path highlighting |

---

## How They Work

Most tools that interact with a running app follow the same pattern:

1. **Revyl CLI** boots a real device in the cloud and installs your app
2. **Natural language targeting** — `revyl device tap --target "Add to Cart button"` — no accessibility IDs, no XPaths, no element inspectors
3. **Screenshots as evidence** — every action is captured and included in reports
4. **AI orchestration** — Claude Code reads context (diffs, configs, designs) and decides what to test

```bash
# This is the entire interaction model
revyl device start --platform android --app-id $APP_ID --json
revyl device tap --target "Add to Cart button" --json
revyl device type --target "Search field" --text "beetles" --json
revyl device screenshot --out evidence.png --json
revyl device stop --json
```

The `--target` flag uses AI to resolve natural language descriptions to screen coordinates. This means these tools work on **any app** without knowing the UI hierarchy.

## Build Your Own

These tools are templates. The pattern for building your own:

1. **Define what to automate** — What does your team do manually today that involves running the app on a device?
2. **Write a script** — Use `revyl device` commands with `--target` for natural language interaction
3. **Add Claude** — Use CLAUDE.md to give Claude context about your app, then let it orchestrate
4. **Wire it up** — GitHub Actions, cron job, Slack bot — whatever fits your workflow

### Ideas for internal tools

| Tool | What it does |
|---|---|
| **Onboarding validator** | New hire pushes their first feature — bot boots device, runs through the feature, posts screenshots in the PR |
| **Localization checker** | Switch device language, screenshot every screen, verify no text overflow or missing translations |
| **Accessibility auditor** | Navigate the app with TalkBack/VoiceOver enabled, verify all elements are labeled |
| **Release smoke test** | Before every release, bot runs through critical user flows and posts a go/no-go report |
| **App store screenshot generator** | Navigate to key screens, capture at multiple device sizes, format for App Store/Play Store |
| **Competitor benchmarking** | Install competitor apps, screenshot their flows, compare side-by-side with yours |
| **Deep link tester** | Open every deep link in your app, verify each one lands on the correct screen |
| **Dark mode validator** | Toggle dark mode, screenshot every screen, diff against light mode to catch unstyled views |

## Getting Started

For local exploration, most cloud-device tools need two things:

1. **A signed-in Revyl CLI session:**
   ```bash
   revyl auth login
   ```
2. **An app uploaded to Revyl:**
   ```bash
   # Create an app and upload a build
   revyl app create --name "MyApp" --platform android --json
   revyl build upload --skip-build --platform android --app "$APP_ID" --file app.apk --json --yes
   ```

For GitHub Actions or other non-interactive environments, create a Revyl API key in [app.revyl.ai](https://app.revyl.ai) under Settings -> API Keys and set it as `REVYL_API_KEY`.

Then fork any tool above and follow its README.

## Sample App

Several repos include sample apps you can use before wiring in your own app. [PR Review Bot](https://github.com/RevylAI/mobile-pr-reviewer/tree/main/sample-app) and [Visual Regression](https://github.com/RevylAI/visual-regression/tree/main/sample-app) include Bug Bazaar, a React Native e-commerce app with intentional bugs for testing. The Figma Design Checker repo includes its own sample app and example report assets.

## Built With

- **[Revyl CLI](https://github.com/RevylAI/revyl-cli)** — Cloud device provisioning and AI-grounded mobile app interaction
- **[Claude Code](https://claude.ai/code)** — AI agent for code analysis and orchestration
- **[Claude Code Action](https://github.com/anthropics/claude-code-action)** — Run Claude in GitHub Actions

## License

MIT
