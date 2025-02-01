# Tailscale No-GUI 版本 for macOS

## 项目说明

本项目旨在解决在部分场景下，需要通过代理访问 Tailscale 登录接口的问题。由于 macOS 下的 Tailscale GUI 版本无法配置代理，因此编译了 macOS ARM 版本的 Tailscale No-GUI 版本。

默认代理配置如下：
- `HTTP_PROXY=http://127.0.0.1:6152`
- `HTTPS_PROXY=http://127.0.0.1:6152`

用户可以通过修改 `com.tailscale.tailscaled.plist` 文件来更改代理配置。

## 使用说明

### 1. 下载并放置二进制文件

首先，下载编译好的 `tailscale` 和 `tailscaled` 文件，并将它们放入 `/usr/local/bin` 目录下。

### 2. 配置执行权限

为 `tailscale` 和 `tailscaled` 配置执行权限：

```bash
chmod +x /usr/local/bin/tailscale
chmod +x /usr/local/bin/tailscaled
xattr -r -d com.apple.quarantine /usr/local/bin/tailscale
xattr -r -d com.apple.quarantine /usr/local/bin/tailscald
```

### 3. 下载并放置 plist 文件

下载 `com.tailscale.tailscaled.plist` 文件，并将其放入 `/Library/LaunchDaemons` 目录下。

### 4. 配置 plist 文件权限

配置 `com.tailscale.tailscaled.plist` 文件的权限：

```bash
sudo chown root:wheel /Library/LaunchDaemons/com.tailscale.tailscaled.plist
sudo chmod 644 /Library/LaunchDaemons/com.tailscale.tailscaled.plist
```

### 5. 加载 plist 文件

加载 `com.tailscale.tailscaled.plist` 文件：

```bash
sudo launchctl unload /Library/LaunchDaemons/com.tailscale.tailscaled.plist
```
```bash
sudo launchctl load /Library/LaunchDaemons/com.tailscale.tailscaled.plist
```

### 6. 启动 Tailscale

最后，启动 Tailscale：

```bash
tailscale up
```

## 修改代理配置

如果需要修改代理配置，可以编辑 `com.tailscale.tailscaled.plist` 文件，找到 `EnvironmentVariables` 部分，修改 `https_proxy` 和 `http_proxy` 的值。

```xml
<key>EnvironmentVariables</key>
<dict>
    <key>HTTP_PROXY</key>
    <string>http://127.0.0.1:6152</string>
    <key>HTTPS_PROXY</key>
    <string>http://127.0.0.1:6152</string>
</dict>
```

修改完成后，重新加载 plist 文件：

```bash
launchctl unload /Library/LaunchDaemons/com.tailscale.tailscaled.plist
launchctl load /Library/LaunchDaemons/com.tailscale.tailscaled.plist
```

## 注意事项

- 本项目基于 Tailscale 官方仓库（[https://github.com/tailscale/tailscale](https://github.com/tailscale/tailscale)）自动编译，确保使用最新版本。
- 请确保代理服务器已正确配置并运行。

## 贡献

欢迎提交 Issue 或 Pull Request 来改进本项目。

## 许可证

本项目遵循 MIT 许可证。详情请参阅 [LICENSE](LICENSE) 文件。


# Tailscaled on macOS 官方介绍

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

