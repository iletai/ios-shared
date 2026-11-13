# ios-shared

One Fastfile and three reusable GitHub Actions workflows for all iletai iOS apps, so a fix lands in every app at once.

- `fastlane/Fastfile`: lanes `bootstrap`, `sync_signing`, `test`, `beta`, `metadata`, `screenshots`, `release` (+ private `asc_api_key`, `build_signed`). All app values come from ENV; see the header of the file.
- `.github/workflows/ios-ci.yml`, `ios-testflight.yml`, `ios-appstore.yml`: `on: workflow_call`, inputs `scheme`, `xcode-version`, `ruby-version`, `xcconfig-secrets` (comma list of secret names written to `xcconfig-secrets-path`, default `Config/Secrets.xcconfig`) (App Store also `submit`, `skip-metadata`, `screenshots`). Callers pass `secrets: inherit`.
- `examples/app/`: everything an app keeps in its own repo.

This repo holds no secrets and must be **public** (or private with Actions access shared to the owner's repos), because callers fetch it from CI without credentials.

## Adopt in an existing app

1. Copy `examples/app/fastlane/{Fastfile,Appfile,Matchfile,.env.default}` and fill `.env.default` (bundle IDs, project, schemes, packages, capabilities).
2. Keep app-only files: `Snapfile`, `metadata/`, `screenshots/`. Move app-only lanes (e.g. a UI gallery lane) into the app Fastfile after the `import_from_git` line.
3. Replace workflows with the callers in `examples/app/.github/workflows/`.
4. Repo secrets (names): `APPLE_TEAM_ID`, `ASC_KEY_ID`, `ASC_ISSUER_ID`, `ASC_KEY_CONTENT_BASE64`, `MATCH_GIT_URL`, `MATCH_PASSWORD`, `MATCH_SSH_KEY`, `FASTLANE_KEYCHAIN_PASSWORD`; optional `APP_REVIEW_FIRST_NAME/LAST_NAME/PHONE/EMAIL` (unset keeps the contact already in ASC). Check with `gh secret list`.
5. Verify locally: `bundle exec fastlane lanes`, then `bundle exec fastlane test`; run `beta` from CI via `workflow_dispatch`.

## Dev / Staging / Prod environments

Apps scaffolded with three environments commit `fastlane/.env.dev`, `.env.staging`, `.env.prod` (each sets `APP_IDENTIFIER`, `EXTRA_APP_IDENTIFIERS`, `SCHEME`, `BUILD_CONFIGURATION`) and select one with fastlane's built-in `--env`:

```sh
bundle exec fastlane bootstrap --env staging   # once per environment
bundle exec fastlane beta --env dev
bundle exec fastlane release --env prod submit:true
```

- `bootstrap`, `sync_signing`, `beta` refuse to run without `--env` when those files exist; `metadata`, `screenshots`, `release` accept only `--env prod`. `test` needs no env (`TEST_SCHEME`).
- Build numbers come from the TestFlight history of the environment's own App Store Connect record.
- Workflows take input `environment`: `ios-testflight.yml` accepts `dev|staging|prod` and runs the job in GitHub Environment `dev` / `staging` / `production`; `ios-appstore.yml` accepts `prod` and always runs in `production`. Empty `environment` keeps the single-environment behaviour.
- Environment secrets override repo secrets of the same name for that job (callers still pass `secrets: inherit`).
- Optional secret `PRIVATE_REPO_TOKEN` (fine-grained PAT, Contents read-only) lets SwiftPM clone private `github.com/iletai/*` packages on CI.

## Release this repo

Callers pin `@main` (workflows) and `branch: "main"` (Fastfile). Work on `develop`, merge to `main` when an app has been verified on CI. Pin a tag instead of `main` if an app must not move.
