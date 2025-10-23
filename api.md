```gdscript
extends Node2D

var swf90
var current_anim  # 当前播放的动画对象
var idle_anim     # 待机动画对象
var attack_anim   # 物理动画对象

func _ready():
    # 初始化SWF
    swf90 = godot_swf_gds_bridge.new()
    swf90.load_swf("res://90.swf")
    add_child(swf90)
    
    # 创建待机动画（循环播放）
    idle_anim = swf90.create_anim(10)
    idle_anim.set_anim_is_looping(true)
    
    # 创建物理动画（播放一次）
    attack_anim = swf90.create_anim(100)
    attack_anim.set_anim_is_looping(false)
    
    # 连接物理动画结束信号
    if attack_anim.has_signal("animation_finished"):
        attack_anim.animation_finished.connect(_on_attack_finished)
    
    # 开始播放待机动画
    play_idle()

func play_idle():
    """播放待机动画"""
    if current_anim and current_anim != idle_anim:
        current_anim.stop()
    
    current_anim = idle_anim
    idle_anim.play()
    print("开始播放待机动画")

func play_attack():
    """播放物理动画"""
    if current_anim:
        current_anim.stop()
    
    current_anim = attack_anim
    attack_anim.play()
    print("开始播放物理动画")

func _on_attack_finished():
    """物理动画播放完毕，切换回待机"""
    print("物理动画结束，切换回待机")
    play_idle()

# 示例：触发物理动画的函数
func trigger_attack():
    """外部调用这个函数来触发物理动画"""
    play_attack()
```