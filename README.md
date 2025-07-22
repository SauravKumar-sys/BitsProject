# BitsProject


import json
import argparse
from datetime import datetime, date

# --- Task Class ---
class Task:
    """
    Represents a single To-Do task.
    Attributes:
        description (str): The main description of the task.
        due_date (date, optional): The date by which the task should be completed.
        priority (str): The urgency level ('low', 'medium', 'high').
        status (str): The completion status ('pending', 'completed').
        created_at (str): Timestamp when the task was created.
    """
    VALID_PRIORITIES = ["low", "medium", "high"]
    VALID_STATUSES = ["pending", "completed"]

    def __init__(self, description: str, due_date: str = None, priority: str = "medium", status: str = "pending"):
        if not isinstance(description, str) or not description.strip():
            raise ValueError("Task description cannot be empty.")
        self.description = description.strip()
        self.due_date = self._parse_date(due_date) if due_date else None
        self.priority = self._validate_priority(priority)
        self.status = self._validate_status(status)
        self.created_at = datetime.now().strftime("%Y-%m-%d %H:%M:%S")

    def _parse_date(self, date_str: str) -> date:
        """Parses a date string (YYYY-MM-DD) into a date object."""
        try:
            return datetime.strptime(date_str, "%Y-%m-%d").date()
        except ValueError:
            raise ValueError(f"Invalid date format for '{date_str}'. Use YYYY-MM-DD.")

    def _validate_priority(self, priority: str) -> str:
        """Validates the given priority string."""
        if priority.lower() not in self.VALID_PRIORITIES:
            raise ValueError(f"Invalid priority '{priority}'. Must be one of {', '.join(self.VALID_PRIORITIES)}.")
        return priority.lower()

    def _validate_status(self, status: str) -> str:
        """Validates the given status string."""
        if status.lower() not in self.VALID_STATUSES:
            raise ValueError(f"Invalid status '{status}'. Must be one of {', '.join(self.VALID_STATUSES)}.")
        return status.lower()

    def mark_completed(self):
        """Changes the task status to 'completed'."""
        self.status = "completed"

    def to_dict(self) -> dict:
        """Converts the Task object to a dictionary for JSON serialization."""
        return {
            "description": self.description,
            "due_date": self.due_date.strftime("%Y-%m-%d") if self.due_date else None,
            "priority": self.priority,
            "status": self.status,
            "created_at": self.created_at
        }

    @classmethod
    def from_dict(cls, data: dict):
        """Creates a Task object from a dictionary, typically for JSON deserialization."""
        # Ensure compatibility with older data formats that might miss new keys
        description = data.get("description")
        if not description:
            raise ValueError("Missing 'description' in task data.")
        
        return cls(
            description=description,
            due_date=data.get("due_date"),
            priority=data.get("priority", "medium"),
            status=data.get("status", "pending")
            # Note: created_at is set by __init__ by default, but we can override it if present
        )._set_created_at(data.get("created_at")) # Internal method to set created_at from loaded data

    def _set_created_at(self, created_at_str: str):
        """Internal helper to set created_at from a string, primarily for loading from file."""
        if created_at_str:
            self.created_at = created_at_str
        return self

    def __str__(self) -> str:
        """Returns a user-friendly string representation of the task."""
        due_str = f" (Due: {self.due_date.strftime('%Y-%m-%d')})" if self.due_date else ""
        return (f"[{'X' if self.status == 'completed' else ' '}] "
                f"Description: {self.description}{due_str}, "
                f"Priority: {self.priority.capitalize()}, Status: {self.status.capitalize()}")

    def __repr__(self) -> str:
        """Returns a developer-friendly string representation of the task."""
        return (f"Task(description='{self.description}', due_date='{self.due_date}', "
                f"priority='{self.priority}', status='{self.status}')")

# --- TodoListManager Class ---
class TodoListManager:
    """
    Manages the collection of Task objects, including loading from and saving to a file.
    Provides methods for adding, viewing, marking, and deleting tasks.
    """
    def __init__(self, filename: str = "tasks.json"):
        self.filename = filename
        self.tasks: list[Task] = self._load_tasks()

    def _load_tasks(self) -> list[Task]:
        """
        Loads tasks from the specified JSON file. Handles file not found
        and JSON decoding errors.
        """
        tasks = []
        try:
            with open(self.filename, 'r', encoding='utf-8') as f:
                data = json.load(f)
                if not isinstance(data, list):
                    print(f"Warning: Expected a list of tasks in {self.filename}, but found a different structure. Starting with an empty list.")
                    return []
                for item in data:
                    try:
                        tasks.append(Task.from_dict(item))
                    except (ValueError, KeyError) as e:
                        print(f"Warning: Could not load malformed task data: {item}. Error: {e}")
        except FileNotFoundError:
            print(f"No task file found at '{self.filename}'. Starting with an empty list.")
        except json.JSONDecodeError:
            print(f"Error: Could not decode JSON from '{self.filename}'. File might be corrupted or empty. Starting with an empty list.")
        except Exception as e:
            print(f"An unexpected error occurred while loading tasks from '{self.filename}': {e}")
        return tasks

    def _save_tasks(self):
        """Saves the current list of tasks to the specified JSON file."""
        try:
            with open(self.filename, 'w', encoding='utf-8') as f:
                json.dump([task.to_dict() for task in self.tasks], f, indent=4)
        except IOError as e:
            print(f"Error: Could not save tasks to '{self.filename}': {e}")
        except Exception as e:
            print(f"An unexpected error occurred while saving tasks: {e}")

    def add_task(self, description: str, due_date: str = None, priority: str = "medium"):
        """Adds a new task to the list and persists the changes."""
        try:
            new_task = Task(description, due_date, priority)
            self.tasks.append(new_task)
            self._save_tasks()
            print(f"Success: Task '{description}' added.")
        except ValueError as e:
            print(f"Error adding task: {e}")

    def view_tasks(self, filter_status: str = None, sort_by: str = None):
        """
        Displays tasks based on optional filters and sorting criteria.
        Args:
            filter_status (str, optional): 'pending' or 'completed' to filter tasks.
            sort_by (str, optional): 'due_date', 'priority', or 'created_at' to sort.
        """
        display_tasks = list(self.tasks) # Create a copy to sort/filter

        if filter_status:
            if filter_status.lower() not in Task.VALID_STATUSES:
                print(f"Error: Invalid status filter '{filter_status}'. Use 'pending' or 'completed'.")
                return
            display_tasks = [task for task in display_tasks if task.status == filter_status.lower()]

        if sort_by:
            if sort_by == "due_date":
                # Sort by completed status first (completed at end), then by due date (None last)
                display_tasks.sort(key=lambda t: (t.status == 'completed', t.due_date is None, t.due_date or date.max))
            elif sort_by == "priority":
                priority_map = {"high": 1, "medium": 2, "low": 3}
                # Sort by completed status, then by numerical priority
                display_tasks.sort(key=lambda t: (t.status == 'completed', priority_map.get(t.priority, 99)))
            elif sort_by == "created_at":
                display_tasks.sort(key=lambda t: datetime.strptime(t.created_at, "%Y-%m-%d %H:%M:%S"))
            else:
                print(f"Error: Invalid sort option '{sort_by}'. Use 'due_date', 'priority', or 'created_at'.")
                return

        if not display_tasks:
            print("No tasks to display.")
            return

        print("\n--- Your To-Do List ---")
        for i, task in enumerate(display_tasks):
            print(f"{i + 1:2}. {task}") # Format index with 2 spaces for alignment
        print("-----------------------\n")

    def mark_task_completed(self, task_id: int):
        """
        Marks a task as completed based on its 1-indexed ID from the current list.
        Args:
            task_id (int): The 1-indexed ID of the task.
        """
        try:
            # Adjust to 0-indexed for list access
            index = task_id - 1 
            if not (0 <= index < len(self.tasks)):
                raise IndexError(f"Task ID {task_id} is out of range.")
            
            task = self.tasks[index]
            if task.status == "completed":
                print(f"Info: Task '{task.description}' is already completed.")
            else:
                task.mark_completed()
                self._save_tasks()
                print(f"Success: Task '{task.description}' marked as completed.")
        except (ValueError, IndexError) as e:
            print(f"Error marking task: {e}")
        except Exception as e:
            print(f"An unexpected error occurred while marking task: {e}")

    def delete_task(self, task_id: int):
        """
        Deletes a task based on its 1-indexed ID from the current list.
        Args:
            task_id (int): The 1-indexed ID of the task.
        """
        try:
            # Adjust to 0-indexed for list access
            index = task_id - 1
            if not (0 <= index < len(self.tasks)):
                raise IndexError(f"Task ID {task_id} is out of range.")
            
            removed_task = self.tasks.pop(index)
            self._save_tasks()
            print(f"Success: Task '{removed_task.description}' deleted.")
        except (ValueError, IndexError) as e:
            print(f"Error deleting task: {e}")
        except Exception as e:
            print(f"An unexpected error occurred while deleting task: {e}")

# --- CLI Parsing with Argparse ---
def main():
    """
    Main function to parse command-line arguments and execute To-Do list operations.
    """
    parser = argparse.ArgumentParser(
        description="""
        Advanced CLI To-Do List Application.
        Manage your tasks from the command line, with features like
        adding, viewing, marking completed, deleting, prioritizing,
        and setting due dates. Tasks are saved to a JSON file.

        Examples:
          python todo_app.py add "Buy groceries" --due 2025-07-30 --priority high
          python todo_app.py view --status pending --sort due_date
          python todo_app.py complete 1
          python todo_app.py delete 2
          python todo_app.py --file my_work_tasks.json add "Finish report"
        """,
        formatter_class=argparse.RawTextHelpFormatter # Preserves newlines in description
    )

    parser.add_argument(
        "--file",
        default="tasks.json",
        help="Specify the JSON file to store tasks (default: tasks.json)."
    )

    subparsers = parser.add_subparsers(dest="command", help="Available commands")

    # --- 'add' command ---
    add_parser = subparsers.add_parser(
        "add", 
        help="Add a new task.",
        description="Add a new task to your to-do list."
    )
    add_parser.add_argument(
        "description", 
        type=str, 
        help="Description of the task (e.g., 'Read Python book')."
    )
    add_parser.add_argument(
        "--due", 
        type=str, 
        help="Due date for the task (format: YYYY-MM-DD).", 
        default=None
    )
    add_parser.add_argument(
        "--priority", 
        type=str, 
        choices=Task.VALID_PRIORITIES,
        default="medium", 
        help=f"Priority of the task (choices: {', '.join(Task.VALID_PRIORITIES)}). Default is 'medium'."
    )

    # --- 'view' command ---
    view_parser = subparsers.add_parser(
        "view", 
        help="View tasks.",
        description="Display tasks with optional filters and sorting."
    )
    view_parser.add_argument(
        "--status", 
        type=str, 
        choices=Task.VALID_STATUSES,
        help=f"Filter tasks by status (choices: {', '.join(Task.VALID_STATUSES)})."
    )
    view_parser.add_argument(
        "--sort", 
        type=str, 
        choices=["due_date", "priority", "created_at"],
        help="Sort tasks by attribute (choices: 'due_date', 'priority', 'created_at')."
    )

    # --- 'complete' command ---
    complete_parser = subparsers.add_parser(
        "complete", 
        help="Mark a task as completed.",
        description="Mark a specific task as completed using its ID from the 'view' list."
    )
    complete_parser.add_argument(
        "id", 
        type=int, 
        help="ID of the task to mark completed (get ID from 'view' command)."
    )

    # --- 'delete' command ---
    delete_parser = subparsers.add_parser(
        "delete", 
        help="Delete a task.",
        description="Delete a specific task using its ID from the 'view' list."
    )
    delete_parser.add_argument(
        "id", 
        type=int, 
        help="ID of the task to delete (get ID from 'view' command)."
    )

    args = parser.parse_args()
    manager = TodoListManager(args.file)

    if args.command == "add":
        manager.add_task(args.description, args.due, args.priority)
    elif args.command == "view":
        manager.view_tasks(args.status, args.sort)
    elif args.command == "complete":
        manager.mark_task_completed(args.id)
    elif args.command == "delete":
        manager.delete_task(args.id)
    else:
        # If no command is provided, or an invalid one, show help and default view
        parser.print_help()
        print("\n--- Default View (All Tasks) ---")
        manager.view_tasks() # Display all tasks by default if no specific command

if __name__ == "__main__":
    main()
