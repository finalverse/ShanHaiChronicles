####  **open /Users/zhangjingran/finalverseProj/mutsea-o-1/MutSea/Region/OptionalModules/World/NPC/NPCModule.cs**
```
public void AddRegion(Scene scene)
{
    if (!Enabled)
        return;

    scene.RegisterModuleInterface<INPCModule>(this);

    scene.EventManager.OnNewClient += (IClientAPI client) =>
{
    if (client.AgentId == UUID.Parse("871e061e-ff29-4a0c-927e-ae9961975b91"))
    {
        m_log.Info("[NPCModule]: ✅ Owner logged in, will delay NPC spawn 500ms.");

        // 延迟 500ms 让 ScenePresence 完全 ready
        Util.FireAndForget((o) =>
        {
            Thread.Sleep(5000);

            ScenePresence ownerPresence = scene.GetScenePresence(client.AgentId);
            AvatarAppearance appearance = null;

            if (ownerPresence != null && !ownerPresence.IsChildAgent)
            {
                appearance = new AvatarAppearance(ownerPresence.Appearance, true);
                m_log.Info("[NPCModule]: ✅ Successfully cloned owner appearance for NPC.");
            }
            else
            {
                m_log.Warn("[NPCModule]: ⚠️ Owner presence still not found even after delay.");
            }

            UUID npcId = CreateNPC(
                "Auto",
                "Bot",
                new Vector3(166, 27, 42),
                UUID.Zero,
                client.AgentId,
                "",
                UUID.Zero,
                true,
                scene,
                appearance
            );

            if (npcId != UUID.Zero)
                m_log.Info("[NPCModule]: ✅ Auto Bot created after delay.");
            else
                m_log.Warn("[NPCModule]: ❌ Auto Bot creation failed after delay.");

        });
    }
};
}
       public void RegionLoaded(Scene scene)
        {
            // 可以保留空实现，或者记录日志
            m_log.InfoFormat("[NPCModule]: RegionLoaded for '{0}'", scene.RegionInfo.RegionName);
        }
```
