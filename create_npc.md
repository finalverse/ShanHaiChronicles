#### **(1) open MutSea/bin/OpenSim.ini **  
- **Find moudle call [NPC]**
- **modify**:  
  ```
  [NPC]
    Enabled = true
    MaxNumberNPCsPerScene = 40 
  ```  
#### **(2) open MutSea/bin/config-include/osslEnable.ini **  
- **Find moudle call [OSSL]**
- **modify**:  
  ```
    [OSSL]
      AllowOSFunctions = true
      OSFunctionThreatLevel = VeryLow
      Allow_osNpcCreate = "PARCEL_OWNER,ESTATE_MANAGER,ESTATE_OWNER"
      Allow_osNpcRemove = "PARCEL_OWNER,ESTATE_MANAGER,ESTATE_OWNER"
      Allow_osNpcSay = "PARCEL_OWNER,ESTATE_MANAGER,ESTATE_OWNER"
      Allow_osNpcMoveToTarget = "PARCEL_OWNER,ESTATE_MANAGER,ESTATE_OWNER"
      Allow_osNpcStand = "PARCEL_OWNER,ESTATE_MANAGER,ESTATE_OWNER"
      Allow_osNpcSit = "PARCEL_OWNER,ESTATE_MANAGER,ESTATE_OWNER"
  ```
#### **(3) Use LSL script to add NPC **
- **Use finalverse storm to login your OpenSim grid and get in your region**
- **Create a prim as a 'script host'**:
  - Press the shortcut key Ctrl+B (or go to the menu Build > Build)
  - Click on the ground with the mouse to place a Box (cube)
  - This is the object that loads the script
- **Right-click on prim and select 'Edit'**:
  - Right-click on prim → Edit
  - In the Edit window, click on the upper Content tab.
- **Right-click on the blank area of the Content tab → New Script At this time, a default script will appear.**:
 ```
key myAvatarKey = "79e617cc-fa02-4f22-b9f2-71b7f33029c4"; // 你的 avatar UUID

default
{
    touch_start(integer total_number)
    {
        vector pos = llGetPos() + <1.0, 0.0, 0.0>;
        key npc = osNpcCreate("精卫", "Jingwei", pos, myAvatarKey);
        llOwnerSay("✅ NPC created: " + (string)npc);
    }
}
 ```
- **save and exit. Then click the box, you can see the NPC.**

#### **(4) If you do not have authority of the land , you can run the following command in your console **
- **Error**:
```
You cannot create objects here.
```
- **solution**:
 - Check land permissions: If you are not the owner and you can not change it , you can ,,,,
    ```
    land clear  //run it in your console,then restart MutSea.
    ```
#### **(5) Ensure your account is owner. If not , you can change it **  
  ```
estate show
show users
estate set owner 101 zaran zhang   //then restart MutSea.
  ```

















