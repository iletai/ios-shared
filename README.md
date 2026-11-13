# ios-shared

One Fastfile and three reusable GitHub Actions workflows for all iletai iOS apps, so a fix lands in every app at once.

- `fastlane/Fastfile`: lanes `bootstrap`, `sync_signing`, `test`, `beta`, `metadata`, `screenshots`, `release` (+ private `asc_api_key`, `build_signed`). All app values come from ENV; see the header of the file.
- `.github/workflows/ios-ci.yml`, `ios-testflight.yml`, `ios-appstore.yml`: `on: workflow_call`, inputs `scheme`, `xcode-version`, `ruby-version` (App Store also `submit`, `skip-metadata`, `screenshots`). Callers pass `secrets: inherit`.
- `examples/app/`: everything an app keeps in its own repo.

This repo holds no secrets and must be **public** (or private with Actions access shared to the owner's repos), because callers fetch it from CI without credentials.

## Adopt in an existing app

1. Copy `examples/app/fastlane/{Fastfile,Appfile,Matchfile,.env.default}` and fill `.env.default` (bundle IDs, project, schemes, packages, capabilities).
2. Keep app-only files: `Snapfile`, `metadata/`, `screenshots/`. Move app-only lanes (e.g. a UI gallery lane) into the app Fastfile after the `import_from_git` line.
3. Replace workflows with the callers in `examples/app/.github/workflows/`.
4. Repo secrets (names): `APPLE_TEAM_ID`, `ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_KEY_CONTENT_BASE64`, `MATCH_GIT_URL`, `MATCH_PASSWORD`, `MATCH_SSH_KEY`, `FASTLANE_KEYCHAIN_PASSWORD`; optional `APP_REVIEW_FIRST_NAME/LAST_NAME/PHONE/EMAIL` (unset keeps the contact already in ASC). Check with `gh secret list`.
5. Verify locally: `bundle exec fastlane lanes`, then `bundle exec fastlane test`; run `beta` from CI via `workflow_dispatch`.

## Release this repo

Callers pin `@main` (workflows) and `branch: "main"` (Fastfile). Work on `develop`, merge to `main` when an app has been verified on CI. Pin a tag instead of `main` if an app must not move.
