# MCP Keyword Search Server: Low-Level Design

For the big picture see the [High-Level Design](hld.md).

## 1. Tool contract

| Field | Value |
|---|---|
| Name | `keyword-search` |
| Description | Search for a keyword within a text file |
| Input | `filePath: string`, `keyword: string` (zod) |
| Output (found) | text: `Found N line(s):` followed by the matching lines |
| Output (none) | text: `No lines found containing '<keyword>'.` |
| Output (error) | text: `Error reading file: <message>`, `isError: true` |

Matching is a case-sensitive substring test on each line (`line.includes(keyword)`).

## 2. Request / response

```mermaid
sequenceDiagram
  participant C as client.js
  participant S as index.js (child process)
  participant FS as File system
  C->>S: spawn node index.js
  Note over C: wait 1.2 s for startup
  C->>S: {"jsonrpc":"2.0","id":1,"method":"tools/call","params":{"name":"keyword-search","arguments":{"filePath":"sample.txt","keyword":"Tushar"}}}
  S->>FS: readFileSync(filePath)
  FS-->>S: contents
  S->>S: split lines, filter by keyword
  S-->>C: {"result":{"content":[{"type":"text","text":"Found 2 line(s): ..."}]}}
  C->>C: print result (or error / raw server log)
```

## 3. Run

```bash
npm install
node client.js      # demo
node index.js       # as a server for any MCP client
```
