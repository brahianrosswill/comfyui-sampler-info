# Release checklist

Internal notes for shipping a new version. Not user-facing — feel free
to delete from the published repo once the workflow is muscle memory.

## One-time setup

### 1. Create the GitHub repo

The scaffolding step has already done `git init`, made the initial
commit on `feature/initial-scaffold` (branch-protection hook blocks
commits straight to `main`), and set `origin` to point at
`https://github.com/laurigates/comfyui-sampler-info.git`.

Two steps to publish:

```sh
cd /mnt/sabrent/ComfyUI/custom_nodes/comfyui-sampler-info

# Rename the feature branch to main (no actual main commit exists, so
# nothing is lost). This is a metadata-only operation.
git branch -M feature/initial-scaffold main

# Create the GitHub repo and push.
gh repo create laurigates/comfyui-sampler-info \
    --public \
    --source . \
    --description "Rich metadata + fuzzy-search picker for ComfyUI sampler/scheduler dropdowns" \
    --push
```

### 2. Register a Comfy Registry publisher

1. Sign in at https://registry.comfy.org/ (GitHub auth).
2. **Create publisher** → pick a publisher ID (e.g. `laurigates`).
3. Edit `pyproject.toml`:
   ```toml
   [tool.comfy]
   PublisherId = "laurigates"   # was: "TODO-publisher-id"
   ```
4. Commit that change — leave the version at `0.1.0` for the first
   publish.

### 3. Issue + store the Registry access token

1. https://registry.comfy.org/ → publisher dashboard → **API Keys** →
   *New token*. Scope: publishing.
2. In the GitHub repo: **Settings → Secrets and variables → Actions →
   New repository secret**:
   - Name: `REGISTRY_ACCESS_TOKEN`
   - Value: paste the token
3. Verify in `.github/workflows/publish.yml` that the secret name
   matches.

### 4. First publish

Pushing the `[tool.comfy] PublisherId` change to `main` triggers the
workflow automatically (the `pyproject.toml` path filter fires).

To trigger manually instead:

```sh
gh workflow run "Publish to Comfy Registry"
gh run watch
```

The action publishes the current `version` to
`registry.comfy.org/nodes/comfyui-sampler-info`.

### 5. ComfyUI Manager listing

Independent of the Comfy Registry. PR to
[Comfy-Org/ComfyUI-Manager](https://github.com/Comfy-Org/ComfyUI-Manager)
adding an entry to `custom-node-list.json`:

```json
{
  "author": "Lauri Gates",
  "title": "Sampler Info",
  "id": "comfyui-sampler-info",
  "reference": "https://github.com/laurigates/comfyui-sampler-info",
  "files": [
    "https://github.com/laurigates/comfyui-sampler-info"
  ],
  "install_type": "git-clone",
  "description": "Rich metadata + fuzzy-search picker dialog for ComfyUI's sampler/scheduler combo widgets. Tooltip with year/family/ODE order/good-for, plus a centered modal that replaces the cryptic native dropdown."
}
```

Place alphabetically by `id`. Validate the JSON before pushing — a
missing comma breaks the whole list for everyone.

## Per-release

1. **Edit `pyproject.toml` `version`** — semver. Bump patch for corpus
   updates, minor for new picker features, major for breaking changes
   to the corpus schema or extension API.
2. **Commit** the version bump alone (or as part of a release commit).
3. **Push to `main`** — the workflow auto-publishes.
4. **Tag** for human reference:
   ```sh
   git tag -a v0.2.0 -m "v0.2.0 — short description"
   git push origin v0.2.0
   ```
5. **GitHub release** (optional but nice):
   ```sh
   gh release create v0.2.0 --generate-notes
   ```

## Things that don't need a release

- README / docs typo fixes — push to `main`, no version bump needed.
- Adding entries to `web/data/samplers.json` / `schedulers.json` — push
  to `main`. **Until users update**, only published-version users see
  the new entries. Bump the patch version when there's a meaningful
  batch of corpus additions.
