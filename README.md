# Renpy Auto Build

This GitHub action allows you to automate the creation of Renpy builds. This builds all distribution types and then uploads them to the corresponding 
Git Release.

```yaml
steps:
  - name: Ren'Py Autobuild and Deploy
    uses: creeeples/renpy-autobuild@v1
    with:
      version: '1.0.0'
```

## Setup
  1. Create a new workflow by clicking 'Actions' in the Repository View of GitHub Web.
  2. Select 'New Workflow' in the top-left corner of the actions page.
  3. Click 'Set Up a Workflow Yourself'
  4.  Insert the contents of Renpy-Build.yml into the file.
  5.  Update 'SDK-Version' to your version of Renpy
  6.  Update 'Project-Dir' to the directory containing your Renpy game files

## Usage

**Required Parameters:**

- `sdk-version`: The version of the Ren'Py SDK to use while building. Will default to `7.3.2` if none is found.
- `project-dir`: The directory where the project exists. Will default to `'.'` (root) if none is found.

**Optional Parameters:**

- `package`: Specific package to build for. Must be one of the following: `pc`, `mac`, `linux`, `market`, `web`, `android`. Will build for all packages if value is not supported.

### Outputs

- `dir`: The directory where the files were built to.
- `version`: The name of the project and version that was built. Often useful in cases where you don't know the version.
