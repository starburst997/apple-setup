# Apple Code Signing Setup

A GitHub composite action to automatically set up Apple code signing certificates and provisioning profiles using fastlane match. This action simplifies the process of managing iOS and macOS code signing in CI/CD pipelines.

## Features

- Automated certificate and provisioning profile management
- Support for both iOS and macOS platforms
- Multiple certificate types: App Store, Developer ID, and PKG installer
- Secure storage of certificates in a private Git repository
- Works with both Apple ID session and App Store Connect API authentication

## Prerequisites

1. **Apple Developer Program**: Active enrollment (~$99/year)
2. **Match Repository**: A private Git repository to store your certificates and provisioning profiles
3. **GitHub PAT**: Personal Access Token with repo access to the match repository
4. **App Store Connect API Key**: For certificate management (recommended) OR Apple ID credentials

## Quick Start

### Most Common Scenario: iOS App Store Distribution

This setup allows you to sign iOS apps for TestFlight and App Store distribution:

```yaml
name: Apple Setup
on:
  workflow_dispatch:
jobs:
  apple-setup:
    runs-on: ubuntu-latest
    steps:
      - name: Setup Apple Code Signing
        uses: starburst997/apple-setup@v1
        with:
          generate_ios: true
          gh_pat: ${{ secrets.GH_PAT }}
          match_repository: "your-org/certificates-repo"
          match_password: ${{ secrets.MATCH_PASSWORD }}
          appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
          appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
          appstore_p8: ${{ secrets.APPSTORE_P8 }}
          ios_bundle_id: "com.company.appname"
```

**Required Secrets for this scenario:**

- `GH_PAT` - GitHub Personal Access Token
- `MATCH_REPOSITORY` - Private repo for certificates (e.g., 'myorg/certificates')
- `MATCH_PASSWORD` - Encryption password for certificates
- `APPSTORE_ISSUER_ID` - App Store Connect API Issuer ID
- `APPSTORE_KEY_ID` - App Store Connect API Key ID
- `APPSTORE_P8` - App Store Connect API private key (.p8 file contents)

**Note:** Bundle ID (`com.company.appname`) is provided inline in the workflow, not as a secret.

## Other Usage Examples

### macOS Developer ID Distribution

For distributing macOS apps outside the Mac App Store (direct download, DMG, etc.):

```yaml
- name: Setup Apple Code Signing
  uses: starburst997/apple-setup@v1
  with:
    generate_macos: true
    generate_developer_id: true
    gh_pat: ${{ secrets.GH_PAT }}
    match_repository: "your-org/certificates-repo"
    match_password: ${{ secrets.MATCH_PASSWORD }}
    match_deploy_key: ${{ secrets.MATCH_DEPLOY_KEY }}
    appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
    appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
    appstore_p8: ${{ secrets.APPSTORE_P8 }}
    mac_bundle_id: "com.company.appname"
```

### macOS App Store Distribution

```yaml
- name: Setup Apple Code Signing
  uses: starburst997/apple-setup@v1
  with:
    generate_macos: true
    generate_appstore: true
    gh_pat: ${{ secrets.GH_PAT }}
    match_repository: "your-org/certificates-repo"
    match_password: ${{ secrets.MATCH_PASSWORD }}
    match_deploy_key: ${{ secrets.MATCH_DEPLOY_KEY }}
    appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
    appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
    appstore_p8: ${{ secrets.APPSTORE_P8 }}
    mac_bundle_id: "com.company.appname"
```

### macOS with PKG Installer

For distributing macOS apps as installer packages:

```yaml
- name: Setup Apple Code Signing
  uses: starburst997/apple-setup@v1
  with:
    generate_macos: true
    generate_developer_id: true
    generate_pkg: true
    gh_pat: ${{ secrets.GH_PAT }}
    match_repository: "your-org/certificates-repo"
    match_password: ${{ secrets.MATCH_PASSWORD }}
    match_deploy_key: ${{ secrets.MATCH_DEPLOY_KEY }}
    appstore_issuer_id: ${{ secrets.APPSTORE_ISSUER_ID }}
    appstore_key_id: ${{ secrets.APPSTORE_KEY_ID }}
    appstore_p8: ${{ secrets.APPSTORE_P8 }}
    mac_bundle_id: "com.company.appname"
```

## Inputs

### Required Inputs

| Input              | Description                                                  |
| ------------------ | ------------------------------------------------------------ |
| `gh_pat`           | GitHub Personal Access Token for accessing match repository  |
| `match_repository` | Git repository for storing certificates (format: `org/repo`) |
| `match_password`   | Password to encrypt/decrypt certificates                     |

### Platform Selection

| Input            | Type    | Default | Description                              |
| ---------------- | ------- | ------- | ---------------------------------------- |
| `generate_ios`   | boolean | `false` | Generate iOS certificates and profiles   |
| `generate_macos` | boolean | `false` | Generate macOS certificates and profiles |

### Certificate Types (macOS only)

| Input                   | Type    | Default | Description                         |
| ----------------------- | ------- | ------- | ----------------------------------- |
| `generate_appstore`     | boolean | `false` | Generate App Store certificates     |
| `generate_developer_id` | boolean | `false` | Generate Developer ID certificates  |
| `generate_pkg`          | boolean | `false` | Generate PKG installer certificates |

### Authentication Method

| Input         | Type    | Default | Description                                           |
| ------------- | ------- | ------- | ----------------------------------------------------- |
| `use_session` | boolean | `false` | Use Apple ID session instead of App Store Connect API |

#### App Store Connect API (default)

Required when `use_session` is `false`:

| Input                | Description                               |
| -------------------- | ----------------------------------------- |
| `appstore_issuer_id` | App Store Connect API issuer ID           |
| `appstore_key_id`    | App Store Connect API key ID              |
| `appstore_p8`        | App Store Connect API private key content |

#### Apple ID Session

Required when `use_session` is `true`:

| Input               | Description             |
| ------------------- | ----------------------- |
| `fastlane_user`     | Apple ID email          |
| `fastlane_password` | Apple ID password       |
| `fastlane_session`  | Fastlane session cookie |

### Bundle Identifiers

| Input           | Description                                                                |
| --------------- | -------------------------------------------------------------------------- |
| `ios_bundle_id` | iOS application bundle identifier (required if `generate_ios` is true)     |
| `mac_bundle_id` | macOS application bundle identifier (required if `generate_macos` is true) |

## Setting Up Secrets

### Step 1: Create App in App Store Connect (Optional)

<table align="center"><tr><td>
<a href="docs/images/create_app_01.png" target="_blank"><img src="docs/images/create_app_01.png" alt="New App" title="New App" width="250" /></a><p align="center">1</p>
</td><td>
<a href="docs/images/create_app_02.png" target="_blank"><img src="docs/images/create_app_02.png" alt="New App Form" title="New App Form" width="250" /></a><p align="center">2</p>
</td><td>
<a href="docs/images/create_app_03.png" target="_blank"><img src="docs/images/create_app_03.png" alt="Create Bundle ID" title="Create Bundle ID" width="250" /></a><p align="center">3</p>
</td></tr></table>

If you haven't already created your app:

1. Register your app in [App Store Connect](https://appstoreconnect.apple.com/apps), click on the **(+)** button and select **New App**

2. You'll be asked to pick a **Bundle ID** which can be created in the **Apple Developer** website under the [Certificates, Identifiers & Profiles](https://developer.apple.com/account/resources/identifiers/bundleId/add/bundle) section

3. Remember its value and pick the **Explicit** option (e.g., `com.company.appname`)

### Step 2: Create a Match Repository

1. Create a new **private** GitHub repository (e.g., `myorg/ios-certificates`)
2. This repository will store your encrypted certificates and provisioning profiles
3. Keep this repository **private** to protect your signing certificates
4. The repository can be empty - fastlane match will populate it automatically

**Secret to save:** `MATCH_REPOSITORY` - Format: `org/repo` (e.g., `myorg/ios-certificates`)

### Step 3: Create Match Password

1. Generate a strong random password (use your password manager)
2. This password encrypts/decrypts your certificates in the match repository
3. Save it securely - you'll need it for future projects or manual certificate access

**Secret to save:** `MATCH_PASSWORD` - Your strong encryption password

### Step 4: Generate App Store Connect API Key

<table align="center"><tr><td>
<a href="docs/images/api_key_01.png" target="_blank"><img src="docs/images/api_key_01.png" alt="New API Key" title="New API Key" width="400" /></a><p align="center">1</p>
</td><td>
<a href="docs/images/api_key_02.png" target="_blank"><img src="docs/images/api_key_02.png" alt="New API Key form" title="New API Key form" width="400" /></a><p align="center">2</p>
</td></tr></table>

App Store Connect API Key is the recommended way to authenticate with Apple services.

1. Go to your [Users and Access](https://appstoreconnect.apple.com/access/integrations/api) page on App Store Connect

   - Click the **Integrations** tab
   - Select **App Store Connect API** on the left panel
   - Select the **Team Keys** tab
   - **Note the Issuer ID** at the top of the page

2. Create a new Key:
   - Click the **(+)** button
   - Give it a name (e.g., "GitHub Actions")
   - Select **Admin** access (required for certificate management)
   - Click **Generate**
   - **Download the .p8 file immediately** (you can only download it once!)
   - **Note the Key ID** shown in the key's row

**Secrets to save:**

- `APPSTORE_ISSUER_ID` - The Issuer ID from the API page
- `APPSTORE_KEY_ID` - The Key ID of your generated key
- `APPSTORE_P8` - The **entire contents** of the `.p8` file (including BEGIN/END lines)

### Step 5: Create GitHub Personal Access Token

<table align="center"><tr><td>
<a href="docs/images/pat_01.png" target="_blank"><img src="docs/images/pat_01.png" alt="Generate New Token (Classic)" title="Generate New Token (Classic)" width="400" /></a><p align="center">1</p>
</td><td>
<a href="docs/images/pat_02.png" target="_blank"><img src="docs/images/pat_02.png" alt="Use repo scope" title="Use repo scope" width="400" /></a><p align="center">2</p>
</td></tr></table>

1. Visit your GitHub [Personal Access Tokens](https://github.com/settings/tokens) settings
2. Click **Generate new token** → Select **Generate new token (classic)**
3. Give it a descriptive name (e.g., "Fastlane Match - Project Name")
4. Select scope: **repo** (Full control of private repositories)
5. Set expiration to **No expiration** (or manage renewals yourself)
6. Click **Generate token**
7. **Copy the token immediately** (you won't see it again!)

**Secret to save:** `GH_PAT` - Your generated GitHub Personal Access Token

### Step 6: Add All Secrets to GitHub

Go to your GitHub repository → **Settings** → **Secrets and variables** → **Actions** → **New repository secret**

Add each of these secrets:

**Required secrets (same for iOS and macOS):**

- ✅ `GH_PAT` - GitHub Personal Access Token
- ✅ `MATCH_REPOSITORY` - Format: `org/repo` (e.g., `myorg/ios-certificates`)
- ✅ `MATCH_PASSWORD` - Strong encryption password
- ✅ `APPSTORE_ISSUER_ID` - From App Store Connect API
- ✅ `APPSTORE_KEY_ID` - From App Store Connect API
- ✅ `APPSTORE_P8` - Entire `.p8` file contents

**Important Notes:**

- For `APPSTORE_P8`: Copy the **entire contents** of the `.p8` file, including `-----BEGIN PRIVATE KEY-----` and `-----END PRIVATE KEY-----`
- For `MATCH_PASSWORD`: Use a strong random password and **save it in your password manager** for reuse in other projects
- These secrets can be reused across multiple projects
- **Bundle IDs** are provided inline in your workflow YAML (e.g., `ios_bundle_id: "com.company.appname"`), not as secrets

## How It Works

1. Sets up Ruby and installs fastlane dependencies
2. Runs `fastlane ios init_ci` which uses the `github_action` plugin to generate an SSH deploy key
3. Adds the SSH deploy key to the SSH agent for authentication
4. Runs fastlane match to sync certificates and provisioning profiles
5. Downloads certificates from the match repository (or creates new ones if needed)
6. Installs certificates and profiles in the CI environment

## Troubleshooting

### SSH Key Issues

If you encounter SSH authentication errors:

- Verify the GH_PAT has access to the match repository
- Ensure the token has appropriate permissions (repo access)
- Check that the match repository URL is correct

### Certificate Generation Fails

- Verify your Apple Developer account has appropriate permissions
- Check that the bundle ID matches your app configuration
- Ensure the App Store Connect API key has the necessary permissions

## License

MIT

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.
