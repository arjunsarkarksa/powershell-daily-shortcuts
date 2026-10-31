# PowerShell Productivity Shortcuts: `cc` and `cf`

Small commands. Everyday productivity.

If you work with PowerShell regularly, you probably copy console output or clear the terminal multiple times a day.

These two simple custom functions make those tasks faster:

| Command | Description |
|---|---|
| `cc` | Copy the currently visible console text to the clipboard |
| `cf` | Clear the console screen |

The goal is simple: spend less time managing your terminal and more time doing actual work.

## Why These Shortcuts?

When debugging an application, running scripts, checking logs, or troubleshooting errors, you often need to:

- Copy console output to share with teammates.
- Paste terminal output into GitHub Issues.
- Share error messages with AI assistants.
- Clear the terminal before starting a new task.

Instead of manually selecting text with your mouse, use `cc`.

Instead of typing `Clear-Host`, use `cf`.

## Installation

### Step 1: Open your PowerShell profile

```powershell
notepad $PROFILE
```

If the profile doesn't exist, create it:

```powershell
New-Item -ItemType File -Path $PROFILE -Force
```

### Step 2: Add the functions

Add the following functions to your PowerShell profile:

```powershell
# cc = Copy visible console content
function cc {
    $raw = $Host.UI.RawUI
    $pos = $raw.WindowPosition
    $size = $raw.WindowSize

    $rect = [System.Management.Automation.Host.Rectangle]::new(
        $pos.X,
        $pos.Y,
        $pos.X + $size.Width - 1,
        $pos.Y + $size.Height - 1
    )

    $buffer = $raw.GetBufferContents($rect)
    $lines = [System.Collections.Generic.List[string]]::new()

    for ($y = 0; $y -lt $size.Height; $y++) {
        $chars = [char[]]::new($size.Width)

        for ($x = 0; $x -lt $size.Width; $x++) {
            $chars[$x] = $buffer[$y, $x].Character
        }

        $line = (-join $chars).TrimEnd()

        if ($line -notmatch '^\s*PS .+>\s*cc\s*$') {
            $lines.Add($line)
        }
    }

    while ($lines.Count -gt 0 -and [string]::IsNullOrWhiteSpace($lines[0])) {
        $lines.RemoveAt(0)
    }

    while ($lines.Count -gt 0 -and [string]::IsNullOrWhiteSpace($lines[$lines.Count - 1])) {
        $lines.RemoveAt($lines.Count - 1)
    }

    Set-Clipboard -Value ($lines -join "`r`n")
}

# cf = Clear console
function cf {
    Clear-Host
    [Console]::Write("`e[3J")
}
```

Save the file.

### Step 3: Reload your profile

```powershell
. $PROFILE
```

Or simply close and reopen PowerShell.

## Usage

### Copy console output

```powershell
cc
```

The visible console text is copied to your clipboard.

Paste it anywhere using:

```text
Ctrl + V
```

Useful for:

- Sharing debugging output.
- Reporting errors.
- Sending logs to teammates.
- Copying terminal output into GitHub Issues or discussions.

### Clear the console

```powershell
cf
```

Clears the visible console screen.

## Example Workflow

```powershell
Get-Process
cc
```

Now paste the copied output into Notepad, a GitHub issue, or a chat.

When starting a new task:

```powershell
cf
```

Clean terminal. Fresh start.

## Compatibility

- PowerShell 7.x
- Windows
- Windows PowerShell console hosts that support `GetBufferContents()`

These functions are intended for interactive console sessions. Console buffer behavior may vary across terminal hosts.

## Contributing

Have a better implementation or another useful PowerShell shortcut?

Feel free to contribute!

Small improvements can make everyday development work much more comfortable.

---

**Keep it simple. Automate the little things.**









## Real-World Test: PowerShell 7 + ADB + Fastboot

I tested `cc` during USB driver troubleshooting on my Motorola Moto G05.

Windows initially detected the phone with driver error Code 28, so `fastboot devices` returned no device. I used PowerShell to locate the Motorola USB driver installer, installed it silently with Windows Installer, and refreshed device detection.

After the driver installation, Windows reported the phone as an OK Android USB device, and Fastboot detected it:

```powershell
fastboot devices
```

```text
ZY32LCMTM2       fastboot
```

I also queried the bootloader's product variable:

```powershell
fastboot getvar product
```

```text
product: lamul
Finished. Total time: 0.003s
```

This was a practical test of the shortcuts in an actual troubleshooting workflow—not just a sample command.

