# PowerShell - System PATH Helpers

*Source: https://gist.github.com/014f18d00bea1c63ca5da34fa66a71e0#file-syspath-psm1*

````powershell
Function Add-SystemPath {
  <#
    .SYNOPSIS
    This function updates enironment PATH variable.
    .DESCRIPTION
    This function in details:
    * adds specified paths to %PATH% environment variable
    .PARAMETER PathToAdd
    Array of paths to add.
    .LINK
    https://github.com/a4099181/vagrant-provvin/blob/master/docs/Add-SystemPath.md
    .LINK
    https://github.com/a4099181/vagrant-provvin/blob/master/modules/extend-PATH-environment-variable.psm1
    .NOTES
    Author: bryjamus <bryjamus@gmail.com>
#>
  Param([array] $PathToAdd)

  $VerifiedPathsToAdd = $Null
  Foreach ($Path in $PathToAdd) {
    $UserPath = [Environment]::GetEnvironmentVariable("Path", [EnvironmentVariableTarget]::User)
    if ($UserPath -like "*$Path*") {
      Write-Host "Currnet item in path is: $Path"
      Write-Host "$Path already exists in Path statement"
    }
    else {
      $VerifiedPathsToAdd += ";$Path"
      Write-Host "`$VerifiedPathsToAdd updated to contain: $Path"
    }

    if ($VerifiedPathsToAdd -ne $null) {
      Write-Host "`$VerifiedPathsToAdd contains: $verifiedPathsToAdd"
      Write-Host "Adding $Path to Path statement now..."
      [Environment]::SetEnvironmentVariable("Path", $UserPath + $VerifiedPathsToAdd, [EnvironmentVariableTarget]::User)
    }
  }
}
````

---

## Appendix: Links

* *Code*
* [Development](../../MOCs/Development.md)
* *Windows*
* [Microsoft DOS](../../../3-Resources/Tools/Developer%20Tools/Shell/Microsoft%20DOS.md)
* *Command Line*
* [2-Areas/MOCs/PowerShell](../../MOCs/PowerShell.md)

*Backlinks:*

````dataview
list from [[PowerShell - System PATH Helpers]] AND -"Changelog"
````
