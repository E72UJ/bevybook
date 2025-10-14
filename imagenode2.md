use bevy::prelude::*;
use bevy::asset::RenderAssetUsages;

fn main() {
    App::new()
        .add_plugins(DefaultPlugins)
        .add_systems(Startup, setup_two_images)
        .add_systems(Update, detailed_memory_monitor)
        .run();
}

fn setup_two_images(mut commands: Commands, asset_server: Res<AssetServer>) {
    commands.spawn(Camera2d);
    
    // 图像1 - 将会被优化
    let image_handle1 = asset_server.load("1.png");
    commands.spawn((
        ImageNode::new(image_handle1.clone()),
        Transform::from_translation(Vec3::new(-200.0, 0.0, 0.0)),
        Visibility::Visible,
        ImageInfo {
            handle: image_handle1,
            name: "1.png".to_string(),
            optimized: false,
        },
    ));
    
    // 图像2 - 将会被优化 
    let image_handle2 = asset_server.load("2.png");
    commands.spawn((
        ImageNode::new(image_handle2.clone()),
        Transform::from_translation(Vec3::new(200.0, 0.0, 0.0)),
        Visibility::Visible,
        ImageInfo {
            handle: image_handle2,
            name: "2.png".to_string(),
            optimized: false,
        },
    ));
}

#[derive(Component)]
struct ImageInfo {
    handle: Handle<Image>,
    name: String,
    optimized: bool,
}

fn detailed_memory_monitor(
    mut images: ResMut<Assets<Image>>,
    mut query: Query<&mut ImageInfo>,
    mut timer: Local<f32>,
    time: Res<Time>,
) {
    *timer += time.delta_secs();
    
    if *timer > 3.0 {
        *timer = 0.0;
        
        println!("\n=== 详细内存使用统计 ===");
        
        let mut total_cpu_memory = 0;
        let mut optimized_images = 0;
        let mut standard_images = 0;
        
        // 显示每个图像的内存使用情况
        for mut info in query.iter_mut() {
            if let Some(image) = images.get_mut(&info.handle) {
                if !info.optimized {
                    let before_size = image.data.as_ref().map(|d| d.len()).unwrap_or(0);
                    
                    println!("🔍 {} - 修改前图像数据大小: {} bytes ({:.2} MB)", 
                        info.name, before_size, before_size as f32 / 1024.0 / 1024.0);
                    
                    image.asset_usage = RenderAssetUsages::RENDER_WORLD;
                    info.optimized = true;
                    
                    let after_size = image.data.as_ref().map(|d| d.len()).unwrap_or(0);
                    println!("✅ {} - 修改后CPU内存使用: {} bytes", info.name, after_size);
                    println!("💾 {} - 内存节省: {} bytes ({:.2} MB)", 
                        info.name,
                        before_size - after_size, 
                        (before_size - after_size) as f32 / 1024.0 / 1024.0);
                }
                
                let memory_usage = image.data.as_ref().map(|d| d.len()).unwrap_or(0);
                total_cpu_memory += memory_usage;
                
                let status = if info.optimized { "🎮 已优化" } else { "💻 未优化" };
                println!("{} {} - CPU内存: {:.2} MB ({}x{}像素)", 
                    status, info.name, memory_usage as f32 / 1024.0 / 1024.0, 
                    image.width(), image.height());
                
                if info.optimized {
                    optimized_images += 1;
                } else {
                    standard_images += 1;
                }
            }
        }
        
        println!("📊 CPU端总内存: {:.2} MB", total_cpu_memory as f32 / 1024.0 / 1024.0);
        println!("🎮 优化图像数量: {}", optimized_images);
        println!("💻 标准图像数量: {}", standard_images);
        println!("========================\n");
    }
}
