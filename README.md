# How to publish plugin on scpslplugins.killers.dev website

1. Your plugin needs to be on github repository which is publicly visbile.
2. Go to your plugin repository and go to **Actions** tab
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/a50ca3bf-39b4-42bf-922d-d8425b6d8856)
3. Press **set up a workflow yourself**
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/c2d4cebf-ef27-4a1f-93f1-b25655a0c4ef)
4.
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/70b75dc0-fd65-49ff-b037-45cd985babf1)
   - 1 Change file name for example **Listing.yml**
   - 2 Paste this code below
   - 3 Commit changes
```yaml
name: Build Listing

env:
  websiteDirectory: Website

on: 
  workflow_dispatch:
  workflow_run:
    workflows: [Build Release]
    types:
      - completed
  release:
     types: [published, created, edited, unpublished, deleted, released]

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Main Repository.
      uses: actions/checkout@v2
    - name: Build listing
      uses: killers0992/scpsl.pluginlisting@master
      with:
        token: ${{ secrets.GITHUB_TOKEN }}
    - name: Setup Pages
      uses: actions/configure-pages@f156874f8191504dae5b037505266ed5dda6c382
    - name: Upload Pages Artifact
      uses: actions/upload-pages-artifact@a753861a5debcf57bf8b404356158c8e1e33150c
      with:
        path: ${{ env.websiteDirectory }}
    - name: Deploy to GitHub Pages
      id: deployment
      uses: actions/deploy-pages@9dbe3824824f8a1377b8e298bafde1a50ede43e5
```
5. Go again to **Actions** tab and press **New workflow**
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/4d4549c1-7323-4ec7-acaf-b75009073010)
6. Press **set up a workflow yourself**
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/fdf3c05d-f422-414e-b66f-94cc1865a1cc)
   - 1 Change file name for example **release.yml**
   - 2 Paste this code below but make sure to modify it for your project.
   - 3 Commit changes

Examples:

When your project **.csproj** is not in root of your github repo which you want to publish then as **PROJECT_NAME** use /Agents
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/e1027cab-a4ad-4b22-acd4-fe56cb5a1913)
But if your **.csproj** is in your root then set **PROJECT_NAME** to PROJECT_NAME: **""**

** ASSEMBLY_NAME** in most cases its just name of project like **Agents**

```yaml
name: Build Release

on: 
  workflow_dispatch:

env:
  NET_FRAMEWORK_VERSION: "net48"
  PROJECT_NAME: "<YOUR PROJECT NAME>"
  ASSEMBLY_NAME: "<YOUR ASSEMBLY NAME>"
  SL_REFERENCES: "${{ github.workspace }}/References"
  UNITY_REFERENCES: "${{ github.workspace }}/References"

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
    - name: Checkout main repository.
      uses: actions/checkout@v4
    - name: Get Release Info
      id: release-info
      uses: zoexx/github-action-json-file-properties@b9f36ce6ee6fe2680cd3c32b2c62e22eade7e590
      with: 
          file_path: "${{ github.workspace }}/releaseInfo.json"
    - name: Set Environment Variables
      run: |
        echo "version=${{ steps.release-info.outputs.version }}" >> $GITHUB_ENV
        echo "gameAssemblyReferences=${{ steps.release-info.outputs.gameAssemblyReferences }}" >> $GITHUB_ENV
        echo "publicizeAssemblies=${{ steps.release-info.outputs.publicizeAssemblies }}" >> $GITHUB_ENV
    - name: Download SCP SL References
      uses: killers0992/scpsl.downloadfiles@master
      with:
        branch: 'public'
        filesToDownload: ${{ env.gameAssemblyReferences }}
    - name: Publicize Assemblies
      uses: killers0992/scpsl.assemblypublicizer@master
      with:
        assemblies: ${{ env.publicizeAssemblies }}
    - name: Setup .NET
      uses: actions/setup-dotnet@v2
      with:
        dotnet-version: 8.0.x
    - name: Build
      run: dotnet build --configuration Release
    - name: Upload
      uses: actions/upload-artifact@v3
      with:
        name: Agents
        path: ${{ github.workspace }}${{ env.PROJECT_NAME }}/bin/Release/${{ env.NET_FRAMEWORK_VERSION }}/${{ env.ASSEMBLY_NAME }}.dll
    - name: Create Tag
      id: tag_version
      uses: mathieudutour/github-tag-action@v6.1
      with:
        github_token: "${{ secrets.GITHUB_TOKEN }}"
        tag_prefix: ""
        custom_tag: "${{ env.version }}"
    - name: Make Release
      uses: softprops/action-gh-release@de2c0eb89ae2a093876385947365aca7b0e5f844
      with:
        files: |
          ${{ github.workspace }}${{ env.PROJECT_NAME }}/bin/Release/${{ env.NET_FRAMEWORK_VERSION }}/${{ env.ASSEMBLY_NAME }}.dll
          ${{ github.workspace }}/releaseInfo.json
        tag_name: ${{ env.version }}
```

7. Go to repository **settings**
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/cf733122-1158-4093-8048-a73e8be6ffb0)
8. Select **pages** tab
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/35bfe870-1929-4c43-8b3d-4b1d7d35774d)
9. Here change **Source** to **Github Actions**
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/6696cad3-4f13-46f1-a41c-6b2bccd28e38)

10. Go to **Code** page, create new file in github repository with name **releaseInfo.json** and paste this text below and make sure to edit this with your plugin information then just commit changes.
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/9f15bdba-57fc-43c8-9c36-3cb225396497)
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/0a4406a5-7bda-4c9b-874d-e2f573dcc146)

```json
{
  "displayname": "<DISPLAY NAME>",
  "description": "<DESCRIPTION>",
  "authorName": "<AUTHOR>",
  "buildType": 0,
  "version": "<VERSION>",
  "scpslVersion": "<GAME VERSION>",
  "loaderType": "<LOADER TYPE>",
  "gameAssemblyReferences": "<GAME REFERENCES>",
  "publicizeAssemblies": "<ASSEMBLIES TO PUBLICIZE>"
}
```
Example [releaseInfo.json](https://raw.githubusercontent.com/Killers0992/Agents/master/releaseInfo.json)

Build Types
   - 0 - **RELEASE**
   - 1 - **Pre Release**
   - 2 - **Beta**
   - 3 - **Alpha**

Supported loader types:
 - **nwapi**
 - **exiled**
 - **labapi**

Game assembly references format its just FileName with extension seperated by comma.

Example

**Assembly-CSharp.dll,Mirror.dll**


Publicize assemblies format also works same as Game assembly references

**Assembly-CSharp.dll** -> outputs **Assembly-CSharp-Publicized.dll**

11. Your **.csproj** file also needs to use env variables like
- SL_REFERENCES

Example [.csproj](https://github.com/Killers0992/Agents/blob/b44e3574b4494b3b1fd96e59f95386976b9124a1/Agents/Agents.csproj#L17)

```yaml
<?xml version="1.0" encoding="utf-8"?>
<Project Sdk="Microsoft.NET.Sdk">
	<PropertyGroup>
		<TargetFramework>net48</TargetFramework>
		<PlatformTarget>x64</PlatformTarget>
		<OutputType>Library</OutputType>

		<LangVersion>latest</LangVersion>

		<GenerateAssemblyInfo>false</GenerateAssemblyInfo>

		<AllowUnsafeBlocks>True</AllowUnsafeBlocks>
	</PropertyGroup>

	<ItemGroup>
		<Reference Include="Mirror" HintPath="$(SL_REFERENCES)\Mirror.dll" />
		<Reference Include="Pooling" HintPath="$(SL_REFERENCES)\Pooling.dll" />
		<Reference Include="PluginAPI" HintPath="$(SL_REFERENCES)\PluginAPI.dll" />
		<Reference Include="Assembly-CSharp-Publicized" HintPath="$(SL_REFERENCES)\Assembly-CSharp-Publicized.dll" />
		<Reference Include="CommandSystem.Core" HintPath="$(SL_REFERENCES)\CommandSystem.Core.dll" />
		<Reference Include="UnityEngine.AIModule" HintPath="$(UNITY_REFERENCES)\UnityEngine.AIModule.dll" />
		<Reference Include="UnityEngine.CoreModule" HintPath="$(UNITY_REFERENCES)\UnityEngine.CoreModule.dll" />
		<Reference Include="UnityEngine.PhysicsModule" HintPath="$(UNITY_REFERENCES)\UnityEngine.PhysicsModule.dll" />
		<Reference Include="UnityEngine.TerrainModule" HintPath="$(UNITY_REFERENCES)\UnityEngine.TerrainModule.dll" />
	</ItemGroup>
</Project>
```

12. After doing everything properly you should be able to just go to **Actions** page and there press **Build release** -> **Run workflow**
![image](https://github.com/Killers0992/SCPSL.Plugins.Guide/assets/38152961/9d1fd986-5737-49cf-b183-28defd5379fa)

13. When build completes you should be able to add your plugin at https://scpslplugins.killers.dev/dashboard/publish but before this you need to login via https://scpslplugins.killers.dev/login



