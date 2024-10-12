---
title: Fixing VS Code Go to Definition Issue on Linux Mint
date: 2024-10-11
tags: ["misc"]
ShowToc: false
---

After switching from Linux Ubuntu to Mint (Cinnamon) on my work laptop, I discovered that the Go to Definition functionality in Visual Studio Code, which uses `Alt+Click`, had stopped working.

This issue had been driving me crazy for weeks. I tried multiple times during work to fix it, but all attempts were in vain. So I decided to dedicate a day on my weekend to investigate and resolve it.

The problem was that Mint, by default, uses `Alt+Click` to move and resize windows, which was overriding and conflicting with VS Code's Go to Definition functionality.

## Solution

In Mint's Windows Settings, change the "Special key to move and resize windows" from `<Alt>` to `<Super>`.
