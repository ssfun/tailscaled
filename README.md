# Tailscaled on macOS

There are multiple ways to use Tailscale on macOS. The recommended way is to always install the Standalone variant, available for download from the Tailscale website.

This page is about how to use the open source, non-GUI `tailscaled` and `tailscale` binaries. This is only recommended for advanced users.

## Requirements

### Install Go

Install Go 1.21 (or whatever the most recently released Go version is) from [https://golang.org/dl/](https://golang.org/dl/) or Homebrew, etc. Tailscale always requires the most recent Go version and doesn't support older ones.

## Compile Tailscale

Run:

```bash
go install tailscale.com/cmd/tailscale{,d}@main
```

That'll put the binaries in `$(go env GOPATH)/bin`, so likely `$HOME/go/bin`. You can copy or symlink those binaries into your `$PATH`, or make your `$PATH` include that directory.

You can also compile from a specific release version. For example, to build from the source code used for Tailscale 1.38.2, use:

```bash
go install tailscale.com/cmd/tailscale{,d}@v1.38.2
```

## Run the `tailscaled` (daemon)

```bash
sudo $HOME/go/bin/tailscaled
```

Or, to run it in the background under `launchd` so it starts at system boot:

```bash
sudo $HOME/go/bin/tailscaled install-system-daemon
```

That copies the binary to `/usr/local/bin` and installs a plist in `/Library/LaunchDaemons/com.tailscale.tailscaled.plist` and starts `com.tailscale.tailscaled`.

(to stop/uninstall, use: `sudo tailscaled uninstall-system-daemon`)

## Use the `tailscale` CLI tool

See [https://tailscale.com/kb/1080/cli](https://tailscale.com/kb/1080/cli) (but ignore the `/Applications/Tailscale.app/Contents/MacOS/Tailscale` part; that's the path to the GUI's CLI)

```bash
tailscale up       # (any optional arguments)
tailscale status
```

Enjoy.

## Comparison to GUI version

Compared to the GUI version of Tailscale, running `tailscaled` instead has the following differences:

- `tailscaled` on macOS is less tested.
- The App Store version uses the Apple Network Extension API; `tailscaled` uses the `/dev/utun` TUN interface.
- MagicDNS works, but you need to set `100.100.100.100` as your DNS server yourself. It doesn't change your DNS config.
- `tailscaled` can run at system boot before any user has logged in (e.g., letting you VNC to your computer after a power outage).
- It is fully open source (Tailscale GUI parts aren't open source on non-free operating systems).

Refer to the comparison available in the Tailscale KB for more details.
```
