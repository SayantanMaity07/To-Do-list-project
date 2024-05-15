# To-Do-list-project
import tkinter as tk
from tkinter import messagebox
from tkinter import simpledialog
import datetime
import json
import os

# Global variables to store the color schemes
light_mode_colors = {
    "bg": "#f0f0f0",
    "fg": "black",
    "entry_bg": "white",
    "entry_fg": "black",
    "button_bg": {
        "add": "#4CAF50",       # Green
        "edit": "#FFC107",      # Amber
        "delete": "#F44336",    # Red
        "complete": "#2196F3",  # Blue
        "search": "#9C27B0",    # Purple
        "switch": "#607D8B"     # Blue Grey
    },
    "button_fg": "white",
    "listbox_bg": "white",
    "listbox_fg": "black"
}

dark_mode_colors = {
    "bg": "#2e2e2e",
    "fg": "white",
    "entry_bg": "#3e3e3e",
    "entry_fg": "white",
    "button_bg": {
        "add": "#4CAF50",
        "edit": "#FFC107",
        "delete": "#F44336",
        "complete": "#2196F3",
        "search": "#9C27B0",
        "switch": "#607D8B"
    },
    "button_fg": "white",
    "listbox_bg": "#3e3e3e",
    "listbox_fg": "white"
}

current_colors = light_mode_colors

def switch_mode():
    global current_colors
    if current_colors == light_mode_colors:
        current_colors = dark_mode_colors
    else:
        current_colors = light_mode_colors
    apply_colors()

def apply_colors():
    app.configure(bg=current_colors["bg"])
    task_entry.configure(bg=current_colors["entry_bg"], fg=current_colors["entry_fg"])
    todo_list.configure(bg=current_colors["listbox_bg"], fg=current_colors["listbox_fg"])

    for button, color in current_colors["button_bg"].items():
        buttons[button].configure(bg=color, fg=current_colors["button_fg"], activebackground=color)

def add_task():
    task_text = task_entry.get()
    if task_text:
        due_date = simpledialog.askstring("Due Date", "Enter due date (YYYY-MM-DD):")
        if due_date:
            try:
                datetime.datetime.strptime(due_date, "%Y-%m-%d")
                category = simpledialog.askstring("Category", "Enter task category:")
                todo_list.insert(tk.END, f"{task_text} (Due: {due_date}) [Category: {category}]")
                save_tasks()
                task_entry.delete(0, tk.END)
            except ValueError:
                messagebox.showwarning("Input Error", "Please enter a valid date in YYYY-MM-DD format.")
        else:
            todo_list.insert(tk.END, task_text)
            save_tasks()
            task_entry.delete(0, tk.END)
    else:
        messagebox.showwarning("Input Error", "Please enter a task.")

def edit_task():
    try:
        selected_task_index = todo_list.curselection()[0]
        edited_task_text = task_entry.get()
        if edited_task_text:
            due_date = simpledialog.askstring("Due Date", "Enter due date (YYYY-MM-DD):")
            if due_date:
                try:
                    datetime.datetime.strptime(due_date, "%Y-%m-%d")
                    category = simpledialog.askstring("Category", "Enter task category:")
                    todo_list.delete(selected_task_index)
                    todo_list.insert(selected_task_index, f"{edited_task_text} (Due: {due_date}) [Category: {category}]")
                    save_tasks()
                    task_entry.delete(0, tk.END)
                except ValueError:
                    messagebox.showwarning("Input Error", "Please enter a valid date in YYYY-MM-DD format.")
            else:
                todo_list.delete(selected_task_index)
                todo_list.insert(selected_task_index, edited_task_text)
                save_tasks()
                task_entry.delete(0, tk.END)
        else:
            messagebox.showwarning("Input Error", "Please enter a task.")
    except IndexError:
        messagebox.showwarning("Selection Error", "Please select a task to edit.")

def delete_task():
    try:
        selected_task_index = todo_list.curselection()[0]
        todo_list.delete(selected_task_index)
        save_tasks()
    except IndexError:
        messagebox.showwarning("Selection Error", "Please select a task to delete.")

def mark_completed():
    try:
        selected_task_index = todo_list.curselection()[0]
        task_text = todo_list.get(selected_task_index)
        todo_list.delete(selected_task_index)
        todo_list.insert(selected_task_index, f"{task_text} - Completed")
        save_tasks()
    except IndexError:
        messagebox.showwarning("Selection Error", "Please select a task to mark as completed.")

def save_tasks():
    tasks = todo_list.get(0, tk.END)
    with open("tasks.json", "w") as file:
        json.dump(tasks, file)

def load_tasks():
    if os.path.exists("tasks.json"):
        with open("tasks.json", "r") as file:
            tasks = json.load(file)
            for task in tasks:
                todo_list.insert(tk.END, task)

def search_tasks():
    search_term = task_entry.get()
    tasks = todo_list.get(0, tk.END)
    todo_list.delete(0, tk.END)
    for task in tasks:
        if search_term.lower() in task.lower():
            todo_list.insert(tk.END, task)

app = tk.Tk()
app.title("Advanced To-Do List Application")
app.geometry("500x600")

header_frame = tk.Frame(app, bg=current_colors["bg"])
header_frame.pack(pady=10)

title_label = tk.Label(header_frame, text="To-Do List", font=("Helvetica", 18, "bold"), bg=current_colors["bg"], fg=current_colors["fg"])
title_label.pack()

top_frame = tk.Frame(app, bg=current_colors["bg"])
top_frame.pack(padx=10, pady=10, fill=tk.X)

middle_frame = tk.Frame(app, bg=current_colors["bg"])
middle_frame.pack(padx=10, pady=10, fill=tk.BOTH, expand=True)

bottom_frame = tk.Frame(app, bg=current_colors["bg"])
bottom_frame.pack(padx=10, pady=10, fill=tk.X)

task_entry = tk.Entry(top_frame, width=40, font=("Helvetica", 12), bg=current_colors["entry_bg"], fg=current_colors["entry_fg"], bd=2, relief=tk.SUNKEN)
task_entry.pack(side=tk.LEFT, padx=5, pady=5)

def create_button(parent, text, command, color_key):
    button = tk.Button(parent, text=text, command=command, font=("Helvetica", 12), bg=current_colors["button_bg"][color_key], fg=current_colors["button_fg"], activebackground=current_colors["button_bg"][color_key], bd=0, padx=10, pady=5)
    button.pack(side=tk.LEFT, padx=5, pady=5)
    return button

add_button = create_button(top_frame, "Add Task", add_task, "add")
search_button = create_button(top_frame, "Search", search_tasks, "search")
switch_button = create_button(top_frame, "Switch Mode", switch_mode, "switch")

todo_list = tk.Listbox(middle_frame, width=50, height=15, font=("Helvetica", 12), bg=current_colors["listbox_bg"], fg=current_colors["listbox_fg"], selectbackground=current_colors["entry_bg"], selectforeground=current_colors["entry_fg"], bd=2, relief=tk.SUNKEN)
todo_list.pack(padx=10, pady=10, fill=tk.BOTH, expand=True)

edit_button = create_button(bottom_frame, "Edit Task", edit_task, "edit")
delete_button = create_button(bottom_frame, "Delete Task", delete_task, "delete")
complete_button = create_button(bottom_frame, "Mark Completed", mark_completed, "complete")

buttons = {
    "add": add_button,
    "edit": edit_button,
    "delete": delete_button,
    "complete": complete_button,
    "search": search_button,
    "switch": switch_button
}

load_tasks()
apply_colors()
app.mainloop()
