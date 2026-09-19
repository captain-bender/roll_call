# ROS 2 Roll Call — Student Guide

**The goal:** get your name to appear on the lecturer's screen, published from a ROS 2 node that you wrote.

Everything runs on the lecturer's laptop. You connect to it over the network and type commands there. Nothing is installed on your own machine.

---

## Before you start

Write these down from the board:

| What | Value |
|---|---|
| WiFi network / password | `.....................` |
| Laptop address (IP) | `.....................` |
| Username | `student` |
| Password | `.....................` |

**Pick your name now.** Use lowercase letters and no spaces, for example `maria`, `john`, `wei_lin`. Everywhere below you see `maria`, type your own name instead. If two people use the same name, things get confusing, so pick something unique in the room.

---

## Step 1 — Connect to the laptop

You are going to open a terminal *on the lecturer's laptop*, from your own machine. This is called SSH (Secure Shell).

### Windows

Option A — PuTTY (no installation needed):

1. Open `putty.exe`.
2. In **Host Name**, type `student@<IP>` using the address from the board.
3. Leave **Port** as `22`, click **Open**.
4. A security warning appears the first time. Click **Accept**.
5. Type the password.

Option B — PowerShell (built into Windows):

```
ssh student@<IP>
```

### macOS / Linux

Open Terminal and type:

```
ssh student@<IP>
```

Type `yes` if it asks about authenticity, then type the password.

> **The password is invisible while you type it.** No dots, no stars, nothing moves. This is normal. Type it carefully and press Enter.

You are connected when the prompt changes to something like `student@bender-XPS:~$`.

> In PuTTY, **paste is right-click** (not Ctrl+V), and **copy is just selecting text with the mouse**. Ctrl+C does *not* copy — it stops whatever is running.

---

## Step 2 — Stage 1: your name on screen in one command

Type this, with your own name in the quotes:

```bash
ros2 topic pub /roll_call std_msgs/msg/String "data: 'Maria'" -r 1
```

Look up at the projector. Your name should appear, once per second.

Press **Ctrl+C** to stop it.

### What that command means

| Part | Meaning |
|---|---|
| `ros2` | The ROS 2 command-line tool. Everything in ROS 2 starts with this. |
| `topic pub` | "Publish onto a topic." A **topic** is a named channel that messages are sent to. |
| `/roll_call` | The name of the channel. The lecturer's program is listening to this exact name. |
| `std_msgs/msg/String` | The **type** of message. Everyone must agree on the type, or the message won't be understood. `String` is the simplest one: it carries text. |
| `"data: 'Maria'"` | The content. A `String` message has one field called `data`, and we put your name in it. |
| `-r 1` | Repeat at a **r**ate of 1 per second. |

> **If your name flashes up once and the command ends,** you forgot `-r 1`. Without it, ROS publishes a single message and exits.

**You have now used ROS.** A program you started sent a message to another program, on a channel, and that program received it — without either of them knowing anything about the other.

---

## Step 3 — Stage 2: write the same thing as a real node

Stage 1 used a ready-made tool. Now you build your own **package** containing your own **node**.

- A **node** is a single program that does one job. Your node's job is to publish your name.
- A **package** is the folder structure ROS 2 expects a node to live in.
- A **workspace** is a folder holding one or more packages, so they can be built together.

### 3.1 Create your workspace and package

```bash
mkdir -p ~/rollcall/ws_maria/src
cd ~/rollcall/ws_maria/src
ros2 pkg create --build-type ament_python --dependencies rclpy std_msgs --node-name hello_maria hello_maria
```

| Part | Meaning |
|---|---|
| `mkdir -p` | Make the folders. `-p` also creates any missing parent folders. |
| `~/rollcall/ws_maria` | Your own workspace. `~` means your home folder. **Use your own name** so you don't clash with anyone else. |
| `src` | Short for "source". Packages always live in a workspace's `src` folder. |
| `cd` | Change directory — move into that folder. |
| `ros2 pkg create` | Generate an empty package with the correct structure. |
| `--build-type ament_python` | This package is written in Python (rather than C++). |
| `--dependencies rclpy std_msgs` | It needs the ROS 2 Python library and the standard message types. |
| `--node-name hello_maria` | Also generate a node called this, and register it so `ros2 run` can find it later. |
| `hello_maria` (last one) | The package name. |

> Package and node names must be **lowercase, no spaces, no hyphens**. Underscores are fine.

### 3.2 Write your node

The command above generated a placeholder file. Empty it, then open it in the `nano` text editor:

```bash
echo -n > hello_maria/hello_maria/hello_maria.py
nano hello_maria/hello_maria/hello_maria.py
```

Type this in (change **both** `maria` and `Maria` to your own name):

```python
import rclpy
from std_msgs.msg import String

def main(args=None):
    rclpy.init(args=args)
    node = rclpy.create_node('hello_maria')
    pub = node.create_publisher(String, '/roll_call', 10)
    msg = String()
    msg.data = 'Maria'
    node.create_timer(1.0, lambda: pub.publish(msg))
    rclpy.spin(node)
```

Save and close: **Ctrl+O**, then **Enter**, then **Ctrl+X**.

> Python cares about indentation. Every line after `def main(...)` must be indented by the same four spaces.

### What each line does

| Line | Meaning |
|---|---|
| `import rclpy` | Load the ROS 2 library for Python. `rclpy` = **R**OS **C**lient **L**ibrary for **Py**thon. |
| `from std_msgs.msg import String` | Load the `String` message type — the same one you used in Stage 1. |
| `rclpy.init(...)` | Start up ROS 2 inside this program. |
| `rclpy.create_node('hello_maria')` | Create the node and give it a name. This name is how your program appears on the ROS network. |
| `create_publisher(String, '/roll_call', 10)` | Announce: "I will send `String` messages on `/roll_call`." The `10` is the queue size — how many messages to hold if the network is briefly busy. |
| `msg = String()` / `msg.data = 'Maria'` | Create an empty message and fill in its one field. |
| `create_timer(1.0, ...)` | Call that function every 1.0 seconds. This is what makes it repeat, like `-r 1` did before. |
| `rclpy.spin(node)` | Hand control to ROS and keep running, so the timer keeps firing. Without this, the program would end immediately. |

### 3.3 Build it

```bash
cd ~/rollcall/ws_maria
colcon build --symlink-install
```

| Part | Meaning |
|---|---|
| `colcon build` | The build tool. It reads every package in `src` and installs it into a runnable form. Always run this from the **top** of your workspace, not from `src`. |
| `--symlink-install` | Link to your Python file instead of copying it, so small edits work without rebuilding. |

You should see `1 package finished`.

### 3.4 Run it

```bash
source install/setup.bash
ros2 run hello_maria hello_maria
```

| Part | Meaning |
|---|---|
| `source install/setup.bash` | Tell this terminal where your new package is. **Without this, the next command fails.** You must do this once in every new terminal session. |
| `ros2 run <package> <node>` | Start the node. First name is the package, second is the node. |

Watch the projector. Your name is on screen — this time from code you wrote.

Press **Ctrl+C** to stop.

---

## If something goes wrong

| Message | What it means |
|---|---|
| `ros2: command not found` | The terminal doesn't know about ROS. Close it and reconnect. |
| `Package 'hello_maria' not found` | You forgot `source install/setup.bash`, or you're in the wrong folder. |
| `0 packages finished` | You ran `colcon build` from the wrong place. Go to `~/rollcall/ws_maria` and try again. |
| `No such file or directory` | A typo in a folder name, or you're not where you think you are. Type `pwd` to see where you are, `ls` to see what's there. |
| `WARNING: The path ... doesn't exist` | Harmless leftover from an earlier build. Ignore it. |
| Nothing appears on the projector | Check the topic name is exactly `/roll_call`, and that you used `-r 1` or a timer. |
| `IndentationError` | Python indentation. Every line inside `main` needs the same four leading spaces. |

Useful anywhere:

- `pwd` — print working directory (where am I?)
- `ls` — list what's in this folder
- `cd ..` — go up one folder
- **Ctrl+C** — stop the running program
- Up arrow — bring back the previous command instead of retyping it

---

## The point of all this

Your node never knew who was listening. The lecturer's listener never knew who was publishing, or how many of you there were. They agreed on one thing only: a topic called `/roll_call` carrying `String` messages.

That is the idea the whole of ROS is built on. Swap your publisher for a laser scanner, and the listener for a mapping program, and nothing else about the picture changes.

Everything you did here ran on one laptop over SSH. If ROS 2 were installed on your own machine, the exact same code would work across the WiFi with no SSH at all — the nodes find each other by themselves.
