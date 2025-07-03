Here’s a professional English version of the workflow for building your *Shan Hai Jing*-themed OpenSimulator world, covering modeling, deployment, and scripting:

---

### **1. Blender Modeling & Export Guidelines**  
#### **(1) Modeling Standards**  
- **Scale**: 1 Blender unit = 1 OpenSim meter (avoid scaling issues).  
- **Material Optimization**:  
  - Use single UV maps (avoid multi-texturing).  
  - Name textures without special characters (e.g., `mountain_diffuse.jpg`).  
- **LOD (Level of Detail)**:  
  ```python  
  # Blender Python script to generate LOD:  
  bpy.ops.object.generate_lod(levels=3, ratio=0.5)  
  ```  

#### **(2) Exporting to DAE**  
1. Select objects → *File* → *Export* → *Collada (.dae)*.  
2. **Critical Settings**:  
   - ☑ *Selected Objects Only*  
   - ☑ *Apply Transforms*  
   - ☐ *Disable Triangulation* (preserve quads)  
   - ☑ *Include UVs/Normals*  

#### **(3) Asset Organization**  
```  
/shanhaijing_assets  
   /zhongshan  
      jianmu_tree.dae  
      kunlun_mountain.dae  
   /nanhai  
      coral_palace.dae  
      dragon_statue.dae  
```  

---

### **2. Local Testing & OAR Packaging**  
#### **(1) Temporary Region Setup**  
```bash  
# Create a test region:  
create region TestRegion 1000 1000  
```  
- **Terrain Preload**:  
  ```bash  
  terrain load test_terrain.raw  
  ```  

#### **(2) Bulk Upload via Firestorm**  
1. **Hotkeys**:  
   - `Ctrl+3`: Open build tools.  
   - `Ctrl+U`: Bulk upload models.  
2. **Permissions**:  
   - Enable *Next Owner Can Copy/Modify*.  
   - Set *Price* to 0 (avoid transaction issues).  

#### **(3) Layout Adjustments**  
- **Precise Placement**:  
  ```lsl  
  llSetPos(<128.0, 128.0, 25.5>);  
  llSetRot(llEuler2Rot(<0, 0, 45> * DEG_TO_RAD));  
  ```  
- **Physics Testing**:  
  - Press `Ctrl+Alt+P` to enable physics.  
  - Verify avatar collision.  

#### **(4) Export OAR Archive**  
```bash  
save oar shanhaijing_temp.oar -m "Initial version"  
```  
**Parameters**:  
- `-m`: Add notes.  
- `--assets`: Include dependencies.  

---

### **3. Production Deployment**  
#### **(1) Region Initialization**  
```bash  
# Create all 6 regions:  
create region ZhongShan 1000 1000  
create region NanShan 1000 1256  
... (repeat for other regions)  
```  

#### **(2) OAR Import by Region**  
```bash  
# Load ZhongShan:  
change region ZhongShan  
load oar zhongshan.oar --merge --force-terrain  

# Configure NanHai (underwater effects):  
set region_environment NanHai --water-level 20 --underwater-fog 0.7  
```  
**Key Flags**:  
- `--merge`: Preserve existing builds.  
- `--force-terrain`: Override terrain.  

#### **(3) Database Optimization (MySQL)**  
```ini  
# Database.ini  
[Storage]  
ConnectionString = "Server=localhost;Database=opensim;Uid=root;Pwd=123456;Pooling=true;"  
AssetCacheSize = 2048  # MB  
```  

---

### **4. Teleport System**  
#### **(1) Mythical Creature Teleporter**  
```lsl  
// Phoenix statue in NanShan:  
default {  
    touch_start(integer num) {  
        key user = llDetectedKey(0);  
        llParticleSystem([PSYS_PART_FLAGS, PSYS_PART_EMISSIVE_MASK]);  
        llSleep(0.5);  
        llTeleportAgent(user, "ZhongShan", <128,128,50>, <1,0,0>);  
    }  
}  
```  

#### **(2) HUD Navigation**  
```lsl  
showRegionMap() {  
    llDialog(llGetOwner(), "Six Divine Realms:",  
        ["ZhongShan", "NanShan", "NanHai", "YunTing", "DongHai", "ZhongHai"], 9);  
}  
```  

---

### **5. NPCs & Dynamic Systems**  
#### **(1) Lore-Driven NPCs**  
```lsl  
string[] lore_msgs = [  
    "I am Bai Ze, knower of all creatures...",  
    "East of ZhongShan lies Qingqiu, home of nine-tailed foxes..."  
];  

default {  
    touch_start(integer num) {  
        llSay(0, lore_msgs[llFloor(llFrand(llGetListLength(lore_msgs))]);  
        llStartAnimation("mouth_open");  
    }  
}  
```  

#### **(2) Weather Control**  
```lsl  
integer is_raining = FALSE;  

toggleWeather() {  
    is_raining = !is_raining;  
    llRegionSetEnvironment(ENV_RAIN, is_raining);  
    llOwnerSay("Weather: " + (is_raining ? "Storm" : "Clear"));  
}  
```  

---

### **6. Maintenance & Monitoring**  
#### **(1) Admin Commands**  
```bash  
# Check resource usage:  
show region resource ZhongShan  

# Backup a region:  
save oar zhongshan_backup.oar --scope Region --assets  
```  

#### **(2) Log Analysis**  
```bash  
# Track script errors:  
tail -f logs/opensim.log | grep "LSL ERROR"  
```  

---

### **Troubleshooting Cheat Sheet**  
| Issue                      | Solution                                  |  
|----------------------------|-------------------------------------------|  
| Missing textures           | Avoid Chinese characters in file paths.  |  
| Avatars stuck after teleport | Increase landing Z-coordinate by +5m.    |  
| Scripts inactive post-OAR  | Run `reset script *`.                    |  

This workflow enables scalable deployment of your *Shan Hai Jing* metaverse. Test thoroughly in a sandbox region before full rollout. Let me know if you need further customization! 🚀
