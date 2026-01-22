---
tags: [schema, chatbotrpg, implementation, save-file, persistence]
created: 2026-01-20
---

# Save File Format (Game State Serialization)

**Primary Files**:
- `src/core/utils.py` (save_game_state, lines 19-156)
- `src/core/utils.py` (load_game_state, lines 158-512)

**Storage Location**: `{workflow_dir}/saves/{save_name}/`

**Format**: Directory-based save containing full game state snapshot

## Purpose

ChatBotRPG uses a directory-based save system that captures the entire runtime game state by copying the `game/` directory to a timestamped save directory.

## Save Directory Structure

```
{workflow_dir}/
├── game/                    # Active game state
│   ├── variables.json       # Global variables
│   ├── actors/              # Runtime actors
│   │   ├── player.json
│   │   ├── innkeeper.json
│   │   └── guard_captain.json
│   ├── settings/            # Runtime settings
│   │   └── world/region/location/
│   │       ├── tavern_setting.json
│   │       └── market_square_setting.json
│   └── timers/              # Timer state
│       └── timer_state.json
└── saves/                   # Save games
    ├── Before Quest/        # Save directory
    │   ├── variables.json
    │   ├── actors/
    │   ├── settings/
    │   └── timers/
    └── After Battle/
        ├── variables.json
        ├── actors/
        ├── settings/
        └── timers/
```

## Save Process

### User-Initiated Save

**Function**: `save_game_state(self)` (utils.py:19-156)

**Flow**:

1. **Get current workflow tab** (lines 20-28)
```python
tab_data = self.get_current_tab_data()
if not tab_data:
    QMessageBox.warning(self, "No Workflow Active", "Please select a workflow tab to save its state.")
    return

tab_name = tab_data.get('name', f"Tab {self.current_tab_index + 1}")
tab_dir = os.path.dirname(tab_data.get('tab_settings_file', ''))
```

2. **Verify game directory exists** (lines 29-33)
```python
game_dir = os.path.join(tab_dir, "game")
saves_dir = os.path.join(tab_dir, "saves")

if not os.path.isdir(game_dir):
    QMessageBox.warning(self, "Nothing to Save", f"The 'game' directory for workflow '{tab_name}' does not exist.")
    return
```

3. **Prompt for save name** (lines 34-123)
```python
dialog = QDialog(self)
# ... frameless themed dialog ...
save_name_input = QLineEdit()
save_name_input.setPlaceholderText("Save name...")
# ... dialog execution ...

save_name = result[1]
sanitized_save_name = sanitize_folder_name(save_name)
save_dest_path = os.path.join(saves_dir, sanitized_save_name)
```

4. **Check for overwrite** (lines 132-141)
```python
if os.path.exists(save_dest_path):
    reply = QMessageBox.question(
        self, "Overwrite Save?",
        f"A save named '{sanitized_save_name}' already exists. Overwrite?",
        QMessageBox.Yes | QMessageBox.No, QMessageBox.No
    )
    if reply == QMessageBox.Yes:
        overwrite = True
    else:
        return
```

5. **Save timer state** (lines 145-146)
```python
if hasattr(self, 'timer_manager'):
    self.timer_manager.save_timer_state(tab_data)
```

6. **Copy game directory to save** (line 147)
```python
shutil.copytree(game_dir, save_dest_path)
```

### What Gets Saved

The entire `game/` directory is copied, including:

- **variables.json**: All global variables
- **actors/*.json**: All runtime actors with current state
  - Player position, health, inventory, equipment
  - NPC locations, relations, variables
- **settings/**/*.json**: All runtime settings
  - Setting states, variables, character lists
  - Connection data
- **timers/timer_state.json**: Active timer states
  - Running timers, intervals, next fire times

### Save File Naming

```python
def sanitize_folder_name(name):
    sanitized = re.sub(r'[^a-zA-Z0-9\- ]', '', name).strip()
    return sanitized or 'Workflow'
```

**Examples**:
- "Before Quest" → `Before Quest/`
- "Chapter 3 - After Battle" → `Chapter 3  After Battle/`
- "Save #1" → `Save 1/`

## Load Process

### User-Initiated Load

**Function**: `load_game_state(self)` (utils.py:158-512)

**Flow**:

1. **Get current workflow tab** (lines 159-167)
```python
tab_data = self.get_current_tab_data()
if not tab_data:
    QMessageBox.warning(self, "No Workflow Active", "Please select a workflow tab to load a state into.")
    return

tab_dir = os.path.dirname(tab_data.get('tab_settings_file', ''))
game_dir = os.path.join(tab_dir, "game")
saves_dir = os.path.join(tab_dir, "saves")
```

2. **List available saves** (lines 173-180)
```python
try:
    available_saves = [d for d in os.listdir(saves_dir) if os.path.isdir(os.path.join(saves_dir, d))]
except OSError as e:
    QMessageBox.critical(self, "Load Error", f"Could not read available saves.\nError: {e}")
    return

if not available_saves:
    QMessageBox.information(self, "No Saves Found", f"No saved states found in '{saves_dir}'.")
    return
```

3. **Display save selection dialog** (lines 181-341)
```python
dialog = QDialog(self)
# ... themed frameless dialog ...
saves_list = QListWidget()
for save in available_saves:
    item = QListWidgetItem(save)
    saves_list.addItem(item)
# ... delete button support ...
```

4. **Clear current tab state** (lines 348-376)
```python
if tab_data:
    # Clear memory
    if 'memory' in tab_data and tab_data['memory']:
        old_memory = tab_data['memory']
        tab_data['memory'] = None
        del old_memory

    # Clear output widget
    output_widget = tab_data.get('output')
    if output_widget:
        output_widget.clear_messages()

    # Clear context
    tab_data['context'] = []
    tab_data['_remembered_selected_message'] = None

    # Clear system editor
    system_editor = tab_data.get('system_context_editor')
    if system_editor:
        system_editor.clear()

gc.collect()
```

5. **Backup old game files** (lines 379-404)
```python
renamed_files = []
if os.path.exists(game_dir):
    timestamp = datetime.now().strftime("%Y%m%d%H%M%S%f")
    for item_name in os.listdir(game_dir):
        item_path = os.path.join(game_dir, item_name)
        if os.path.isfile(item_path):
            base, ext = os.path.splitext(item_name)
            new_name = f"{base}{ext}_old_{timestamp}"
            new_path = os.path.join(game_dir, new_name)
            os.rename(item_path, new_path)
            renamed_files.append(new_path)
```

6. **Copy save files to game directory** (lines 406-446)
```python
copied_files = []
if not os.path.exists(game_dir):
    os.makedirs(game_dir)

for item_name in os.listdir(save_src_path):
    source_item_path = os.path.join(save_src_path, item_name)
    dest_item_path = os.path.join(game_dir, item_name)

    if os.path.isfile(source_item_path):
        shutil.copy2(source_item_path, dest_item_path)
        copied_files.append(dest_item_path)
    elif os.path.isdir(source_item_path):
        if os.path.exists(dest_item_path):
            shutil.rmtree(dest_item_path)
        shutil.copytree(source_item_path, dest_item_path)
```

7. **Reload game state** (lines 447-470)
```python
# Load conversation history
self.load_conversation_for_tab(self.current_tab_index)

# Load variables
self._load_variables(self.current_tab_index)

# Reload system context
system_context_content = self.get_system_context(self.current_tab_index)
if tab_data and system_context_content is not None:
    system_editor = tab_data.get('system_context_editor')
    if system_editor:
        system_editor.setPlainText(system_context_content)

# Reload timers
if hasattr(self, 'timer_manager'):
    self.timer_manager.stop_all_timers()
    self.timer_manager.load_timer_state(tab_data)

# Clear actor cache
if hasattr(self, '_actor_name_to_file_cache'):
    self._actor_name_to_file_cache.clear()
```

8. **Update UI** (lines 471-503)
```python
# Check if game has content
loaded_context_check = tab_data.get('context', [])
top_splitter = tab_data.get('top_splitter')

if loaded_context_check:
    if tab_data.get('input'):
        tab_data['input'].set_input_state('normal')
    if top_splitter:
        top_splitter.setVisible(True)
else:
    if top_splitter:
        top_splitter.setVisible(False)

# Update right splitter (player panel)
right_splitter = tab_data.get('right_splitter')
workflow_data_dir = tab_data.get('workflow_data_dir')
if right_splitter and workflow_data_dir:
    current_setting_name = _get_player_current_setting_name(workflow_data_dir)
    right_splitter.update_setting_name(current_setting_name, workflow_data_dir)
    right_splitter.load_character_data()
    right_splitter.update_game_time()
```

9. **Schedule old file cleanup** (lines 482-486)
```python
if renamed_files:
    if not hasattr(self, 'files_to_delete_on_exit'):
        self.files_to_delete_on_exit = []
    self.files_to_delete_on_exit.extend(renamed_files)
    _cleanup_old_backup_files_in_directory(game_dir)
```

### What Gets Loaded

The entire save directory is copied back to `game/`, restoring:

- All global variables
- All actor states (position, inventory, health, etc.)
- All setting states (variables, character lists)
- All timer states
- Conversation history
- System context

### Rollback on Error

If load fails, the system attempts to restore the previous state (lines 504-512):

```python
except Exception as e:
    QMessageBox.critical(self, "Load Error", f"Could not complete load operation.\nError: {e}\n\nAttempting to restore clean state.")
    try:
        if os.path.exists(game_dir):
            shutil.rmtree(game_dir)
        self.load_conversation_for_tab(self.current_tab_index)
        self._load_variables(self.current_tab_index)
    except Exception as restore_err:
        QMessageBox.critical(self, "Restore Error", f"Failed to restore tab to empty state after load error.\nPlease restart the application.\nError: {restore_err}")
```

## Timer State Persistence

**File**: `{workflow_dir}/game/timers/timer_state.json`

**Format**:
```json
{
  "timers": [
    {
      "rule_id": "hourly_announcement",
      "character": "Town Crier",
      "is_running": true,
      "interval_ms": 3600000,
      "start_time": "2024-01-20T15:30:00.000000",
      "next_fire_time": "2024-01-20T16:30:00.000000"
    }
  ]
}
```

**Saved**: Automatically during `save_game_state()` via `timer_manager.save_timer_state()`

**Loaded**: Automatically during `load_game_state()` via `timer_manager.load_timer_state()`

## Conversation History Persistence

**File**: `{workflow_dir}/game/conversation_history.json`

**Format**:
```json
[
  {
    "role": "system",
    "content": "You are a narrator in a fantasy RPG...",
    "timestamp": "2024-01-20T15:00:00Z"
  },
  {
    "role": "user",
    "content": "I enter the tavern.",
    "metadata": {
      "character": "Player"
    },
    "timestamp": "2024-01-20T15:05:00Z"
  },
  {
    "role": "assistant",
    "content": "The warm glow of the fireplace greets you as you step inside...",
    "metadata": {
      "character": "Narrator"
    },
    "timestamp": "2024-01-20T15:05:15Z"
  }
]
```

**Saved**: Part of standard save process (copied with game/ directory)

**Loaded**: Via `load_conversation_for_tab()` during load process

## Backup File Cleanup

Old backup files are automatically cleaned up (utils.py:1379-1402):

```python
def _cleanup_old_backup_files_in_directory(directory):
    cutoff_time = datetime.now() - timedelta(minutes=10)
    files_cleaned = 0

    for filename in os.listdir(directory):
        match = re.search(r'_old_(\d{20})$', filename)
        if match:
            timestamp_str = match.group(1)
            file_time = datetime.strptime(timestamp_str, '%Y%m%d%H%M%S%f')

            if file_time < cutoff_time:
                file_path = os.path.join(directory, filename)
                os.remove(file_path)
                files_cleaned += 1
```

**Backup File Pattern**: `{filename}_old_{YYYYMMDDHHMMSSSSSSSS}`

**Cleanup Trigger**: Files older than 10 minutes

## Save/Load Guarantees

### Atomicity

- Save creates complete copy of game/ directory
- Load replaces game/ directory entirely
- Old files are renamed (not deleted) until load succeeds

### Data Integrity

- All JSON files validated on load
- Rollback to previous state if load fails
- Actor cache cleared after load to prevent stale data

### State Consistency

- Timers paused during save/load
- UI state updated after load
- Character locations validated
- Setting character lists synchronized

## Delete Save

**Location**: Load dialog (utils.py:311-333)

**Process**:
```python
def on_delete():
    current_item = saves_list.currentItem()
    if not current_item:
        return

    save_name = current_item.text()
    save_path = os.path.join(saves_dir, save_name)

    if os.path.exists(save_path):
        shutil.rmtree(save_path)  # Delete entire save directory

    # Remove from list
    row = saves_list.row(current_item)
    saves_list.takeItem(row)

    # Close dialog if no saves left
    if saves_list.count() == 0:
        dialog.reject()
```

## Cross-References

- **Used By**: [[Game Engine]], [[Main UI]]
- **Contains**: [[Actor Schema]], [[Setting Schema]], [[Variables Schema]], [[Timer Schema]]
- **Validates**: Discord claims about persistence and save system
- **Related Patterns**: [[Three-Tier Persistence Pattern]], [[State Management]]

## Notes

- Save is a complete snapshot - no incremental or delta saves
- Save directory names are sanitized (alphanumeric, dash, space only)
- Old backup files (_old_*) are cleaned up after 10 minutes
- Load process is transactional with rollback on failure
- Conversation history is preserved in saves
- Timer states are persisted and restored
- Actor cache must be cleared after load to prevent stale references
- UI state (visibility, enabled/disabled) updated based on loaded state
