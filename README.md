# Elbecity-System-
Elbecity System 
// ==========================================
// ElbeCity RP Discord Bot
// Discord.js v14
// ==========================================

const {
  Client,
  GatewayIntentBits,
  PermissionsBitField,
  EmbedBuilder,
  ActionRowBuilder,
  ButtonBuilder,
  ButtonStyle,
  ChannelType,
  SlashCommandBuilder,
  REST,
  Routes
} = require("discord.js");

// ==============================
// EINSTELLUNGEN
// ==============================

const TOKEN = "2a8cce4178e38907d9107be2fdb0f5946d652d1182669893dfe7c83a909d2335";
const CLIENT_ID = "1537496767233261700";
const GUILD_ID = "1537182681039773877";

// Kanal-IDs hier eintragen
const WELCOME_CHANNEL = "WELCOME_CHANNEL_ID";
const LOG_CHANNEL = "LOG_CHANNEL_ID";

// ==============================
// CLIENT
// ==============================

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMembers,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent,
    GatewayIntentBits.GuildModeration
  ]
});

// ==============================
// SLASH COMMANDS
// ==============================

const commands = [

  new SlashCommandBuilder()
    .setName("serverinfo")
    .setDescription("Zeigt Informationen über ElbeCity RP."),

  new SlashCommandBuilder()
    .setName("announce")
    .setDescription("Erstellt eine Ankündigung.")
    .addStringOption(option =>
      option
        .setName("text")
        .setDescription("Text der Ankündigung")
        .setRequired(true)
    ),

  new SlashCommandBuilder()
    .setName("kick")
    .setDescription("Kickt einen Benutzer.")
    .addUserOption(option =>
      option
        .setName("user")
        .setDescription("Benutzer")
        .setRequired(true)
    )
    .addStringOption(option =>
      option
        .setName("grund")
        .setDescription("Grund")
        .setRequired(false)
    ),

  new SlashCommandBuilder()
    .setName("ban")
    .setDescription("Bannt einen Benutzer.")
    .addUserOption(option =>
      option
        .setName("user")
        .setDescription("Benutzer")
        .setRequired(true)
    )
    .addStringOption(option =>
      option
        .setName("grund")
        .setDescription("Grund")
        .setRequired(false)
    ),

  new SlashCommandBuilder()
    .setName("timeout")
    .setDescription("Gibt einem Benutzer einen Timeout.")
    .addUserOption(option =>
      option
        .setName("user")
        .setDescription("Benutzer")
        .setRequired(true)
    )
    .addIntegerOption(option =>
      option
        .setName("minuten")
        .setDescription("Dauer in Minuten")
        .setRequired(true)
    ),

  new SlashCommandBuilder()
    .setName("warn")
    .setDescription("Warnt einen Benutzer.")
    .addUserOption(option =>
      option
        .setName("user")
        .setDescription("Benutzer")
        .setRequired(true)
    )
    .addStringOption(option =>
      option
        .setName("grund")
        .setDescription("Grund")
        .setRequired(true)
    ),

  new SlashCommandBuilder()
    .setName("ticket")
    .setDescription("Öffnet ein Support-Ticket."),

  new SlashCommandBuilder()
    .setName("bewerbung")
    .setDescription("Öffnet ein Bewerbungs-Ticket."),

  new SlashCommandBuilder()
    .setName("fraktion")
    .setDescription("Zeigt die Fraktionsverwaltung."),

  new SlashCommandBuilder()
    .setName("support")
    .setDescription("Zeigt das Support-System.")

].map(command => command.toJSON());

// ==============================
// COMMANDS REGISTRIEREN
// ==============================

const rest = new REST({ version: "10" }).setToken(TOKEN);

(async () => {
  try {
    console.log("Registriere Slash Commands...");

    await rest.put(
      Routes.applicationGuildCommands(CLIENT_ID, GUILD_ID),
      { body: commands }
    );

    console.log("Slash Commands wurden registriert!");
  } catch (error) {
    console.error(error);
  }
})();

// ==============================
// BOT READY
// ==============================

client.once("ready", () => {
  console.log("--------------------------------");
  console.log("ElbeCity RP Bot ist online!");
  console.log(`Eingeloggt als ${client.user.tag}`);
  console.log("--------------------------------");

  client.user.setPresence({
    activities: [
      {
        name: "ElbeCity RP 🚔",
        type: 3
      }
    ],
    status: "online"
  });
});

// ==============================
// WILLKOMMEN
// ==============================

client.on("guildMemberAdd", async member => {

  const channel = member.guild.channels.cache.get(WELCOME_CHANNEL);

  if (!channel) return;

  const embed = new EmbedBuilder()
    .setTitle("🌊 Willkommen bei ElbeCity RP!")
    .setDescription(
      `Willkommen ${member}!\n\n` +
      `Schön, dass du unserem Discord beigetreten bist.\n\n` +
      `📜 Lies dir bitte die Regeln durch.\n` +
      `🎫 Bei Fragen kannst du ein Ticket erstellen.\n` +
      `🎮 Viel Spaß bei ElbeCity RP!`
    )
    .setThumbnail(member.user.displayAvatarURL())
    .setColor("Blue")
    .setTimestamp();

  channel.send({ embeds: [embed] });
});

// ==============================
// LOG SYSTEM
// ==============================

async function sendLog(guild, title, description) {

  const channel = guild.channels.cache.get(LOG_CHANNEL);

  if (!channel) return;

  const embed = new EmbedBuilder()
    .setTitle(title)
    .setDescription(description)
    .setColor("Red")
    .setTimestamp();

  channel.send({ embeds: [embed] });
}

// ==============================
// INTERACTIONS
// ==============================

client.on("interactionCreate", async interaction => {

  if (!interaction.isChatInputCommand()) return;

  const { commandName } = interaction;

  // ==========================
  // SERVERINFO
  // ==========================

  if (commandName === "serverinfo") {

    const guild = interaction.guild;

    const embed = new EmbedBuilder()
      .setTitle("🌊 ElbeCity RP")
      .setDescription("Informationen über unseren Discord-Server.")
      .addFields(
        {
          name: "👥 Mitglieder",
          value: `${guild.memberCount}`,
          inline: true
        },
        {
          name: "💬 Kanäle",
          value: `${guild.channels.cache.size}`,
          inline: true
        },
        {
          name: "🎭 Rollen",
          value: `${guild.roles.cache.size}`,
          inline: true
        }
      )
      .setColor("Blue")
      .setTimestamp();

    return interaction.reply({ embeds: [embed] });
  }

  // ==========================
  // ANNOUNCE
  // ==========================

  if (commandName === "announce") {

    if (
      !interaction.member.permissions.has(
        PermissionsBitField.Flags.ManageGuild
      )
    ) {
      return interaction.reply({
        content: "❌ Du hast keine Berechtigung.",
        ephemeral: true
      });
    }

    const text = interaction.options.getString("text");

    const embed = new EmbedBuilder()
      .setTitle("📢 ElbeCity RP – Ankündigung")
      .setDescription(text)
      .setColor("Blue")
      .setFooter({
        text: `Ankündigung von ${interaction.user.tag}`
      })
      .setTimestamp();

    await interaction.channel.send({
      content: "@everyone",
      embeds: [embed]
    });

    interaction.reply({
      content: "✅ Ankündigung wurde gesendet.",
      ephemeral: true
    });
  }

  // ==========================
  // KICK
  // ==========================

  if (commandName === "kick") {

    if (
      !interaction.member.permissions.has(
        PermissionsBitField.Flags.KickMembers
      )
    ) {
      return interaction.reply({
        content: "❌ Du darfst keine Mitglieder kicken.",
        ephemeral: true
      });
    }

    const user = interaction.options.getUser("user");
    const grund =
      interaction.options.getString("grund") || "Kein Grund angegeben";

    const member = await interaction.guild.members.fetch(user.id);

    if (!member.kickable) {
      return interaction.reply({
        content: "❌ Ich kann diesen Benutzer nicht kicken.",
        ephemeral: true
      });
    }

    await member.kick(grund);

    await sendLog(
      interaction.guild,
      "👢 Mitglied gekickt",
      `**Benutzer:** ${user}\n**Moderator:** ${interaction.user}\n**Grund:** ${grund}`
    );

    interaction.reply({
      content: `✅ ${user.tag} wurde gekickt.\n**Grund:** ${grund}`
    });
  }

  // ==========================
  // BAN
  // ==========================

  if (commandName === "ban") {

    if (
      !interaction.member.permissions.has(
        PermissionsBitField.Flags.BanMembers
      )
    ) {
      return interaction.reply({
        content: "❌ Du darfst keine Mitglieder bannen.",
        ephemeral: true
      });
    }

    const user = interaction.options.getUser("user");
    const grund =
      interaction.options.getString("grund") || "Kein Grund angegeben";

    const member = await interaction.guild.members.fetch(user.id);

    if (!member.bannable) {
      return interaction.reply({
        content: "❌ Ich kann diesen Benutzer nicht bannen.",
        ephemeral: true
      });
    }

    await member.ban({ reason: grund });

    await sendLog(
      interaction.guild,
      "🔨 Mitglied gebannt",
      `**Benutzer:** ${user}\n**Moderator:** ${interaction.user}\n**Grund:** ${grund}`
    );

    interaction.reply({
      content: `🔨 ${user.tag} wurde gebannt.\n**Grund:** ${grund}`
    });
  }

  // ==========================
  // TIMEOUT
  // ==========================

  if (commandName === "timeout") {

    if (
      !interaction.member.permissions.has(
        PermissionsBitField.Flags.ModerateMembers
      )
    ) {
      return interaction.reply({
        content: "❌ Du darfst keine Timeouts vergeben.",
        ephemeral: true
      });
    }

    const user = interaction.options.getUser("user");
    const minuten = interaction.options.getInteger("minuten");

    if (minuten < 1 || minuten > 40320) {
      return interaction.reply({
        content: "❌ Erlaubt sind 1–40320 Minuten.",
        ephemeral: true
      });
    }

    const member = await interaction.guild.members.fetch(user.id);

    if (!member.moderatable) {
      return interaction.reply({
        content: "❌ Ich kann diesen Benutzer nicht timeouten.",
        ephemeral: true
      });
    }

    await member.timeout(
      minuten * 60 * 1000,
      `Timeout von ${interaction.user.tag}`
    );

    await sendLog(
      interaction.guild,
      "⏱️ Timeout",
      `**Benutzer:** ${user}\n**Moderator:** ${interaction.user}\n**Dauer:** ${minuten} Minuten`
    );

    interaction.reply({
      content: `⏱️ ${user.tag} wurde für **${minuten} Minuten** timeoutet.`
    });
  }

  // ==========================
  // WARN
  // ==========================

  if (commandName === "warn") {

    if (
      !interaction.member.permissions.has(
        PermissionsBitField.Flags.ModerateMembers
      )
    ) {
      return interaction.reply({
        content: "❌ Du darfst keine Verwarnungen vergeben.",
        ephemeral: true
      });
    }

    const user = interaction.options.getUser("user");
    const grund = interaction.options.getString("grund");

    await sendLog(
      interaction.guild,
      "⚠️ Verwarnung",
      `**Benutzer:** ${user}\n**Moderator:** ${interaction.user}\n**Grund:** ${grund}`
    );

    interaction.reply({
      content:
        `⚠️ ${user.tag} wurde verwarnt.\n` +
        `**Grund:** ${grund}`
    });
  }

  // ==========================
  // TICKET
  // ==========================

  if (commandName === "ticket") {

    const existing = interaction.guild.channels.cache.find(
      channel =>
        channel.name === `ticket-${interaction.user.username.toLowerCase()}`
    );

    if (existing) {
      return interaction.reply({
        content: `❌ Du hast bereits ein Ticket: ${existing}`,
        ephemeral: true
      });
    }

    const channel = await interaction.guild.channels.create({
      name: `ticket-${interaction.user.username}`,
      type: ChannelType.GuildText,
      permissionOverwrites: [
        {
          id: interaction.guild.id,
          deny: [PermissionsBitField.Flags.ViewChannel]
        },
        {
          id: interaction.user.id,
          allow: [
            PermissionsBitField.Flags.ViewChannel,
            PermissionsBitField.Flags.SendMessages
          ]
        }
      ]
    });

    const embed = new EmbedBuilder()
      .setTitle("🎫 ElbeCity RP Support")
      .setDescription(
        `Hallo ${interaction.user}!\n\n` +
        `Beschreibe hier dein Anliegen.\n` +
        `Ein Teammitglied wird sich schnellstmöglich darum kümmern.`
      )
      .setColor("Blue");

    const button = new ActionRowBuilder().addComponents(
      new ButtonBuilder()
        .setCustomId("ticket_close")
        .setLabel("Ticket schließen")
        .setEmoji("🔒")
        .setStyle(ButtonStyle.Danger)
    );

    await channel.send({
      content: `${interaction.user}`,
      embeds: [embed],
      components: [button]
    });

    interaction.reply({
      content: `✅ Dein Ticket wurde erstellt: ${channel}`,
      ephemeral: true
    });
  }

  // ==========================
  // BEWERBUNG
  // ==========================

  if (commandName === "bewerbung") {

    const channel = await interaction.guild.channels.create({
      name: `bewerbung-${interaction.user.username}`,
      type: ChannelType.GuildText,
      permissionOverwrites: [
        {
          id: interaction.guild.id,
          deny: [PermissionsBitField.Flags.ViewChannel]
        },
        {
          id: interaction.user.id,
          allow: [
            PermissionsBitField.Flags.ViewChannel,
            PermissionsBitField.Flags.SendMessages
          ]
        }
      ]
    });

    const embed = new EmbedBuilder()
      .setTitle("📋 ElbeCity RP Bewerbung")
      .setDescription(
        `Hallo ${interaction.user}!\n\n` +
        `Bitte beantworte folgende Fragen:\n\n` +
        `1️⃣ Wie heißt du?\n` +
        `2️⃣ Wie alt bist du?\n` +
        `3️⃣ Für welchen Bereich bewirbst du dich?\n` +
        `4️⃣ Warum möchtest du ins Team?\n` +
        `5️⃣ Was sind deine Stärken?\n` +
        `6️⃣ Hast du bereits Erfahrung?\n`
      )
      .setColor("Blue");

    await channel.send({
      content: `${interaction.user}`,
      embeds: [embed]
    });

    interaction.reply({
      content: `✅ Deine Bewerbung wurde geöffnet: ${channel}`,
      ephemeral: true
    });
  }

  // ==========================
  // FRAKTION
  // ==========================

  if (commandName === "fraktion") {

    const embed = new EmbedBuilder()
      .setTitle("🎭 ElbeCity RP Fraktionen")
      .setDescription(
        "Hier kannst du deine RP-Fraktion verwalten.\n\n" +
        "🚓 Polizei\n" +
        "🚑 Rettungsdienst\n" +
        "🚒 Feuerwehr\n" +
        "🔧 Abschleppdienst\n" +
        "🏢 Unternehmen\n" +
        "🔫 Kriminelle Fraktionen"
      )
      .setColor("Blue");

    interaction.reply({ embeds: [embed] });
  }

  // ==========================
  // SUPPORT
  // ==========================

  if (commandName === "support") {

    const embed = new EmbedBuilder()
      .setTitle("🚨 ElbeCity RP Support")
      .setDescription(
        "Du benötigst Hilfe?\n\n" +
        "🎫 Nutze `/ticket`, um ein Support-Ticket zu erstellen.\n\n" +
        "📋 Für eine Team-Bewerbung nutze `/bewerbung`.\n\n" +
        "⚠️ Bitte eröffne keine Tickets ohne Grund."
      )
      .setColor("Blue");

    interaction.reply({ embeds: [embed] });
  }
});

// ==============================
// TICKET BUTTON
// ==============================

client.on("interactionCreate", async interaction => {

  if (!interaction.isButton()) return;

  if (interaction.customId === "ticket_close") {

    await interaction.reply({
      content: "🔒 Ticket wird in 5 Sekunden geschlossen..."
    });

    setTimeout(async () => {

      try {
        await interaction.channel.delete();
      } catch (error) {
        console.log(error);
      }

    }, 5000);
  }
});

// ==============================
// EINFACHE AUTO-MODERATION
// ==============================
client.on("messageCreate", async message => {

  if (message.author.bot) return;

  const content = message.content.toLowerCase();

  const found = badWords.some(word =>
    content.includes(word)
  );

  if (!found) return;

  try {
    await message.delete();

    await message.channel.send(
      `⚠️ ${message.author}, deine Nachricht wurde wegen unangemessener Sprache entfernt.`
    );

    await sendLog(
      message.guild,
      "🤖 Auto-Moderation",
      `**Benutzer:** ${message.author}\n` +
      `**Kanal:** ${message.channel}\n` +
      `Eine Nachricht wurde automatisch entfernt.`
    );

  } catch (error) {
    console.log(error);
  }
});

// ==============================
// BOT STARTEN
// ==============================

client.login(TOKEN);
