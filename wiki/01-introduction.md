# 01. Introduction

MGD App Updater Skill is a practical guide for planning software update systems with AI coding agents.

It is not a finished updater library. It is a planning skill that helps an agent choose a safe update strategy before writing code.

## The basic idea

```text
Installed app
↓
Update manifest
↓
Update dialog
↓
Download or installation
```

A useful first update system can be very simple. The app checks a JSON file, compares versions and shows a download link.

## Why planning matters

Update systems can replace software on a user's computer. That makes them security-sensitive. A bad updater can become a dangerous remote code execution mechanism.

This skill keeps the first implementation simple and prepares later security steps such as checksums, signatures, forced updates and rollback.
