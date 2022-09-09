```powershell
Function Get-ExeIcon {
	<#
		.SYNOPSIS
		Get-Icon extracts the icon image from an exe file 			and saves it as a .ico file in the same directory as
the .exe file.
.DESCRIPTION
Get-DiskInventory will run on every .exe file in the specified
directory by the folder PARAMETER then save a .ico file for
every .exe discovered.
.PARAMETER folder
The directory containing the .exe files.
.EXAMPLE
Get-Icon -folder c:\exelocation -name
	#>

    Param ( 
    [Parameter(Mandatory=$true)]
    [string]$folder
    )

    [System.Reflection.Assembly]::LoadWithPartialName('System.Drawing')  | Out-Null

    md $folder -ea 0 | Out-Null

    dir $folder *.exe -ea 0 -rec |
      ForEach-Object { 
        $baseName = [System.IO.Path]::GetFileNameWithoutExtension($_.FullName)
        Write-Progress "Extracting Icon" $baseName
        [System.Drawing.Icon]::ExtractAssociatedIcon($_.FullName).ToBitmap().Save("$folder\$BaseName.ico")
    }

}
```