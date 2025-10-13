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


## 测试留存
``` rust
use bevy::prelude::*;

#[derive(Component)]
struct TextboxBackground;

#[derive(Component)]
struct SidebarLayer;

#[derive(Component)]
struct CharacterImage;

#[derive(Component)]
struct SizeDisplay;

#[derive(Component)]
struct EntityDisplay;

fn setup(mut commands: Commands, asset_server: Res<AssetServer>) {
    // 添加摄像机
    commands.spawn(Camera2d);
    
    // 创建textbox UI层（底层）
    commands.spawn((
        Node {
            width: Val::Percent(100.0),   
            height: Val::Percent(25.0),   
            position_type: PositionType::Absolute,
            left: Val::Px(0.0),
            bottom: Val::Px(0.0),
            ..default()
        },
        BackgroundColor(Color::srgba(0.2, 0.2, 0.2, 0.9)),
        ZIndex(1),
        Name::new("TextboxUI"),
        TextboxBackground,
    )).with_children(|parent| {
        // 添加textbox背景图片
        parent.spawn((
            ImageNode::new(asset_server.load("textbox.png")),
            Node {
                width: Val::Percent(100.0),
                height: Val::Percent(100.0),
                ..default()
            },
            Name::new("TextboxBackground"),
        ));
        
        // 添加大小显示文本
        parent.spawn((
            Text::new("加载中..."),
            TextFont {
                font_size: 24.0,
                ..default()
            },
            TextColor(Color::WHITE),
            Node {
                position_type: PositionType::Absolute,
                left: Val::Px(20.0),
                top: Val::Px(20.0),
                ..default()
            },
            Name::new("SizeDisplayText"),
            SizeDisplay,
        ));
        
        // 添加实体信息显示文本
        parent.spawn((
            Text::new("实体信息加载中..."),
            TextFont {
                font_size: 20.0,
                ..default()
            },
            TextColor(Color::srgb(0.8, 1.0, 0.8)), // 淡绿色
            Node {
                position_type: PositionType::Absolute,
                left: Val::Px(20.0),
                top: Val::Px(120.0),
                ..default()
            },
            Name::new("EntityDisplayText"),
            EntityDisplay,
        ));
    });

    // 创建sidebar层（上层）- 容纳角色图片
    commands.spawn((
        Node {
            width: Val::Percent(100.0),
            height: Val::Percent(100.0),
            position_type: PositionType::Absolute,
            left: Val::Px(0.0),
            top: Val::Px(20.0),
            padding: UiRect::all(Val::Px(50.0)),
            ..default()
        },
        ZIndex(10),
        Name::new("SidebarLayer"),
        SidebarLayer,
    )).with_children(|parent| {
        // 添加你的机械天使角色图片
        parent.spawn((
            ImageNode::new(asset_server.load("sidebar.png")),
            Node {
                width: Val::Px(350.0),  
                height: Val::Px(450.0), 
                ..default()
            },
            Name::new("MechanicalAngel"),
            CharacterImage,
        ));
    });
}

fn update_size_display(
    mut text_query: Query<&mut Text, With<SizeDisplay>>,
    windows: Query<&Window>,
) {
    if let Ok(window) = windows.single() {
        if let Ok(mut text) = text_query.single_mut() {
            let textbox_width = window.width();
            let textbox_height = window.height() * 0.25;
            
            text.0 = format!(
                "窗口: {:.0}x{:.0}px\nTextbox: {:.0}x{:.0}px (25%高度)\n角色图片: 350x450px (右下角)", 
                window.width(), window.height(),
                textbox_width, textbox_height
            );
        }
    }
}

fn update_entity_display(
    mut entity_text_query: Query<&mut Text, With<EntityDisplay>>,
    all_entities: Query<(Entity, &Name)>,
    character_query: Query<&Node, With<CharacterImage>>,
) {
    if let Ok(mut text) = entity_text_query.single_mut() {
        let total_entities = all_entities.iter().count();
        
        // 获取角色图片的当前状态
        let character_status = if let Ok(node) = character_query.single() {
            match node.display {
                Display::None => "隐藏",
                _ => "显示",
            }
        } else {
            "未找到"
        };
        
        // 获取一些有趣的实体名称
        let mut entity_names = Vec::new();
        for (entity, name) in all_entities.iter().take(5) {
            entity_names.push(format!("{}: {}", entity.index(), name.as_str()));
        }
        
        text.0 = format!(
            "总实体数: {}\n机械天使: {}\n实体示例:\n{}", 
            total_entities,
            character_status,
            entity_names.join("\n")
        );
    }
}

fn on_window_resize(
    mut resize_events: EventReader<bevy::window::WindowResized>,
) {
    for event in resize_events.read() {
        let textbox_height = event.height * 0.25;
        
        println!("窗口改变: {:.0}x{:.0}px -> Textbox: {:.0}x{:.0}px", 
            event.width, event.height, event.width, textbox_height);
    }
}

fn character_interaction(
    mut character_query: Query<&mut Node, With<CharacterImage>>,
    input: Res<ButtonInput<KeyCode>>,
) {
    if input.just_pressed(KeyCode::Space) {
        if let Ok(mut node) = character_query.single_mut() {
            if node.width == Val::Px(350.0) {
                node.width = Val::Px(450.0);
                node.height = Val::Px(550.0);
                println!("机械天使放大！");
            } else {
                node.width = Val::Px(350.0);
                node.height = Val::Px(450.0);
                println!("机械天使缩小");
            }
        }
    }
    
    if input.just_pressed(KeyCode::KeyH) {
        if let Ok(mut node) = character_query.single_mut() {
            if node.display == Display::Flex {
                node.display = Display::None;
                println!("隐藏角色");
            } else {
                node.display = Display::Flex;
                println!("显示角色");
            }
        }
    }
}

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup)
        .add_systems(Update, (
            update_size_display,
            update_entity_display,
            on_window_resize,
            character_interaction,
        ))
        .run();
}
```