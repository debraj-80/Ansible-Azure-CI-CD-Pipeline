# 📓 Learnings & Challenges

A honest reflection on what I learned, what broke, and what I'd do differently.

---

## ✅ What Went Well

- **Ansible's agentless approach** made it surprisingly easy to connect to Azure VMs — just SSH and Python, nothing else needed on the managed nodes
- **Inventory management** was straightforward once SSH key-based auth was set up correctly between the control node and both managed nodes
- **Maven + Tomcat** worked smoothly together — the `.war` file dropped straight into Tomcat's webapps directory and the app came up without manual config
- Seeing the **same app go live on two different nodes** from a single playbook run was the moment it all clicked

---

## 🔥 Challenges I Hit

### 1. SSH Key Permissions
The first time I ran the playbook, Ansible couldn't connect because the private key file had permissions that were too open (`chmod 644`). SSH requires `chmod 600` on private keys. Simple fix, but took a while to debug.

### 2. Tomcat Not Starting After Deploy
The `.war` file deployed fine but the app wasn't accessible. Turned out Tomcat needed a **restart** after deployment — added a restart task at the end of the playbook.

### 3. Azure NSG (Network Security Group) Rules
The app was running but not reachable from the browser. The Azure Network Security Group was blocking port `8080`. Had to add an inbound rule in the Azure portal to allow traffic on that port.

### 4. Maven Build Time
The first Maven build took much longer than expected because it was downloading all dependencies from scratch. On subsequent runs it was faster due to the local Maven cache.

---

## 💡 Key Takeaways

- **Idempotency matters** — Ansible lets you re-run the same playbook safely. This saved me multiple times when I had to fix a mid-run failure and restart.
- **SSH key hygiene is critical** — Managing keys properly between control and managed nodes is foundational. Get this wrong and nothing works.
- **Cloud networking adds a layer** — It's not just about the app running; ports, firewalls, and NSG rules are part of the deployment story.
- **Automation reveals assumptions** — Writing the playbook forced me to think through every manual step I had been doing without thinking.

---

## 🔭 What I'd Do Differently

- Set up **Ansible Vault** from the start to manage any sensitive variables securely
- Use **Ansible Roles** to keep the playbook modular and reusable across projects
- Add a **smoke test task** at the end of the playbook to verify the app is actually responding before declaring success
- Use **Azure Managed Identities** instead of SSH keys for a more production-appropriate auth approach

---

> These learnings came from actually breaking things and fixing them — which is the best way to learn DevOps.
