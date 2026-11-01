# Practice 4. Network traffic analysis

[Русская версия](README.md)

**70 minutes, Wireshark.** We walk through a recording of a real computer infection (the Trickbot trojan, 2017): we find a program disguised
as a picture, a check of the external address, non-standard ports and a beacon. The second recording contains a DNS tunnel. The teacher shows
each step on the projector, you repeat it on your own machine. No programming is needed.

## 0. Start Wireshark

Open a Codespace from this repository (**Code → Codespaces → Create codespace on main**) or clone it into a Codespace you already have open:

```bash
git clone https://github.com/ObeWan99/P4-traffic-analysis.git
cd P4-traffic-analysis
```

Start Wireshark — it runs in a container and opens in the browser:

```bash
docker compose up -d
```

The first start downloads the image (about 2 GB) — 2–3 minutes. Then the **Ports** tab → port **3000** → **Open in Browser**.
The new browser tab shows the Wireshark window.

Open a recording: **File → Open**, type `/pcap/c2-beacon.pcap` in the file name field → **Open**. The second recording is `/pcap/dns-tunnel.pcap`.

> `c2-beacon.pcap` contains real Windows malware. **You may look, but do not save anything from the traffic:**
> leave **File → Export Objects** alone.

## Filters used in the lesson

Type a filter into the bar above the packet list and press **Enter**. A red bar means a typo.

| Step | What we look at | Filter or menu |
|---|---|---|
| 1 | Who talked to whom | **Statistics → Conversations**, **IPv4** tab |
| 2 | What the computer downloaded | `http.request` |
| 3 | What the "picture" is | right-click the row → **Follow → TCP Stream** |
| 4 | What the program did first | `dns.flags.response == 0` · `http.request.uri == "/raw"` |
| 5 | Where it connects | `tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| 5 | Connection rhythm | **View → Time Display Format → Seconds Since Previous Displayed Packet**, then `ip.dst == 67.21.90.105 && tcp.flags.syn == 1 && tcp.flags.ack == 0` |
| 5 | One more server | `ip.dst == 72.211.215.68 && tcp.flags.syn == 1` |
| 6 | What the traffic consists of | file `/pcap/dns-tunnel.pcap`, **Statistics → Protocol Hierarchy** |
| 6 | Queries to one domain | `dns.flags.response == 0 && dns.qry.name contains "onetwogleb"` |
| 6 | All other queries | `dns.flags.response == 0 && !(dns.qry.name contains "onetwogleb") && !mdns` |
| 6 | Query rhythm | **Statistics → I/O Graphs** |

After **Follow → TCP Stream** the bar keeps `tcp.stream eq …` — clear it and press Enter.

## Lesson result

A table "observation → hypothesis → how we checked → confirmed?" for all findings. We fill it in together at the end of the lesson.

## Files

| File | Contents |
|---|---|
| `pcap/c2-beacon.pcap` | recording of the infected computer, 21 minutes |
| `pcap/dns-tunnel.pcap` | recording of the second computer, 27 minutes |
| `pcap/SOURCES.md` | where the recordings come from and their checksums |
| `compose.yml` | Wireshark in the browser |

Stop Wireshark: `docker compose down`.
