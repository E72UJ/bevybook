```csharp
using Godot;
using System;
using GodotSWF;
public partial class playbox : Node2D
{
	public SWF Swf;
	public override void _Ready()
	{
		Swf = new SWF("res://swf/90.swf", this);
		AddChild(Swf);
		Swf.Parse();
		Swf.Play(234);
		GD.Print("Hello Godot swf！");
	}
}
```

## 注意，默认的生成头部分需要手动修改成node2d，否则无法运行。