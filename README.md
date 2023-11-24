# OS_File_Explorer

```
import os
import tkinter as tk
from tkinter import messagebox, filedialog, Text, Scrollbar
import datetime


# Define the log file path
log_file = "file_manager_log.txt"
# Define the "deleted_files" folder path
deleted_files_folder = r"C:\Users\admin\deleted_files"

# Function to list all the directories
def list_directory(path):
    f = []
    for files in os.listdir(path):
        if not files.startswith('.'):
            f.append(files)
    return f
def move_file():
    selected_indices = file_listbox.curselection()
    if selected_indices:
        selected_item = file_listbox.get(selected_indices[0])
        source_path = os.path.join(current_path.get(), selected_item)

        # Prompt the user to choose the destination directory
        destination_dir = filedialog.askdirectory(title="Select Destination Directory")
        if destination_dir:
            destination_path = os.path.join(destination_dir, selected_item)
            try:
                os.rename(source_path, destination_path)
                refresh_display(current_path.get())
                log_operation("Move", source_path, destination_path)
            except Exception as e:
                messagebox.showerror("Error", str(e))


# Function to view the contents when it is double-clicked
def navigate_directory(event):
    selected_item = file_listbox.get(file_listbox.curselection())
    new_path = os.path.join(current_path.get(), selected_item)
    refresh_display(new_path)

# Function to copy a file
def copy_file():
    selected_indices = file_listbox.curselection()
    if selected_indices:
        selected_item = file_listbox.get(selected_indices[0])
        source_path = os.path.join(current_path.get(), selected_item)
        destination_path = os.path.join(current_path.get(), "Copy_" + selected_item)
        try:
            import shutil
            shutil.copy(source_path, destination_path)
            refresh_display(current_path.get())
            log_operation("Copy", source_path, destination_path)
        except Exception as e:
            messagebox.showerror("Error", str(e))
# Function to display the contents of a file in a new window
def show_file_content(file_path, content):
    file_content_window = tk.Toplevel(root)
    file_content_window.title(f"File Content - {os.path.basename(file_path)}")

    text_widget = Text(file_content_window, wrap=tk.WORD)
    text_widget.pack(expand=tk.YES, fill=tk.BOTH)

    scrollbar = Scrollbar(file_content_window, command=text_widget.yview)
    scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
    text_widget.config(yscrollcommand=scrollbar.set)

    text_widget.insert(tk.END, content)
    text_widget.config(state=tk.DISABLED)

# Function to move a file to the "deleted_files" folder
def delete_file():
    selected_item = file_listbox.get(file_listbox.curselection())
    file_path = os.path.join(current_path.get(), selected_item)
    deleted_file_path = os.path.join(deleted_files_folder, selected_item)
    try:
        os.rename(file_path, deleted_file_path)
        refresh_display(current_path.get())
        log_operation("Delete", file_path)
    except Exception as e:
        messagebox.showerror("Error", str(e))

# Function to refresh the display
def refresh_display(new_path):
    current_path.set(new_path)
    file_listbox.delete(0, tk.END) #delete all contents
    for item in list_directory(new_path):
        file_listbox.insert(tk.END, item)

# Function to log file operations
def log_operation(operation, *paths):
    timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")
    with open(log_file, "a") as log:
        log.write(f"{timestamp} - {operation}: {' | '.join(paths)}\n") 

# Function to retrieve a file from the "deleted_files" folder
def retrieve_file():
    selected_indices = deleted_files_listbox.curselection()
    if selected_indices:
        selected_item = deleted_files_listbox.get(selected_indices[0])
        source_path = os.path.join(deleted_files_folder, selected_item)
        destination_path = os.path.join(current_path.get(), selected_item)
        try:
            os.rename(source_path, destination_path)
            refresh_deleted_files_list()
        except Exception as e:
            messagebox.showerror("Error", str(e))

# Function to permanently delete a file from the "deleted_files" folder
def delete_permanently():
    selected_indices = deleted_files_listbox.curselection()
    if selected_indices:
        selected_item = deleted_files_listbox.get(selected_indices[0])
        file_path = os.path.join(deleted_files_folder, selected_item)
        try:
            os.remove(file_path)
            refresh_deleted_files_list()
        except Exception as e:
            messagebox.showerror("Error", str(e))
# Function to open the journal window and display log file operations
def open_journal_window():
    if hasattr(open_journal_window, 'journal_window') and open_journal_window.journal_window:
        open_journal_window.journal_window.deiconify()
        journal_text.delete(1.0, tk.END)
        try:
            with open(log_file, "r") as log:
                log_contents = log.read()
            journal_text.insert(tk.END, log_contents)
        except Exception as e:
            journal_text.insert(tk.END, f"Error reading log file: {str(e)}")
# Function to move up one directory
def move_up_directory():
    current_dir = os.path.abspath(current_path.get())
    parent_dir = os.path.dirname(current_dir)
    refresh_display(parent_dir)

# Function to refresh the deleted files list
def refresh_deleted_files_list():
    deleted_files_listbox.delete(0, tk.END)
    for item in list_directory(deleted_files_folder):
        deleted_files_listbox.insert(tk.END, item)

# Function to open the "Deleted Files" window
def open_deleted_files_window():
    deleted_files_window.deiconify()
    refresh_deleted_files_list()

# Create the main window
root = tk.Tk()
root.title("File Manager")

# Initialize the current path to the user's home directory
current_path = tk.StringVar()
current_path.set(os.path.expanduser("~"))

# Create a label to display the current path
path_label = tk.Label(root, textvariable=current_path)
path_label.pack()

# Create a listbox to display the files and folders
file_listbox = tk.Listbox(root, selectmode=tk.SINGLE)
file_listbox.pack()

# Bind double-click event to navigate_directory function
file_listbox.bind("<Double-1>", navigate_directory)

# Create buttons for file operations
button_frame = tk.Frame(root)
copy_button = tk.Button(button_frame, text="Copy", command=copy_file)
delete_button = tk.Button(button_frame, text="Delete", command=delete_file)
move_button = tk.Button(button_frame, text="Move", command=move_file)
journal_button = tk.Button(button_frame, text="Journal", command=open_journal_window)
move_up_button=tk.Button(button_frame,text="Move up",command=move_up_directory);

copy_button.pack(side=tk.LEFT)
delete_button.pack(side=tk.LEFT)
move_button.pack(side=tk.LEFT)
journal_button.pack(side=tk.LEFT)
move_up_button.pack(side=tk.LEFT)
button_frame.pack()

# Create a "Deleted Files" button
deleted_files_button = tk.Button(root, text="Deleted Files", command=open_deleted_files_window)
deleted_files_button.pack()

# Initial file list
refresh_display(current_path.get())

# Create the "Deleted Files" window
deleted_files_window = tk.Toplevel(root)
deleted_files_window.title("Deleted Files")
deleted_files_window.withdraw()

# Create a listbox to display deleted files
deleted_files_listbox = tk.Listbox(deleted_files_window, selectmode=tk.SINGLE)
deleted_files_listbox.pack()

# Create buttons for retrieval and permanent deletion
retrieve_button = tk.Button(deleted_files_window, text="Retrieve", command=retrieve_file)
delete_permanent_button = tk.Button(deleted_files_window, text="Delete Permanently", command=delete_permanently)

retrieve_button.pack()
delete_permanent_button.pack()

# Function to refresh the deleted files list
def refresh_deleted_files_list():
    deleted_files_listbox.delete(0, tk.END)
    for item in list_directory(deleted_files_folder):
        deleted_files_listbox.insert(tk.END, item)

"""# Create a "Journal" button
journal_button = tk.Button(root, text="Journal", command=open_journal_window)
journal_button.pack(side=tk.LEFT)"""

root.mainloop();
```
