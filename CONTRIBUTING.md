# Contributing to People of City

Thanks for wanting to make the city better. Developers, designers, 3D artists and builders are all welcome.

## Two ways to take part

1. **Build your own city.** Fork the repository and make People of _your_ city. Independent city implementations generally stay in **your own fork**: that's where your streets, landmarks and branding live. Show it off through [SUBMISSIONS.md](SUBMISSIONS.md).
2. **Improve the shared foundation.** Fix a bug or add something **every** city can use (a better building kit, a performance win, a new interaction, a config option). These come back here through a **pull request**.

Not sure which one your change is? Ask yourself whether a People of Tokyo would want it. If yes, it belongs in the foundation.

## Before you start

- For anything bigger than a small fix, **[open an issue](https://github.com/peopleofcity/peopleofcity/issues/new/choose) first** so we can agree on the approach before you spend time on it.
- Read [ARCHITECTURE.md](ARCHITECTURE.md) to find your way around.
- Read [`design.md`](design.md) and [`sounddesign.md`](sounddesign.md) before changing how the city looks or sounds. They're specifications, not inspiration.

## Making a change

### 1. Fork and branch

```bash
git clone https://github.com/<you>/peopleofcity.git
cd peopleofcity
git checkout -b fix/portal-flicker      # or feat/…, docs/…
npm install
npm run dev
```

### 2. Follow the existing conventions

- **TypeScript everywhere**, strict. No `any` unless there's truly no type.
- **Match the code around you:** small components, plain functions, and comments that say *why*. Look at a neighbouring file before writing a new one.
- **The world is built in code** from the kit in `src/game/world/kit.tsx`. Use the kit and the palette (`src/lib/palette.ts`), not raw boxes and one-off colours.
- **Every screen works on touch, keyboard and the on-screen keyboard**, and typing in a field never moves the player.
- **Performance matters:** check `?debug` (draw calls, triangles, frame time) before and after a visual change, on desktop and on a phone.
- **No new dependencies** without discussing them in an issue first.

### 3. Test it

```bash
npm test                  # unit tests
npx tsc --noEmit -p .     # type-check
npm run build             # production build
```

If you touched `realtime/` or `shared/net/`:

```bash
npm --prefix realtime install
npm --prefix realtime run typecheck
```

For anything visual, try it in the browser on desktop and in a phone-sized window, by day and by night (`?time=21:00`).

### 4. Open a pull request

- One change per pull request. Small PRs get reviewed fast.
- Fill in the template: what changed, how you checked it, and screenshots or a short clip for anything visual.
- The same checks run on your pull request, and must be green.
- A maintainer reviews it. `main` is protected: every change lands through a reviewed pull request and is squash-merged.

## Reporting bugs

[Open a bug report](https://github.com/peopleofcity/peopleofcity/issues/new/choose) with what happened, the steps to reproduce it, your device and browser, and a screenshot or console error if you have one.

**Security issues are different:** never post them publicly. Report them privately, see [SECURITY.md](SECURITY.md).

## Suggesting features

[Open an idea](https://github.com/peopleofcity/peopleofcity/issues/new/choose): what should be possible, and why it makes the city better to walk through. For open-ended questions, use [Discussions](https://github.com/peopleofcity/peopleofcity/discussions).

## Never commit

Keys, tokens, `.env` files, or anyone's personal data. `.env*` is git-ignored, and GitHub's secret scanning blocks known key formats on push.

## License

By contributing, you agree your contribution is licensed under the project's [GNU AGPL-3.0](LICENSE).

Please be kind: [Code of Conduct](CODE_OF_CONDUCT.md).
