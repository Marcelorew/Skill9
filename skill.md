---
name: android-reverse-ide
description: Guide an AI agent to work with the Android Reverse app (UltraSina/androidReverse), an on-device Android reverse engineering suite, through its built-in MCP server. Use whenever the user mentions Android Reverse, androidReverse, editing an APK on the phone, Smali editing, AXML or AndroidManifest edits, ARSC resources, Jadx/CFR/Vineflower decompilers, Radare2 native analysis, Flutter (Unflutter) or Unity il2cpp analysis, or connecting an AI client to the app over MCP, even if they do not name the app explicitly.
---

# Android Reverse (on-device APK analysis and editing)

Android Reverse is an Android app that does APK extraction, decompilation, native binary analysis, editing and rebuilding entirely on the phone, with no PC or ADB. Repository: https://github.com/UltraSina/androidReverse

The app ships a built-in Model Context Protocol (MCP) server. An MCP-compatible client can connect to it and edit resources, AXML, the manifest and Smali inside an open project. Compilation and rebuild are always a manual step done by the user inside the app.

## Ground rules

1. Work only on apps the user owns or has explicit permission to analyze. If the user says otherwise or the context suggests unauthorized targets, stop and say so.
2. Never assume an MCP tool name or parameter. List the tools exposed by the connected server first and use exactly what it reports. If the server is not connected, tell the user to enable the MCP server in the app and connect the client.
3. Never claim to have rebuilt, signed or installed an APK. Those steps are done by the user in the app. Finish edits, then tell the user to rebuild.
4. Make one focused change at a time, and re-read the edited file after each write to confirm the result.
5. Before editing, record the original content of the file or method so the change can be reverted. State the original and the new version to the user.

## Capabilities of the app

| Area | What exists |
| --- | --- |
| Input | .apk, .xapk, split .apks, from storage or installed apps |
| Java decompilers | CFR, Procyon, JD-Core, Krakatau, Vineflower, Jadx (primary), Jadx Fallback, Jadx IR, selectable per class |
| Native | Embedded Radare2: CFG, call graph, xrefs, Pseudo-C, assembly, hex, binary info, function rename and comments |
| Resources | ARSC tree view, string pools, resource editing, AXML editing, manifest editing, layout preview, image editor |
| Smali | Editing, formatter, LSP, import and delete, jump-to-definition, class structure compass, Smart Smali Explainer |
| XML | LSP, formatter, search, data converter |
| Cross-platform | Flutter (Dart AOT via Unflutter), Unity il2cpp metadata and function recovery |
| Rebuild and signing | APK rebuild, JKS keystore create/import/export, signing |
| Utilities | Data Converter (Base64, Hex, URL, Unicode escape, XOR, color), Global Notebook, Class Bookmarking, regex Deep Memory Search, certificate details |

## Standard workflow

1. Confirm which project is open in the app and what the user wants to achieve (a resource change, a manifest change, a Smali patch).
2. Discover the MCP tools available and map each one to the step it serves.
3. Inspect before editing: read the manifest, the target resource or the target Smali class and method.
4. Locate the exact target with the search tools or by reading the class. Prefer the exact class and method path over broad search results.
5. Apply the edit with the smallest possible change.
6. Verify by reading the file back. For Smali, check syntax, register counts and labels.
7. Report what changed and ask the user to rebuild and sign in the app.

## Smali editing checklist

- Keep `.locals` or `.registers` consistent with the registers the edited code uses.
- Do not break `:label` targets, `.catch` ranges or `.line` and `.param` directives.
- Match types exactly in descriptors, for example `Ljava/lang/String;`, `I`, `Z`, `[B`.
- Use `p0` for `this` in instance methods, and remember that `p` registers follow `v` registers.
- When replacing a return value, keep the return opcode compatible with the method signature (`return-void`, `return v0`, `return-object v0`, `return-wide v0`).
- If the code depends on the decompiled Java, cross-check with Jadx output, and use another decompiler when Jadx fails or produces unreadable output.

## Manifest, AXML and resources

- Edit the manifest only for the exact attribute, permission, component or flag the user asked for.
- Keep namespaces (`android:`) and resource references (`@string/...`, `@drawable/...`) valid.
- When changing resource values, confirm the resource ID and type in the ARSC tree first.
- Do not remove components or permissions the app clearly depends on without warning the user.

## Native, Flutter and Unity

- Native work happens through Radare2 inside the app: use CFG, xrefs and call graph views to trace a function, and report addresses together with the function names.
- For Flutter apps, expect Dart AOT and use the Unflutter analysis rather than Java decompilation.
- For Unity il2cpp apps, use the il2cpp metadata and recovered functions. Keep addresses and offsets with the module they belong to.
- Always state the architecture (arm64-v8a or armeabi-v7a) that an address or offset refers to.

## Output format

When finishing a task, reply with:

1. What was changed (file, class, method or resource).
2. Original and new version of the changed part.
3. What the user must do next in the app (rebuild, sign, install and test).
4. Any risk or thing to verify after installing.

## Troubleshooting

- MCP tools not visible: the MCP server is not enabled in the app, or the client is not connected. Ask the user to check both.
- Jadx failed on a class: the app can fall back automatically to Jadx Fallback. Try CFR, Vineflower or Procyon, or read the Smali directly.
- Manifest looks obfuscated: the app's AXML decoder handles most manifest obfuscation, so prefer reading through the app rather than a generic parser.
- Rebuild fails after an edit: revert the last edit and re-check syntax, register counts, labels and resource references.

## Disclaimer

Educational use and security research on applications where the user has explicit permission from the author, following the project's own disclaimer.
