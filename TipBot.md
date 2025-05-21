# Integrating Cardano Tip Bot with Minecraft Server Using EssentialsX Discord

This guide outlines how to integrate the Cardano Tip Bot with your Minecraft server using EssentialsX Discord, enabling players to send Cardano Tip Bot commands from Minecraft to Discord.

## Overview
- **EssentialsX Discord**: A plugin that bridges Minecraft and Discord, relaying in-game chat and commands to Discord channels and vice versa.
- **Cardano Tip Bot**: A Discord bot (Cardano Tip Bot#7235) that processes tipping commands (e.g., `/tip @DiscordUser <amount> <asset>`) in public Discord channels.
- **Goal**: Allow players to send commands like `/tip @user 1 ADA` from Minecraft, relayed via EssentialsX Discord to the Discord server for processing by the Cardano Tip Bot.

The challenge is ensuring EssentialsX Discord relays commands in a format the Cardano Tip Bot can process.

## Prerequisites
- **Cardano Tip Bot**: Installed and active in a public Discord channel (e.g., #tipping-channel).
- **EssentialsX and EssentialsX Discord**: Installed on a Spigot/Paper Minecraft server.
- **Permissions Plugin**: LuckPerms recommended for managing command access.
- **Minecraft Server**: Running Spigot/Paper, preferably Paper for performance.
- **Discord Developer Portal Access**: Required for configuring the EssentialsX Discord bot.

## Steps to Integrate

### 1. Verify Cardano Tip Bot Setup
- Ensure the Cardano Tip Bot is active in a public Discord channel (e.g., #tipping-channel).
- Test the bot in Discord with a command like `/tip @CardanoTipBot 1 ADA` to confirm functionality.
- Note the channel ID where the bot accepts commands for later configuration.

### 2. Configure EssentialsX Discord
EssentialsX Discord relays Minecraft chat and commands to a specified Discord channel.

**Steps**:
1. **Set Up EssentialsX Discord**:
   - Follow the setup guide at [essentialsx.net/discord.html](https://essentialsx.net/discord.html).
   - Create a Discord bot at [discord.com/developers/applications](https://discord.com/developers/applications), copy its Application ID and Token, and add it to your Discord server.
   - Enable Developer Mode in Discord (Settings > Advanced > Enable Developer Mode) to copy the server ID and channel IDs.
   - Copy the ID of the Discord channel where the Cardano Tip Bot is active.
   - In `plugins/EssentialsDiscord/config.yml`, set:
     ```yaml
     guild-id: <your-discord-server-id>
     primary-channel: <tipping-channel-id>
     ```

2. **Configure Message Types**:
   - In `plugins/EssentialsDiscord/config.yml`, configure the `message-types` section:
     ```yaml
     message-types:
       player-chat: "<tipping-channel-id>"
       player-command: "<tipping-channel-id>"
     ```
   - This ensures commands or chat from Minecraft are sent to the Cardano Tip Bot’s channel.

3. **Enable Command Relaying**:
   - In `config.yml`, ensure `show-commands` is enabled for the `tip` command:
     ```yaml
     commands:
       tip:
         enabled: true
         show-commands: true
         allowed-roles: ["*"]
     ```

4. **Restart the Server**:
   - Restart your Minecraft server to apply changes (`stop` and `start` in the server console).

### 3. Create a Custom Command in Minecraft
The Cardano Tip Bot expects commands in the format `/tip @DiscordUser <amount> <asset>`. Create a custom Minecraft command to match this format.

**Option 1: EssentialsX Custom Command**:
- Edit `plugins/Essentials/config.yml` and add an alias:
  ```yaml
  aliases:
    tip:
      - "msg $1 /tip @$2 $3 $4"
  
This alias allows players to use `/tip <player> <amount> <asset>` in Minecraft, which sends a private message to the EssentialsX Discord bot, formatted as a `/tip` command for the Cardano Tip Bot. For example, `/tip Steve 1 ADA` in Minecraft would send `/tip @Steve 1 ADA` to Discord.

**Option 2: Use a Command Plugin**  
If EssentialsX aliases don’t work as expected, consider using a plugin like CommandAPI or MyCommand to create a custom command that sends a formatted message to the Discord channel.

**Example with MyCommand**:
- Install [MyCommand](https://www.spigotmc.org/resources/mycommand.22272/).
- Create a command in `plugins/MyCommand/commands.yml`:
  ```yaml
  tip:
    command: tip
    type: RUN_COMMAND
    runcmd:
      - '/discord sendmessage <tipping-channel-id> /tip @$1 $2 $3'
    permission-required: true
    permission: cardano.tip
This sends the command directly to the Discord channel (replace `<tipping-channel-id>` with the actual channel ID).

### 4. Link Minecraft and Discord Accounts (Optional)
To ensure the Cardano Tip Bot recognizes the Discord user associated with a Minecraft player, use the EssentialsX Discord Link addon to link player accounts:
- Install the EssentialsX Discord Link addon (`EssentialsXDiscordLink.jar`) in your `plugins/` folder.
- Start the server to generate the configuration file (`plugins/EssentialsDiscordLink/config.yml`).
- In Minecraft, players run `/link` to receive a code, then run `/link <code>` in Discord to link their accounts.
- This step ensures the Cardano Tip Bot can associate the Minecraft player’s command with their Discord user (e.g., mapping `Steve` to `@Steve#1234`).

### 5. Test the Integration
- In Minecraft, have a player run the custom command, e.g., `/tip Steve 1 ADA`.
- Check the Discord channel (#tipping-channel) to confirm that the EssentialsX Discord bot relayed the command as `/tip @Steve 1 ADA`.
- Verify that the Cardano Tip Bot processes the command and responds in Discord (e.g., confirming the tip was sent).
- **If the bot doesn’t respond, ensure**:
  - The command format matches exactly what the Cardano Tip Bot expects (`/tip @DiscordUser <amount> <asset>`).
  - The Discord channel ID in `config.yml` is correct.
  - The EssentialsX Discord bot has permissions to send messages in the channel (Manage Channels, Send Messages, Read Messages).
  - The Cardano Tip Bot is online and has the necessary permissions in the Discord channel.

### 6. Handle Permissions
- Use LuckPerms to restrict who can use the `/tip` command in Minecraft:
  ```bash
  /lp group default permission set cardano.tip true

  This ensures only authorized players can send tipping commands.

- In Discord, ensure the Cardano Tip Bot has permissions to read and respond to messages in the target channel. You may also restrict who can use the `/tip` command in Discord by setting role-based permissions in the EssentialsX Discord config (`allowed-roles`).

### 7. Troubleshooting
- **Command Not Relaying**: Check the `message-types` and `commands` sections in `plugins/EssentialsDiscord/config.yml`. Ensure `show-commands` is `true` and the channel ID is correct.
- **Cardano Tip Bot Not Responding**: Verify the command format matches the bot’s requirements. Test the command directly in Discord to isolate issues. Ensure the bot is online (Cardano Tip Bot#7235).
- **Version Mismatch**: Ensure EssentialsX and EssentialsX Discord are the same version (e.g., 2.20.1). Mismatched versions can cause issues.
- **Permissions Issues**: Confirm the EssentialsX Discord bot has the necessary Discord permissions (Manage Webhooks, Send Messages, Read Messages) and that Privileged Intents are enabled in the Discord Developer Portal.
- **Logs**: Check the server logs (`logs/latest.log`) for errors related to EssentialsX Discord or command execution.

### 8. Additional Considerations
- **Security**: The Cardano Tip Bot warns about scams. Ensure players only interact with the official bot (Cardano Tip Bot#7235). Warn players not to send funds to unknown addresses.
- **Fees**: The Cardano Tip Bot requires sufficient ADA to cover transaction fees (e.g., ~0.3 ADA per transaction). Ensure players deposit enough ADA (recommended: 3 ADA) to avoid issues.
- **Alternative Plugin**: If EssentialsX Discord doesn’t meet your needs, consider [DiscordSRV](https://www.spigotmc.org/resources/discordsrv.18494/), which some users report as more robust for Discord-Minecraft bridging. It supports similar chat/command relaying and may offer more flexibility.

## Example Workflow
1. Player Steve runs `/tip Alex 1 ADA` in Minecraft.
2. EssentialsX Discord relays the command to the #tipping-channel in Discord as `/tip @Alex 1 ADA`.
3. The Cardano Tip Bot processes the command, checks Steve’s linked Discord account balance, and sends 1 ADA to Alex’s wallet, responding in Discord with a confirmation.
4. The confirmation message is relayed back to Minecraft via EssentialsX Discord (if configured).

## Limitations
- **Command Format**: The Cardano Tip Bot requires Discord-specific user tags (e.g., `@Alex#1234`). If players’ Minecraft usernames don’t match their Discord tags, you may need a mapping system (e.g., via EssentialsX Discord Link) to resolve this.
- **Bot Restrictions**: The Cardano Tip Bot only processes commands in public Discord channels, so ensure the relay targets a public channel.
- **Performance**: Relaying commands adds slight latency, and frequent tipping could strain the server or bot if not optimized. Consider rate-limiting commands with a permissions plugin.

## Resources
- [EssentialsX Discord Tutorial](https://essentialsx.net/discord.html)
- [Cardano Tip Bot Guide](https://github.com/QCPOLstakepool/cardano-tip-bot)
- [EssentialsX Discord Link](https://github.com/EssentialsX/Essentials/pull/4155)
- [Shockbyte Guide for EssentialsX](https://shockbyte.com/billing/knowledgebase/143/How-to-Install-and-Configure-EssentialsX-on-Your-Minecraft-Server.html)
- [DiscordSRV (Alternative)](https://www.spigotmc.org/resources/discordsrv.18494/)
