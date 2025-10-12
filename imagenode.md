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

## 工具栏测试
use bevy::prelude::*;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, handle_button_hover)
        .run();
}

// 工具栏按钮类型
#[derive(Component, Clone, Copy)]
enum ToolbarButton {
    Rollback,
    History,
    Skip,
    Auto,
    Save,
    Load,
    Settings,
}

// 工具栏容器标记
#[derive(Component)]
struct ToolbarContainer;

// 按钮图标组件
#[derive(Component)]
struct ButtonIcon {
    idle_texture: Handle<Image>,
    hover_texture: Handle<Image>,
}

// 按钮图标标记组件
#[derive(Component)]
struct ButtonIconImage;

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    // 创建2D相机
    commands.spawn(Camera2d);
    
    // 名字框容器
    let namebox_container = commands.spawn((
        Name::new("namebox_container"),
        Node {
            position_type: PositionType::Absolute,
            bottom: Val::Px(450.0),
            left: Val::Px(50.0),
            width: Val::Px(450.0),
            height: Val::Px(70.0),
            display: Display::Flex,
            align_items: AlignItems::Center,
            justify_content: JustifyContent::Center,
            ..default()
        },
        GlobalZIndex(2),
    )).id();
    
    // 背景图
    let namebox_bg = commands.spawn((
        ImageNode::new(asset_server.load("namebox.png")),
        Node {
            position_type: PositionType::Absolute,
            width: Val::Percent(100.0),
            height: Val::Percent(100.0),
            ..default()
        }
    )).id();
    
    // 文字
    let namebox_text = commands.spawn((
        Text::new("戴安娜"),
        TextFont {
            font: asset_server.load("font.ttf"),
            font_size: 27.0,
            ..default()
        },
        TextLayout::new_with_justify(Justify::Center),
        Node {
            margin: UiRect {
                left: Val::Px(120.0),
                right: Val::Px(0.0),
                top: Val::Px(12.0),
                bottom: Val::Px(0.0),
            },
            ..default()
        }
    )).id();
    
    // 建立父子关系
    commands.entity(namebox_container).add_children(&[namebox_bg, namebox_text]);

    // 文本框背景
    commands.spawn((
        ImageNode::new(asset_server.load("textbox.png")),
        Node {
            width: Val::Percent(100.0),
            position_type: PositionType::Absolute,
            left: Val::Px(0.0),
            bottom: Val::Px(0.0),
            ..default()
        },
        GlobalZIndex(1),
    ));

    // 工具栏
    setup_toolbar(&mut commands, &asset_server);
}

fn setup_toolbar(commands: &mut Commands, asset_server: &AssetServer) {
    // 工具栏容器
    let toolbar_container = commands.spawn((
        Name::new("toolbar_container"),
        Node {
            width: Val::Percent(100.0),
            height: Val::Px(80.0),
            position_type: PositionType::Absolute,
            bottom: Val::Px(1.0),
            left: Val::Px(0.0),
            justify_content: JustifyContent::Center,
            align_items: AlignItems::End,
            padding: UiRect::all(Val::Px(5.0)),
            margin: UiRect::all(Val::Px(0.0)),
            border: UiRect::all(Val::Px(0.0)),
            flex_direction: FlexDirection::Row,
            ..default()
        },
        BackgroundColor(Color::NONE),
        ToolbarContainer,
        GlobalZIndex(3),
    )).id();

    let button_style = Node {
        width: Val::Px(70.0),
        height: Val::Px(70.0),
        margin: UiRect::horizontal(Val::Px(5.0)),
        justify_content: JustifyContent::Center,
        align_items: AlignItems::Center,
        padding: UiRect::all(Val::Px(0.0)), // 移除内边距
        ..default()
    };

    // 按钮配置：只需要类型和图标编号，不需要文字
    let button_configs = [
        (ToolbarButton::Rollback, 12),
        (ToolbarButton::History, 13),
        (ToolbarButton::Skip, 14),
        (ToolbarButton::Auto, 15),
        (ToolbarButton::Save, 16),
        (ToolbarButton::Load, 17),
        (ToolbarButton::Settings, 18),
    ];

    let mut button_entities = Vec::new();

    for (button_type, icon_number) in button_configs.iter() {
        let idle_path = format!("{}.png", icon_number);
        let hover_path = format!("{}1.png", icon_number);
        
        let button_entity = create_icon_button(
            commands,
            *button_type,
            button_style.clone(),
            asset_server.load(&idle_path),
            asset_server.load(&hover_path),
        );
        
        button_entities.push(button_entity);
    }

    // 将所有按钮添加到工具栏容器
    commands.entity(toolbar_container).add_children(&button_entities);
}

fn create_icon_button(
    commands: &mut Commands,
    button_type: ToolbarButton,
    button_style: Node,
    idle_icon: Handle<Image>,
    hover_icon: Handle<Image>,
) -> Entity {
    // 创建透明按钮
    let button = commands.spawn((
        Button,
        button_style,
        BackgroundColor(Color::NONE), // 完全透明的背景
        BorderRadius::all(Val::Px(0.0)), // 移除圆角
        button_type,
        ButtonIcon {
            idle_texture: idle_icon.clone(),
            hover_texture: hover_icon,
        },
    )).id();

    // 创建图标，填满整个按钮区域
    let icon = commands.spawn((
        ImageNode::new(idle_icon),
        Node {
            width: Val::Px(70.0),  // 图标大小等于按钮大小
            height: Val::Px(70.0),
            ..default()
        },
        ButtonIconImage,
    )).id();

    // 只将图标添加到按钮
    commands.entity(button).add_children(&[icon]);

    button
}

// 处理按钮悬停状态的系统
fn handle_button_hover(
    mut interaction_query: Query<
        (&Interaction, &ButtonIcon, &Children),
        (Changed<Interaction>, With<Button>),
    >,
    mut image_query: Query<&mut ImageNode, With<ButtonIconImage>>,
) {
    for (interaction, button_icon, children) in interaction_query.iter_mut() {
        // 直接查找带有 ButtonIconImage 标记的子元素
        for child in children.iter() {
            if let Ok(mut image) = image_query.get_mut(child) {
                match interaction {
                    Interaction::Hovered => {
                        image.image = button_icon.hover_texture.clone();
                    }
                    Interaction::None => {
                        image.image = button_icon.idle_texture.clone();
                    }
                    Interaction::Pressed => {
                        image.image = button_icon.hover_texture.clone();
                    }
                }
                break;
            }
        }
    }
}
