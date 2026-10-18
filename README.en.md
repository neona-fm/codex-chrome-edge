# How to connect Codex to Chrome and Edge

[Русская версия](README.md)

This guide explains how to connect Codex Desktop on Windows to Google Chrome or Microsoft Edge, verify that the browser connection really works, and recover it when Codex shows **Connected** but can no longer see or control browser tabs.

It also includes ready-to-copy prompts that you can give directly to Codex so it can diagnose and perform the setup itself.

> Tested in practice by NeonaFM. Updated October 18, 2026. UI labels and internal paths may change between Codex versions.

**Original article:** https://n-fm.ru/codex-chrome-edge/

**Related OpenAI Codex issue:** https://github.com/openai/codex/issues/45625

## Contents

1. [What you need](#1-what-you-need)
2. [Connect Chrome or Edge](#2-connect-chrome-or-edge)
3. [If the extension store is unavailable](#3-if-the-extension-store-is-unavailable)
4. [Verify that it really works](#4-verify-that-it-really-works)
5. [Connected, but Codex cannot see browser tabs](#5-connected-but-codex-cannot-see-browser-tabs)
6. [Recovery after a VPN, proxy, network-route, or IP change](#6-recovery-after-a-vpn-proxy-network-route-or-ip-change)
7. [Full setup prompt for Codex](#7-full-setup-prompt-for-codex)
8. [Recovery-only prompt for Codex](#8-recovery-only-prompt-for-codex)

---

## 1. What you need

For this setup you need:

- the OpenAI desktop app with Codex;
- Google Chrome or Microsoft Edge;
- the official ChatGPT browser extension from OpenAI.

A normal ChatGPT tab in a browser is not enough for the local-browser integration described here.

1. Open the official Codex page: https://openai.com/codex/
2. Install the desktop app for your system and sign in.
3. Install Chrome or Edge if needed.
4. In the browser profile you plan to use, sign in to the websites Codex should work with.

Browser authentication is profile-specific. A website session in Edge does not automatically exist in Chrome.

---

## 2. Connect Chrome or Edge

1. Open the desktop app.
2. Go to **Settings → Computer Use**.
3. Select Chrome or Edge.
4. Click **Install** next to the browser.
5. Install the official ChatGPT extension from OpenAI.
6. Return to **Computer Use**. The browser should now show a **Manage** option.
7. Enable the browser.
8. In a Codex chat, select `@Chrome` or `@Edge` from the mention menu.
9. If Codex asks for access to a website, allow access only to the site you actually need.

Official browser-extension documentation:
https://learn.chatgpt.com/docs/chrome-extension

---

## 3. If the extension store is unavailable

A browser-extension store failing to load and Codex failing to connect to an already installed extension are two different problems. Identify which one you have before changing anything.

Use the official installation flow first. Network access to the store, account eligibility, and service availability are separate things.

OpenAI supported countries:
https://developers.openai.com/api/docs/supported-countries

### If you already have a verified copy of the official extension

This repository does **not** provide a repackaged ChatGPT extension. Do not download random builds from search results: a browser extension can read page contents.

If you already have a verified official unpacked extension:

1. Put it in a permanent folder, for example:

```text
C:\BrowserExtensions\ChatGPT
```

2. Make sure that folder contains `manifest.json`.
3. Open:
   - Chrome: `chrome://extensions`
   - Edge: `edge://extensions`
4. Enable **Developer mode**.
5. Choose **Load unpacked** and select the folder.
6. Return to Codex and test the browser connection.

Manual loading does not guarantee compatibility. The extension version, origin and ID still need to match the desktop integration expected by your Codex version.

---

## 4. Verify that it really works

The word **Connected** is not proof that Codex can actually use the browser.

Run a real test:

```text
@Chrome open https://example.com, read the page heading and text, click Learn more, and tell me the heading of the page after navigation.
```

For Edge, replace `@Chrome` with `@Edge`.

A successful test means Codex actually opened the page, read it, navigated, and read the second page.

Then test the real website you need in the browser profile where you are signed in.

---

## 5. Connected, but Codex cannot see browser tabs

This failure has been reported by multiple Windows users. One public OpenAI Codex issue describes Chrome and Edge being open while browser discovery returns no usable browser and errors such as:

```text
Unexpected token '<', "<html>..." is not valid JSON
```

See:
https://github.com/openai/codex/issues/45625

### `Browser is not available`

The browser connection may have received a new internal identifier. Ask Codex to refresh the browser list and use the current connection. Do not copy browser IDs from another chat.

### `Unexpected token '<' ... is not valid JSON`

The browser tool expected structured data but received HTML. This is a symptom, not proof of one specific root cause. A network route, proxy, server response or connector layer may be involved.

### `nodeRepl.fetch request failed`

The browser runtime request failed. Check connectivity and proxy settings before assuming the browser extension itself is broken.

Before editing internal files:

1. refresh the browser connection list;
2. verify site permissions;
3. try a new Codex chat;
4. fully exit and restart the desktop app;
5. preserve the exact error text for diagnostics.

Do not reinstall or patch everything blindly.

---

## 6. Recovery after a VPN, proxy, network-route, or IP change

This section is the workaround that fixed our own installation.

In our case, changing VPN also changed the local HTTP proxy. One configuration used a local endpoint such as:

```text
http://127.0.0.1:12334
```

Another used a different local port such as:

```text
http://127.0.0.1:10801
```

Those are examples only.

The browser runtime was still using the old proxy information. After updating the active runtime to the currently working local HTTP proxy and fully restarting Codex, browser access returned.

### Important: this is not only about VPN

A similar failure can appear after any network change that changes the route or proxy used by the browser runtime, for example:

- switching VPN servers or VPN clients;
- switching proxy profiles;
- reconnecting to another network;
- changing a local proxy port;
- a network configuration refresh;
- a dynamic public IP change **if it coincides with a changed proxy or route**.

A public IP address by itself is **not** the value used in this workaround. Do not paste your external VPN/server/public IP into `HTTP_PROXY`.

The important question is: **did the local HTTP proxy or route used by the Codex browser runtime change while the runtime still points to the previous one?**

### Step 1. Find the current proxy configuration

On Windows, open:

**Settings → Network & Internet → Proxy**

Also inspect the HTTP or mixed-proxy settings of your VPN/proxy client.

You can give Codex this PowerShell command:

```powershell
Get-ItemProperty -LiteralPath 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings' |
  Select-Object ProxyEnable, ProxyServer, AutoConfigURL
```

If no local proxy is actually in use, do not invent one and do not apply this workaround automatically.

### Step 2. Verify the proxy itself

Codex should verify both:

1. that the local proxy port is reachable;
2. that a real external HTTP request succeeds through that proxy.

An open local port alone is not proof that the proxy can reach the internet.

### Step 3. Identify the active browser runtime

In our Windows installation, the browser runtime file followed a path similar to:

```text
%LOCALAPPDATA%\OpenAI\Codex\runtimes\cua_node\<current_version>\bin\node_modules\@oai\cua-repl\bin\cua-repl.mjs
```

Do not blindly edit the first folder you find. Determine which runtime is actually active.

In our version, runtime information also appeared in:

```text
%LOCALAPPDATA%\OpenAI\Codex\chrome-native-hosts-v2.json
```

That file may contain stale entries, so it must be compared with the runtime actually used by the current process.

### Step 4. Apply the temporary proxy fix only if diagnostics confirm it

> **Warning:** this is a user workaround, not an official OpenAI configuration method. It edits an internal runtime file and may stop working after an update. Back up the original file first.

In a compatible version, before:

```js
await cua_repl.launch();
```

we used the following block, replacing `CURRENT_HTTP_PROXY` with the verified local HTTP proxy, for example `http://127.0.0.1:12334`:

```js
process.env.HTTP_PROXY = 'CURRENT_HTTP_PROXY';
process.env.HTTPS_PROXY = 'CURRENT_HTTP_PROXY';
process.env.NO_PROXY = 'localhost,127.0.0.1,::1';
process.env.NODE_USE_ENV_PROXY = '1';
process.env.NODE_REPL_UNTRUSTED_ENV_ALLOWLIST = [...new Set([
  ...(process.env.NODE_REPL_UNTRUSTED_ENV_ALLOWLIST || '')
    .split(',').filter(Boolean),
  'HTTP_PROXY', 'HTTPS_PROXY', 'NO_PROXY', 'NODE_USE_ENV_PROXY'
])].join(',');
```

Do not replace the whole file with this snippet. Preserve the rest of the code and preserve existing allowlist entries.

Then validate the file syntax using the Node.js environment that belongs to that runtime:

```text
node --check <path-to-cua-repl.mjs>
```

If validation fails, restore the backup.

### Step 5. Fully restart Codex

Closing the window may leave the app running in the background. A JavaScript session reset is also not the same as restarting the desktop process.

Save your work, fully exit the desktop app, then start it again while the browser and required VPN/proxy are active.

Do **not** kill every Node.js process on the machine.

### Step 6. Test the browser again

Repeat the real browser test from section 4.

After a Codex update, check the active runtime path again. A new version may create a new runtime folder, leaving the previous patched file unused.

---

## 7. Full setup prompt for Codex

Copy this into Codex if you want it to guide you through the complete setup and diagnose browser failures:

```text
Help me connect Codex Desktop on Windows to Google Chrome and/or Microsoft Edge and verify that the browser integration actually works.

Work carefully and diagnose before changing anything.

1. Inspect the current Codex/Desktop environment and identify which browser integration is available in this version.
2. Help me install or verify the official ChatGPT browser extension from OpenAI for Chrome and/or Edge.
3. Verify that the correct browser profile is used and that Codex has permission for the target website.
4. Do not treat a Connected/Manage label as proof of success. Run a real read-only browser test: open a public page, read it, follow one link, and read the destination page.
5. If Codex cannot see the browser or tabs, refresh browser discovery and use the current browser connection rather than IDs copied from another chat.
6. If errors include Browser is not available, Unexpected token '<' ... is not valid JSON, nodeRepl.fetch request failed, an empty browser list, or similar connector failures, preserve the exact error and diagnose the network/browser-runtime path before reinstalling everything.

If the failure appeared after a VPN, proxy, network, or IP/route change, determine the current local HTTP proxy used by Windows and the VPN/proxy client. Do not confuse the local proxy endpoint with the external/public IP. Verify both the local port and a real external request through the proxy. Compare that with the configuration used by the actually active Codex browser runtime.

Only if diagnostics confirm that the active browser runtime is still using an outdated proxy and the current Codex version is compatible with the workaround, back up the active cua-repl.mjs file before editing it. A path like %LOCALAPPDATA%\OpenAI\Codex\runtimes\cua_node\<version>\bin\node_modules\@oai\cua-repl\bin\cua-repl.mjs is only an example; determine the runtime that is actually active. chrome-native-hosts-v2.json may contain stale entries.

In a compatible version, before await cua_repl.launch(); the workaround sets HTTP_PROXY and HTTPS_PROXY to the verified current local HTTP proxy, NO_PROXY to localhost,127.0.0.1,::1, and NODE_USE_ENV_PROXY=1, while preserving all existing NODE_REPL_UNTRUSTED_ENV_ALLOWLIST values and adding HTTP_PROXY, HTTPS_PROXY, NO_PROXY and NODE_USE_ENV_PROXY if missing.

Do not add duplicate blocks. Do not change security settings, credentials or unrelated configuration. Validate syntax with node --check using the Node.js environment for that runtime. If validation fails, restore the original file.

If a full restart is required, tell me clearly. Closing the window may not terminate the app, and resetting a JS session is not the same as restarting the desktop process. Do not kill every Node.js process and do not ask for a full computer reboot unless there is a separate reason.

After every change, repeat a real browser test. Tell me exactly what was verified in this chat, what was changed, where the backup is, and what remains unverified.
```

---

## 8. Recovery-only prompt for Codex

Use this shorter prompt when Chrome/Edge was already configured and working, but the browser connection stopped working after a VPN, proxy, network, route, or IP-related change.

```text
My Chrome/Edge integration in Codex was already configured and working, but Codex can no longer see or control the browser. The UI may still show Connected/Manage.

Do not reinstall everything first. Diagnose the current browser connection and network path.

1. Refresh browser discovery and verify the current Chrome/Edge connection and profile.
2. Run a real read-only browser test and preserve the exact error if it fails.
3. Check whether the failure followed a VPN, proxy, network, route, or IP-related change.
4. Determine the current local HTTP proxy used by Windows and the VPN/proxy client. Do not confuse it with the external/public IP.
5. Verify that the local proxy port is reachable and that a real external request succeeds through it.
6. Identify the actually active Codex cua_node runtime and compare its proxy/runtime configuration with the current working network configuration. Do not blindly use the first runtime folder you find; chrome-native-hosts-v2.json may contain stale entries.
7. Only if diagnostics confirm an outdated-proxy mismatch and this Codex version is compatible with the workaround, back up the active cua-repl.mjs file. Before await cua_repl.launch(); add or update the following block, replacing CURRENT_HTTP_PROXY with the verified current local HTTP proxy:

process.env.HTTP_PROXY = 'CURRENT_HTTP_PROXY';
process.env.HTTPS_PROXY = 'CURRENT_HTTP_PROXY';
process.env.NO_PROXY = 'localhost,127.0.0.1,::1';
process.env.NODE_USE_ENV_PROXY = '1';
process.env.NODE_REPL_UNTRUSTED_ENV_ALLOWLIST = [...new Set([
  ...(process.env.NODE_REPL_UNTRUSTED_ENV_ALLOWLIST || '').split(',').filter(Boolean),
  'HTTP_PROXY', 'HTTPS_PROXY', 'NO_PROXY', 'NODE_USE_ENV_PROXY'
])].join(',');

Do not create duplicate blocks, do not remove existing allowlist values, and do not modify unrelated settings. Validate syntax with node --check using the Node.js environment for that runtime. Restore the backup if validation fails.

If the change requires a restart, tell me to save active work, fully exit Codex Desktop, and start it again with the required browser/VPN/proxy active. Closing the window may leave the process running. Do not kill all Node.js processes.

After restart, continue from the current state and run a real browser test again. Report exactly what was verified, what was changed, where the backup is, and what remains unconfirmed.
```

---

## Important

The `cua-repl.mjs` modification is a user-discovered workaround for a specific class of proxy/runtime mismatch problems. It is **not** an official OpenAI configuration method and it is not a universal fix for every browser-connector failure.

Always diagnose first. A Codex update may replace or relocate the internal runtime, so the path and compatibility must be checked again after updates.
