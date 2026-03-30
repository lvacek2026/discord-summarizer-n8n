# 🤖 Discord Summarizer — n8n workflow

Automatický AI summarizér Discord serverů. Dvakrát denně projde všechny textové kanály, vytvoří přehledné shrnutí pomocí Claude AI a pošle ho jako Discord DM a/nebo Slack zprávu.

---

## 📋 Ukázka výstupu

```
🤖 Discord Shrnutí — 30. 3. 2026 08:30
Aktivní kanály: 5

## #iotstack
s474n se ptal na BYOD přístup k AI nástrojům. dittmanncz vyjádřil frustraci 
že mají v práci zakázaný Claude a k dispozici jen ChatGPT. Otázka zůstala nevyřešena.

## #energie  
otava5 se zeptal na průměrné ceny elektřiny Kč/kWh. lipown odpověděl konkrétními 
čísly: 3,2 Kč/kWh včetně daní. Zmínil také topení.

## #ev-a-vse-okolo
sumerian666 sdílel dvě videa o elektromobilech — Video o Škodě PEAQ a srovnání 
Tesly vs. tradičních aut. Bez diskuze.
```

---

## 🗺️ Jak workflow funguje

```
⏰ Schedule Trigger (2× denně)
  │
  ▼
🌐 Načte seznam všech kanálů ze serveru (Discord API)
  │
  ▼
⚙️ Filtruje pouze textové kanály
   (přeskočí hlasové, kategorie, nsfw, admin, archivní...)
  │
  ▼
🌐 Pro každý kanál načte nové zprávy od posledního běhu
   (pamatuje si kde skončil — nenačítá znovu to samé)
  │
  ▼
⚙️ Zpracuje zprávy (text, přílohy, embedy, obrázky)
  │
  ▼
⚙️ Agreguje vše dohromady (max 20 000 znaků)
  │
  ▼
❓ Jsou nějaké zprávy?
  ├── NE → konec
  └── ANO ↓
       │
       ▼
  🤖 Claude AI vytvoří shrnutí v češtině
       │
       ├──▶ 💬 Slack webhook (zpráva do kanálu nebo DM)
       └──▶ 📨 Discord DM každému příjemci ze seznamu
```

---

## ✅ Co budeš potřebovat

| Co | Kde získat | Cena |
|---|---|---|
| **n8n** (self-hosted) | [n8n.io](https://n8n.io) nebo Docker | Zdarma |
| **Discord Bot Token** | [discord.com/developers](https://discord.com/developers/applications) | Zdarma |
| **Anthropic API klíč** | [console.anthropic.com](https://console.anthropic.com) | ~$0,10–0,20/měsíc |
| **Slack Webhook** (volitelné) | [api.slack.com/apps](https://api.slack.com/apps) | Zdarma |

---

## 🚀 Instalace krok za krokem

### Krok 1 — Vytvoř Discord bota

1. Jdi na [discord.com/developers/applications](https://discord.com/developers/applications)
2. Klikni **"New Application"** → zadej název (např. `Discord Summarizer`)
3. V levém menu klikni na **"Bot"**
4. Klikni **"Add Bot"** → potvrď
5. Klikni **"Reset Token"** → **okamžitě zkopíruj a ulož token** (uvidíš ho jen jednou!)
6. Níže v sekci **"Privileged Gateway Intents"** zapni:
   - ✅ **Message Content Intent** ← BEZ TOHO NEBUDE ČÍST ZPRÁVY!
7. Klikni **"Save Changes"**

**Vytvoř pozvánkový odkaz pro bota:**
1. Stále na Developer Portalu → vlevo klikni **"OAuth2"**
2. V sekci **"Scopes"** zaškrtni: `bot`
3. V sekci **"Bot Permissions"** zaškrtni: `View Channels` + `Read Message History`
4. Zkopíruj vygenerovaný odkaz ze spodní části stránky

**⚠️ Důležité:** Na serverech kde nejsi admin musíš poslat tento odkaz správci serveru a požádat ho o přidání bota. Bez jeho souhlasu a akce bot na server nezíská přístup.

---

### Krok 2 — Zjisti Guild ID (ID serveru)

1. Otevři Discord
2. Jdi do **Nastavení → Pokročilé → zapni Vývojářský režim**
3. Pravý klik na ikonu serveru v levém panelu → **"Kopírovat ID serveru"**
4. Ulož si toto číslo — budeš ho potřebovat

---

### Krok 3 — Anthropic API klíč

1. Jdi na [console.anthropic.com](https://console.anthropic.com) a přihlas se
2. Vlevo klikni **"API Keys"** → **"Create Key"**
3. Pojmenuj ho (např. `n8n-discord`) → zkopíruj a ulož
4. Jdi na **Billing** → přidej platební kartu → kup kredit za $5
   - Při 2× denním běhu ti vydrží **roky**
   - Kredity vyprší za 1 rok od nákupu — doporučuji zapnout auto-reload na $5

---

### Krok 4 — Slack Webhook (volitelné)

Pokud nechceš Slack, tento krok přeskoč a v n8n uzel "Slack - Odesli" jednoduše smaž.

1. Jdi na [api.slack.com/apps](https://api.slack.com/apps) → **"Create New App"** → **"From scratch"**
2. Pojmenuj app (např. `Discord Summaries`) → vyber svůj workspace
3. Vlevo klikni **"Incoming Webhooks"** → přepni na **ON**
4. Klikni **"Add New Webhook to Workspace"** → vyber kanál nebo DM sobě
5. Zkopíruj webhook URL (začíná `https://hooks.slack.com/...`)

---

### Krok 5 — Import do n8n

1. Otevři n8n v prohlížeči
2. Jdi na stránku **Workflows**
3. Klikni na existující workflow nebo vytvoř nový
4. Stiskni **Cmd+V** (Mac) nebo **Ctrl+V** (Windows) a vlož obsah souboru `Discord_Summarizer.json`
5. Workflow se importuje automaticky

---

### Krok 6 — Nastav credentials v n8n

Workflow používá n8n Credentials pro bezpečné uložení tokenů.

**Discord Bot Token:**
1. Klikni na uzel **"Nacti seznam kanalu"**
2. V poli **Authentication** → **Generic Credential Type** → **Header Auth**
3. Klikni **"Set up credential"**
4. Vyplň: **Name** = `Authorization`, **Value** = `Bot TVŮJ_TOKEN` (s mezerou!)
5. Pojmenuj credential `Discord Bot Token` → **Save**
6. Stejný credential přiřaď i uzlům: **Nacti zpravy kanalu**, **Discord - Otevri DM**, **Discord - Posli DM**

**Anthropic API Key:**
1. Klikni na uzel **"Anthropic - Shrnuti"**
2. Stejně — Generic Credential Type → Header Auth
3. **Name** = `x-api-key`, **Value** = `sk-ant-...`
4. Pojmenuj `Anthropic API Key` → **Save**

**Slack Webhook URL:**
1. Klikni na uzel **"Slack - Odesli"**
2. Do pole **URL** vlož svoji webhook URL

---

### Krok 7 — Uprav konfiguraci

**Guild ID serveru:**
V uzlu **"Nacti seznam kanalu"** nahraď `YOUR_GUILD_ID` v URL svým Guild ID ze Kroku 2.

**Seznam příjemců Discord DM:**
V uzlu **"seznam prijemcu"** uprav pole `recipients`:
```javascript
const recipients = [
  'TVOJE_DISCORD_USER_ID',      // Ty
  // 'DALSI_USER_ID',           // Další příjemce (odkomentuj)
];
```

Jak zjistit Discord User ID:
- Discord → Nastavení → Pokročilé → zapni Vývojářský režim
- Pravý klik na své jméno v chatu → **"Kopírovat ID uživatele"**

**Přeskakované kanály:**
V uzlu **"Filtruj textove kanaly"** uprav pole `SKIP_NAMES` — přidej nebo odeber názvy kanálů které nechceš sledovat.

**Čas spuštění:**
V uzlu **"Schedule Trigger"** je nastaven cron `30 8,20 * * *` = každý den v 8:30 a 20:30. Uprav podle potřeby.

---

### Krok 8 — Otestuj a aktivuj

1. Klikni **"Execute workflow"** — workflow se spustí jednorázově
2. Zkontroluj jestli ti přišlo shrnutí na Discord a/nebo Slack
3. Pokud vše funguje, klikni **"Publish"** — workflow poběží automaticky

**Tip pro první test:** V uzlu "Filtruj textove kanaly" dočasně odkomentuj řádek `staticData.lastIds = {};` — tím načteš zprávy za posledních 12 hodin místo jen od posledního běhu. Po testu řádek znovu zakomentuj.

---

## ❓ Časté problémy

**"Unknown Channel" chyba**
→ Bot není na serveru. Pošli adminovi pozvánkový odkaz ze Kroku 1.

**Zprávy se načítají ale jsou prázdné**
→ Není zapnutý Message Content Intent. Jdi na Discord Developer Portal → Bot → zapni Message Content Intent.

**Workflow nic nenajde (prázdný výstup)**
→ Za posledních 12 hodin nebyly žádné zprávy, nebo static data jsou zastaralá. Dočasně odkomentuj reset v uzlu "Filtruj textove kanaly".

**Bot vidí kanály ale ne zprávy v nich**
→ Bot nemá oprávnění číst konkrétní kanál. Admin serveru musí boту dát přístup k daným kanálům.

---

## 💰 Náklady

| Služba | Cena |
|---|---|
| Discord API | Zdarma |
| Slack Webhooks | Zdarma |
| Anthropic Haiku (2× denně) | ~$0,10–0,20/měsíc |
| n8n self-hosted | Zdarma |

---

## 📄 Licence

MIT — použij, uprav, sdílej jak chceš.

---

*Vytvořeno s pomocí Claude AI. Máš dotaz nebo nápad na vylepšení? Otevři Issue nebo Pull Request.*
