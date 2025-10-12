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


## 大小设定
```rust
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
    commands.spawn((
        ImageNode::new(asset_server.load("textbox.png")),
        Node {
            // 只设置宽度，高度自动调整保持比例
            width: px(1920),
            position_type: PositionType::Absolute,
            left: px(0),
            bottom: px(0),
            
            ..default()
        },
    ));
}
```