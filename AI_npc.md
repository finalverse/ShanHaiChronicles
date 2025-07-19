### Here is a method for everyone. Make API requests to the local MutSea proxy: 
set up a small proxy locally, for example http://myproxy.local/openai, let llHTTPRequest access the proxy → the proxy forwards to OpenAI.
#### **(1) open MutSea/bin/config-include/osslEnable.ini**  
- **Find moudle call [OSSL]**
- **modify**:  
```
[OSSL]
AllowOSFunctions = true
OSFunctionThreatLevel = High
```
#### **(2) open MutSea.ini**
```
OutboundDisallowForUserScriptsExcept = 127.0.0.1:3000
```
#### **(3) Use node.js to build a simply local proxy**
- **Install Node.js if you server don't have**
```
sudo apt install nodejs npm    # Ubuntu/Debian
# if macOS you can use brew:
brew install node
```
- **create a file named openai-proxy.js**
```
const express = require('express');
const bodyParser = require('body-parser');

const app = express();
const PORT = 3000;

// set your OpenAI API Key！
const OPENAI_API_KEY = 'sk-xxxxxxx';

app.use(bodyParser.json());

app.post('/chat', async (req, res) => {
  const prompt = req.body.prompt || "Hello!";

  try {
    const openaiRes = await fetch('https://api.openai.com/v1/chat/completions', {
      method: 'POST',
      headers: {
        'Authorization': `Bearer ${OPENAI_API_KEY}`,
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        model: "gpt-3.5-turbo",
        messages: [{ role: "user", content: prompt }]
      })
    });

    const data = await openaiRes.json();

    if (data.choices && data.choices.length > 0) {
      console.log('✅ GPT Reply:', data.choices[0].message.content);
      res.json({ reply: data.choices[0].message.content.trim() });
    } else {
      console.error('❌ GPT Error Response:', data);
      res.status(500).json({ error: 'No valid reply from GPT' });
    }
  } catch (err) {
    console.error('❌ Proxy Server Error:', err);
    res.status(500).json({ error: 'Internal Proxy Error' });
  }
});

app.listen(PORT, () => {
  console.log(`✅ OpenAI Proxy running at http://localhost:${PORT}`);
});
```
- **install dependences and run it**
```
npm init -y
npm install express axios
node openai-proxy.js
```
- **change your mutsea prim's script. (prim I mensioned in ./create_npc.md**
```
key npc = NULL_KEY;
key myAvatarKey = "your UUID";
integer listener;

default
{
    state_entry()
    {
        vector pos = llGetPos() + <1, 0, 0>;
        npc = osNpcCreate("Lucy", "Lee", pos, myAvatarKey);
        llOwnerSay("✅ NPC is ready");
        
        listener = llListen(0, "", NULL_KEY, "");  // Listen public channel 0 speaking
    }

listen(integer channel, string name, key id, string message)
{
    if (id == npc) {
        // this is NPC own，don't handle，directly ignore！
        return;
    }

    if (llToLower(message) != "")
    {
        string json_body = "{\"prompt\":\"" + llEscapeURL(message) + "\"}";

        llHTTPRequest("http://127.0.0.1:3000/chat",
            [HTTP_METHOD, "POST",
             HTTP_MIMETYPE, "application/json"],
            json_body);
    }
}

    http_response(key request_id, integer status, list metadata, string body)
    {
        llOwnerSay("✅ HTTP status: " + (string)status);
        llOwnerSay("📦 GPT Response: " + body);

        // extract reply content（there suppose body is a json like {"reply":"..."} ）
        string reply = llJsonGetValue(body, ["reply"]);
        if (reply != "")
        {
            osNpcSay(npc, reply);
        }
    }
}
```

  
