# Industrial Cybersecurity — Book Materials

Lab configurations, scripts, and captured artifacts that accompany
**_Industrial Cybersecurity_** by [Pascal Ackerman](https://github.com/SackOfHacks), published by Packt.

These are the working files from the ICS/OT lab used throughout the books — the Modbus
device simulator, the Security Onion ingest pipelines, the Wazuh agent configuration, and the
scan and system-inventory output the exercises analyze.

---

## 📚 The Books

### Industrial Cybersecurity — Second Edition (2021)

> *Efficiently monitor the cybersecurity posture of your ICS environment*

The second edition is a ground-up expansion of the original, shifting the focus from building a
secure ICS architecture to **actively monitoring, assessing, and defending** one. It covers ICS
network monitoring and visibility, security assessments and penetration testing of industrial
environments, threat hunting in OT networks, incident response, and digital forensics — built
around a hands-on lab you construct as you read.

| | |
|---|---|
| **Publisher** | Packt Publishing |
| **Published** | 2021 |
| **Pages** | 800 |
| **ISBN-13** | 978-1-80020-209-2 |
| **Get it** | [Packt](https://www.packtpub.com/en-us/product/industrial-cybersecurity-9781800202092) · [Amazon](https://www.amazon.com/dp/1800202091) |

### Industrial Cybersecurity — First Edition (2017)

> *Efficiently secure critical infrastructure systems*

The original edition is the foundational text: ICS and Purdue-model fundamentals, industrial
protocols, risk assessment and threat modeling, and the design of defense-in-depth architectures
for critical infrastructure. It remains the best starting point if you are new to OT security, and
complements rather than duplicates the second edition.

| | |
|---|---|
| **Publisher** | Packt Publishing |
| **Published** | October 2017 |
| **Pages** | 456 |
| **ISBN-13** | 978-1-78839-515-1 |
| **Get it** | [Packt](https://www.packtpub.com/en-us/product/industrial-cybersecurity-9781788395151) · [Amazon](https://www.amazon.com/dp/1788395158) · [O'Reilly](https://www.oreilly.com/library/view/industrial-cybersecurity/9781788395151/) |

**New to the series?** Read the first edition to learn how to *build* a defensible ICS
environment, then the second to learn how to *watch, test, and defend* it.

---

## 📁 What's in this repository

| Directory | Contents |
|---|---|
| `lab-setup/modbus-server/` | A [pymodbus](https://github.com/pymodbus-dev/pymodbus)-based Modbus TCP server that simulates an industrial device on the lab network. |
| `lab-setup/malware/` | Malware sample used by the detection and incident-response exercises. |
| `security-onion-logstash/` | Elasticsearch ingest pipelines for [Security Onion](https://securityonion.net/): a `syslog` router and a `silentdefense` pipeline that parses CEF alerts from the SilentDefense OT IDS. |
| `elasticsearch-tweaks/` | Index template for the `so-syslog-*` indices. |
| `wazuh-config/` | The [Wazuh](https://wazuh.com/) Windows agent `ossec.conf` deployed to the lab endpoints — Sysmon, PowerShell and Security eventchannel collection, FIM, and registry monitoring. |
| `lab-artifacts/` | Data collected *from* the lab: `msinfo32` system reports per host, and Nmap host-discovery and service-detection scans of the ICS subnet. |

---

## 🏭 The lab environment

The exercises run against a simulated industrial network on **`172.25.100.0/24`**, combining OT
devices, a Windows domain, and the monitoring stack:

| Host | Role |
|---|---|
| `172.25.100.11–.12` | Rockwell Automation PLCs — EtherNet/IP (44818) |
| `172.25.100.20` | Siemens S7 controller — ISO-TSAP (102) |
| `172.25.100.21` | Modbus TCP device (502) — simulated by `modbus-server.py` |
| `172.25.100.23` | EtherNet/IP device (44818) |
| `172.25.100.24` | OPC UA server (4840) |
| `172.25.100.100` | `OT-DC1` — Active Directory domain controller for `OT-Domain.local` |
| `172.25.100.105/.110` | `FT-DIR1` / `FT-DIR2` — FactoryTalk Directory servers |
| `172.25.100.201–.212` | Windows engineering workstations |
| `172.25.100.250` | Wazuh manager |
| `HMI-1`, `HMI-2` | Windows XP human-machine interfaces |

---

## 🚀 Using the materials

### Modbus device simulator

The simulator exposes discrete inputs, coils, holding registers and input registers so the lab's
scanning, enumeration and traffic-analysis exercises have a real Modbus endpoint to work against.

```bash
cd lab-setup/modbus-server
pip install -r requirements.txt
python modbus-server.py
```

> **Dependency note.** This script targets the **pymodbus 2.x** API (`pymodbus.server.sync`), which
> was removed in pymodbus 3.0. `requirements.txt` pins a compatible 2.x release — install it into a
> virtualenv rather than upgrading to the latest pymodbus, or the imports will fail.

The listener binds to `172.25.100.21:502` to match the lab topology. If you are running outside
that network, change the `address=` argument in `run_server()` to an address your host owns
(`0.0.0.0` binds all interfaces). Port 502 is privileged, so on Linux run with `sudo` or grant the
capability with `setcap`.

### Security Onion ingest pipelines

Install the pipelines on the Security Onion manager, then apply the index template:

```bash
# copy the pipeline definitions into place, then load them
curl -XPUT "localhost:9200/_ingest/pipeline/syslog"        -H 'Content-Type: application/json' -d @security-onion-logstash/syslog
curl -XPUT "localhost:9200/_ingest/pipeline/silentdefense" -H 'Content-Type: application/json' -d @security-onion-logstash/silentdefense
```

Then run the `PUT _template/so-syslog` request in `elasticsearch-tweaks/configure-templates.txt`
from Kibana's Dev Tools console.

The `syslog` pipeline is the entry point: it parses incoming syslog and CEF messages, tags the
originating vendor and product, and routes events on to the matching downstream pipeline. The
`silentdefense` pipeline dissects SilentDefense CEF alert and network logs into ECS-style fields
(`source.ip`, `destination.ip`, `network.community_id`, and so on).

### Wazuh agent

Deploy `wazuh-config/ossec.conf` to `C:\Program Files (x86)\ossec-agent\ossec.conf` on the Windows
lab endpoints and restart the agent. Update the `<address>` element to point at your own Wazuh
manager.

---

## ⚠️ A note on the lab

Everything here is built for an **isolated, disposable lab**. The configurations deliberately favor
visibility over hardening, and the exercises involve real attack tooling and malware. Never deploy
these files to a production control system, and keep the lab network segmented from anything you
care about.

---

## 👤 About the author

**Pascal Ackerman** is a seasoned industrial security professional with a degree in electrical
engineering and over two decades of experience in industrial network design and support,
information and network security, risk assessment, penetration testing, threat hunting, and
forensics.

- GitHub: [@SackOfHacks](https://github.com/SackOfHacks)

## 📄 License

Released under the GNU General Public License v3.0 — see [LICENSE](LICENSE).
