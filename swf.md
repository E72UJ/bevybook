
```python
extends Node2D
var swf: godot_swf_gds_bridge
var aid:int
var aid165 :int
var aid256 :int
var is256: bool
func _ready() -> void:
	swf = godot_swf_gds_bridge.new()
	swf.class_init()
	swf.load_swf("res://90.swf")
	swf.parse()
	add_child(swf)
	swf.scale /= 20.0
	swf.animation_finished.connect(动画结束)
	aid165 = swf.play(165)
func 动画结束(id:int):
	print(id,"动画结束")
	play(1)
func play(cid:int):
	if !(is256):
		aid256 = swf.play(256)
		swf.stop(aid165)
		is256 = true
	else:
		aid165 = swf.play(165)
		swf.stop(aid256)
		is256 = false
```