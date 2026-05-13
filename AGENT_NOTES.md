# UnrealMCP — Agent Notes

This plugin runs a TCP JSON server inside the Unreal Editor at **127.0.0.1:55557**.
It starts automatically when the editor opens (via `UUnrealMCPBridge` EditorSubsystem).

Agents can connect and drive the editor programmatically without human interaction.

---

## Protocol

```json
// Request
{"type": "command_name", "params": {...}}

// Response
{"status": "success", "result": {...}}
{"status": "error",   "error":  "..."}
```

Messages are newline-terminated. Connection is persistent.

---

## Commands

### execute_python ⭐ most powerful
Run arbitrary Unreal Python on the game thread. Use this for anything not covered
by the other commands — asset editing, data asset population, batch operations, etc.

```json
{"type": "execute_python", "params": {"script": "import unreal\nunreal.log('hello')"}}
```

### Editor / Level
| command | key params | notes |
|---|---|---|
| `get_actors_in_level` | — | list all actors |
| `find_actors_by_name` | `pattern` | substring search |
| `spawn_actor` | `type`, `name`, `location`, `rotation`, `scale` | types: StaticMeshActor, PointLight, SpotLight, DirectionalLight, CameraActor |
| `delete_actor` | `name` | |
| `set_actor_transform` | `name`, `location`/`rotation`/`scale` | |
| `get_actor_properties` | `name` | |
| `set_actor_property` | `name`, `property`, `value` | |
| `spawn_blueprint_actor` | `blueprint_path`, `name`, `location`/`rotation`/`scale` | |
| `focus_viewport` | `actor_name` or `location` | |
| `take_screenshot` | `filepath` | saves PNG |

### Blueprint Authoring
| command | key params |
|---|---|
| `create_blueprint` | `name`, `parent_class` |
| `add_component_to_blueprint` | `blueprint_name`, `component_type`, `component_name` |
| `set_component_property` | `blueprint_name`, `component_name`, `property`, `value` |
| `set_static_mesh_properties` | `blueprint_name`, `component_name`, `mesh_path`, `material_path` |
| `set_physics_properties` | `blueprint_name`, `component_name`, `simulate_physics`, `mass`, `damping` |
| `set_blueprint_property` | `blueprint_name`, `property`, `value` |
| `set_pawn_properties` | `blueprint_name`, ... |
| `compile_blueprint` | `blueprint_name` |

### Blueprint Graph
| command | key params |
|---|---|
| `add_blueprint_event_node` | `blueprint_name`, `event_name` |
| `add_blueprint_function_node` | `blueprint_name`, `target`, `function_name`, `params` |
| `connect_blueprint_nodes` | `blueprint_name`, `source_node`, `source_pin`, `target_node`, `target_pin` |
| `add_blueprint_variable` | `blueprint_name`, `variable_name`, `variable_type` |
| `add_blueprint_input_action_node` | `blueprint_name`, `action_name` |
| `find_blueprint_nodes` | `blueprint_name`, `node_type` |

### UMG Widgets
| command | key params |
|---|---|
| `create_umg_widget_blueprint` | `name` |
| `add_text_block_to_widget` | `widget_name`, `text`, `position` |
| `add_button_to_widget` | `widget_name`, `text`, `position` |
| `bind_widget_event` | `widget_name`, `widget_component`, `event_name` |
| `set_text_block_binding` | `widget_name`, `text_block_name`, `function_name` |

### Project
| command | key params |
|---|---|
| `create_input_mapping` | `action_name`, `key`, `shift`/`ctrl`/`alt` |

---

## Useful Procedures — Update This Section

### Populate a UDataAsset via MCP
Use `execute_python` with the `unreal` module. Example: editing a `UServeGISRoadStyleTable`:
```python
import unreal
table = unreal.EditorAssetLibrary.load_asset("/MyPlugin/Data/MyTable")
entries = []
e = unreal.ServeGISRoadStyleEntry()
e.set_editor_property("tag_value", "residential")
e.set_editor_property("two_way_preset", unreal.load_class(None, "/Path/To/Preset_C"))
entries.append(e)
table.set_editor_property("entries", entries)
unreal.EditorAssetLibrary.save_asset("/MyPlugin/Data/MyTable")
```
See `ServeGISTools/Scripts/mcp_setup_road_style_table.py` for a full working example.

### Hot-reload after C++ rebuild
After rebuilding the UnrealMCP plugin, the editor must reload it before new commands
are available. Either restart the editor or use Edit → Plugins to disable/re-enable UnrealMCP.

---

*Update this file whenever you discover a new useful procedure or capability.*
