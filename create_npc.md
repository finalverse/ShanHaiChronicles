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

#### **(6) NPC walk and speak if there have somebody. I can speak with her , but it is static.**
```
key myAvatarKey = "871e061e-ff29-4a0c-927e-ae9961975b91";  // 你的 avatar UUID
key npc = NULL_KEY;
integer npc_created = FALSE;
vector home_pos;
integer listener = 0;

default
{
    state_entry()
    {
        llOwnerSay("💡 脚本初始化，检查是否需要创建 NPC...");
        home_pos = llGetPos();
        vector pos = home_pos + <1.0, 0.0, 0.0>;

        if (!npc_created)
        {
            npc = osNpcCreate("Butler", "Lee", pos, myAvatarKey);
            llOwnerSay("✅ NPC 已自动创建: " + (string)npc);
            listener = llListen(0, "", NULL_KEY, "");
            npc_created = TRUE;
            llSetTimerEvent(10.0);  // 10 秒循环
        }
    }

    touch_start(integer total_number)
    {
        if (npc_created)
        {
            llOwnerSay("✅ NPC 已经存在，无需重复创建。");
        }
    }

    listen(integer channel, string name, key id, string message)
    {
        if (npc != NULL_KEY)
        {
            if (llToLower(message) == "你好")
            {
                osNpcSay(npc, "你好，" + name + "！欢迎你！");
            }
            else if (llToLower(message) == "再见")
            {
                osNpcSay(npc, "再见，" + name + "！期待下次见面！");
            }
        }
    }

    timer()
    {
        llSensor("", NULL_KEY, AGENT, 20.0, PI);

        vector offset = <llFrand(6.0) - 3.0, llFrand(6.0) - 3.0, 0.0>;  // 3 米半径
        vector target = home_pos + offset;

        osNpcMoveToTarget(npc, target, OS_NPC_NO_FLY | OS_NPC_LAND_AT_TARGET);
    }

    sensor(integer num_detected)
    {
        integer i;
        for (i = 0; i < num_detected; ++i)
        {
            if (llDetectedKey(i) != npc)
            {
                osNpcSay(npc, "I am checking this lawn.");
                return;
            }
        }
    }

    no_sensor()
    {
        // 没有 avatar 静默
    }
}
```
#### **(7) NPC walk and speak. second method**
```
key myAvatarKey = "871e061e-ff29-4a0c-927e-ae9961975b91";  // 你的 avatar UUID
key npc = NULL_KEY;
integer npc_created = FALSE;

list waypoints;  // 预定义巡逻点
integer current_wp = 0;

default
{
    touch_start(integer total_number)
    {
        if (!npc_created)
        {
            vector base_pos = llGetPos();

            // ⚠️ 定义巡逻路线（相对 base_pos 的偏移）
            waypoints = [
                base_pos + <2.0, 0.0, 0.0>,
                base_pos + <2.0, 2.0, 0.0>,
                base_pos + <0.0, 2.0, 0.0>,
                base_pos + <0.0, 0.0, 0.0>
            ];

            vector start_pos = llList2Vector(waypoints, 0);
            npc = osNpcCreate("Tourist", "Lee", start_pos, myAvatarKey);
            llOwnerSay("✅ NPC 已创建并准备开始巡逻。");

            npc_created = TRUE;
            current_wp = 1;  // 下一个 waypoint 的索引
            llSetTimerEvent(10.0);
        }
        else
        {
            llOwnerSay("✅ NPC 已经创建，不会重复创建。");
        }
    }

    timer()
    {
        if (npc != NULL_KEY)
        {
            // 检测附近 avatar
            llSensor("", NULL_KEY, AGENT, 20.0, PI);
            vector target = llList2Vector(waypoints, current_wp);
            osNpcMoveToTarget(npc, target, OS_NPC_NO_FLY | OS_NPC_LAND_AT_TARGET);

            // 下一个 waypoint，循環
            current_wp = (current_wp + 1) % llGetListLength(waypoints);
        }
    }
    
        sensor(integer num_detected)
    {
        integer i;
        for (i = 0; i < num_detected; ++i)
        {
            if (llDetectedKey(i) != npc)
            {
                osNpcSay(npc, "There are so beautiful!");
                return;
            }
        }
    }

    no_sensor()
    {
        // 无 avatar 不说话
    }
    
}
```

















