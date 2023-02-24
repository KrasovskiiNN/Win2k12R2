# Win2k12R2
Manual for build current ISO-image 

Source ISO: 9600.17050.WINBLUE_REFRESH.140317-1640_X64FRE_SERVER_EVAL_EN-US-IR3_SSS_X64FREE_EN-US_DV9.iso

<# Create directories for work  
	ISO - for copy files from official disk with operating system  
	MOUNT -   
	RU - RollUp Updates  
	SU - Security Updates  
	WO - WSUS Offline Updates  
	NET35 - .NET Framework 3.5 Updates  
#>  

New-Item -Path 'D:\WS2012R2\ISO' -ItemType Directory
New-Item -Path 'D:\WS2012R2\MOUNT' -ItemType Directory
New-Item -Path 'D:\WS2012R2\RU' -ItemType Directory
New-Item -Path 'D:\WS2012R2\SU' -ItemType Directory
New-Item -Path 'D:\WS2012R2\WO' -ItemType Directory
New-Item -Path 'D:\WS2012R2\NET35' -ItemType Directory

cd D:\WS2012R2

Mount-DiskImage -ImagePath D:\WS2012R2\9600.17050.WINBLUE_REFRESH.140317-1640_X64FRE_SERVER_EVAL_EN-US-IR3_SSS_X64FREE_EN-US_DV9.iso
Copy-Item "E:\*" -Destination "D:\WS2012R2\ISO" -Recurse -Force
Dismount-DiskImage -ImagePath D:\WS2012R2\9600.17050.WINBLUE_REFRESH.140317-1640_X64FRE_SERVER_EVAL_EN-US-IR3_SSS_X64FREE_EN-US_DV9.iso

# For modifying install.wim by intagration of updates delete Read-Only checkbox in properties

Set-ItemProperty -Path "D:\WS2012R2\ISO\sources\install.wim" -Name IsReadOnly -Value $False

# List editions in wim-file. Standard Full need only. Not Standard Core.

DISM.exe /Get-WimInfo /wimfile:D:\WS2012R2\ISO\sources\install.wim
DISM.exe /Delete-Image /ImageFile:D:\WS2012R2\ISO\sources\install.wim /Index:4 /CheckIntegrity
DISM.exe /Delete-Image /ImageFile:D:\WS2012R2\ISO\sources\install.wim /Index:3 /CheckIntegrity
DISM.exe /Delete-Image /ImageFile:D:\WS2012R2\ISO\sources\install.wim /Index:1 /CheckIntegrity
DISM.exe /Get-WimInfo /wimfile:D:\WS2012R2\ISO\sources\install.wim

#

DISM.exe /Mount-Wim /wimfile:"D:\WS2012R2\ISO\sources\install.wim" /Index:1 /mountdir:D:\WS2012R2\MOUNT

# Enable .NET Framework 3.5 before updates integration, or we have error after installation.

DISM.exe /Image:D:\WS2012R2\MOUNT /Enable-Feature /FeatureName:NetFx3 /All /LimitAccess /Source:D:\WS2012R2\ISO\sources\sxs

.\1-RUadd.ps1 
.\2-SUadd.ps1  
.\3-WOadd.ps1  
.\4-NET35add.ps1 

#

DISM.exe /Image:D:\WS2012R2\MOUNT /Cleanup-Image /ScanHealth

# Unmount WIM-file and commit all modifications

DISM.exe /Unmount-Wim /MountDir:D:\WS2012R2\MOUNT /Commit

#

DISM.exe /Export-Image /SourceImageFile:D:\WS2012R2\ISO\sources\install.wim /SourceIndex:1 /DestinationImageFile:D:\WS2012R2\ISO\sources\install1.wim  /Compress:max /CheckIntegrity
DISM.exe /Get-WimInfo /wimfile:D:\WS2012R2\ISO\sources\install1.wim

DISM.exe /Mount-Wim /wimfile:"D:\WS2012R2\ISO\sources\install.wim" /Index:1 /mountdir:D:\WS2012R2\MOUNT
DISM.exe /Append-Image /imagefile:D:\WS2012R2\ISO\sources\install1.wim /captureDir:mount /name:"Windows Server 2012 R2 Standard Evaluation" /description:"WS2k12R2 Std"
DISM.exe /Unmount-Wim /MountDir:D:\WS2012R2\MOUNT /Discard

# Move old wim-file and rename new wim-file

Move-Item -Path "D:\WS2012R2\ISO\sources\install.wim" -Destination "D:\WS2012R2"
Rename-Item -Path "D:\WS2012R2\ISO\sources\install1.wim" -NewName install.wim

# Delete evaluation CORE

DISM.exe /Get-WimInfo /wimfile:D:\WS2012R2\ISO\sources\install.wim
DISM.exe /Delete-Image /ImageFile:D:\WS2012R2\ISO\sources\install.wim /Index:1 /CheckIntegrity


New-Item -Path 'D:\WS2012R2\BootOrder.txt' -ItemType File

Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "#BootOrder"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value ""
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\bcd"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\boot.sdi"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\bootfix.bin"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\bootsect.exe"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\etfsboot.com"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\memtest.exe"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\en-us\bootsect.exe.mui"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\fonts\chs_boot.ttf"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\fonts\cht_boot.ttf"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\fonts\jpn_boot.ttf"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\fonts\kor_boot.ttf"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "boot\fonts\wgl4_boot.ttf"
Add-Content -Path 'D:\WS2012R2\BootOrder.txt' -Value "sources\boot.wim"

<# BootOrder.txt

boot\bcd
boot\boot.sdi
boot\bootfix.bin
boot\bootsect.exe
boot\etfsboot.com
boot\memtest.exe
boot\en-us\bootsect.exe.mui
boot\fonts\chs_boot.ttf
boot\fonts\cht_boot.ttf
boot\fonts\jpn_boot.ttf
boot\fonts\kor_boot.ttf
boot\fonts\wgl4_boot.ttf
sources\boot.wim

#>

# Create ISO with new WIM-file

.\oscdimg -m -u2 -udfver102 -yoD:\WS2012R2\bootorder.txt -bootdata:2#p0,e,bD:\WS2012R2\ISO\BOOT\Etfsboot.com#pEF,e,bD:\WS2012R2\ISO\EFI\microsoft\BOOT\efisys.bin -lIR3_SSS_X64FREE_EN-US_DV9 D:\WS2012R2\ISO EN_WS2k12R2_Feb2023.iso
