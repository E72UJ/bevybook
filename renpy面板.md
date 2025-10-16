```rust
use bevy::prelude::*;

// Ren'Py风格的虚拟分辨率配置
const VIRTUAL_WIDTH: f32 = 1920.0;
const VIRTUAL_HEIGHT: f32 = 1080.0;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, update_virtual_scale)
        .run();
}

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(Camera2d);
    
    // 虚拟屏幕容器 - 就像Ren'Py的screen
    commands.spawn((
        Node {
            position_type: PositionType::Absolute,
            width: Val::Percent(100.0),
            height: Val::Percent(100.0),
            justify_content: JustifyContent::Center,
            align_items: AlignItems::Center,
            ..default()
        },
        BackgroundColor(Color::BLACK),
    )).with_children(|parent| {
        
        // 虚拟分辨率画布 - 这就是你的"设计画布"
        parent.spawn((
            Node {
                // 永远按虚拟分辨率设计！
                width: Val::Px(VIRTUAL_WIDTH),
                height: Val::Px(VIRTUAL_HEIGHT),
                position_type: PositionType::Relative,
                ..default()
            },
            BackgroundColor(Color::srgb(0.1, 0.1, 0.2)),
            VirtualScreen,
            Transform::from_scale(Vec3::ONE),
        )).with_children(|parent| {
            
            // 对话框 - 就像Ren'Py的textbox
            parent.spawn((
                Node {
                    position_type: PositionType::Absolute,
                    left: Val::Px(100.0),        // 在1920宽度中的位置
                    bottom: Val::Px(50.0),       // 在1080高度中的位置
                    width: Val::Px(1720.0),      // 在1920宽度中的大小
                    height: Val::Px(200.0),      // 在1080高度中的大小
                    padding: UiRect::all(Val::Px(30.0)),
                    justify_content: JustifyContent::FlexStart,
                    align_items: AlignItems::FlexEnd,
                    ..default()
                },
                BackgroundColor(Color::srgba(0.0, 0.0, 0.0, 0.8)),
            )).with_children(|parent| {
                parent.spawn((
                    Text::new("这是对话文本，就像Ren'Py中的对话框一样。\n无论窗口大小如何变化，它都会保持相同的相对位置和比例。"),
                    TextFont {
                        font_size: 36.0,     // 在1080高度中的字体大小
                        ..default()
                    },
                    TextColor(Color::WHITE),
                ));
            });
            
            // 角色立绘 - 就像Ren'Py的sprite
            parent.spawn((
                Node {
                    position_type: PositionType::Absolute,
                    right: Val::Px(200.0),       // 在1920宽度中的位置
                    bottom: Val::Px(250.0),      // 在1080高度中的位置  
                    width: Val::Px(400.0),       // 在1920宽度中的大小
                    height: Val::Px(600.0),      // 在1080高度中的大小
                    ..default()
                },
                BackgroundColor(Color::srgb(0.8, 0.6, 0.9)), // 演示用颜色
            ));
            
            // 选择按钮 - 就像Ren'Py的menu选项
            let choices = vec!["选择项 1", "选择项 2", "选择项 3"];
            for (i, choice_text) in choices.iter().enumerate() {
                parent.spawn((
                    Button,
                    Node {
                        position_type: PositionType::Absolute,
                        right: Val::Px(100.0),
                        top: Val::Px(200.0 + i as f32 * 80.0), // 在1080高度中垂直排列
                        width: Val::Px(400.0),       // 在1920宽度中的大小
                        height: Val::Px(60.0),       // 在1080高度中的大小
                        justify_content: JustifyContent::Center,
                        align_items: AlignItems::Center,
                        ..default()
                    },
                    BackgroundColor(Color::srgba(0.2, 0.4, 0.8, 0.9)),
                )).with_children(|parent| {
                    parent.spawn((
                        Text::new(*choice_text),
                        TextFont {
                            font_size: 32.0, // 在1080高度中的字体大小
                            ..default()
                        },
                        TextColor(Color::WHITE),
                    ));
                });
            }
            
            // UI面板 - 就像Ren'Py的screen元素
            parent.spawn((
                Node {
                    position_type: PositionType::Absolute,
                    left: Val::Px(50.0),         // 在1920宽度中的位置
                    top: Val::Px(50.0),          // 在1080高度中的位置
                    width: Val::Px(300.0),       // 在1920宽度中的大小
                    height: Val::Px(150.0),      // 在1080高度中的大小
                    flex_direction: FlexDirection::Column,
                    justify_content: JustifyContent::SpaceAround,
                    align_items: AlignItems::Center,
                    padding: UiRect::all(Val::Px(20.0)),
                    ..default()
                },
                BackgroundColor(Color::srgba(0.1, 0.3, 0.1, 0.9)),
            )).with_children(|parent| {
                parent.spawn((
                    Text::new("状态面板"),
                    TextFont {
                        font_size: 28.0,     // 在1080高度中的字体大小
                        ..default()
                    },
                    TextColor(Color::srgb(0.5, 1.0, 0.5)), // 改为自定义的亮绿色
                ));
                parent.spawn((
                    Text::new("生命值: 100/100"),
                    TextFont {
                        font_size: 24.0,
                        ..default()
                    },
                    TextColor(Color::WHITE),
                ));
            });
        });
    });
}

#[derive(Component)]
struct VirtualScreen;

fn update_virtual_scale(
    mut virtual_screen_query: Query<&mut Transform, With<VirtualScreen>>,
    window_query: Query<&Window>,
) {
    if let Ok(window) = window_query.single() {
        if let Ok(mut transform) = virtual_screen_query.single_mut() {
            let window_width = window.resolution.width();
            let window_height = window.resolution.height();
            
            // 计算缩放比例 - 就像Ren'Py一样
            let scale_x = window_width / VIRTUAL_WIDTH;
            let scale_y = window_height / VIRTUAL_HEIGHT;
            
            // 可以选择等比缩放或拉伸填充
            let scale = scale_x.min(scale_y); // 等比缩放，保持宽高比
            // let scale = scale_x; // 如果想要拉伸填充，可以分别设置scale_x和scale_y
            
            transform.scale = Vec3::splat(scale);
            
            println!("虚拟分辨率: {}x{} -> 实际窗口: {}x{}, 缩放: {:.2}x", 
                VIRTUAL_WIDTH as i32, VIRTUAL_HEIGHT as i32,
                window_width as i32, window_height as i32, scale);
        }
    }
}
```