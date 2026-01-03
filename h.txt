import tkinter as tk
from tkinter import messagebox
from tkinter import ttk



TEAM_COUNT = 9
DEFAULT_MAX_STONES = 10
UPGRADED_MAX_STONES = 15


class TeamStoneApp:
    def upgrade_storage(self, idx):
        self.team_max_stones[idx] = UPGRADED_MAX_STONES
        # Change color and text of upgrade button
        self.upgrade_btns[idx].config(state=tk.DISABLED, text="Storage Upgraded!", bg=self.upgrade_btns[idx].master.cget('bg'), fg="#c0392b", activebackground=self.upgrade_btns[idx].master.cget('bg'), activeforeground="#e74c3c")
        # Enable and color the delete button
        btn = self.delete_btns[idx]
        btn.pack(side=tk.LEFT)
        btn.config(state=tk.NORMAL, bg=btn.master.cget('bg'), fg="#c0392b", activebackground=btn.master.cget('bg'), activeforeground="#e74c3c")

    def delete_upgrade(self, idx):
        self.team_max_stones[idx] = DEFAULT_MAX_STONES
        # Reset color and text of upgrade button
        btn_upg = self.upgrade_btns[idx]
        btn_upg.config(
            state=tk.NORMAL,
            text="Upgrade Storage to 15",
            bg=btn_upg.master.cget('background'),
            fg=btn_upg.master.cget('foreground'),
            activebackground=btn_upg.master.cget('background'),
            activeforeground=btn_upg.master.cget('foreground')
        )
        # Hide the delete button visually and disable it
        btn = self.delete_btns[idx]
        parent_bg = btn.master.cget('background')
        btn.config(
            state=tk.DISABLED,
            bg=parent_bg,
            fg=parent_bg,
            activebackground=parent_bg,
            activeforeground=parent_bg
        )
        btn.pack_forget()

    def apply_theme(self):
        if self.mode == 'dark':
            bg = '#222222'
            fg = '#ffffff'
            btn_bg = '#333333'
            btn_fg = '#ffffff'
            entry_bg = '#111111'
            self.switch_btn.config(text="Switch to Light Mode", bg=btn_bg, fg=btn_fg, activebackground=bg, activeforeground=fg)
        else:
            bg = '#f0f0f0'
            fg = '#000000'
            btn_bg = '#e0e0e0'
            btn_fg = '#000000'
            entry_bg = '#ffffff'
            self.switch_btn.config(text="Switch to Dark Mode", bg=btn_bg, fg=btn_fg, activebackground=bg, activeforeground=fg)

        self.root.configure(bg=bg)
        # Update team tabs
        for i, frame in enumerate(self.tabs):
            frame.configure(bg=bg)
            for widget in frame.winfo_children():
                if isinstance(widget, tk.Label):
                    widget.configure(bg=bg, fg=fg)
                elif isinstance(widget, tk.Button):
                    widget.configure(bg=btn_bg, fg=btn_fg, activebackground=bg, activeforeground=fg)
            if i < len(self.upgrade_btns):
                self.upgrade_btns[i].configure(bg=btn_bg, fg=btn_fg, activebackground=bg, activeforeground=fg)
            if hasattr(self, 'delete_btns') and i < len(self.delete_btns):
                btn = self.delete_btns[i]
                if btn['state'] == tk.NORMAL:
                    btn.configure(bg=btn_bg, fg="#c0392b", activebackground=btn_bg, activeforeground="#e74c3c")
                else:
                    btn.configure(bg=btn_bg, fg=btn_fg, activebackground=btn_bg, activeforeground=btn_fg)

        # Update Start and Admin Mode tabs
        # Start tab is always the first tab (index 0), Admin Mode is always the last
        if hasattr(self, 'notebook'):
            start_tab = self.notebook.winfo_children()[0]
            for widget in start_tab.winfo_children():
                if isinstance(widget, tk.Label):
                    widget.configure(bg=bg, fg=fg)
                elif isinstance(widget, tk.Button):
                    widget.configure(bg=btn_bg, fg=btn_fg, activebackground=bg, activeforeground=fg)
            start_tab.configure(bg=bg)
            admin_tab = self.notebook.winfo_children()[-1]
            for widget in admin_tab.winfo_children():
                if isinstance(widget, tk.Label):
                    widget.configure(bg=bg, fg=fg)
                elif isinstance(widget, tk.Button):
                    widget.configure(bg=btn_bg, fg=btn_fg, activebackground=bg, activeforeground=fg)
            admin_tab.configure(bg=bg)
        style = ttk.Style()
        try:
            style.theme_use('clam')
        except Exception:
            try:
                style.theme_use('default')
            except Exception:
                pass
        if self.mode == 'dark':
            style.configure('TNotebook', background='#000000', borderwidth=0)
            style.configure('TNotebook.Tab', background=btn_bg, foreground=fg)
            style.map('TNotebook.Tab', background=[('selected', '#222222')])
        else:
            style.configure('TNotebook', background=bg)
            style.configure('TNotebook.Tab', background=btn_bg, foreground=fg)
            style.map('TNotebook.Tab', background=[('selected', bg)])
    def __init__(self, root):
        self.root = root
        self.root.title("Ressourcenstation App")
        # Start in normal windowed mode
        # Set app icon (pixel art coal mine)
        try:
            import os
            icon_ico = os.path.join(os.path.dirname(__file__), "coal_mine_logo.ico")
            if os.path.exists(icon_ico):
                self.root.iconbitmap(icon_ico)
            else:
                # Fallback: try PNG for in-app display (not window icon)
                from tkinter import PhotoImage
                icon_png = os.path.join(os.path.dirname(__file__), "coal_mine_logo.png")
                if os.path.exists(icon_png):
                    self.icon_img = PhotoImage(file=icon_png)
                    self.root.iconphoto(False, self.icon_img)
        except Exception as e:
            pass  # If icon fails, continue without it
        self.team_counts = [0] * TEAM_COUNT
        self.labels = []
        self.tabs = []
        self.upgrade_btns = []
        self.team_max_stones = [DEFAULT_MAX_STONES] * TEAM_COUNT
        self.team_items = [[] for _ in range(TEAM_COUNT)]
        self.max_storage = [DEFAULT_MAX_STONES] * TEAM_COUNT
        self.production_rates = [1.0] * TEAM_COUNT
        self.production_upgraded = [False] * TEAM_COUNT
        self.production_labels = []
        self.mode = 'dark'  # start in dark mode
        self.create_ui()
        self.apply_theme()

    def create_ui(self):
        # Theme switch button
        self.switch_btn = tk.Button(self.root, text="Switch to Light Mode", command=self.toggle_theme)
        self.switch_btn.pack(pady=5)

        self.notebook = ttk.Notebook(self.root)
        self.notebook.pack(expand=True, fill='both')

        # Start tab
        start_frame = tk.Frame(self.notebook, pady=40)
        self.notebook.add(start_frame, text="Start")
        start_label = tk.Label(start_frame, text="Welcome! Please select a team tab or Admin Mode.", font=("Arial", 18))
        start_label.pack(pady=40)
        reset_btn = tk.Button(start_frame, text="Reset All Stones", font=("Arial", 14), command=self.reset_all_stones)
        reset_btn.pack(pady=10)
        reset_upgrades_btn = tk.Button(start_frame, text="Reset All Upgrades", font=("Arial", 14), command=self.reset_all_upgrades)
        reset_upgrades_btn.pack(pady=10)

        # Team tabs
        for i in range(TEAM_COUNT):
            frame = tk.Frame(self.notebook, pady=20)
            self.notebook.add(frame, text=f"Team {i+1}")
            self.tabs.append(frame)
            label_title = tk.Label(frame, text=f"Team {i+1}", font=("Arial", 16))
            label_title.pack(pady=10)
            count_label = tk.Label(frame, text="0", width=5, font=("Arial", 24), relief=tk.SUNKEN)
            count_label.pack(pady=10)
            self.labels.append(count_label)
            # Production rate label
            prod_label = tk.Label(frame, text="Production: 1.0x", font=("Arial", 12))
            prod_label.pack(pady=2)
            self.production_labels.append(prod_label)
            # Frame for upgrade and delete buttons side by side
            btn_frame = tk.Frame(frame, bg=frame.cget('bg'), highlightthickness=0, bd=0)
            btn_frame.pack(pady=5)
            # Upgrade storage button
            upgrade_btn = tk.Button(
                btn_frame, text="Upgrade Storage to 15", width=22, height=2, command=lambda idx=i: self.upgrade_storage(idx)
            )
            upgrade_btn.pack(side=tk.LEFT, padx=0)
            self.upgrade_btns.append(upgrade_btn)
            # Delete upgrade button, do not pack initially
            delete_btn = tk.Button(
                btn_frame, text="✖", width=2, command=lambda idx=i: self.delete_upgrade(idx),
                bg=frame.cget('bg'), fg=frame.cget('bg'), activebackground=frame.cget('bg'), activeforeground=frame.cget('bg'),
                state=tk.DISABLED, borderwidth=0, highlightthickness=0
            )
            if not hasattr(self, 'delete_btns'):
                self.delete_btns = []
            self.delete_btns.append(delete_btn)
            # Production upgrade and delete buttons
            prod_btn_frame = tk.Frame(frame, bg=frame.cget('bg'), highlightthickness=0, bd=0)
            prod_btn_frame.pack(pady=2)
            prod_btn = tk.Button(prod_btn_frame, text="Upgrade Production (+50%)", width=22, command=lambda idx=i: self.upgrade_production(idx))
            prod_btn.pack(side=tk.LEFT)
            if not hasattr(self, 'prod_btns'):
                self.prod_btns = []
            self.prod_btns.append(prod_btn)
            prod_del_btn = tk.Button(prod_btn_frame, text="✖", width=2, command=lambda idx=i: self.delete_production_upgrade(idx),
                                    bg=frame.cget('bg'), fg=frame.cget('bg'), activebackground=frame.cget('bg'), activeforeground=frame.cget('bg'),
                                    state=tk.DISABLED, borderwidth=0, highlightthickness=0)
            if not hasattr(self, 'prod_del_btns'):
                self.prod_del_btns = []
            self.prod_del_btns.append(prod_del_btn)
            # Clear button below
            clear_btn = tk.Button(frame, text="Clear", width=12, command=lambda idx=i: self.clear_count(idx))
            clear_btn.pack(pady=5)

        # Admin Tab
        admin_frame = tk.Frame(self.notebook, pady=40)
        self.notebook.add(admin_frame, text="Admin Tab")
        admin_label = tk.Label(admin_frame, text="Admin Tab", font=("Arial", 18, "bold"))
        admin_label.pack(pady=40)
        # Admin Tab item management UI
        self.admin_selected_team = tk.IntVar(value=0)
        admin_item_frame = tk.Frame(admin_frame, bg=admin_frame.cget('bg'))
        admin_item_frame.pack(pady=10)
        tk.Label(admin_item_frame, text="Select Team:").pack(side=tk.LEFT)
        team_names = [f"Team {i+1}" for i in range(TEAM_COUNT)]
        def set_team(idx):
            self.admin_selected_team.set(idx)
            team_menu.config(text=team_names[idx])
            self.update_admin_item_view()
        team_menu = tk.Menubutton(admin_item_frame, text=team_names[0], relief=tk.RAISED)
        menu = tk.Menu(team_menu, tearoff=0)
        team_menu.config(menu=menu)
        for idx, name in enumerate(team_names):
            menu.add_command(label=name, command=lambda i=idx: set_team(i))
        team_menu.pack(side=tk.LEFT, padx=5)
        self.admin_add_btn = tk.Button(admin_item_frame, text="+", width=3, command=self.admin_add_stone)
        self.admin_add_btn.pack(side=tk.LEFT, padx=(0, 5))
        self.admin_del_btn = tk.Button(admin_item_frame, text="-", width=3, command=self.admin_delete_selected_item)
        self.admin_del_btn.pack(side=tk.LEFT)
        self.admin_item_list = tk.Listbox(admin_frame, width=30, height=5)
        self.admin_item_list.pack(pady=2)
        set_team(0)
        self.start_auto_add()

    def upgrade_production(self, idx):
        if not self.production_upgraded[idx]:
            self.production_rates[idx] *= 1.5
            self.production_upgraded[idx] = True
            self.production_labels[idx].config(text=f"Production: {self.production_rates[idx]:.1f}x")
            # Change color and text of upgrade button
            self.prod_btns[idx].config(state=tk.DISABLED, text="Production Upgraded!", bg=self.prod_btns[idx].master.cget('bg'), fg="#c0392b", activebackground=self.prod_btns[idx].master.cget('bg'), activeforeground="#e74c3c")
            # Show and enable delete button
            btn = self.prod_del_btns[idx]
            btn.pack(side=tk.LEFT)
            btn.config(state=tk.NORMAL, bg=btn.master.cget('bg'), fg="#c0392b", activebackground=btn.master.cget('bg'), activeforeground="#e74c3c")

    def delete_production_upgrade(self, idx):
        self.production_rates[idx] = 1.0
        self.production_upgraded[idx] = False
        self.production_labels[idx].config(text="Production: 1.0x")
        # Reset color and text of upgrade button
        btn_prod = self.prod_btns[idx]
        btn_prod.config(
            state=tk.NORMAL,
            text="Upgrade Production (+50%)",
            bg=btn_prod.master.cget('background'),
            fg=btn_prod.master.cget('foreground'),
            activebackground=btn_prod.master.cget('background'),
            activeforeground=btn_prod.master.cget('foreground')
        )
        # Hide and disable delete button
        btn = self.prod_del_btns[idx]
        parent_bg = btn.master.cget('background')
        btn.config(
            state=tk.DISABLED,
            bg=parent_bg,
            fg=parent_bg,
            activebackground=parent_bg,
            activeforeground=parent_bg
        )
        btn.pack_forget()


    def reset_all_stones(self):
        for idx in range(TEAM_COUNT):
            self.team_counts[idx] = 0
            self.labels[idx].config(text="0")

    def reset_all_upgrades(self):
        for idx in range(TEAM_COUNT):
            # Reset storage upgrade
            self.team_max_stones[idx] = DEFAULT_MAX_STONES
            btn_upg = self.upgrade_btns[idx]
            # Use default colors for reset
            btn_upg.config(
                state=tk.NORMAL,
                text="Upgrade Storage to 15",
                bg="#e0e0e0" if self.mode == 'light' else "#333333",
                fg="#000000" if self.mode == 'light' else "#ffffff",
                activebackground="#f0f0f0" if self.mode == 'light' else "#222222",
                activeforeground="#000000" if self.mode == 'light' else "#ffffff"
            )
            btn = self.delete_btns[idx]
            parent_bg = "#e0e0e0" if self.mode == 'light' else "#333333"
            btn.config(
                state=tk.DISABLED,
                bg=parent_bg,
                fg=parent_bg,
                activebackground=parent_bg,
                activeforeground=parent_bg
            )
            btn.pack_forget()
            # Reset production upgrade
            self.production_rates[idx] = 1.0
            self.production_upgraded[idx] = False
            self.production_labels[idx].config(text="Production: 1.0x")
            btn_prod = self.prod_btns[idx]
            btn_prod.config(
                state=tk.NORMAL,
                text="Upgrade Production (+50%)",
                bg="#e0e0e0" if self.mode == 'light' else "#333333",
                fg="#000000" if self.mode == 'light' else "#ffffff",
                activebackground="#f0f0f0" if self.mode == 'light' else "#222222",
                activeforeground="#000000" if self.mode == 'light' else "#ffffff"
            )
            btn2 = self.prod_del_btns[idx]
            parent_bg2 = "#e0e0e0" if self.mode == 'light' else "#333333"
            btn2.config(
                state=tk.DISABLED,
                bg=parent_bg2,
                fg=parent_bg2,
                activebackground=parent_bg2,
                activeforeground=parent_bg2
            )
            btn2.pack_forget()
        # Force UI refresh
        self.apply_theme()


    def update_admin_item_view(self):
        idx = self.admin_selected_team.get()
        self.admin_item_list.delete(0, tk.END)
        for item in self.team_items[idx]:
            self.admin_item_list.insert(tk.END, item)
        # Disable Add button if at max items
        max_items = self.max_storage[idx]
        if len(self.team_items[idx]) >= max_items:
            self.admin_add_btn.config(state=tk.DISABLED)
        else:
            self.admin_add_btn.config(state=tk.NORMAL)

    def admin_add_stone(self):
        idx = self.admin_selected_team.get()
        max_stones = self.team_max_stones[idx] if hasattr(self, 'team_max_stones') else self.max_storage[idx]
        if self.team_counts[idx] < max_stones:
            self.team_counts[idx] += 1
            self.labels[idx].config(text=str(self.team_counts[idx]))

    def admin_delete_selected_item(self):
        idx = self.admin_selected_team.get()
        if self.team_counts[idx] > 0:
            self.team_counts[idx] -= 1
            self.labels[idx].config(text=str(self.team_counts[idx]))

    def toggle_theme(self):
        self.mode = 'light' if self.mode == 'dark' else 'dark'
        self.apply_theme()

    def start_auto_add(self):
        pass

    def auto_add_stones(self):
        for idx in range(TEAM_COUNT):
            if self.team_counts[idx] < self.team_max_stones[idx]:
                add_amt = max(1, int(self.production_rates[idx]))
                new_count = min(self.team_counts[idx] + add_amt, self.team_max_stones[idx])
                self.team_counts[idx] = new_count
                self.labels[idx]["text"] = str(self.team_counts[idx])
        self.root.after(60000, self.auto_add_stones)

    def add_stone(self, idx):
        if self.team_counts[idx] < self.team_max_stones[idx]:
            self.team_counts[idx] += 1
            self.labels[idx]["text"] = str(self.team_counts[idx])
        else:
            messagebox.showinfo("Max Stones", f"Team {idx+1} already has the maximum of {self.team_max_stones[idx]} stones.")

    def clear_count(self, idx):
        self.team_counts[idx] = 0
        self.labels[idx]["text"] = "0"

if __name__ == "__main__":
    root = tk.Tk()
    app = TeamStoneApp(root)
    root.mainloop()
