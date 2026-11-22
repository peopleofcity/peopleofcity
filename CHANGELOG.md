# Changelog

All notable changes to the People of City foundation are recorded here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and versions follow [Semantic Versioning](https://semver.org).

## [Unreleased]

## [0.1.0] - 2026-11-23

The first public release: the open foundation of People of Bengaluru, as of 10 November 2026.

### Added
- The open-source foundation under the GNU AGPL-3.0.
- Documentation: README, [ARCHITECTURE.md](ARCHITECTURE.md), [CUSTOMIZATION.md](CUSTOMIZATION.md), [CONTRIBUTING.md](CONTRIBUTING.md), [SUBMISSIONS.md](SUBMISSIONS.md), Code of Conduct and security policy.
- Issue forms, a pull request template, code owners, and CI checks on every pull request.
- Fillers for email (`src/lib/server/email/send.ts`) and payments (`src/lib/server/dodo.ts`), so the foundation runs without either.
- A starter database (`supabase/schema.sql`, `supabase/migrations/`): every table, row-level security policy and function the app uses, with no data.

### Included from People of Bengaluru
- A walkable, stylised 3D city: Koramangala 5th Block and HSR Layout, with portal travel between them.
- Day and night on the city's real clock; generative music and ambience.
- Characters, sample residents, name plates, and find-someone search.
- Desktop and touch controls.
- With Supabase: email-code sign-in, invites, applications with admin review, profiles, status, connect and chat, Day Zero missions.
- The live city: real-time positions through a Cloudflare Workers server (`realtime/`).

### Not included
- People of Bengaluru's data, email templates and sending, payments, and the scheduled reminder job.

### Changed
- Analytics are off by default (`GA_ID` is empty in `src/lib/site.ts`).
- Applications are free: a saved application goes straight to review.
