# Parallel FOSSA Analysis for Large RPMs

This GitHub Actions workflow allows you to analyze large `.src.rpm` packages with [FOSSA CLI](https://github.com/fossas/fossa-cli) by splitting the extracted contents into smaller fragments (≤200MB) and scanning them in parallel.

---

## 🛠 What It Does

1. **Extracts** the specified `.src.rpm` file from Fedora EPEL.
2. **Splits** the contents into multiple 200MB-or-less directories ("fragments").
3. **Generates** individual `fossa-deps.yml` entries for each fragment.
4. **Scans** each fragment in parallel using the FOSSA CLI.
5. **Merges** the fragment entries and runs a full `fossa analyze`.

---

## 🚀 Usage

Trigger the workflow manually via the **Actions > Parallel FOSSA Analysis for Large RPMs > Run workflow** button.

### Required Input

- **rpm** (string): The full filename of the `.src.rpm` to analyze, for example: chromium-133.0.6943.141-1.el8.src.rpm


---

## 📦 Workflow Overview

### Step 1: `extract-and-split`

- Downloads and extracts the `.src.rpm`.
- Splits files into ≤200MB chunks (to avoid FOSSA file upload limits).
- Generates a JSON array of fragments with `name`, `path`, and `version`.
- Uploads:
- Fragment directories
- Metadata file (`deps.json`)

### Step 2: `scan-fragments`

- Runs in parallel per fragment.
- Downloads and prepares a mini `fossa-deps` entry for each fragment.
- Uploads the resulting `.yml` file.

### Step 3: `merge-and-analyze`

- Downloads all `.yml` fragments and combines them into one `fossa-deps.yml`.
- Installs the latest FOSSA CLI.
- Runs `fossa analyze` using the merged dependency file.
- Uploads final output.

---

## 📁 Outputs

- `fossa-deps.yml`: Final vendored dependency list for the RPM.
- `extracted-fragments/`: All split directories.
- `fossa-deps-entries/`: Individual fragment `.yml` files.
- `fossa-fragments-meta/`: Metadata JSON of fragments.

---

## 🔐 Secrets

Ensure your repository has the following GitHub secret:

- `FOSSA_API_KEY`: A valid API key to authenticate with FOSSA.

---

## ⚠️ Notes

- This workflow assumes the `.src.rpm` is available from the Fedora EPEL v8 repository and begins with `c/` (modify the path if analyzing different packages).
- It's designed for large RPMs like Chromium that exceed single-directory limits.
- Each fragment must be independently analyzable as a vendored source.
