# xrp — Sovereign XRP Terminal Wallet

Part of **AIOS**: bare-metal, local-first operating system primitives for the sovereign Linux user.

No phone app. No exchange account. No cloud. Your keys live encrypted on your machine. The tool talks directly to the XRP Ledger over WebSocket.

---

## Install

```bash
git clone https://github.com/yourusername/xrp
cd xrp
bash install.sh
```

**Dependencies:** Python 3.10+, pip

---

## Usage

```bash
xrp init                                  # Generate new wallet (shown once: seed phrase)
xrp balance                               # Show balance
xrp address                               # Print your address
xrp receive                               # Show receive QR
xrp receive --amount 5.0 --label "1hr"   # Request specific amount
xrp send rXXX...XXX 1.5                  # Send 1.5 XRP
xrp send rXXX...XXX 1.5 --tag 4821       # Send with destination tag
xrp history                               # Recent transactions
xrp watch                                 # Watch for incoming payments
xrp watch --amount 1.5 --on-payment "./unlock.sh"   # Trigger script on payment
xrp export                                # Show seed phrase (careful)
xrp node https://xrplcluster.com          # Set custom XRPL node
```

---

## Watch Mode — IoT / Access Control

`xrp watch` subscribes to your address on the XRP Ledger in real time. When a qualifying payment arrives, it fires a shell command with full payment context in environment variables:

```bash
xrp watch \
  --amount 1.5 \
  --tag 4821 \
  --on-payment "gpio_set.sh 17 1"
```

**Environment variables passed to your command:**

| Variable | Value |
|---|---|
| `XRP_AMOUNT` | Amount received in XRP |
| `XRP_SENDER` | Sender's XRP address |
| `XRP_TAG` | Destination tag |
| `XRP_HASH` | Transaction hash |
| `XRP_DESTINATION` | Your address |

**Examples:**

```bash
# Unlock a GPIO relay (Raspberry Pi / any SBC)
xrp watch --amount 1.0 --on-payment "python3 relay.py on 30"

# Start a service
xrp watch --on-payment "systemctl start my-service"

# Log to file
xrp watch --on-payment 'echo "$XRP_AMOUNT XRP from $XRP_SENDER" >> payments.log'

# Run once and exit (vending / single-use)
xrp watch --amount 2.0 --on-payment "./dispense.sh" --once
```

**Run as a systemd service:**

```ini
[Unit]
Description=XRP Payment Watcher
After=network.target

[Service]
ExecStart=/usr/local/bin/xrp watch --amount 1.5 --on-payment "/home/pi/unlock.sh"
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```

---

## Security

- Private key is encrypted with AES-256-GCM + PBKDF2 (600k iterations)
- Key is derived from your password — never stored in plaintext
- `xrp receive`, `xrp watch`, `xrp balance`, `xrp history` never ask for your password — they only need your public address
- Only `xrp send` and `xrp export` decrypt the key

---

## Key Storage

```
~/.config/xrp/
├── wallet.enc      # AES-256-GCM encrypted seed
├── config.json     # node URL, public address
└── history.db      # SQLite local tx log
```

---

## Part of AIOS

| Tool | Role |
|---|---|
| `ai` | Local LLM assistant (DeepSeek R1 via Ollama) |
| `scrape` | Playwright-based recursive web scraper with AI |
| `xrp` | Sovereign XRP payment layer |

---

## XRPL Node

Defaults to `https://xrplcluster.com` (community-run, reliable). You can point it at any public or self-hosted rippled node:

```bash
xrp node https://s1.ripple.com
```

---

## License

MIT
