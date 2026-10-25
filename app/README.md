# LifeMe Dost Android app module

This directory restores a minimal Android application module so the repository has an Android manifest, Gradle module configuration, and a Jetpack Compose launcher activity.

## Current status

- Package/application ID: `com.lifemetech.dost`
- Minimum SDK: 26
- Target SDK: 34
- UI: minimal Compose welcome screen

**Important:** This is a starter reconstruction, not a restoration of the missing original application source. Features described in the root README (AI agent, chat, memory, reminders, voice, Firebase, and Drive backup) are not implemented by these starter files.

The root Gradle configuration also references a `:shared` module. That module must exist or be removed from `settings.gradle.kts` before the complete project can build. Configure Firebase and other service credentials separately; do not commit private keys.
