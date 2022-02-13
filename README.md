# DiskPartShrinkVolume
While attempting to shrink a volume using diskpart in Windows you receive an error stating: The specified shrink size is too big and will cause the volume to be smaller than the minimum volume size. This is what you should do in this scenario.  

## Make sure ...  
   - That the partition you are shrinking is (*for lack of better words*) the **last** partition on the disk. Open **diskmgmt.msc**, observe you're target partition on your target disk from left to right. Is there anything to the **right** of your target partition that isn't unallocated space?
   - You've explicitly told Windows **not** to use pagefile.sys.
       - This can be accomplished via: <img height=16px width=auto src="https://raw.githubusercontent.com/nstevens1040/images/main/winkey.svg"/>  → sysdm.cpl → Advanced → Performance \[Settings\] → Advanced → Change → No paging file → **Don't forget to click 'Set'!**
   - You have the correct volume selected? ```SELECT VOLUME 0```
   - Can the volume reasonably accommodate the shrink operation?  
      - To be specific, here is an example. Let's say I want to shink my current volume by 20GB (20480MB). If the following PowerShell command evaluates to **True**, then you're **"not crazy"**: ```(((Get-Partition -DriveLetter C | % Size) - (Get-PSDrive -Name C | % Used)) / 1MB) -gt 20480```
   - Building on the example above, the correct **diskpart** command would be: ```SHRINK DESIRED=20480``` (**shrink the currently selected volume by 20GB**)
   - You observe the shrink operation's progress by launching **%windir%\system32\dfrgui.exe**  
  
## Still seeing errors?  
Specifically, if you are seeing this error,  
  
```
The specified shrink size is too big and will cause the volume to be smaller than the minimum volume size
```  
  
then you can view relevant event logs using the **Windows PowerShell** script below (**Run As Admin**).  
```ps1
@((Get-WinEvent -FilterXml "`n<QueryList>`n  <Query Id=`"0`" Path=`"Application`">`n    <Select Path=`"Application`">*[System[(EventID=259 or EventID=260 or EventID=261)]]</Select>`n  </Query>`n</QueryList>") | Sort TimeCreated -Descending)
```  
  
If you're certain that the most recent event that the script returns is relevant, then the script below will tell you the exact file or folder path that is causing problems (**Run As Admin**).  
```ps1
@((Get-WinEvent -FilterXml "`n<QueryList>`n  <Query Id=`"0`" Path=`"Application`">`n    <Select Path=`"Application`">*[System[(EventID=259 or EventID=260 or EventID=261)]]</Select>`n  </Query>`n</QueryList>") | Sort TimeCreated -Descending)[0].Properties[2].Value
```  
  
Personally, I simply deleted the problem files/folders and then proceeded to successfully shrink the volume.  
