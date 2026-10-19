# Nuticast for Claude

Let Claude get an iPhone on a different network ready for Xcode. This plugin
connects Claude Code and Claude Desktop to [Nuticast](https://www.nuticast.com),
a Mac app that makes an iPhone on another Wi-Fi show up in Xcode as a wireless
debug device, over your own Tailscale, ZeroTier or Netbird network.

Ask Claude to "run the tests on my iPhone" and it can list your saved devices,
start the bridge, wait until Xcode sees the phone, and build against the
device with the right `xcodebuild` destination.

## Requirements

- A Mac with the [Nuticast app](https://www.nuticast.com/download) installed in
  `/Applications` (the plugin starts the `nuticast` command inside it), version
  1.0.15 or later. With Homebrew:
  `brew install --cask webwarrior06/tap/nuticast`
- A Nuticast license (the bridge needs one; reading devices does not)
- An iPhone saved in Nuticast and a mesh VPN (Tailscale, ZeroTier or Netbird)
  on both devices
- Claude Code or Claude Desktop on that Mac. This is a local server, so it does
  not run in claude.ai chat or on mobile.

## What it adds

A skill that tells Claude how to use the tools in the right order, and a local
MCP server (`nuticast mcp`, started by Claude over stdio) with these tools:

| Tool | What it does | Changes anything |
| --- | --- | --- |
| `nuticast_list_devices` | Lists saved iPhones and how each is reached | No |
| `nuticast_list_mesh_peers` | Asks Tailscale and Netbird for the devices on your network | No |
| `nuticast_scan_lan` | Finds iPhones advertising wireless debugging on this Wi-Fi | No |
| `nuticast_list_xcode_devices` | Lists the devices Xcode can see, with UDIDs | No |
| `nuticast_bridge_status` | Shows which devices are bridged by the command line | No |
| `nuticast_wait_for_bridge` | Waits for a bridge to come up | No |
| `nuticast_get_logs` | Reads the last lines of the Nuticast log (at most 200) | No |
| `nuticast_license_status` | Shows whether this Mac is licensed | No |
| `nuticast_diagnose` | Probes a device over the mesh; saves a moved port | Saves a port |
| `nuticast_start_bridge` | Starts a bridge for a saved device | Starts a process |
| `nuticast_ensure_device_ready` | Starts the bridge if needed and waits for Xcode | Starts a process |
| `nuticast_stop_bridge` | Stops a command-line bridge | Ends a debug session |

It cannot delete devices, activate or deactivate a license, or install apps.

## Install

Install the plugin from the Claude plugin directory (Customize, then Plugins).

To use only the tools, without the skill, add the server by hand in Claude Code:

```
claude mcp add --scope user nuticast -- /Applications/Nuticast.app/Contents/Helpers/nuticast mcp
```

## What it runs and what it sends

The plugin contains no program of its own. It only starts the signed,
notarized `nuticast` command that ships inside Nuticast.app, and talks to it
over standard input and output. While tools run, that command may run
`tailscale`, `netbird`, `zerotier-cli`, `dns-sd` and `devicectl` on your Mac, as
the Nuticast app does.

The only network request Nuticast makes because of this plugin is the license
check against `nuticast.com`, when you start a bridge. It sends your license key,
the activation's instance ID and a hardware identifier of your Mac.

Tool results (device names, addresses, peer names, log lines) are returned to
Claude and become part of your conversation. Log lines and peer names can
contain mesh IP addresses and host names.

## Privacy Policy

Nuticast collects no usage data and has no account. The full policy, including
what the MCP tools return, what is stored and for how long, third parties, and
how to contact us, is at <https://www.nuticast.com/privacy>.

- Data collection: none by us from this plugin.
- Usage and storage: device profiles and the log stay on your Mac.
- Third-party sharing: tool output goes only to the MCP client you use; the
  license check goes only to our license service.
- Retention: we keep nothing from MCP calls. Your client keeps what it keeps.
- Contact: hello@nuticast.com

## License

The files in this repository are released under the MIT License. The Nuticast
app and the `nuticast` command are proprietary and licensed separately.
