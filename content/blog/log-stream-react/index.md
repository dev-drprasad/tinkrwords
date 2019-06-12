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

```jsx
import React, { useEffect, useState } from "react";

const decoder = new TextDecoder("utf-8");

async function* logReader() {
  const url = 'http://localhost:8000/api';
  const response = await fetch(url);
  const reader = response.body.getReader();


  while (true) {
    const {done, value} = await reader.read();
    if (done) break;
    const decoded =  decoder.decode(value);
    yield decoded;
  }
}

const initialState = [];

const reducer = (state, action) => {
  switch (action.type) {
    case 'RESET':
      return initialState;
    case 'APPEND_LOG':
      return [...state, ...action.payload];
    default:
      return state;
  }
};

const LogViewer = () => {
  const [logs, dispatch] = React.useReducer(reducer, initialState);
  const ref = React.useRef();

  useEffect(() => {
    (async function () {
      for await(const log of logReader()) {
        dispatch({ type: "APPEND_LOG", payload: log.split("\r").filter(Boolean) })
      }
    })()
  }, []);

  useEffect(() => {
    ref.current.scrollTo(0, ref.current.scrollHeight);
  }, [logs])

  return (
    <div id="LogViewer"
      ref={ref}
      style={{
        whiteSpace: "pre",
        fontFamily: "Fira Code",
        fontWeight: 500,
        margin: 15,
        border: "1px solid #e1e2e3",
        height: 400,
        padding: 15,
        borderRadius: 4,
        overflow: "auto"
      }}
    >{logs.join("\n")}</div>
  )
}

export default LogViewer;
```

output

![git pull command logs](./log-stream-viewer.gif)
