
## Product attributes

### UpgradeCode
GUID defining the product across versions. E.g. a previous version is uninstalled during upgrade.
In other words:
For update (or upgrade), Windows Installer relies on the UpgradeCode attribute of the Product tag.
Keep the same UpgradeCode GUID as long as you want the products to be upgraded by the installer.

### Id
[wixtoolset](https://wixtoolset.org/documentation/manual/v3/xsd/wix/product.html): The product code GUID for the product. Type: AutogenGuid

[MS](https://docs.microsoft.com/en-us/windows/win32/msi/product-codes): The product code is a GUID that is the principal identification of an application or product.

[MS](https://docs.microsoft.com/en-us/windows/win32/msi/productcode): This ID must vary for different versions and languages.

[MS](https://docs.microsoft.com/en-us/windows/win32/msi/changing-the-product-code): The product code must be changed if any of the following are true for the update:
- The name of the .msi file has been changed.

[MS](https://docs.microsoft.com/en-us/windows/win32/msi/major-upgrades):
A major upgrade is a comprehensive update of a product that needs a change of the ProductCode Property.
A typical major upgrade removes a previous version of an application and installs a new version.

A constant Product code GUID is (only) useful for a subsequent mst (transform).
To be safe for a major upgrade, the Id (product code GUI) is dynamic/autogenerated: * (star)

Therefore: we use dynamic/autogenerated: * (star)


## Conditions (for install)

[doc](https://wixtoolset.org/documentation/manual/v3/xsd/wix/condition.html)

[expression-syntax](https://www.firegiant.com/wix/tutorial/com-expression-syntax-miscellanea/expression-syntax)

The XML CDATA Section <![CDATA[ and ]]> is safer.

## Properties
Most important [Naming conventions](https://docs.microsoft.com/en-us/windows/win32/msi/restrictions-on-property-names): 

- Public properties may be changed by the user and must be upper-case.

Logic value and checkboxes:

-  A msi property is false if and only if it is unset, undefined, missing, the empty string (msi properties are strings).
-  A checkbox is empty if and only if the relevant msi property is false.


[OS Properties](http://wixtoolset.org/documentation/manual/v3/howtos/redistributables_and_install_checks/block_install_on_os.html)

- MsiNTProductType:  1=Workstation  2=Domain controller  3=Server
- VersionNT:
  - Windows  7=601   [msdn](https://msdn.microsoft.com/library/aa370556.aspx)
  - Windows 10=603   [ms](https://support.microsoft.com/en-us/help/3202260/versionnt-value-for-windows-10-and-windows-server-2016)
- PhysicalMemory     [ms](https://docs.microsoft.com/en-us/windows/desktop/Msi/physicalmemory)




msi properties, use in custom actions:
-  DECAC = "Deferred cusmtom action in C#"
-  CADH  = "Custom action data helper"
-  The CADH helper must mention each msi property or the DECAC function will crash:
-  A DECAC that tries to use a msi property not listed in its CADH crashes.

Example:

In the CADH:

    master=[MASTER];minion_id=[MINION_ID]

In the DECAC:

    session.CustomActionData["master"]      THIS IS OK
    session.CustomActionData["mister"]      THIS WILL CRASH

### Delete minion_id file
Alternatives

https://wixtoolset.org/documentation/manual/v3/xsd/wix/removefile.html

https://stackoverflow.com/questions/7120238/wix-remove-config-file-on-install


### user interface
[WIXUI_INSTALLDIR](https://wixtoolset.org//documentation/manual/v3/wixui/dialog_reference/wixui_installdir.html)

[customizing](http://www.dizzymonkeydesign.com/blog/misc/adding-and-customizing-dlgs-in-wix-3/)

## Properties for Custom actions

We uninstall a NSIS Salt-minion v2017.5 or newer with a deferred Quiet Execution Custom Action

[WixQuietExec](https://wixtoolset.org/documentation/manual/v3/customactions/qtexec.html)

https://wixtoolset.org/documentation/manual/v3/xsd/wix/customaction.html

NSIS Salt Minion v2017.5 added parameters to uninst.exe

    uninst /S                    leaves the installdir
    uninst /S /DeleteInstallDir  deletes the installdir, both silently
    uninst /s is not /S


uninst.exe _?= waits before returning:

    MSI (s) (88:A4) [16:41:46:795]: Hello, I'm your 32bit Elevated Non-remapped custom action server.
    WixQuietExec:  Entering WixQuietExec in C:\Windows\Installer\MSICED6.tmp, version 3.11.2318.0
    WixQuietExec:  "C:\salt\uninst.exe" /S _?=C:\salt
    MSI (s) (88:F0) [16:41:53:435]: Executing op: ActionStart(Name=uninst_NSIS2_DECAX,,)


    MSI (s) (C4:00) [16:54:13:482]: Hello, I'm your 32bit Elevated Non-remapped custom action server.
    WixQuietExec:  Entering WixQuietExec in C:\Windows\Installer\MSI33C4.tmp, version 3.11.2318.0
    WixQuietExec:  "C:\salt\uninst.exe" /S _?=C:\salt
    MSI (s) (C4:30) [16:54:19:701]: Executing op: ActionStart(Name=uninst_NSIS2_DECAX,,)


## Sequences
An msi is no linear program.
To understand when custom actions will be executed, one must look at the condition within the tag and Before/After:

On custom action conditions:
[Common-MSI-Conditions.pdf](http://resources.flexerasoftware.com/web/pdf/archive/IS-CHS-Common-MSI-Conditions.pdf)
[ms](https://docs.microsoft.com/en-us/windows/win32/msi/property-reference)

On the upgrade custom action condition:

|  Property   |  Comment  |
| --- |  --- |
|  UPGRADINGPRODUCTCODE | does not work
|  Installed            | the product is installed per-machine or for the current user
|  Not Installed        | there is no previous version with the same UpgradeCode
|  REMOVE ~= "ALL"      | Uninstall

[Custom action introduction](https://docs.microsoft.com/en-us/archive/blogs/alexshev/from-msi-to-wix-part-5-custom-actions-introduction)

## Standard action sequence

[Standard actions reference](https://docs.microsoft.com/en-us/windows/win32/msi/standard-actions-reference)

[Standard actions WiX default sequence](https://www.firegiant.com/wix/tutorial/events-and-actions/queueing-up/)

[coding bee on Standard actions WiX default sequence](https://codingbee.net/wix/wix-the-installation-sequence)

You get error LGHT0204 when  After or Before are wrong. Example:

    del_NSIS_DECAC is a in-script custom action.  It must be sequenced between InstallInitialize and InstallFinalize in the InstallExecuteSequence

Notes on ReadConfig_IMCAC

    Note 1:
      Problem: INSTALLFOLDER was not set in ReadConfig_IMCAC
      Solution:
      ReadConfig_IMCAC must not be called BEFORE FindRelatedProducts, but BEFORE MigrateFeatureStates because
      INSTALLFOLDER in only set in CostFinalize, which comes after FindRelatedProducts
      Maybe one could call ReadConfig_IMCAC AFTER FindRelatedProducts
    Note 2:
      ReadConfig_IMCAC is in both InstallUISequence and InstallExecuteSequence,
      but because it is declared Execute='firstSequence', it will not be repeated in InstallExecuteSequence if it has been called in InstallUISequence.


## Don't allow downgrade
http://wixtoolset.org/documentation/manual/v3/howtos/updates/major_upgrade.html 

## How to Delete a folder

    Delete a folder recursivly
    Inspiration 1: https://www.hass.de/content/wix-how-use-removefolderex-your-xml-scripts
    Inspiration 2: http://robmensching.com/blog/posts/2010/5/2/the-wix-toolsets-remember-property-pattern/   Simple Remember Pattern SRP
    Learn: Condition with KEEP_CONFIG is useless on Components and on Features
    SRP pattern
       We cannot recursivly delete the var folder because it contains the cache, we regard as configuration and optionally keep.
         It is the task of Uninstall_excl_Config_DECAC to selectivly delete the var folder
        The use of the SRP Pattern is limited.


## VC++ for Python

Why does the Salt-Minion needs the C++ runtime:

  - TODO/unknown/???

Quote from [PythonWiki](https://wiki.python.org/moin/WindowsCompilers): 
Even though Python is an interpreted language, you may need to install Windows C++ compilers in some cases.
For example, you will need to use them if you wish to:

- Install a non-pure Python package from sources with Pip (if there is no Wheel package provided).
- Compile a Cython or Pyrex file.

Microsoft provides official C++ compilers called Visual C++, you can find them bundled with Visual Studio.

Which Microsoft Visual C++ compiler to use with a specific Python version?

|Python     | VC++        | Visual Studio
|---        |---          |---
|  2.7      | VC90_CRT    | 2008
|  3.5-8    | VC140_CRT   | 2015

Merge modules (*.msm) are msi 'library' databases that can be included ('merged') into a (single) msi databases.

The msi incorporates the VC*_CRT as merge modules (*.msm), following the [how-to](https://wixtoolset.org/documentation/manual/v3/howtos/redistributables_and_install_checks/install_vcredist.html)


## Images
Images:

- Dimensions of images must follow [WiX rules](http://wixtoolset.org/documentation/manual/v3/wixui/wixui_customizations.html)
- WixUIDialogBmp must be transparent

Create imgLeft.png from panel.bmp:

- Open paint3D:
  - new image, ..., canvas options: Transparent canvas off, Resize image with canvas NO, Width 493 Height 312
  - paste panel.bmp, move to the left, save as



## Note on Create folder

          Function win_verify_env()  in  salt/slt/utils/verify.py sets permissions on each start of the salt-minion services.
          The installer must create the folder with the same permissions, so you keep sets of permissions in sync.

          The Permission element(s) below replace any present permissions,
          except NT AUTHORITY\SYSTEM:(OI)(CI)(F), which seems to be the basis.
          Therefore, you don't need to specify User="[WIX_ACCOUNT_LOCALSYSTEM]"  GenericAll="yes"

          Use icacls to test the result:
            C:\>icacls salt
            salt BUILTIN\Administrators:(OI)(CI)(F)
                 NT AUTHORITY\SYSTEM:(OI)(CI)(F)
            ~~ read ~~
            (object inherit)(container inherit)(full access)

            C:\>icacls salt\bin\include
            salt\bin\include BUILTIN\Administrators:(I)(OI)(CI)(F)
                             NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
                             w7h64\Markus:(I)(OI)(CI)(F)
            ~~ read ~~
            (permission inherited from parent container)(object inherit)(container inherit)(full access)

          Maybe even the Administrator group full access is "basis", so there is no result of the instruction,
          I leave it for clarity, and potential future use.
          
## On servicePython.wxs

      Experimental. Intended to replace nssm (ssm) with the Windows service control.
           Maybe, nssm (ssm) cannot be replaced, because it indefineiy starts the salt-minion python exe over and over again,
           whereas the Windows method only starts an exe only a limited time and then stops.
           Also goto BuildDistFragment.xsl and remove python.exe
      <ComponentRef Id="servicePython" />

## Set permissions of the install folder with WixQueryOsWellKnownSID 

[doc](http://wixtoolset.org/documentation/manual/v3/customactions/osinfo.html)

