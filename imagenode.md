```
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .run();
}

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    // 创建2D相机
    commands.spawn(Camera2d);
    
    // 创建ImageNode显示图像
    commands.spawn(ImageNode::new(asset_server.load("my_image.png")));
}
```