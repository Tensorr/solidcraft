# Architecture

Every entry point drives one engine. Arrows show who calls whom at run time; layers follow the
[README](../README.md#architecture) (L0 `geom` up to L5 front ends).

```mermaid
flowchart TB
    subgraph apps["Apps · entry points"]
        direction LR
        desktop["<b>Desktop &amp; web apps</b><br/><code>solidcraft · solidcraft-web</code>"]
        cli["<b>Headless CLI</b><br/><code>solidcraft-cli</code>"]
    end

    subgraph front["Front ends · L5"]
        direction LR
        ui["<b>User interface</b><br/><code>ui-egui</code> · egui + wgpu"]
        gpu["<b>GPU rendering</b><br/><code>gpu</code> · pipelines"]
        mcp["<b>MCP server</b><br/><code>mcp</code> · JSON-RPC stdio"]
    end

    engine["<b>Engine · L4</b><br/><code>engine</code> · session · commands · undo"]

    subgraph libs["Libraries · L0–L3"]
        direction LR
        io["<b>File I/O · L3</b><br/><code>io</code> · STEP · STL · 3MF · glTF"]
        doc[("<b>Document · L2</b><br/><code>doc</code> · params · timeline")]
        core["<b>Modeling core · L0–L1</b><br/><code>sketch · kernel · render · geom</code>"]
    end

    desktop -- hosts --> ui
    cli -- "mcp subcommand" --> mcp
    cli -- snapshots --> gpu
    mcp -. "--connect (TCP control port)" .-> desktop
    ui -- draw --> gpu
    ui -- commands --> engine
    mcp -- commands --> engine
    engine -- scene --> gpu
    engine -- "open · save" --> io
    engine -- edits --> doc
    io -- serialize --> doc
    doc -- evaluate --> core

    classDef focal stroke:#bf4520,stroke-width:2px
    class engine focal
```

| Node | Crates | Role |
|---|---|---|
| Desktop & web apps | `solidcraft`, `solidcraft-web` | Native and browser (WebGPU / WebGL2) shells around the egui UI |
| Headless CLI | `solidcraft-cli` | Command scripts, export, snapshots, and the `mcp` subcommand |
| User interface | `ui-egui` | The swappable desktop front end |
| GPU rendering | `gpu` | Viewport pipelines, offscreen captures, compute-shader tessellation |
| MCP server | `mcp` | Lets agents drive a headless session, or the running app over the [control protocol](control-protocol.md) |
| Engine | `engine` | The session and command registry: every action, from any front end, is a command |
| File I/O | `io` | Design files plus STL/OBJ/STEP/IGES/3MF/glTF, FreeCAD and Rhino import |
| Document | `doc` | Parameters, expressions, the feature timeline and its incremental evaluation |
| Modeling core | `sketch`, `kernel`, `render`, `geom` | Constraint solver, B-rep kernel boundary (truck), view math, geometry foundation |

Not drawn: `derive` (proc macros used by `doc`) and `space-mouse` / `3dx-client` (3D-mouse
input, desktop app only).
