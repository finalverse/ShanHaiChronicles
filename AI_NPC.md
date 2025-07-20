//gpt
using System.Net.Http;
using System.Text;
using System.Text.Json;
using System.Threading.Tasks;
 private UUID m_autoNpcId = UUID.Zero;
 //gpt
        scene.EventManager.OnChatFromClient += async (sender, chat) =>
        {
            IClientAPI client = sender as IClientAPI;
            if (client == null)
                return;

            if (m_autoNpcId == UUID.Zero)
                return;

            if (client.AgentId == m_autoNpcId)
                return;  // 🛑 ignore NPC speaking

            if (chat.Type != ChatTypeEnum.Say && chat.Type != ChatTypeEnum.Shout)
                return;

            ScenePresence npcPresence = scene.GetScenePresence(m_autoNpcId);
            ScenePresence senderPresence = scene.GetScenePresence(client.AgentId);

            if (npcPresence != null && senderPresence != null)
            {
                float distance = Vector3.Distance(npcPresence.AbsolutePosition, senderPresence.AbsolutePosition);
                if (distance <= 20f)
                {
                    m_log.Info($"[NPCModule]: 💬 '{client.Name}' says near NPC: {chat.Message}");

                    string reply = await QueryOllamaAsync(chat.Message);
                    if (!string.IsNullOrWhiteSpace(reply))
                    {
                        Say(m_autoNpcId, scene, reply);
                    }
                }
            }
        };

         //gpt
        private async Task<string> QueryOllamaAsync(string prompt)
    {
        try
        {
            using (HttpClient client = new HttpClient())
            {
                var payload = new
                {
                    model = "llama3.2",
                    prompt = prompt,
                    stream = false
                };
                string json = JsonSerializer.Serialize(payload);
                var content = new StringContent(json, Encoding.UTF8, "application/json");
                var response = await client.PostAsync("http://localhost:11434/api/generate", content);
                string responseBody = await response.Content.ReadAsStringAsync();
                var doc = JsonDocument.Parse(responseBody);
                return doc.RootElement.GetProperty("response").GetString();
            }
        }
        catch (Exception e)
        {
            m_log.Warn($"[NPCModule]: ⚠️ Ollama call failed: {e.Message}");
            return "Sorry, I couldn't understand that.";
        }
    }

        
