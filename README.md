# Renpy Auto Build

This GitHub action allows you to automate the creation of Renpy builds. This builds all distribution types and then uploads them to the corresponding 
Git Release.

```yaml
name: Ren'Py Autobuild + Deploy

on:
  push:
    branches:
      - 'release/*'

jobs:
  build-renpy:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout repository
      uses: actions/checkout@v2

    - name: Extract Version from Branch Name
      id: extract_version
      run: |
          echo "old GITHUB_REF is $GITHUB_REF"
          GITHUB_REF=$(echo $GITHUB_REF | sed -e "s#refs/heads/release/##g")
          echo "::set-output name=ref::$GITHUB_REF"
      env:
        GITHUB_REF: ${{ github.ref }}

    - name: Format Release Name
      id: format_release_name
      run: |
        echo "::set-output name=release_name::Release-${{ steps.extract_version.outputs.ref }}"
      env:
        VERSION: ${{ steps.extract_version.outputs.ref }}
        
    - name: Format Release Tag
      id: format_tag_name
      run: |
        echo "::set-output name=tag_name::v${{ steps.extract_version.outputs.ref }}"
      env:
        VERSION: ${{ steps.extract_version.outputs.ref }}
        
    - name: Build VN Project
      id: build_project
      uses: creeeples/renpy-docker-builder@v1.0.1
      with:
        sdk-version: '8.1.1'
        project-dir: './game/'
      env:
        SDL_AUDIODRIVER: dummy
        SDL_VIDEODRIVER: dummy

    - name: Setup GitHub Release
      uses: softprops/action-gh-release@v1
      with:
        token: ${{ github.token }}
        tag_name: ${{ steps.format_tag_name.outputs.tag_name }}
      env:
          GH_TOKEN: ${{ github.token }}

    - name: Upload All Build Folders
      run: |
        for file in ${{ steps.build_project.outputs.dir }}/*.zip; do
        echo "Uploading $file"
        asset_name=$(basename "$file")
        gh release upload ${{ steps.format_tag_name.outputs.tag_name }} "$file" --clobber
        done
      env:
          GH_TOKEN: ${{ github.token }}
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
