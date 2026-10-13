# Raspberry Pi 5 Local AI Setup Guide (4GB RAM & OS Lite)

A complete guide to running a local Large Language Model (LLM) on a **Raspberry Pi 5 (4GB RAM)** using **Raspberry Pi OS Lite (64-bit)** with an interactive Terminal User Interface (TUI).

---

## 🛠️ Hardware Requirements
* **Raspberry Pi 5 (4GB RAM)**
* **Active Cooler / Fan** (*Essential: LLM inference runs CPU at 100%*)
* **32GB+ MicroSD Card** (or NVMe SSD)
* **MicroSD Card Reader** (for initial setup)

---

## Step 1: Flash Raspberry Pi OS Lite

1. Download and install **Raspberry Pi Imager** on your PC/Mac from [raspberrypi.com/software](https://www.raspberrypi.com/software/).
2. Insert your MicroSD card and open Raspberry Pi Imager.
3. Configure target device and OS:
   * **Device:** `Raspberry Pi 5`
   * **OS:** `Raspberry Pi OS (other)` $\rightarrow$ `Raspberry Pi OS Lite (64-bit)`
4. Click **NEXT** and select **EDIT SETTINGS**:
   * **Hostname:** Set a hostname (e.g., `raspberrypi`).
   * **User Credentials:** Set your preferred username and password.
   * **Wi-Fi:** Fill in your Wi-Fi SSID, password, and country code.
   * **Services Tab:** Check **Enable SSH** (use password authentication).
5. Click **SAVE**, confirm write operation, and wait for process completion.

---

## Step 2: First Boot & Network Connection

1. Insert MicroSD card into the Pi 5 and power it on.
2. Wait ~60 seconds for initial initialization.
3. Open terminal/command prompt on your primary computer and log in via SSH:
   ```bash
   ssh username@raspberrypi.local
   ```
4. Update system packages:
   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

---

## Step 3: Install Ollama & Pull Light Model

1. Install Ollama via official installation script:
   ```bash
   curl -fsSL https://ollama.com/install.sh | sh
   ```
2. Verify Ollama service status:
   ```bash
   sudo systemctl status ollama
   ```
3. Download a lightweight quantized model optimized for 4GB RAM:
   ```bash
   # Meta's Llama 3.2 1B (Fastest & low memory overhead)
   ollama pull llama3.2:1b
   ```
   *(Alternative models: `deepseek-r1:1.5b` or `qwen2.5-coder:1.5b`)*

---

## Step 4: Install Terminal Interface Dependencies

Install Python 3 package manager along with the `textual` framework and `requests` library:

```bash
sudo apt update && sudo apt install -y python3-pip python3-textual python3-requests
```

---

## Step 5: Create the Terminal Chat Interface (TUI)

Create a Python script named `pi_chat.py`:

```bash
nano pi_chat.py
```

Paste the following code into `pi_chat.py`:

```python
import requests
from textual.app import App, ComposeResult
from textual.containers import VerticalScroll
from textual.widgets import Header, Footer, Input, Static

OLLAMA_URL = "http://localhost:11434/api/generate"
MODEL_NAME = "llama3.2:1b"

class ChatApp(App):
    CSS = """
    VerticalScroll { height: 1fr; border: solid green; padding: 1; }
    Input { dock: bottom; border: solid border; }
    .user { color: cyan; margin-bottom: 1; }
    .bot { color: green; margin-bottom: 1; }
    """

    def compose(self) -> ComposeResult:
        yield Header()
        with VerticalScroll(id="chat_history"):
            yield Static("[bold green]Pi 5 Local AI Terminal Interface[/bold green]\nType a message and press Enter.\n", classes="bot")
        yield Input(placeholder="Type your message here...", id="user_input")
        yield Footer()

    async def on_input_submitted(self, event: Input.Submitted) -> None:
        user_text = event.value.strip()
        if not user_text:
            return

        input_widget = self.query_one("#user_input", Input)
        chat_box = self.query_one("#chat_history", VerticalScroll)

        input_widget.value = ""
        chat_box.mount(Static(f"[bold cyan]You:[/bold cyan] {user_text}", classes="user"))
        chat_box.scroll_end(animate=False)

        # Query Ollama local API
        try:
            response = requests.post(
                OLLAMA_URL,
                json={"model": MODEL_NAME, "prompt": user_text, "stream": False},
                timeout=30
            )
            if response.status_code == 200:
                reply = response.json().get("response", "No response received.")
            else:
                reply = f"Error: Received HTTP {response.status_code}"
        except Exception as e:
            reply = f"Connection failed: {str(e)}"

        chat_box.mount(Static(f"[bold green]AI:[/bold green] {reply}", classes="bot"))
        chat_box.scroll_end(animate=False)

if __name__ == "__main__":
    app = ChatApp()
    app.run()
```

Save and exit: Press `Ctrl+O`, `Enter`, then `Ctrl+X`.

---

## Step 6: Test & Optional Auto-Launch Setup

1. **Test the interface manually:**
   ```bash
   python3 pi_chat.py
   ```
   *(Press `Ctrl+C` to exit the application)*

2. **(Optional) Launch automatically on SSH login:**
   Add execution command to `.bashrc`:
   ```bash
   echo 'python3 ~/pi_chat.py' >> ~/.bashrc
   ```

---

## 💡 Performance Optimization Tips

1. **Avoid Heavy Swapping:** Do not run multiple large tasks simultaneously; 4GB RAM is tightly allocated when loaded with model weights.
2. **Temperature Management:** Check temperature periodically using:
   ```bash
   vcgencmd measure_temp
   ```
   Ensure temperature stays under $80^\circ\text{C}$ to prevent thermal throttling.