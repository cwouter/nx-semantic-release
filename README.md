<p style="text-align: center;"><img src="https://raw.githubusercontent.com/cwouter/nx-semantic-release/main/assets/nx.png" 
width="200" alt="Nx - Smart, Fast and Extensible Build System"></p>

# Nx + semantic release pipeline

**Prerequisites**
- Create personal access token on github
- Create NPM access token (automation)

### 1. Build all or affected
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

### 2. Trigger semantic release
_file: .github/workflows/main.yml_

```yaml
- name: Publish all packages
  run: npx semantic-release
```

### 3. Semantic release configuration
_file: .releaserc_

- Eslint config
- Github 

...
