---
status: working
tags:
  - tooling
  - setup
  - environment
  - python
---
> *Environment Setup:* The one-time configuration that trips people up mid-engagement if not done ahead. Python isolation, BloodHound CE, and proxychains. Related to the thechunk workstation build, but scoped to the attack-tooling config rather than the OS.

## Python venv & pipx (PEP 668 workaround)

Kali enforces PEP 668, so a bare `pip install` fails. Use a venv for project/PoC dependencies and pipx for standalone CLI tools.

```bash
sudo apt install -y python3-venv python3-pip pipx
mkdir -p ~/tools/proj && cd ~/tools/proj && python3 -m venv .venv
source .venv/bin/activate          # modifies the current shell
pip install -r requirements.txt    # a PoC's deps
deactivate
# standalone CLI tools, each in its own hidden venv, entry point on PATH:
pipx install impacket
pipx install netexec
pipx install bloodyAD
```

## BloodHound CE (Docker)

Set it up once before you need it live.

```bash
curl -L https://ghst.ly/getbhce -o docker-compose.yml
docker compose up -d
docker compose logs | grep -i "Initial Password"     # grab the generated admin password
```

> [!note]
> Browse `http://localhost:8080`, log in as `admin` with that password. Collect with `bloodhound-python` ([[03D - AD Enumeration]]) and drag the `.zip` onto the UI. Legacy neo4j build: `sudo neo4j start`, browse `http://localhost:7474` to set the password, launch `bloodhound`, connect to `bolt://localhost:7687`.

## ProxyChains & Reusable Listeners

```bash
sudo sed -i 's/^socks4.*/socks5 127.0.0.1 1080/' /etc/proxychains4.conf
```

> [!warning]
> The port must match your tunnel, chisel and `ssh -D` use 1080 here, some setups default to 9050. Verify it or every proxied command silently fails. Uncomment `dynamic_chain` in that file so a dead proxy does not kill the whole chain. Keep a `/opt/tools` dir pre-stocked with transfer-ready binaries (linpeas, winpeas, the Potatoes, SharpHound, Rubeus, chisel, ligolo, mimikatz, PowerView, nc.exe) and serve them with the HTTP one-liner ([[11AA - High-Frequency One-Liners]]).

## See Also

- [[03G - Pivoting & Tunneling]] for what the proxychains config is for
- [[11AAAAA - Tool Map]] and [[00B - References]] for what to install
- Source: [[OSCP Field Manual]] sections 10.2 to 10.4
