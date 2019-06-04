---
title: "Use case: Async generators, streams in JavaScript"
date: "2019-06-04T11:07:31.394Z"
description: Understand JavaScript Async generators and streams with pracitical example. We will create simple log viewer using React
---

This post helps you to understand async generators practical use case.

We need simple API which gives stream output. Lets do that.

```javascript
import { spawn } from 'child_process';

server.on('request', (req, res) => {

  if (req.url === "/api" && req.method === "GET") {
    res.writeHead(200, {
      'Content-Type': 'text/html; charset=UTF-8',
      'Connection': 'Keep-Alive',
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Methods": "HEAD, GET",
    });
    const gitPull = spawn("git", ["clone", "--progress", "https://github.com/microsoft/vscode-docs.git"]);
  
    gitPull.stdout.pipe(res);
    gitPull.stderr.pipe(res);
    gitPull.on("exit", (code) => res.end());

    return;
  }

  return res.writeHead(404).end();
});

server.listen(8000);
```

I am using react to consume API
