<p style="text-align: center;"><img src="https://raw.githubusercontent.com/cwouter/nx-semantic-release/main/assets/nx.png" 
width="200" alt="Nx - Smart, Fast and Extensible Build System"></p>

# Nx + semantic release pipeline

**Prerequisites**
- Create personal access token on github
- Create NPM access token (automation)

## 1. Build all or affected
_file: .github/workflows/main.yml_

```yaml
- name: Build all packages
  run: npx nx run-many --target=build --all
```

```yaml
- name: Build affected packages
  run: npx nx affected --target=build
```

In case of 'affected' flow: don't forget to set the BASE and HEAD for Nx.
More info: https://github.com/marketplace/actions/nx-set-shas

## 2. Trigger semantic release
_file: .github/workflows/main.yml_

```yaml
- name: Publish all packages
  run: npx semantic-release
```

## 3. Configure semantic release
_file: .releaserc_

## 3.1 Commit analyzer
Analyzes the commits and determines the version bump that will be applied according to the configured rules
Ex: "Docs: added commit analyzer to README.md" will trigger a patch version bump

```json5
{
  plugins: ["@semantic-release/commit-analyzer", {
    "preset": "ESLint",
    "releaseRules": [{ tag: 'Docs', release: 'patch' }]
  }]
}
```

## 3.2 Release notes & changelog
Creates a formatted changelog file based on the commit messages (according the ESLint rules)


```json5
{
  plugins: [
    ["@semantic-release/release-notes-generator", {"preset": "ESLint"}],
    ["@semantic-release/changelog", {"changelogFile": "CHANGELOG.md"}]
  ]
}
```

## 3.3 Bump version & publish 
Semantic release triggers an Nx `pre-release` and Nx `release` script. The actual scripts are defined in the `project.json` files 

_file: .releaserc_

```json5
{
  plugins: ["@semantic-release/exec", {
    "prepareCmd": "nx run-many --target=pre-release --all --args=\"--version=${nextRelease.version}\"",
    "publishCmd": "nx run-many --target=release --all"
  }],
}
```

In case of 'affected' flow: use BASE and HEAD instead of `--all`

_file: packages/\_\_PACKAGE_NAME\_\_/project.json_

```json5
{
  targets: {
    "pre-release": {
      "executor": "@nrwl/workspace:run-commands",
      "options": {
        "commands": [
          "cd packages/core && npm version {args.version} --allow-same-version",
          "cd dist/packages/core && npm version {args.version} --allow-same-version"
        ],
        "cwd": "./",
        "parallel": true
      }
    },
    "release": {
      "executor": "@nrwl/workspace:run-commands",
      "options": {
        "cwd": "./",
        "commands": [
          "cd dist/packages/core && npm publish --access public --dry-run"
        ]
      }
    }
  }
}
```

### Pre-release

The pre-release script will bump the package version in the dist folder as well as in the source folder.
This will make sure that the version in the source code and the published package are both in sync.

### Release

The release script will publish the dist folder to the configured registry.

Note: don't forget to remove `--dry-run`.

## 4. Create a GitHub release

```json5
{
  plugins: [
    "@semantic-release/github",
  ]
}
```

