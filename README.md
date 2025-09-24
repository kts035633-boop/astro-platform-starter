{
  "name": "max-vault-proxy",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "node-fetch": "^3.4.0"
  },
  "type": "module"
}import express from "express";
import fetch from "node-fetch";
import { URL } from "url";

const app = express();
app.use(express.static("public"));

// Proxy endpoint
app.get("/proxy", async (req, res) => {
  const target = req.query.url;
  if (!target) return res.status(400).send("Missing url");

  try {
    const u = new URL(target);
    // only allow DuckDuckGo (duck.com or duckduckgo.com)
    if (!["duck.com", "duckduckgo.com", "www.duckduckgo.com"].includes(u.hostname)) {
      return res.status(403).send("Only DuckDuckGo allowed for this demo");
    }

    const response = await fetch(target);
    res.status(response.status);

    response.headers.forEach((val, name) => {
      if (name.toLowerCase() === "set-cookie") return; // block cookies
      res.setHeader(name, val);
    });

    response.body.pipe(res);
  } catch (e) {
    console.error(e);
    res.status(500).send("Proxy error");
  }
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => console.log(`Max Vaul<!doctype html>
<html>
<head>
  <meta charset="utf-8" />
  <title>Max Vault</title>
  <style>
    body {
      background: black;
      color: white;
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 0;
      padding: 0;
    }
    h1 {
      margin-top: 40px;
      font-size: 2.5em;
    }
    form {
      margin-top: 20px;
    }
    input {
      width: 60%;
      padding: 10px;
      border: none;
      border-radius: 5px;
    }
    button {
      padding: 10px 20px;
      margin-left: 10px;
      border: none;
      border-radius: 5px;
      background: #444;
      color: white;
      cursor: pointer;
    }
    button:hover {
      background: #666;
    }
    iframe {
      width: 90%;
      height: 70vh;
      margin-top: 20px;
      border: none;
      background: white;
    }
  </style>
</head>
<body>
  <h1>Max Vault</h1>
  <form id="f">
    <input id="url" value="https://duck.com" />
    <button>Go</button>
  </form>
  <iframe id="frame"></iframe>

  <script>
    const form = document.getElementById("f");
    const urlInput = document.getElementById("url");
    const frame = document.getElementById("frame");

    form.addEventListener("submit", e => {
      e.preventDefault();
      const targetUrl = urlInput.value.trim();
      frame.src = "/proxy?url=" + encodeURIComponent(targetUrl);
    });
  </script>
</body>
</html> proxy running on port ${PORT}`));