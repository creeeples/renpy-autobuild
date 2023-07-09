# Renpy Auto Build

This GitHub action allows you to automate the creation of Renpy builds. This builds all distribution types and then uploads them to the corresponding 
Git Release.

## Setup 
  1. Create a new workflow by clicking 'Actions' in the Repository View of GitHub Web.
  2. Select 'New Workflow' in the top-left corner of the actions page.
  3. Click 'Set Up a Workflow Yourself'
  4. Insert the contents of the action.yml into the file.
  5. Update 'SDK-Version' to your version of Renpy.
  6. Update 'Project-Dir' to the directory containing your Renpy game files.

## Usage
**Instructions**
After setting up the action in your GitHub repository, kicking off the action is simply. This action makes a few assumptions
  1. You have a Master/Main/Develop branch that you work and branch off of.
  2. You follow semantic versioning standards (v X.X.X) where X is a numerical value.

If so, running this action is quite simple. To kick off the action, you simply will want to create a branch in GitHub following the format release/X.X.X . When doing this, GitHub will automatically kick off the action, build your files, create a release, and then upload the assets to that newly created release.

NOTE: The action can be modified to utilize any kind of prefix you would like. Please note that you MUST follow the versioning rules where it is `<prefix>/X.X.X` otherwise the action will not properly run!

**Required Parameters:**
- `sdk-version`: The version of the Ren'Py SDK to use while building. This will default to v8.1.1 of Renpy if not specified.
- `project-dir`: The directory where the project exists. Will default to `'.'` (root) if none is found. Typically, you will want to have this set to ./game/ or the directory that you would normally run build actions in the Ren'Py client.

**Modifiable Parameters:**
- `package`: Specific package(s) to build for. Must be one of the following supported Ren'Py build packages: `pc`, `mac`, `linux`, `market`, `web`, `android`. If a package is not detected or supported, all will be built and uploaded.
- `branches`: The trigger prefix may be updated if repository owners so choose. NOTE: This must be updated in multiple areas.
- `sdk-version`: Specific version of Ren'Py to build with.
- `project-dir`: Directory that houses the files Ren'Py would normally build.

### Outputs
- `dir`: The directory where the files were built to.
- `version`: The name of the project and version that was built. Often useful in cases where you don't know the version.
