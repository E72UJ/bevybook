```GDScript
extends Node2D

class_name godot_swf_gds_bridge
var cs
var swf_node

signal animation_finished(id : int)

func class_init() -> void:
	cs = load("res://Godot-SWF/bridge/GodotSwfNetBridge.cs")
	swf_node = cs.new()
	swf_node.AnimationFinished.connect(func(id : int):
		animation_finished.emit(id)
		)

func load_swf(path : String) -> void:
	
	swf_node.Load(path, self)
	add_child(swf_node)
	print(path)

func parse() -> void:
	swf_node.Parse()

func play(id : int) -> int:
	var i : int = swf_node.Play(id);
	print(i)
	return i
	
func stop(id : int) -> void:
	swf_node.Stop(id)

func stop_all() -> void:
	swf_node.StopAll()

func pause(id : int) -> void:
	swf_node.Pause(id)

func pause_and_play_children(id : int) -> void:
	swf_node.PauseAndPlayChildren(id)

func continue_anim(id : int) -> void:
	swf_node.Continue(id)

func set_anim_transform(id : int, transform : Transform2D) -> void:
	swf_node.SetTransform(id, transform)

func set_anim_color(id : int, color : Color) -> void:
	swf_node.SetColor(id, color)

func set_anim_visible(id : int, visible : bool) -> void:
	swf_node.SetVisible(id, visible)

func is_finished(id : int) -> bool:
	return swf_node.IsFinished(id)
```