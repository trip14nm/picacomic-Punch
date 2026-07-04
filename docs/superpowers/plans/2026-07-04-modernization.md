# Picasign Modernization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Upgrade every outdated versioned project component to the latest stable baseline approved in the design while preserving the existing sign-in behavior.

**Architecture:** Keep the single .NET console application unchanged at the business-logic level. Update the project manifest, GitHub Actions workflow, and user documentation independently, then verify restore, compilation, dependency health, startup validation, and YAML structure.

**Tech Stack:** .NET 10, C#, NuGet, GitHub Actions, Python 3 with PyYAML

---

### Task 1: Upgrade the .NET project baseline

**Files:**
- Modify: `picacomic-Punch.csproj`

- [ ] **Step 1: Confirm the pre-upgrade dependency report**

Run:

```powershell
& 'C:\Program Files\dotnet\dotnet.exe' list .\picacomic-Punch.csproj package --outdated --include-transitive
```

Expected: reports `Newtonsoft.Json` 13.0.4 and `picacomic` 1.0.7 as available updates.

- [ ] **Step 2: Replace the project manifest with the approved versions**

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net10.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.4" />
    <PackageReference Include="picacomic" Version="1.0.7" />
  </ItemGroup>

</Project>
```

- [ ] **Step 3: Restore the upgraded project**

Run:

```powershell
& 'C:\Program Files\dotnet\dotnet.exe' restore .\picacomic-Punch.csproj
```

Expected: exit code 0 and assets restored for `net10.0`.

- [ ] **Step 4: Build the upgraded project**

Run:

```powershell
& 'C:\Program Files\dotnet\dotnet.exe' build .\picacomic-Punch.csproj -c Release --no-restore
```

Expected: exit code 0, 0 warnings, and 0 errors.

- [ ] **Step 5: Commit the project upgrade**

```powershell
git add -- picacomic-Punch.csproj
git commit -m "build: upgrade to .NET 10"
```

### Task 2: Upgrade the GitHub Actions runtime

**Files:**
- Modify: `.github/workflows/picacgPunch.yml`

- [ ] **Step 1: Update the workflow action and SDK versions**

Keep all triggers, the timezone, the `ACCOUNTS` secret, and the run command unchanged. Make these exact replacements:

```yaml
- uses: actions/checkout@v6
```

```yaml
- name: .NET
  uses: actions/setup-dotnet@v5
  with:
    dotnet-version: '10.0.x'
```

- [ ] **Step 2: Parse the workflow as YAML**

Run:

```powershell
python -c "import pathlib, yaml; data=yaml.safe_load(pathlib.Path('.github/workflows/picacgPunch.yml').read_text(encoding='utf-8')); assert data['jobs']['build']['steps'][0]['uses'] == 'actions/checkout@v6'; assert data['jobs']['build']['steps'][2]['uses'] == 'actions/setup-dotnet@v5'; assert data['jobs']['build']['steps'][2]['with']['dotnet-version'] == '10.0.x'"
```

Expected: exit code 0 with no assertion or YAML parsing error.

- [ ] **Step 3: Confirm obsolete Action versions are absent**

Run:

```powershell
rg -n "actions/checkout@v[1-5]|actions/setup-dotnet@v[1-4]|dotnet-version:\s*['\"]?[1-9]\." .github/workflows
```

Expected: no matches and `rg` exit code 1.

- [ ] **Step 4: Commit the workflow upgrade**

```powershell
git add -- .github/workflows/picacgPunch.yml
git commit -m "ci: upgrade Actions to Node.js 24"
```

### Task 3: Update the user documentation

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Update runtime and workflow instructions**

Replace the old VS2019 and .NET 5 bullets with:

```markdown
- **每天定时运行**
- **GitHub Actions**
- **.NET 10 LTS**
```

Update product names to `Fork`, `Settings`, `Secrets and variables > Actions`, and `Actions`. Explain that a push affecting files other than `README.md` and `.gitignore` triggers the workflow, while scheduled execution remains automatic. Preserve the `ACCOUNTS` format and existing screenshots.

- [ ] **Step 2: Check stale version text is gone**

Run:

```powershell
rg -n -i "vs2019|\.net5|net5\.0|checkout@v2|setup-dotnet@v1" README.md .github picacomic-Punch.csproj
```

Expected: no matches and `rg` exit code 1.

- [ ] **Step 3: Commit the documentation update**

```powershell
git add -- README.md
git commit -m "docs: update setup instructions"
```

### Task 4: Run full regression verification

**Files:**
- Verify: `picacomic-Punch.csproj`
- Verify: `Program.cs`
- Verify: `.github/workflows/picacgPunch.yml`
- Verify: `README.md`

- [ ] **Step 1: Restore and build from the final project state**

Run:

```powershell
& 'C:\Program Files\dotnet\dotnet.exe' restore .\picacomic-Punch.csproj
& 'C:\Program Files\dotnet\dotnet.exe' build .\picacomic-Punch.csproj -c Release --no-restore
```

Expected: both commands exit 0; build reports 0 warnings and 0 errors.

- [ ] **Step 2: Confirm dependency freshness and health**

Run:

```powershell
& 'C:\Program Files\dotnet\dotnet.exe' list .\picacomic-Punch.csproj package --outdated --include-transitive
& 'C:\Program Files\dotnet\dotnet.exe' list .\picacomic-Punch.csproj package --vulnerable --include-transitive
& 'C:\Program Files\dotnet\dotnet.exe' list .\picacomic-Punch.csproj package --deprecated
```

Expected: no outdated, vulnerable, or deprecated packages are reported.

- [ ] **Step 3: Verify startup and existing missing-configuration behavior**

Run:

```powershell
& 'C:\Program Files\dotnet\dotnet.exe' run --project .\picacomic-Punch.csproj -c Release --no-build
```

Expected: the application starts and exits nonzero with `请查看文档设置账号密码`, demonstrating that the existing argument guard still runs.

- [ ] **Step 4: Check repository diffs and whitespace**

Run:

```powershell
git diff --check HEAD~3..HEAD
git status --short
git log --oneline -5
```

Expected: no whitespace errors, no uncommitted implementation files, and the three implementation commits are present after the design and plan commits.
