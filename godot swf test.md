```C#
using Godot;
using GodotSWF;
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Net.Security;
using System.Text;
using System.Threading.Tasks;

public partial class SwfPlayer : Node2D
{
	public SWF Swf;
	public Aid Aid184;
	public Aid Aid256;
	public Aid Aid274;
	public Aid Aid342;
	public Aid Aid528;
	public bool IsIdle;

	public List<SWF> SwfList = [];

	public override void _Process(double delta)
	{
		if (IsIdle)
		{

		}
	}

	public override void _Ready()
	{
		Swf = new SWF("res://swf/90.swf", this);

		for (int i = 50; i < 200; i++)
		{
			if (i % 3 == 0)
				continue;
			String swfName = "res://swf/" + i + ".swf";
			SWF swf = new SWF(swfName, this);
			GD.Print("第", i ,"个swf文件成功读取");

			SwfList.Add(swf);
			AddChild(swf);
			//swf.Parse();
			ParseSWFAsync(swf, i);
		}

		AddChild(Swf);
		// 使用Stopwatch来测量耗时
		Stopwatch stopwatch = new();

		// 开始计时
		stopwatch.Start();
		Swf.Parse();
		stopwatch.Stop();

		// 输出耗时
		GD.Print("解析耗时：" + stopwatch.Elapsed.TotalMilliseconds + "ms");
		
		Swf.AnimationFinished += _swf_AnimationFinished;
		//Swf.Pause(Aid);

	}
	private async void ParseSWFAsync(SWF swf, int value)
	{
		await Task.Run(swf.Parse);
		GD.Print("第" + value + "个swf 文件解析完成");
	}

	private void _swf_AnimationFinished(Aid aid)
	{
		//throw new System.NotImplementedException();
		GD.Print("动画结束：" + aid.Id);
		IsIdle = true;
	}

	public void Play()
	{
		Aid256 = Swf.Play(256, 2.0f);
		Swf.Play(184);
		Swf.Play(274);
		Swf.Play(342);
		Swf.Play(528);
	}

	public void PlayTo184()
	{
		Swf.StopAll();
		if (Aid184.Id == 0)
		{
			Aid184 = Swf.Play(184, playScale: 1.1f, transform: new(0.0f, new Vector2(3.0f, 3.0f), 0f, new Vector2(1.0f, 1.0f)));
		}
		else
		{
			Aid184 = Swf.Play(184, transform: new(0.0f, new Vector2(3.0f, 3.0f), 0f, new Vector2(1.0f, 1.0f)));
}
	}

	public void PlayTo256()
	{
		Swf.StopAll();
		if (Aid256.Id == 0)
		{
			Aid256 = Swf.Play(256);
		}
		else
		{
			//Swf.Continue(Aid256);
			Aid256 = Swf.Play(256);
		}
	}

	public void PlayTo274()
	{
		Swf.StopAll();
		if (Aid274.Id == 0)
		{
			Aid274 = Swf.Play(274);
		}
		else
		{
			//Swf.Continue(Aid274);
			Aid274 = Swf.Play(274);
		}
	}

	public void PlayTo342()
	{
		Swf.StopAll();
		if (Aid342.Id == 0)
		{
			Aid342 = Swf.Play(342);
		}
		else
		{
			//Swf.Continue(Aid342);
			Aid342 = Swf.Play(342);
		}
	}

	public void PlayTo528()
	{
		Swf.StopAll();
		if (Aid528.Id == 0)
		{
			Aid528 = Swf.Play(528);
		}
		else
		{
			//Swf.Continue(Aid528);
			Aid528 = Swf.Play(528);
		}
	}



}
```