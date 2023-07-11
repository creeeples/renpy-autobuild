# Renpy Auto Builder Action

This action is made to help Ren'Py project developers quickly stage files to Itch and Github. 
This action runs on a publish of a branch prefixed with release/ and will stage the files to the appropriate locations. 
Please review the documentation below for setup, troubleshooting, and more! 

If you like my work, consider checking me out on Twitter @Creeeples or at www.creeeples.com. Thanks and feel free to reach out if you encounter any issues!

## Dependencies

This action requires the following dependencies. This is handled by GitHub and does not need any configuration by the user.

- actions/checkout@v2
- creeeples/renpy-docker-builder@latest
- softprops/action-gh-release@v1
- Ayowel/butler-to-itch@v1.0.0

## Setup

### Preparing your Repository
  1. Go to your repository settings and then 'Actions'.
  2. Under Actions and General, make sure that actions and reusable workflows are enabled.
  3. Under Workflow Permissions, make sure that actions can read repository contents and package registry.

### Setting up your Secrets
  1. Navigate to Itch and then go to your user settings.
  2. Go to API Keys and generate a new key.
    - NOTE: KEEP THIS KEY A SECRET! Sharing this key can cause irrepairable damage to your projects and account.
  3. Go to your repository settings and then 'Secrets'.
  4. Add a new secret called BUTLER_KEY and paste your key into the value field.
        - NOTE: You must name the secret BUTLER_KEY or the action will not work properly.

### Adding the Action
  1. Go to your repository and click on the 'Actions' tab.
  2. Click on 'Skips this Step and Set Up a Workflow Yourself'.
  3. Copy and paste the following code into the editor and update the following values...
      - `project-dir` - The directory of your Ren'Py project. This is relative to the root of your repository. 
      The default value is `.` and most projects will not need to be modified -- This assumes that ./game/ is where all your game files are stored. If you used Renpy to initialize your project, you should be ok!
      - itch_user - Your itch.io username. This is the username you use to log into itch.io.
      - itch_game - The name of your game on itch.io. This is the name of the game in the URL.
  4. Click 'Commit Changes...' and commit your changes to the repository.
  5. Action should run whenever a branch prefixed with release/ is pushed to the repository.

```yaml
name: Ren'Py Autobuild + Deploy (Creeeples v1.0.3)

on:
  push:
    branches:
      - 'release/*'
      - 'prerelease/*'

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
          GITHUB_REF=$(echo $GITHUB_REF | sed -e "s#refs/heads/\(release\|prerelease\)/##g")
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
      uses: creeeples/renpy-docker-builder@latest
      with:
        sdk-version: '8.1.1'
        project-dir: '.'
      env:
        SDL_AUDIODRIVER: dummy
        SDL_VIDEODRIVER: dummy

    - name: Setup GitHub Release
      uses: softprops/action-gh-release@v1
      with:
        token: ${{ github.token }}
        tag_name: ${{ steps.format_tag_name.outputs.tag_name }}
        prerelease: ${{ startsWith(github.ref, 'refs/heads/prerelease/') }} # Set prerelease flag conditionally
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
          
    - name: ItchIO Upload
      if: startsWith(github.ref, 'refs/heads/release/') # Skip this step for prerelease branch
      uses: Ayowel/butler-to-itch@v1.0.0
      with:
        action: "push"
        install_dir: ~/.butler

        ###### Push options ######
        # Your butler key (see https://itch.io/user/settings/api-keys)
        # IMPORTANT NOTE: Do NOT Expose Your Butler Key! Doing so can cause VERY significant issues and allow someone to compromise your projects.
        butler_key: ${{ secrets.BUTLER_KEY }}
        
        # The itch username of the user that distributes the game
        itch_user: ""
        
        # The name of the game in the project's url
        itch_game: ""
        
        # The game version number
        version: "${{ steps.format_tag_name.outputs.tag_name }}"
        
        # The files to push to itch.
        # File paths support globing and may start with a channel name.
        # NOTE: ONLY ONE CHANNEL IS ALLOWED AT THIS TIME. 
        files: |
          mac ${{ steps.build_project.outputs.dir }}/*-mac.zip
          linux ${{ steps.build_project.outputs.dir }}/*-linux.tar.bz2
          windows ${{ steps.build_project.outputs.dir }}/*-pc.zip
          
        # If no channel is provided for a file, to generate one from the
        # file's name.
        # NOTE: Enabling this may break the integration! Please use defined file types!
        auto_channel: false

        ###### Install options ######
        # Whether to verify the downloaded Butler archive's signature
        check_signature: true
        
        # Whether to update the PATH variable to include Butler's
        # install directory
        update_path: false
        butler_version: "latest"
```

### Running the Action
  1. This action will run whenever a branch prefixed with release/ is pushed to the repository. 
  The format understood by the action is release/x.x.x where x is a number. For example, release/1.0.0 would be a valid branch name.
      - NOTE: The release name must be unique. If you try to push a release with the same name as an existing release, the action will fail. 
  Additionally, the branch name should follow semantic versioning where the first number is the major version, 
  the second number is the minor version, and the third number is the patch version. Lastly, it is recommended that you DO NOT capitalize the 
  first letter of prefix -- this can cause significant issues with Git. If you have done this, feel free to
  DM me on Twitter and I am more than happy to help you fix.
  2. Either from the CLI, Application, or Web Interface, create a new branch from the main branch of your repository, prefixed with release/<VERSION NUM>.
  3. Push the branch to your repository if locally creating a branch.
  4. The action should run and build your project -- this may take a few minutes depending on the size of your project and you can check build progress by 
  clicking 'Actions' at the top of your Git repository.
  5. If the action ran successfully, you should see a new release on GitHub and a new build uploaded Itch.io.

## Usage
After setting up the action in your GitHub repository, kicking off the action is simply. This action makes a few assumptions
  1. You have a Master/Main/Develop branch that you work and branch off of.
  2. You follow semantic versioning standards (v X.X.X) where X is a numerical value.

If so, running this action is quite simple. To kick off the action, you simply will want to create a branch in GitHub following the format release/X.X.X . 
When doing this, GitHub will automatically kick off the action, build your files, create a release, and then upload the assets to that newly created release.

NOTE: The action can be modified to utilize any kind of prefix you would like. Please note that you MUST follow the versioning 
rules where it is `<prefix>/X.X.X` otherwise the action will not properly run!

**Required Parameters:**
- `sdk-version`: The version of the Ren'Py SDK to use while building. This will default to v8.1.1 of Renpy if not specified.
- `project-dir`: The directory where the project exists. Will default to `'.'` (root) if none is found. Typically, you will want to have this set to ./game/ or
  the directory that you would normally run build actions in the Ren'Py client.

**Modifiable Parameters:**
- `package`: Specific package(s) to build for. Must be one of the following supported Ren'Py build packages: `pc`, `mac`, `linux`, `market`, `web`, `android`.
  If a package is not detected or supported, all will be built and uploaded.
- `branches`: The trigger prefix may be updated if repository owners so choose. NOTE: This must be updated in multiple areas.
- `sdk-version`: Specific version of Ren'Py to build with.
- `project-dir`: Directory that houses the files Ren'Py would normally build.

### Outputs
- `dir`: The directory where the files were built to.
- `version`: The name of the project and version that was built. Often useful in cases where you don't know the version.

