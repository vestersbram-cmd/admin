# Tkinter App Style

Standards for Tkinter GUI apps.

## Structure
- `gui/` with `main_app.py`, `ui_utils.py`, `views/`
- Views: `{name}_view.py`
- Separate UI and logic
- Handlers in separate modules for complex apps

## Example Structure
```
gui/
├── main_app.py           # Entry point
├── ui_utils.py           # Shared UI utilities
├── handlers/           # Event handlers (for complex apps)
│   ├── __init__.py
│   └── misc_handlers.py
└── views/              # UI views (or panels/)
    ├── __init__.py
    └── example_view.py
```
- Filenames: snake_case, views as `{name}_view.py`
- No global state in views; use app_context for shared data

## Window Setup

### Basic Window
```python
class MainWindow(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("App Title")
        self.geometry("1200x800")
        self.minsize(800, 600)
        
        # Configure grid for resizing
        self.columnconfigure(0, weight=1)
        self.rowconfigure(0, weight=1)
        
        # Bind close event
        self.protocol("WM_DELETE_WINDOW", self.on_close)
        
        # Bind keyboard shortcuts
        self.bind("<F5>", lambda e: self.refresh())
        self.bind("<Control-q>", lambda e: self.quit())
        
    def on_close(self):
        self.quit()
        self.destroy()
```

## Layout Patterns

### PanedWindow (Split View)
```python
# Horizontal split
paned = ttk.PanedWindow(self, orient=tk.HORIZONTAL)
paned.pack(fill=tk.BOTH, expand=True)

left = ttk.Frame(paned)
paned.add(left, weight=1)

right = ttk.Frame(paned)
paned.add(right, weight=2)
```

### Grid Layout (2x2 panels)
```python
# _build_four_panels() pattern
self.root.columnconfigure(0, weight=1)
self.root.rowconfigure(1, weight=1)

# Creates a 2x2 grid of panels
```

## Status Bar

### Simple StatusBar (info left + error right)
```python
class StatusBar(ttk.Frame):
    def __init__(self, parent):
        super().__init__(parent, relief="sunken", padding=(8, 2))
        self._info_var = tk.StringVar(value="Ready")
        self._err_var = tk.StringVar()
        
        ttk.Label(self, textvariable=self._info_var).pack(side=tk.LEFT)
        ttk.Label(self, textvariable=self._err_var, foreground="red").pack(side=tk.RIGHT, padx=8)

    def info(self, msg):
        self._info_var.set(msg)
        self._err_var.set("")

    def error(self, msg):
        self._err_var.set(msg)
        # Also log to console
        print(f"[ERROR] {msg}", flush=True)

    def clear(self):
        self._info_var.set("Ready")
        self._err_var.set("")
```

### Alternative StatusBar (grid layout)
```python
class StatusBar(ttk.Label):
    def __init__(self, parent):
        super().__init__(parent, text="Ready", anchor="w", relief="sunken")
        self.pack(side=tk.BOTTOM, fill=tk.X)

    def set(self, text, error=False):
        self.config(text=text, foreground="red" if error else "black")
```

## Log Viewer / Scrolled Text

```python
class LogViewer(ttk.LabelFrame):
    def __init__(self, parent, height=6):
        super().__init__(parent, text="Log", padding=5)
        self.columnconfigure(0, weight=1)

        self._text = tk.Text(self, height=height, wrap=tk.WORD, state="disabled", font=("Consolas", 9))
        self._text.grid(row=0, column=0, sticky="ew")

        scrollbar = ttk.Scrollbar(self, orient="vertical", command=self._text.yview)
        scrollbar.grid(row=0, column=1, sticky="ns")
        self._text.configure(yscrollcommand=scrollbar.set)

    def log(self, message):
        self._text.configure(state="normal")
        self._text.insert(tk.END, f"{message}\n")
        self._text.see(tk.END)
        self._text.configure(state="disabled")
        print(f"[LOG] {message}", flush=True)
```

## Scrollable Content

### Canvas + Vertical Scrollbar
```python
canvas = tk.Canvas(frame)
scrollbar = ttk.Scrollbar(frame, orient="vertical", command=canvas.yview)
scroll_frame = ttk.Frame(canvas)

scroll_frame.bind("<Configure>", lambda e: canvas.configure(scrollregion=canvas.bbox("all")))
canvas.create_window((0, 0), window=scroll_frame, anchor="nw")
canvas.configure(yscrollcommand=scrollbar.set)

canvas.grid(row=0, column=0, sticky="nsew")
scrollbar.grid(row=0, column=1, sticky="ns")
```

### ScrollableFrame with Mousewheel
```python
class ScrollableFrame(ttk.Frame):
    def __init__(self, parent):
        super().__init__(parent)
        
        self.canvas = tk.Canvas(self, highlightthickness=0)
        self.vscrollbar = ttk.Scrollbar(self, orient="vertical", command=self.canvas.yview)
        self.canvas.configure(yscrollcommand=self.vscrollbar.set)
        
        self.scrollable_frame = ttk.Frame(self.canvas)
        self.scrollable_frame.bind("<Configure>", 
            lambda e: self.canvas.configure(scrollregion=self.canvas.bbox("all")))
        
        self.canvas.create_window((0, 0), window=self.scrollable_frame, anchor="nw")
        
        self.vscrollbar.pack(side="right", fill="y")
        self.canvas.pack(side="left", fill="both", expand=True)
        
        # Bind mousewheel
        self.bind_mousewheel()

    def bind_mousewheel(self):
        def on_wheel(event):
            if event.delta:
                self.canvas.yview_scroll(int(-1 * (event.delta / 120)), "units")
        
        self.canvas.bind("<MouseWheel>", on_wheel)
        self.scrollable_frame.bind("<MouseWheel>", on_wheel)
```

## Tree View (ttk.Treeview)

### Treeview with Scrollbars
```python
def create_treeview(parent, **options) -> ttk.Treeview:
    frame = ttk.Frame(parent)
    frame.pack(fill=tk.BOTH, expand=True, padx=5, pady=5)
    
    tree = ttk.Treeview(frame, **options)
    vsb = ttk.Scrollbar(frame, orient=tk.VERTICAL, command=tree.yview)
    tree.configure(yscrollcommand=vsb.set)
    hsb = ttk.Scrollbar(frame, orient=tk.HORIZONTAL, command=tree.xview)
    tree.configure(xscrollcommand=hsb.set)
    
    tree.grid(row=0, column=0, sticky="nsew")
    vsb.grid(row=0, column=1, sticky="ns")
    hsb.grid(row=1, column=0, sticky="ew")
    
    frame.grid_rowconfigure(0, weight=1)
    frame.grid_columnconfigure(0, weight=1)
    
    return tree
```

### Treeview with Columns
```python
columns = ("name", "size", "date")
tree = ttk.Treeview(frame, columns=columns, show="headings")

tree.heading("name", text="Name", command=lambda: sort("name"))
tree.heading("size", text="Size")
tree.heading("date", text="Date")

tree.column("name", width=300, anchor=tk.W)
tree.column("size", width=100, anchor=tk.E)
tree.column("date", width=150)

# Tag colors for rows
tree.tag_configure("up_to_date", foreground="green")
tree.tag_configure("out_of_date", foreground="red")
```

## Tabs (ttk.Notebook)

```python
notebook = ttk.Notebook(root)
notebook.grid(row=0, column=0, sticky="nsew", padx=10, pady=10)

tab1 = ttk.Frame(notebook)
notebook.add(tab1, text="Tab 1")

tab2 = ttk.Frame(notebook)
notebook.add(tab2, text="Tab 2")
```

## Right-Click Context Menu

```python
def show_context_menu(self, event):
    menu = tk.Menu(self, tearoff=0)
    menu.add_command(label="Open", command=self.open_item)
    menu.add_command(label="Edit", command=self.edit_item)
    menu.add_command(label="Delete", command=self.delete_item)
    menu.add_separator()
    menu.add_command(label="Cancel")
    menu.tk_popup(event.x_root, event.y_root)

# Bind to tree/widget
self.tree.bind("<Button-3>", self.show_context_menu)
```

## Double-Click Action

```python
def on_double_click(self, event):
    selection = self.tree.selection()
    if selection:
        item_id = selection[0]
        self.open_item(item_id)

self.tree.bind("<Double-1>", self.on_double_click)
```

## Keyboard Shortcuts

### Bind in _bind_shortcuts or __init__
```python
def _bind_shortcuts(self):
    self.bind("<Control-q>", lambda e: self.quit())
    self.bind("<Control-n>", lambda e: self.new_item())
    self.bind("<Control-s>", lambda e: self.save())
    self.bind("<Control-f>", lambda e: self.search())
    self.bind("<Control-c>", lambda e: self.copy())
    self.bind("<Control-v>", lambda e: self.paste())
    self.bind("<Escape>", lambda e: self.cancel())
    self.bind("<F5>", lambda e: self.refresh())
```

### Common Keyboard Bindings

| Action | Binding |
|:-------|:--------|
| Quit | `<Control-q>` |
| New | `<Control-n>` |
| Save | `<Control-s>` |
| Find | `<Control-f>` |
| Copy | `<Control-c>` |
| Paste | `<Control-v>` |
| Refresh | `<F5>` |
| Close/Cancel | `<Escape>` |

## Tooltips

```python
def set_tooltip(widget, text):
    """Set tooltip on widget hover."""
    tooltip = tk.Toplevel()
    tooltip.withdraw()
    tooltip.overraise = True
    
    label = ttk.Label(tooltip, text=text, background="#ffffe0", 
                     relief=tk.SOLID, borderwidth=1)
    label.pack()
    
    def show(event):
        x, y, _, _ = widget.bbox("insert")
        x += widget.winfo_rootx()
        y += widget.winfo_rooty()
        tooltip.winfo_geometry(f"+{x}+{y+y}")
        tooltip.deiconify()
    
    def hide():
        tooltip.withdraw()
    
    widget.bind("<Enter>", show)
    widget.bind("<Leave>", hide)
```

## Dialogs

### Simple Dialog
```python
class ArgsDialog:
    def __init__(self, parent, title, params, on_execute=None):
        self.dialog = tk.Toplevel(parent)
        self.dialog.title(title)
        self.dialog.transient(parent)
        self.dialog.grab_set()
        
        # Center on parent
        self.dialog.update_idletasks()
        x = parent.winfo_x() + (parent.winfo_width() - self.dialog.winfo_width()) // 2
        y = parent.winfo_y() + (parent.winfo_height() - self.dialog.winfo_height()) // 2
        self.dialog.geometry(f"+{x}+{y}")
        
        # Build form
        self._entries = {}
        for i, (name, default) in enumerate(params):
            ttk.Label(self.dialog, text=f"{name}:").grid(row=i, column=0, padx=10, pady=5, sticky="e")
            var = tk.StringVar(value=str(default) if default is not None else "")
            self._entries[name] = var
            ttk.Entry(self.dialog, textvariable=var, width=30).grid(row=i, column=1, padx=10, pady=5, sticky="w")
        
        # Buttons
        btn_frame = ttk.Frame(self.dialog)
        btn_frame.grid(row=len(params), column=0, columnspan=2, pady=20)
        
        ttk.Button(btn_frame, text="Execute", command=self._execute).pack(side=tk.LEFT, padx=5)
        ttk.Button(btn_frame, text="Cancel", command=self.dialog.destroy).pack(side=tk.LEFT, padx=5)

    def _execute(self):
        if self._on_execute:
            args = {name: var.get() for name, var in self._entries.items()}
            self._on_execute(args)
        self.dialog.destroy()
```

## Async/Threads

### Thread + Queue Pattern
```python
import threading, queue

def start_task(self):
    q = queue.Queue()
    
    def worker():
        # Do work
        result = do_work()
        q.put(result)
    
    threading.Thread(target=worker, daemon=True).start()
    self.poll_queue(q)

def poll_queue(self, q):
    try:
        result = q.get_nowait()
        self.handle_result(result)
    except queue.Empty:
        self.after(100, self.poll_queue, q)
```

### Run Async in Thread
```python
def run_async(self, coro, callback=None):
    """Schedule coroutine on background loop."""
    def done(fut):
        try:
            fut.result()
            if callback:
                self.after(0, callback, True, None)
        except Exception as e:
            if callback:
                self.after(0, callback, False, str(e))
    
    asyncio.run_coroutine_threadsafe(coro, self.async_loop).add_done_callback(done)
```

## Server-Based GUI

If the GUI depends on a backend API server, the `main()` function should automatically check and start the server if not running.

### Server Auto-Start Pattern

```python
import socket
import subprocess
import sys
import time

API_BASE = "http://localhost:8000"


def _is_server_running(host: str = "localhost", port: int = 8000) -> bool:
    """Check if the API server is already running"""
    try:
        with socket.create_connection((host, port), timeout=1):
            return True
    except OSError:
        return False


def _start_server() -> subprocess.Popen | None:
    """Start the API server in a background process"""
    try:
        logger.info("Starting API server...")
        # Use DEVNULL to avoid blocking on pipe buffers
        process = subprocess.Popen(
            [sys.executable, "-m", "llm_panel.cli"],
            stdout=subprocess.DEVNULL,
            stderr=subprocess.DEVNULL,
            start_new_session=True,
        )
        # Wait for server to be ready (up to 30 seconds with retry loop)
        for i in range(60):
            time.sleep(0.5)
            if _is_server_running():
                logger.info("Server is ready after %.1fs", (i + 1) * 0.5)
                return process
        logger.warning("Server startup timeout after 30s")
        return process
    except Exception as e:
        logger.error("Failed to start server: %s", e)
        return None


def main():
    """Entry point for GUI"""
    server_process: subprocess.Popen | None = None

    # Check if server is running, start if not
    if not _is_server_running():
        logger.info("Server not running, starting it...")
        server_process = _start_server()
        if not server_process:
            logger.error("Could not start server, continuing anyway...")
        elif not _is_server_running():
            logger.warning("Server may still be starting, continuing...")

    try:
        app = MainWindow()
        app.mainloop()
    finally:
        # Clean up server process if we started it
        if server_process:
            logger.info("Shutting down server...")
            server_process.terminate()
            try:
                server_process.wait(timeout=5)
            except subprocess.TimeoutExpired:
                server_process.kill()


if __name__ == "__main__":
    main()
```

### Key Points

- **Check first**: Use `socket.create_connection()` to test if server is already running
- **Auto-start**: Launch server via `subprocess.Popen()` with the same Python interpreter
- **Use DEVNULL**: Set `stdout=subprocess.DEVNULL, stderr=subprocess.DEVNULL` to avoid pipe buffer blocking
- **Retry loop**: Poll until server is ready (30s max) - use `_is_server_running()` in a loop
- **Cleanup in `finally`**: Always terminate the server on GUI exit if you started it
- **Graceful fallback**: Continue anyway if server fails to start (user may start it manually)

## View Pattern

```python
# views/example_view.py
import tkinter as tk
from tkinter import ttk

def create_view(parent, ctx):
    view = ttk.Frame(parent)
    ttk.Label(view, text="Example").pack(padx=20, pady=20)
    return view
```
- One view per file
- Pass ctx (app context) for shared data

## Additional UI Components

### StatusIndicator (Running/Stopped dot)
```python
class StatusIndicator(ttk.Frame):
    def __init__(self, parent, running=False, text=None):
        super().__init__(parent)
        
        self.dot = tk.Canvas(self, width=10, height=10, highlightthickness=0)
        self.dot.pack(side="left")
        
        color = "#22C55E" if running else "#EF4444"
        self.dot.create_oval(2, 2, 8, 8, fill=color, outline="")
        
        display = text or ("Running" if running else "Stopped")
        ttk.Label(self, text=display).pack(side="left", padx=(4, 0))
```

### Card (LabelFrame)
```python
class Card(ttk.LabelFrame):
    def __init__(self, parent, title):
        super().__init__(parent, text=title)
        self.configure(padding=10)
```

### StatCard
```python
def create_stat_card(parent, label, value, color=None):
    card = ttk.Frame(parent, relief="solid", borderwidth=1, padding=15)
    
    ttk.Label(card, text=label, font=("TkDefaultFont", 9), foreground="gray").pack(anchor="w")
    
    val_color = color or "black"
    ttk.Label(card, text=value, font=("TkDefaultFont", 18, "bold"), 
             foreground=val_color).pack(anchor="w", pady=(5, 0))
    
    return card
```

## Anti-patterns

- Never use `time.sleep()` in main thread
- Never update widgets directly from worker threads
- Don't hardcode window sizes - use geometry + minsize
- Don't forget to bind keyboard events to specific widgets (bind to root for global)