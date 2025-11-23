import tkinter as tk
from tkinter import filedialog, messagebox, ttk
import subprocess
import os

class SimpleTextEditor:
    """
    A simple text editor application built using tkinter, enhanced with
    a dark theme and a feature to compile and run C code using gcc.
    """
    def __init__(self, master):
        self.master = master
        master.title("Python Code Editor & Runner")
        self.current_file = None
        
        # --- 1. Apply Dark Theme and Configuration ---
        self._apply_dark_theme()
        
        # Configure layout weights for resizing
        master.grid_rowconfigure(0, weight=3)  # Text editor takes most space
        master.grid_rowconfigure(1, weight=1)  # Console takes less space
        master.grid_columnconfigure(0, weight=1)

        # --- 2. Text Editor Area ---
        self.text_area = tk.Text(
            master, 
            wrap=tk.WORD, 
            font=("Consolas", 12),
            bg="#2e2e2e", 
            fg="#ffffff", 
            insertbackground="#ffffff", # Cursor color
            bd=0, 
            relief=tk.FLAT, 
            undo=True
        )
        self.text_area.grid(row=0, column=0, sticky="nsew")

        # Scrollbar
        self.scrollbar = ttk.Scrollbar(master, command=self.text_area.yview)
        self.scrollbar.grid(row=0, column=1, sticky="ns")
        self.text_area.config(yscrollcommand=self.scrollbar.set)
        
        # --- 3. Output Console (for C execution) ---
        ttk.Label(master, text="Output Console:", style="TLabel").grid(row=1, column=0, sticky="sw", padx=5)
        self.output_console = tk.Text(
            master, 
            height=10, 
            font=("Consolas", 10),
            bg="#1e1e1e", 
            fg="#cccccc", 
            state=tk.DISABLED, # Read-only
            bd=0, 
            relief=tk.FLAT
        )
        self.output_console.grid(row=2, column=0, columnspan=2, sticky="nsew", padx=5, pady=5)
        master.grid_rowconfigure(2, weight=0) # Console row keeps fixed height relative to text area

        # --- 4. Create Menu Bar ---
        self.menu_bar = tk.Menu(master)
        master.config(menu=self.menu_bar)

        # File Menu
        self.file_menu = tk.Menu(self.menu_bar, tearoff=0)
        self.menu_bar.add_cascade(label="File", menu=self.file_menu)
        self.file_menu.add_command(label="New", command=self.new_file)
        self.file_menu.add_command(label="Open...", command=self.open_file)
        self.file_menu.add_command(label="Save", command=self.save_file)
        self.file_menu.add_command(label="Save As...", command=self.save_file_as)
        self.file_menu.add_separator()
        self.file_menu.add_command(label="Exit", command=master.quit)
        
        # Execution Menu (New)
        self.execution_menu = tk.Menu(self.menu_bar, tearoff=0)
        self.menu_bar.add_cascade(label="Run", menu=self.execution_menu)
        self.execution_menu.add_command(label="Run C Code (F5)", command=self.run_c_code)

        # --- 5. Bind Keyboard Shortcuts ---
        master.bind('<Control-n>', lambda event: self.new_file())
        master.bind('<Control-o>', lambda event: self.open_file())
        master.bind('<Control-s>', lambda event: self.save_file())
        master.bind('<Control-S>', lambda event: self.save_file_as()) 
        master.bind('<F12>', lambda event: self.run_c_code())

    def _apply_dark_theme(self):
        """Sets up the ttk style for a dark mode interface."""
        style = ttk.Style(self.master)
        
        # Set the general theme to 'clam' for more customization options
        style.theme_use("clam") 
        
        # Define dark colors
        BACKGROUND_COLOR = "#1e1e1e"
        FOREGROUND_COLOR = "#cccccc"
        ACCENT_COLOR = "#007acc"

        # Apply styles to standard widgets
        style.configure(".", background=BACKGROUND_COLOR, foreground=FOREGROUND_COLOR, fieldbackground=BACKGROUND_COLOR)
        style.configure("TLabel", background=BACKGROUND_COLOR, foreground=FOREGROUND_COLOR, font=("Arial", 10, "bold"))
        style.configure("TButton", background="#3e3e3e", foreground="#ffffff", borderwidth=0, padding=5)
        style.map("TButton", 
                  background=[('active', '#5e5e5e'), ('pressed', ACCENT_COLOR)],
                  foreground=[('active', '#ffffff')])
        
        # Apply the dark background to the main window
        self.master.config(bg=BACKGROUND_COLOR)

    def _update_output_console(self, text, is_error=False):
        """Utility function to update the read-only output console."""
        self.output_console.config(state=tk.NORMAL)
        self.output_console.delete(1.0, tk.END)
        
        if is_error:
            self.output_console.insert(tk.END, "--- ERROR / COMPILER MESSAGE ---\n", 'error')
            self.output_console.tag_config('error', foreground='#ff4444')
            self.output_console.insert(tk.END, text)
        else:
            self.output_console.insert(tk.END, "--- PROGRAM OUTPUT ---\n", 'success')
            self.output_console.tag_config('success', foreground='#44ff44')
            self.output_console.insert(tk.END, text)
            
        self.output_console.config(state=tk.DISABLED)

    def run_c_code(self):
        """
        Compiles and runs the currently open file, assuming it is C code, using gcc.
        The output/errors are displayed in the console area.
        """
        if not self.save_file():
            self._update_output_console("Could not save the file. Aborting run.", is_error=True)
            return

        if not self.current_file or not self.current_file.lower().endswith(('.c', '.cpp', '.h')):
            self._update_output_console("File is not a recognized C/C++ source file (.c, .cpp, .h).", is_error=True)
            return

        source_path = self.current_file
        # Determine the executable path based on OS (a.out for Linux/Mac, a.exe for Windows)
        executable_name = "a.out"
        if os.name == 'nt': # Windows
            executable_name = "a.exe"
            
        # Get the directory of the source file
        output_dir = os.path.dirname(source_path)
        executable_path = os.path.join(output_dir, executable_name)
        
        # --- Step 1: Compilation ---
        self._update_output_console("Compiling code...\n")
        
        # Use a relative path for the output file
        compile_command = ['gcc', source_path, '-o', executable_path]
        
        try:
            # Run the compilation command
            compile_result = subprocess.run(
                compile_command, 
                capture_output=True, 
                text=True, 
                check=False
            )
            
            if compile_result.returncode != 0:
                # Compilation failed
                self._update_output_console(f"Compilation Failed (Exit Code {compile_result.returncode})\n\n{compile_result.stderr}", is_error=True)
                return
            
            # --- Step 2: Execution ---
            self._update_output_console("Compilation successful. Running program...\n")
            
            # Run the executable
            # Need to execute the file from its directory
            run_result = subprocess.run(
                [executable_path], 
                cwd=output_dir, # Set current working directory for execution
                capture_output=True, 
                text=True, 
                check=False,
                timeout=5 # Add a timeout to prevent infinite loops from hanging the editor
            )
            
            if run_result.returncode != 0:
                # Execution failed (e.g., runtime error)
                self._update_output_console(f"Program Execution Failed (Exit Code {run_result.returncode})\n\n{run_result.stderr}", is_error=True)
            else:
                # Success
                self._update_output_console(run_result.stdout, is_error=False)

        except FileNotFoundError:
            self._update_output_console("Error: 'gcc' command not found. Ensure GCC is installed and in your system PATH.", is_error=True)
        except subprocess.TimeoutExpired:
            self._update_output_console("Error: Program timed out after 5 seconds.", is_error=True)
        except Exception as e:
            self._update_output_console(f"An unexpected error occurred: {e}", is_error=True)
        finally:
            # Clean up the executable if it exists
            try:
                if os.path.exists(executable_path):
                    os.remove(executable_path)
            except Exception as e:
                # Print a console message about cleanup failure, but don't show it to the user
                print(f"Cleanup warning: Could not remove temporary executable at {executable_path}. Error: {e}")

    def new_file(self):
        """Creates a new, empty file instance."""
        if self._check_unsaved_changes():
            self.text_area.delete(1.0, tk.END)
            self.current_file = None
            self.master.title("Python Code Editor & Runner - Untitled")
            self._update_output_console("", is_error=False) # Clear console

    def open_file(self):
        """Opens a file dialog and loads the selected file's content."""
        if not self._check_unsaved_changes():
            return

        file_path = filedialog.askopenfilename(
            defaultextension=".c",
            filetypes=[("C Source Files", "*.c"), ("C++ Source Files", "*.cpp"), ("Text files", "*.txt"), ("All files", "*.*")]
        )
        if file_path:
            try:
                with open(file_path, "r") as file:
                    content = file.read()
                    self.text_area.delete(1.0, tk.END)
                    self.text_area.insert(1.0, content)
                self.current_file = file_path
                self.master.title(f"Python Code Editor & Runner - {os.path.basename(self.current_file)}")
                self._update_output_console(f"File opened: {os.path.basename(self.current_file)}", is_error=False)
            except Exception as e:
                messagebox.showerror("Error", f"Could not open file: {e}")

    def save_file(self):
        """Saves the current content to the current file path, or calls save_file_as if no path exists."""
        if self.current_file:
            try:
                content = self.text_area.get(1.0, tk.END)
                with open(self.current_file, "w") as file:
                    file.write(content)
                self._update_output_console(f"File saved: {os.path.basename(self.current_file)}", is_error=False)
                return True
            except Exception as e:
                messagebox.showerror("Error", f"Could not save file: {e}")
                return False
        else:
            return self.save_file_as()

    def save_file_as(self):
        """Opens a save dialog to choose a path for saving the file."""
        file_path = filedialog.asksaveasfilename(
            defaultextension=".c",
            filetypes=[("C Source Files", "*.c"), ("C++ Source Files", "*.cpp"), ("Text files", "*.txt"), ("All files", "*.*")]
        )
        if file_path:
            self.current_file = file_path
            success = self.save_file()
            if success:
                self.master.title(f"Python Code Editor & Runner - {os.path.basename(self.current_file)}")
            return success
        return False

    def _check_unsaved_changes(self):
        """
        Checks if the document has unsaved content before performing actions like 'New' or 'Open'.
        Returns True if safe to proceed (either no changes or user confirmed discard), False otherwise.
        """
        # A very basic check: if the content is not empty
        current_content = self.text_area.get(1.0, tk.END).strip()
        if current_content:
            response = messagebox.askyesnocancel(
                "Unsaved Changes",
                "You have unsaved changes. Do you want to save before proceeding?"
            )
            if response is True:
                return self.save_file() # Save and then continue
            elif response is False:
                return True # Discard changes and continue
            else: # response is None (Cancel)
                return False
        return True # No content, safe to continue

# Application entry point
if __name__ == "__main__":
    root = tk.Tk()
    editor = SimpleTextEditor(root)
    # Set a minimum size for usability
    root.geometry("800x600") 
    root.mainloop()
