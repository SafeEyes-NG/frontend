# SafeEyes NG: Frontend

> The offline-first mobile app for reporters and the web dashboard for verifiers and responders.

Part of the [SafeEyes NG](https://github.com/safeeyes-ng/docs) project: an open-source, verified crowd-sighting and alert network for kidnapping cases in Nigeria. Read the [docs repo](https://github.com/safeeyes-ng/docs) first for the mission, threat model and design principles.

> **Status:** Phase 0 / early scaffolding. This README describes the intended design. Update it as things are built.

---

## What lives here

| App / package | Who uses it | Purpose |
|---|---|---|
| `apps/mobile` | Residents, family members, volunteers | Receive alerts, submit sightings (photo, video, voice note, text, location), opt in to live location sharing, work offline |
| `apps/dashboard` | Verifiers, responders, family liaisons | Verification queue, live case map and timeline, sighting review, case close-out |
| `packages/ui` | Both | Shared components, design tokens, low-bandwidth and low-literacy patterns |
| `packages/api-client` | Both | Typed API client **generated** from `openapi.yaml` in the docs repo |

## Tech stack (proposed defaults, open to discussion via ADR)

- **Mobile:** React Native with Expo, TypeScript. Local storage with SQLite for the offline queue
- **Dashboard:** React with Vite, TypeScript, MapLibre GL for maps
- **Shared UI:** React Native Web compatible components where practical
- **State and data:** TanStack Query for server state
- **API types:** generated from OpenAPI (never hand-written)
- **i18n:** i18next with English, Hausa, Yoruba, Igbo and Nigerian Pidgin
- **Testing:** Jest and React Native Testing Library (mobile), Vitest and Testing Library (dashboard), Playwright for dashboard end-to-end
- **Tooling:** pnpm workspaces, ESLint, Prettier, Husky, commitlint

## Repository layout

```
frontend/
├── apps/
│   ├── mobile/
│   │   ├── src/
│   │   │   ├── screens/        # Alert, Report sighting, Settings, Consent
│   │   │   ├── features/
│   │   │   │   ├── capture/    # photo, video, voice note
│   │   │   │   ├── location/   # consent-based live location
│   │   │   │   ├── offline/    # upload queue, retry, sync
│   │   │   │   └── hashing/    # on-device SHA-256 of media
│   │   │   ├── i18n/
│   │   │   └── App.tsx
│   │   └── app.json
│   └── dashboard/
│       ├── src/
│       │   ├── pages/          # Queue, Case, Map, Audit
│       │   ├── components/
│       │   └── main.tsx
│       └── vite.config.ts
├── packages/
│   ├── ui/
│   └── api-client/             # generated, do not edit by hand
├── .env.example
├── CLAUDE.md
├── pnpm-workspace.yaml
└── README.md
```

## Getting started

### Prerequisites
- Node.js (current LTS) and pnpm
- For mobile: Expo Go on a phone, or an Android emulator
- A running backend (see the `backend` repo) or the mock server (`pnpm mock`)

### Setup
```bash
git clone https://github.com/safeeyes-ng/frontend.git
cd frontend
pnpm install
cp .env.example .env            # point to your local or mock API
pnpm gen:api                    # generate the typed client from openapi.yaml
```

### Common scripts (define these in `package.json`)
| Script | Purpose |
|---|---|
| `pnpm dev:mobile` | Start Expo for the mobile app |
| `pnpm dev:dashboard` | Start the dashboard dev server |
| `pnpm mock` | Run a mock API from the OpenAPI spec |
| `pnpm gen:api` | Regenerate the API client |
| `pnpm test` | Run all unit tests |
| `pnpm lint` | Lint and format check |
| `pnpm e2e` | Dashboard end-to-end tests |

> In a Codespace, run the dashboard on a forwarded port. For the mobile app, use Expo's tunnel mode so a real phone can connect.

## Core user flows

### Mobile
1. **Receive alert:** shows minimal, expiring details and **a request for sightings only**. Never a call to go and confront anyone.
2. **Report a sighting:** capture photo, short video or voice note, add a location pin and optional text. Works offline. Items queue and upload when signal returns.
3. **Hash on device:** SHA-256 is computed before upload, and the backend checks it.
4. **Consent-based live location:** an explicit opt-in screen explaining what is shared, with whom and for how long. One tap to stop. Auto-stops when the case closes.
5. **Anonymous mode:** report without an identifiable account.

### Dashboard
1. **Verification queue:** review new cases, approve or reject. Two-person rule for high-impact alerts.
2. **Case view:** map and timeline of clustered sightings, with confidence scores and direction of travel.
3. **Evidence review:** view media, see the on-chain anchor status for each file.
4. **Audit view:** who viewed or changed what.

## UX and design requirements

Many users will be on cheap Android phones, patchy 3G and prepaid data.

- **Low bandwidth first:** compress media on-device, show text-first views, avoid heavy assets. Provide a data-saver toggle
- **Offline first:** every reporting action must work without signal and sync later
- **Low literacy friendly:** large touch targets, icons with labels, voice-note reporting, simple language
- **Multilingual:** all user-facing strings go through i18n. No hard-coded text
- **Accessibility:** screen-reader labels, sufficient contrast, scalable text
- **Performance budget:** set and track bundle size and cold-start time targets
- **Calm design:** alerts should inform, not panic. No flashing, countdown pressure or sensational copy

## Safety and privacy rules (non-negotiable)

1. **Alert copy never tells people to confront, chase or approach anyone.** It asks for sightings only. Review all alert templates with the docs team.
2. **Never display suspect photos or identifying details on public screens.** Those are for verified responders only.
3. **Location is consent-only,** visibly active whenever on, and stoppable in one tap.
4. **Strip metadata** (EXIF and similar) from media before upload where possible.
5. **No analytics or trackers** that send personal or location data to third parties.
6. **Secure storage:** tokens in the OS secure store, never plain local storage. Encrypt queued media at rest.
7. **No real data in the repo.** Mocks, screenshots and tests use synthetic data only.
8. **Role-gated UI is not security.** The backend enforces permissions. The UI only reflects them.

## Contributing

1. Read the [docs repo CONTRIBUTING.md](https://github.com/safeeyes-ng/docs/blob/main/CONTRIBUTING.md).
2. Pick an issue labelled `good first issue`, `mobile`, `dashboard` or `help wanted`, and comment to claim it.
3. Branch naming: `feat/…`, `fix/…`, `docs/…`. Conventional commits.
4. Include tests, and screenshots or screen recordings for UI changes (synthetic data only).
5. PRs need passing CI and one review. Changes to consent, location or media-capture flows need two.

**Good first issues to open:**
- Scaffold the Expo app with navigation and i18n setup
- Scaffold the Vite dashboard with routing and auth shell
- Build the `ui` package with buttons, inputs and alert card components
- Offline queue: store pending sightings in SQLite and retry with backoff
- Voice note recorder component with a size limit
- On-device SHA-256 hashing helper with tests
- Consent screen for live location with one-tap stop
- Hausa, Yoruba, Igbo and Pidgin translation files for core strings
- Map component with clustered sighting markers (MapLibre)

## Working with Claude Code in a Codespace

1. Open the repo in a Codespace. Add a `.devcontainer/devcontainer.json` with Node and pnpm.
2. Install Claude Code (`npm install -g @anthropic-ai/claude-code`, or see Anthropic's docs for the current method) and run `claude` in the repo root.
3. Create a `CLAUDE.md` with:
   - The mission and *eyes, not fists* principle
   - The **Safety and privacy rules** section above, verbatim
   - The stack, folder layout and scripts table
   - "Never hand-edit `packages/api-client`. Regenerate it from the spec."
   - "All user-facing strings go through i18n"
   - "Synthetic data only"
4. Example first prompts:
   - *"Scaffold the Expo mobile app with TypeScript, navigation, i18next and a placeholder alert screen."*
   - *"Implement an offline upload queue backed by SQLite with exponential backoff and tests."*
   - *"Build the consent screen for live location sharing with an always-visible stop control."*
5. Review everything yourself. Test mobile changes on a real low-end device or throttled network, not just the emulator.

## Security

Never open a public issue for a vulnerability. See `SECURITY.md` in the docs repo for private reporting.

## License

Apache-2.0 (confirm and add a `LICENSE` file before the first release).
