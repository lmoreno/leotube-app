# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is an **OwnTube Branded App Template** - a configuration-only repository for creating customized branded apps. This repo contains **no source code**, only:

- `.customizations` file with branding configuration
- GitHub Actions workflows for building and deploying
- Optional `/assets` folder for custom branding files (icons, splash screens, etc.)

**For comprehensive architecture, development patterns, and technical details, see the upstream repository:**
- **Documentation:** [OwnTube-tv/web-client CLAUDE.md](https://github.com/OwnTube-tv/web-client/blob/main/CLAUDE.md)
- **Repository:** [OwnTube-tv/web-client](https://github.com/OwnTube-tv/web-client)

## How This Repository Works

### Build Architecture

1. GitHub Actions workflows in this repo trigger builds via `workflow_dispatch` (manual push-button releases)
2. Workflows read `.customizations` file from this repository
3. Workflows call reusable workflows from `OwnTube-tv/web-client` repository with `use_parent_repo_customizations: true`
4. Upstream workflows clone web-client source code and apply branding from `.customizations`
5. Platform-specific artifacts are built and deployed:
   - **Web:** Static export to GitHub Pages (automatic after workflow completes)
   - **Android/Android TV:** APK/AAB artifacts with optional Google Play upload
   - **iOS/tvOS:** Simulator builds + optional TestFlight upload

### Available Workflows

- **`build_branded_main.yml`**: Main workflow - builds all platforms, deploys web to GitHub Pages, optionally uploads to TestFlight and Google Play
- **`google_play_initial_bundle.yml`**: Generates AAB bundles for first-time manual upload to Google Play Console (required before automatic uploads work)

## Triggering Builds

All builds are manual (no automatic CI/CD):

1. Navigate to GitHub Actions tab
2. Select desired workflow
3. Click "Run workflow"
4. Configure options (TestFlight uploads, Google Play track selection, etc.)
5. Wait for build to complete (can take 30-60+ minutes for full multi-platform builds)

## Configuration

### Editing Customizations

All branding is controlled by the `.customizations` file in the repository root. This file contains environment variables with `EXPO_PUBLIC_*` and `EXPO_APP_*` prefixes.

After editing `.customizations`, commit changes and trigger a new build.

### Asset Paths

Custom assets (icons, splash screens, banners) go in `/assets` folder at repository root. Reference them in `.customizations` using:

```bash
EXPO_PUBLIC_ICON=../customizations/assets/icon.png
EXPO_PUBLIC_SPLASH_IMAGE=../customizations/assets/splashScreen.png
```

**Critical:** The `../customizations/` prefix is required - this is how the upstream build process locates files from this repository.

### Key Variables

**Required:**
- `EXPO_PUBLIC_APP_NAME`: Display name
- `EXPO_PUBLIC_APP_SLUG`: Unique identifier
- `EXPO_PUBLIC_IOS_BUNDLE_IDENTIFIER`: iOS bundle ID
- `EXPO_PUBLIC_ANDROID_PACKAGE`: Android package name
- `EXPO_PUBLIC_PRIMARY_BACKEND`: PeerTube instance hostname (restricts app to single instance)

**Assets** (with recommended dimensions):
- `EXPO_PUBLIC_ICON`: Main app icon (512x512)
- `EXPO_PUBLIC_FAVICON_URL`: Web favicon (32x32)
- `EXPO_PUBLIC_SPLASH_IMAGE`: Splash screen (1152x1152, transparent background)
- `EXPO_PUBLIC_SPLASH_BG_COLOR`: Splash background color
- TV-specific icons and banners (see README.md for all dimensions)

**Optional:**
- `EXPO_PUBLIC_HIDE_VIDEO_SITE_LINKS`: Set to `1` to hide PeerTube site links
- `EXPO_PUBLIC_HIDE_GIT_DETAILS`: Set to `1` to hide git commit info in app
- `EXPO_PUBLIC_LANGUAGE_OVERRIDE`: Force specific language code
- `EXPO_PUBLIC_CUSTOM_DEPLOYMENT_URL`: Custom domain for web deployment
- `EXPO_PUBLIC_POSTHOG_API_KEY`: PostHog analytics key (or `null` to disable)
- Legal/contact info: `EXPO_PUBLIC_PROVIDER_LEGAL_ENTITY`, `EXPO_PUBLIC_PROVIDER_LEGAL_EMAIL`, `EXPO_PUBLIC_PROVIDER_CONTACT_EMAIL`

For complete list of available variables, see the example `.customizations` file in this repository or consult [docs/customizations.md](https://github.com/OwnTube-tv/web-client/blob/main/docs/customizations.md) in the upstream repository.

## Deployment Setup

### GitHub Secrets Required

All secrets must be configured in the `owntube` GitHub environment:

**Android/Google Play:**
- `ANDROID_KEYSTORE_PASSWORD`
- `ANDROID_SIGNING_KEY_ALIAS`
- `ANDROID_SIGNING_KEY_PASSWORD`
- `ANDROID_RELEASE_KEYSTORE_CONTENT_BASE64`
- `ANDROID_SERVICE_ACCOUNT_JSON`

**iOS/TestFlight:**
- `IOS_BUILD_CERTIFICATE_BASE64`
- `IOS_BUILD_PROVISION_PROFILE_BASE64`
- `IOS_P12_PASSWORD`
- `IOS_PROVISIONING_PROFILE_SPECIFIER`
- `APPLE_DEVELOPMENT_TEAM`
- `APPLE_API_PRIVATE_KEY`
- `APPLE_API_KEY`
- `APPLE_API_KEY_ISSUER`

**tvOS/TestFlight:**
- `TVOS_BUILD_CERTIFICATE_BASE64`
- `TVOS_BUILD_PROVISION_PROFILE_BASE64`
- `TVOS_P12_PASSWORD`
- `TVOS_PROVISIONING_PROFILE_SPECIFIER`
- Plus the same Apple API keys as iOS

For detailed code signing setup instructions, see [docs/pipeline.md](https://github.com/OwnTube-tv/web-client/blob/main/docs/pipeline.md) in the upstream repository.

### Web Deployment

Web builds deploy to GitHub Pages automatically after workflow completion. The URL is:
- Custom domain: Configure via `EXPO_PUBLIC_CUSTOM_DEPLOYMENT_URL` or `CUSTOM_DEPLOYMENT_URL` in `github-pages` environment
- Default: `https://{owner}.github.io/{repo-name}`

### First-Time Google Play Setup

Before automatic Google Play uploads work, you must:

1. Run the `google_play_initial_bundle.yml` workflow to generate AAB
2. Download AAB artifact from workflow run
3. Manually upload to Google Play Console to create the app listing
4. After first manual upload, automatic uploads via `build_branded_main.yml` workflow will work

## Relationship to Upstream

This repository is part of OwnTube's distributed branded app architecture:

1. **OwnTube-tv/web-client** (main repo): Contains all source code, continuous auto-deployment with vanilla OwnTube.tv branding
2. **OwnTube-tv/cust-app-template** (this template): GitHub template for creating new branded apps
3. **Branded app repos** (e.g., cust-app-blender, cust-app-xrtube): Production apps created from this template with manual deployment model

**Philosophy:** "Your videos, your user experience, on your apps!" - Each content distributor maintains their own app store presence, review processes, and content responsibility.

## Testing Changes

**There is no support for testing branded apps locally.** To test configuration changes:

1. **Commit changes** to `.customizations` file or assets
2. **Trigger build** via GitHub Actions workflow_dispatch
3. **Test the build:**
   - **Web:** Automatic deployment to GitHub Pages after workflow completes
   - **iOS/tvOS:** Upload to TestFlight (enable option in workflow), test via TestFlight app
   - **Android:** Upload to Google Play Internal Test track (select in workflow), test via Google Play
   - **APK:** Download APK artifacts from workflow run, sideload for testing

## Developing with Custom Features

**To develop new features or experiment with customizations that require upstream code changes, use a branded app repo together with a fork of the OwnTube web-client:**

1. **Fork the web-client:** Create your own fork of [OwnTube-tv/web-client](https://github.com/OwnTube-tv/web-client)
2. **Develop locally in fork:** Work in your forked web-client repository with full local development support (npm start, npm test, etc.)
3. **Point branded app to fork:** Edit `.github/workflows/*.yml` in this branded app repo to use your fork:
   ```yaml
   # Change from:
   owntube_source: OwnTube-tv/web-client

   # To:
   owntube_source: your-username/web-client
   ```
4. **Test integration:** Trigger workflows in the branded app repo to build with your forked code + branded customizations
5. **Test on devices:** Use TestFlight/Google Play Internal Test to verify features work with branding applied
6. **Contribute upstream:** Once features are working, submit PR to OwnTube-tv/web-client
7. **Restore to canonical:** After upstream merge, revert workflows back to `owntube_source: OwnTube-tv/web-client`

This workflow lets you develop features locally in the forked web-client, then test the integration with your branded app configuration through CI/CD before contributing changes upstream.

## Code Style & Commits

- **Commit Messages:** NEVER mention AI agents, Claude, or AI assistance in commit messages. Write commits as if they were written by a human developer.
- **Formatting:** Follow conventions from the upstream repository for consistency

For comprehensive code style guidelines, see the [upstream CLAUDE.md](https://github.com/OwnTube-tv/web-client/blob/main/CLAUDE.md).

## Important Notes

- **No local development:** This repo has no source code to run locally. Testing happens via GitHub Actions + TestFlight/Google Play.
- **Manual deployments only:** Branded apps deploy on their own schedule, not automatically on every commit.
- **Build times:** Full multi-platform builds take 30-60+ minutes (iOS/tvOS require macOS runners).
- **Single-instance UX:** `EXPO_PUBLIC_PRIMARY_BACKEND` + hidden instance switcher UI creates single-instance experience for App Store/Google Play review requirements, but deep links with different backends still work technically.
- **Customization testing:** The [mykhailodanilenko/web-client](https://github.com/mykhailodanilenko/web-client) fork ("Misha Tube") serves as a testing vehicle for customization mechanism with continuous deployment.

## For More Information

- **Architecture & development patterns:** [OwnTube-tv/web-client CLAUDE.md](https://github.com/OwnTube-tv/web-client/blob/main/CLAUDE.md)
- **Customization options:** [docs/customizations.md](https://github.com/OwnTube-tv/web-client/blob/main/docs/customizations.md)
- **CI/CD & code signing setup:** [docs/pipeline.md](https://github.com/OwnTube-tv/web-client/blob/main/docs/pipeline.md)
- **Diagnostics/analytics:** [docs/diagnostics.md](https://github.com/OwnTube-tv/web-client/blob/main/docs/diagnostics.md)
- **Project website:** [www.owntube.tv](https://www.owntube.tv)
- **Example branded apps:** See "Example Branded Apps" section in [upstream CLAUDE.md](https://github.com/OwnTube-tv/web-client/blob/main/CLAUDE.md)
