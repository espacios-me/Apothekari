# Objective

Document the official Cloudflare Wrangler **Install/Update Wrangler** workflow in a practical way so this repository has a clear reference for the setup logic, the expected commands, the common errors, and the exact reasons Cloudflare recommends local project installation.

## Background

The official Cloudflare page is intentionally short. Its main job is to define the supported installation model:

- install **Wrangler locally in each project**
- run it through your package manager
- use a Node version manager such as **Volta** or **nvm**
- verify the installed version before using it
- update Wrangler using the same dependency-install command used for the initial setup

Official sources used for this document:

- Install/Update Wrangler: https://developers.cloudflare.com/workers/wrangler/install-and-update/
- Wrangler commands: https://developers.cloudflare.com/workers/wrangler/commands/
- General Wrangler commands: https://developers.cloudflare.com/workers/wrangler/commands/general/
- Wrangler v2 to v3 migration: https://developers.cloudflare.com/workers/wrangler/migration/update-v2-to-v3/
- Wrangler v3 to v4 migration: https://developers.cloudflare.com/workers/wrangler/migration/update-v3-to-v4/

## Method

### 1. What the official page literally says

The page describes Wrangler as:

> Wrangler is a command-line tool for building with Cloudflare developer products.

It then instructs you to ensure **Node.js** and **npm** are installed, preferably through **Volta** or **nvm**. Cloudflare explains that a version manager helps avoid permission issues and makes it easier to change Node.js versions.

The page includes a **Wrangler System Requirements** section stating that:

- Wrangler supports the **Current, Active, and Maintenance** Node.js release lines.
- Your Worker runs in **workerd**, not in Node.js itself.
- Wrangler is supported on **macOS 13.5+**, **Windows 11**, and Linux distributions that support **glib 2.35**.

### 2. Official installation commands

Cloudflare’s package-manager tabs render these commands:

```bash
npm i -D wrangler@latest
```

```bash
yarn add -D wrangler@latest
```

```bash
pnpm add -D wrangler@latest
```

### 3. Official version check

```bash
npx wrangler --version
# or
npx wrangler -v
```

### 4. Official update command

Cloudflare uses the same commands for updating:

```bash
npm i -D wrangler@latest
```

```bash
yarn add -D wrangler@latest
```

```bash
pnpm add -D wrangler@latest
```

### 5. Official warning that causes many real-world mistakes

Cloudflare explicitly warns:

> If Wrangler is not installed, running `npx wrangler` will use the latest version of Wrangler.

That warning is easy to miss but extremely important.

## Results

## Why Cloudflare wants local install only

Cloudflare recommends local project installation because it gives each repository its own Wrangler version.

That solves four practical problems:

1. **Reproducibility**
   - Every teammate uses the same Wrangler dependency declared by the repo.
   - CI can use the same version the team uses locally.

2. **Version control per project**
   - One project can stay on one Wrangler version while another project upgrades.
   - You do not have to make your whole machine match a single global CLI version.

3. **Safer debugging**
   - Local install makes it easier to answer: “Which Wrangler version is this project actually using?”
   - That removes many “works on my machine” failures.

4. **Rollback capability**
   - If an update causes issues, you can pin an older Wrangler version in the project and reinstall dependencies.

## Why global install is discouraged

The official page does not spend time teaching global installation. That omission is deliberate.

Global installs usually create one or more of these issues:

- version drift between projects
- confusion between the global `wrangler` binary and the local project dependency
- permission failures when using system-wide npm directories
- harder debugging because the active CLI version depends on the machine instead of the repository

## Why Node version managers matter

Cloudflare specifically mentions **Volta** and **nvm** because many “Wrangler install problems” are really **Node/npm environment problems**.

Using a version manager helps by:

- avoiding `EACCES` permission errors
- making it easy to switch Node versions
- keeping the Node toolchain in user space instead of protected system directories

## How to run Wrangler correctly after installation

Because Cloudflare recommends local installation, the practical command differs by package manager.

### npm

```bash
npx wrangler dev
npx wrangler deploy
npx wrangler init
npx wrangler login
```

### yarn

```bash
yarn wrangler dev
yarn wrangler deploy
yarn wrangler init
yarn wrangler login
```

### pnpm

```bash
pnpm wrangler dev
pnpm wrangler deploy
pnpm wrangler init
pnpm wrangler login
```

## Recommended `package.json` scripts

Cloudflare’s command docs explicitly recommend adding frequently used Wrangler commands to your project scripts.

```json
{
  "scripts": {
    "dev": "wrangler dev",
    "deploy": "wrangler deploy"
  }
}
```

Then run them like this:

```bash
npm run dev
npm run deploy
```

## Common Errors & Fixes

The official install page is intentionally minimal, so the table below combines Cloudflare’s guidance with the real npm/Node/Wrangler issues people commonly hit while following it.

> Note: not every exact error string below appears on the official page. These are common real-world symptoms around the setup flow the page documents.

| Error or symptom | Why it happens | Fix |
| --- | --- | --- |
| `npm: command not found` | Node.js/npm is not installed or not available on PATH | Install Node.js first, preferably using Volta or nvm, then verify with `node -v` and `npm -v`. |
| `node: command not found` | Node.js is missing | Install a supported Node version and reopen the terminal. |
| `npm ERR! code EACCES` / `permission denied` | npm is writing to a protected location, often because of a system-wide install or global package usage | Stop relying on global installs. Use Volta or nvm. Then install Wrangler locally with `npm i -D wrangler@latest`. |
| `wrangler: command not found` | Wrangler is installed locally, not globally | Run it with `npx wrangler ...`, `yarn wrangler ...`, `pnpm wrangler ...`, or package scripts. |
| `npx wrangler` suddenly behaves differently than expected | Wrangler is not installed locally, so `npx` fetched the latest available version | Install Wrangler into the project first, then re-run `npx wrangler --version`. |
| Unsupported Node or engine errors | The current Node version is outside Cloudflare’s supported policy | Upgrade to a currently supported Node release line, reinstall dependencies, and check `node -v`. |
| Teammates see different Wrangler behavior | Different local or global versions are being used | Commit `package.json` and the lockfile, install dependencies from the repo, and verify with `npx wrangler --version`. |
| Update appears not to work | You may be in the wrong directory, using a stale lockfile, or invoking a different install than you think | Check the current repo, verify with `npx wrangler --version`, and reinstall dependencies if necessary. |

## Step-by-step fixes for the most common failures

### Missing Node or npm

```bash
node -v
npm -v
```

If either command fails, install Node.js with a version manager and try again.

### Permission denied / EACCES

```bash
npm uninstall -g wrangler
npm i -D wrangler@latest
npx wrangler --version
```

If npm itself still throws permission errors, fix the Node installation first by moving to Volta or nvm.

### Wrangler command not found

```bash
npx wrangler --version
npx wrangler dev
npx wrangler deploy
```

## Update logic

Updating Wrangler is the same operation as installing it because npm-style dependency management uses the same install command to refresh a package version.

### Upgrade to the newest version

```bash
npm i -D wrangler@latest
```

### Pin a specific major version

```bash
npm i -D wrangler@4
```

### Pin an exact version

```bash
npm i -D wrangler@<exact-version>
```

That is useful when you want to:

- avoid a major upgrade during active development
- align every team member with a fixed version
- roll back after a regression

## Recommended workflow from zero

```bash
# verify Node and npm
node -v
npm -v

# create project
mkdir my-worker
cd my-worker
npm init -y

# install Wrangler locally
npm i -D wrangler@latest

# verify version
npx wrangler --version

# initialize a project
npx wrangler init

# log in
npx wrangler login

# develop locally
npx wrangler dev

# deploy
npx wrangler deploy
```

## Extra best practices

- Commit `package.json` and your lockfile (`package-lock.json`, `yarn.lock`, or `pnpm-lock.yaml`).
- Treat Wrangler as a **project dependency**, not a machine dependency.
- Use local scripts for day-to-day commands.
- Use a Node version manager if you work across multiple projects or teams.
- For multiple Cloudflare repos on one machine, keep each repo’s Wrangler version local to that repo.

## What the official page deliberately does not cover

The current page is intentionally focused. It does **not** teach:

- global `npm install -g wrangler`
- Homebrew installation
- Cargo installation
- a machine-wide preferred binary strategy

That is consistent with the page’s philosophy: Wrangler should live inside the project’s dependency graph.

## Migration notes

Cloudflare’s official Wrangler v2 to v3 migration page says there are **no special migration instructions** beyond following the normal install/update flow.

Cloudflare’s official Wrangler v3 to v4 migration page highlights these relevant changes:

- Node.js v16 is no longer supported in Wrangler v4.
- Wrangler v4 follows Node.js’s official lifecycle policy.
- Some commands now default to **local mode** and require `--remote` when remote access is intended.
- Several deprecated commands and configurations were removed.

## Verification checklist

Use this list to confirm your setup matches Cloudflare’s intended workflow:

- [ ] Node.js is installed
- [ ] npm, yarn, or pnpm is available
- [ ] Node is managed by Volta or nvm, or another user-space version manager
- [ ] Wrangler is installed in the project, not globally
- [ ] `npx wrangler --version` reports the expected version
- [ ] frequently used Wrangler commands are added to `package.json` scripts
- [ ] lockfile is committed for team reproducibility

# Conclusion

The official Cloudflare page is short because it is setting policy, not trying to be a giant troubleshooting manual.

Its core message is simple:

- install Wrangler **locally inside each project**
- run it through your package manager
- use a Node version manager to avoid environment problems
- verify the active version
- update using the same dependency command you used to install it

Following that model gives teams repeatability, cleaner debugging, safer upgrades, and fewer permission problems. That is the real logic behind the page, and it is the right default for this repository whenever Cloudflare Workers tooling is involved.
