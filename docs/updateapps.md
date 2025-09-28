---
layout: doc
---
# Update supported apps

## Using Update-Evergreen

After the Evergreen module is installed, the supported application functions need to be downloaded with `Update-Evergreen`.

::: warning
The Evergreen module requires the application functions and manifests to work. Ensure that you run `Update-Evergreen` as the first action after installing Evergreen.
:::

A new function has been added to Evergreen named `Update-Evergreen`. This downloads the latest release from the [evergreen-apps](https://github.com/EUCPilots/evergreen-apps) repository, unpacks the files and stores them locally.

![Running Update-Evergreen for the first time](/img/update-evergreen.gif)

This function supports the `-Force` parameter to force the download of the latest release of the per-application functions even if you already have these locally. This approach should enable the administrator to update Evergreen where the locally cached copies of the functions are perhaps broken.

![Running Update-Evergreen with the -Force parameter](/img/update-evergreen-force.gif)

When importing Evergreen where the local cache is out of date, you will be prompted to update:

![Importing Evergreen and being prompted to run Update-Evergreen](/img/import-evergreen.gif)

## Cache location

Evergreen stores application functions and manifests in the following locations:

* Windows - `%LocalAppData%\Evergreen`
* macOS and Linux - `~\.evergreen`

The current cache location can be returned with `Get-EvergreenAppsPath`:

```powershell
PS C:\> Get-EvergreenAppsPath
C:\Users\aaron\AppData\Local\Evergreen
```

## Setting a custom cache location

A custom cache location can be set with the `EVERGREEN_APPS_PATH` environment variable. Configure this variable in the user or system context and ensure it points to a valid path.

![Setting the EVERGREEN_APPS_PATH on Windows](/img/environment-variable.png)
