```rust
// src/raven/mod.rs

// ============================================================================
// 模块声明
// ============================================================================

// 故事模块 - 包含Story结构体和story!宏
pub mod story {
    use std::collections::HashMap;
    use crate::raven::character::Character;
    use crate::raven::scene::Scene;
    use crate::raven::background::Background;

    #[derive(Debug, Clone)]
    pub struct Story {
        pub characters: HashMap<String, Character>,
        pub scenes: HashMap<String, Scene>,
        pub backgrounds: HashMap<String, Background>,
        pub start_scene: Option<String>,
    }

    impl Story {
        pub fn new() -> Self {
            Self {
                characters: HashMap::new(),
                scenes: HashMap::new(),
                backgrounds: HashMap::new(),
                start_scene: None,
            }
        }

        pub fn add_character(&mut self, id: String, character: Character) {
            self.characters.insert(id, character);
        }

        pub fn add_scene(&mut self, id: String, scene: Scene) {
            if self.start_scene.is_none() {
                self.start_scene = Some(id.clone());
            }
            self.scenes.insert(id, scene);
        }

        pub fn add_background(&mut self, id: String, background: Background) {
            self.backgrounds.insert(id, background);
        }

        pub fn get_character(&self, id: &str) -> Option<&Character> {
            self.characters.get(id)
        }

        pub fn get_scene(&self, id: &str) -> Option<&Scene> {
            self.scenes.get(id)
        }

        pub fn get_background(&self, id: &str) -> Option<&Background> {
            self.backgrounds.get(id)
        }
    }
}

// 角色模块
pub mod character {
    #[derive(Debug, Clone)]
    pub struct Character {
        pub name: String,
        pub sprite: String,
        pub color: Option<String>,
    }

    impl Character {
        pub fn new(name: String, sprite: String) -> Self {
            Self {
                name,
                sprite,
                color: None,
            }
        }

        pub fn with_color(mut self, color: String) -> Self {
            self.color = Some(color);
            self
        }
    }
}

// 场景模块
pub mod scene {
    #[derive(Debug, Clone)]
    pub struct Scene {
        pub commands: Vec<SceneCommand>,
    }

    #[derive(Debug, Clone)]
    pub enum SceneCommand {
        PlayMusic {
            file: String,
        },
        ShowBackground {
            background: String,
        },
        ShowCharacter {
            character: String,
            emotion: Option<String>,
        },
        HideCharacter {
            character: String,
        },
        Dialogue {
            speaker: String,
            text: String,
        },
        PlayerThinks {
            text: String,
        },
        PlayerSays {
            text: String,
        },
        ShowChoices {
            choices: Vec<Choice>,
        },
        Jump {
            scene: String,
        },
        EndWith {
            ending: String,
        },
    }

    #[derive(Debug, Clone)]
    pub struct Choice {
        pub text: String,
        pub scene: String,
    }

    impl Scene {
        pub fn new() -> Self {
            Self {
                commands: Vec::new(),
            }
        }

        pub fn add_command(&mut self, command: SceneCommand) {
            self.commands.push(command);
        }
    }

    impl Choice {
        pub fn new(text: String, scene: String) -> Self {
            Self { text, scene }
        }
    }
}

// 背景模块
pub mod background {
    #[derive(Debug, Clone)]
    pub struct Background {
        pub image: String,
        pub music: Option<String>,
    }

    impl Background {
        pub fn new(image: String) -> Self {
            Self {
                image,
                music: None,
            }
        }

        pub fn with_music(mut self, music: String) -> Self {
            self.music = Some(music);
            self
        }
    }
}

// 游戏状态模块 - 重构为使用安全的全局状态管理
pub mod game {
    use crate::raven::story::Story;
    use std::sync::{Mutex, OnceLock};

    #[derive(Debug, Clone, PartialEq)]
    pub enum GameResult {
        Playing,
        Ending(String),
        Quit,
        Error(String),
    }

    // 使用 OnceLock 和 Mutex 来安全地管理全局状态
    static GAME_STATE: OnceLock<Mutex<GameState>> = OnceLock::new();

    #[derive(Debug)]
    struct GameState {
        story: Option<Story>,
        current_scene: Option<String>,
        result: GameResult,
    }

    impl GameState {
        fn new() -> Self {
            Self {
                story: None,
                current_scene: None,
                result: GameResult::Playing,
            }
        }
    }

    // 初始化游戏状态
    fn init_game_state() -> &'static Mutex<GameState> {
        GAME_STATE.get_or_init(|| Mutex::new(GameState::new()))
    }

    pub fn run_raven_game_with_story(story: Option<Story>) {
        let game_state = init_game_state();
        
        {
            let mut state = game_state.lock().unwrap();
            state.current_scene = story.as_ref().and_then(|s| s.start_scene.clone());
            state.story = story;
            state.result = GameResult::Playing;
        }
        
        println!("Raven 游戏引擎已启动！");
        
        // 获取故事的只读引用来执行游戏逻辑
        let state = game_state.lock().unwrap();
        if let Some(story) = &state.story {
            println!("故事已加载，包含 {} 个角色，{} 个场景，{} 个背景", 
                story.characters.len(), 
                story.scenes.len(), 
                story.backgrounds.len()
            );
            
            // 简单执行第一个场景来验证功能
            if let Some(start_scene_id) = &story.start_scene {
                if let Some(scene) = story.get_scene(start_scene_id) {
                    println!("开始执行场景: {}", start_scene_id);
                    for command in &scene.commands {
                        execute_command(command, story);
                    }
                }
            }
        }
        // 这里 state 会自动释放锁
    }

    fn execute_command(command: &crate::raven::scene::SceneCommand, story: &Story) {
        use crate::raven::scene::SceneCommand;
        
        match command {
            SceneCommand::PlayMusic { file } => {
                println!("🎵 播放音乐: {}", file);
            },
            SceneCommand::ShowBackground { background } => {
                if let Some(bg) = story.get_background(background) {
                    println!("🖼️ 显示背景: {} ({})", background, bg.image);
                }
            },
            SceneCommand::ShowCharacter { character, emotion } => {
                if let Some(char) = story.get_character(character) {
                    let emotion_text = emotion.as_ref().map(|e| format!(" [{}]", e)).unwrap_or_default();
                    println!("👤 显示角色: {}{} ({})", char.name, emotion_text, char.sprite);
                }
            },
            SceneCommand::HideCharacter { character } => {
                println!("👻 隐藏角色: {}", character);
            },
            SceneCommand::Dialogue { speaker, text } => {
                if let Some(char) = story.get_character(speaker) {
                    println!("💬 {}: \"{}\"", char.name, text);
                } else {
                    println!("💬 {}: \"{}\"", speaker, text);
                }
            },
            SceneCommand::PlayerThinks { text } => {
                println!("💭 (内心想法): {}", text);
            },
            SceneCommand::PlayerSays { text } => {
                println!("🗣️ 玩家: \"{}\"", text);
            },
            SceneCommand::ShowChoices { choices } => {
                println!("🔘 选择:");
                for (i, choice) in choices.iter().enumerate() {
                    println!("  {}. {} -> {}", i + 1, choice.text, choice.scene);
                }
            },
            SceneCommand::Jump { scene } => {
                println!("↗️ 跳转到场景: {}", scene);
            },
            SceneCommand::EndWith { ending } => {
                println!("🏁 游戏结束: {}", ending);
                set_game_ending(ending.clone());
            },
        }
    }

    pub fn get_game_result() -> GameResult {
        let game_state = init_game_state();
        let state = game_state.lock().unwrap();
        state.result.clone()
    }

    pub fn end_raven_game() {
        let game_state = init_game_state();
        let mut state = game_state.lock().unwrap();
        if state.result == GameResult::Playing {
            state.result = GameResult::Quit;
        }
        println!("Raven 游戏引擎已关闭");
    }

    pub fn set_game_ending(ending: String) {
        let game_state = init_game_state();
        let mut state = game_state.lock().unwrap();
        state.result = GameResult::Ending(ending);
    }

    pub fn get_current_story() -> Option<Story> {
        let game_state = init_game_state();
        let state = game_state.lock().unwrap();
        state.story.clone()
    }
}

// ============================================================================
// 宏定义区域
// ============================================================================

// 主要的story宏
#[macro_export]
macro_rules! story {
    ($($item:tt)*) => {{
        let mut story = $crate::raven::story::Story::new();
        $crate::parse_story_items!(story, $($item)*);
        story
    }};
}

// 解析故事项目的辅助宏
#[macro_export]
macro_rules! parse_story_items {
    ($story:ident,) => {};
    
    ($story:ident, character $char_id:ident { $($char_content:tt)* } $($rest:tt)*) => {
        let character = $crate::parse_character!($($char_content)*);
        $story.add_character(stringify!($char_id).to_string(), character);
        $crate::parse_story_items!($story, $($rest)*);
    };
    
    ($story:ident, background $bg_id:ident { $($bg_content:tt)* } $($rest:tt)*) => {
        let background = $crate::parse_background!($($bg_content)*);
        $story.add_background(stringify!($bg_id).to_string(), background);
        $crate::parse_story_items!($story, $($rest)*);
    };
    
    ($story:ident, scene $scene_id:ident { $($scene_content:tt)* } $($rest:tt)*) => {
        let scene = $crate::parse_scene!($($scene_content)*);
        $story.add_scene(stringify!($scene_id).to_string(), scene);
        $crate::parse_story_items!($story, $($rest)*);
    };
}

// 解析角色的宏
#[macro_export]
macro_rules! parse_character {
    ($($content:tt)*) => {{
        let mut name = String::new();
        let mut sprite = String::new();
        let mut color: Option<String> = None;
        $crate::parse_character_fields!(name, sprite, color, $($content)*);
        
        let mut character = $crate::raven::character::Character::new(name, sprite);
        if let Some(c) = color {
            character = character.with_color(c);
        }
        character
    }};
}

// 解析角色字段的宏
#[macro_export]
macro_rules! parse_character_fields {
    ($name:ident, $sprite:ident, $color:ident,) => {};
    
    ($name:ident, $sprite:ident, $color:ident, name = $value:expr; $($rest:tt)*) => {
        $name = $value.to_string();
        $crate::parse_character_fields!($name, $sprite, $color, $($rest)*);
    };
    
    ($name:ident, $sprite:ident, $color:ident, sprite = $value:expr; $($rest:tt)*) => {
        $sprite = $value.to_string();
        $crate::parse_character_fields!($name, $sprite, $color, $($rest)*);
    };
    
    ($name:ident, $sprite:ident, $color:ident, color = $value:expr; $($rest:tt)*) => {
        $color = Some($value.to_string());
        $crate::parse_character_fields!($name, $sprite, $color, $($rest)*);
    };
}

// 解析背景的宏
#[macro_export]
macro_rules! parse_background {
    (image = $image:expr; $($rest:tt)*) => {{
        let mut background = $crate::raven::background::Background::new($image.to_string());
        $crate::parse_background_fields!(background, $($rest)*);
        background
    }};
}

// 解析背景字段的宏
#[macro_export]
macro_rules! parse_background_fields {
    ($bg:ident,) => {};
    
    ($bg:ident, music = $value:expr; $($rest:tt)*) => {
        $bg = $bg.with_music($value.to_string());
        $crate::parse_background_fields!($bg, $($rest)*);
    };
}

// 解析场景的宏
#[macro_export]
macro_rules! parse_scene {
    ($($content:tt)*) => {{
        let mut scene = $crate::raven::scene::Scene::new();
        $crate::parse_scene_commands!(scene, $($content)*);
        scene
    }};
}

// 场景命令解析宏
#[macro_export]
macro_rules! parse_scene_commands {
    ($scene:ident,) => {};
    
    // 播放音乐 - 使用字符串字面量
    ($scene:ident, play music $file:literal $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::PlayMusic {
            file: $file.to_string(),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 显示背景
    ($scene:ident, show background $bg:ident $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::ShowBackground {
            background: stringify!($bg).to_string(),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 显示角色（无表情）
    ($scene:ident, show character $char:ident $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::ShowCharacter {
            character: stringify!($char).to_string(),
            emotion: None,
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 显示角色（带表情）- 使用字符串字面量
    ($scene:ident, show character $char:ident as $emotion:literal $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::ShowCharacter {
            character: stringify!($char).to_string(),
            emotion: Some($emotion.to_string()),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 角色对话 - 使用字符串字面量
    ($scene:ident, $char:ident says $text:literal $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::Dialogue {
            speaker: stringify!($char).to_string(),
            text: $text.to_string(),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 玩家内心想法 - 使用字符串字面量
    ($scene:ident, player thinks $text:literal $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::PlayerThinks {
            text: $text.to_string(),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 玩家说话 - 使用字符串字面量
    ($scene:ident, player says $text:literal $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::PlayerSays {
            text: $text.to_string(),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 显示选择
    ($scene:ident, show choices { $($choice_content:tt)* } $($rest:tt)*) => {
        let mut choices = Vec::new();
        $crate::parse_choices!(choices, $($choice_content)*);
        $scene.add_command($crate::raven::scene::SceneCommand::ShowChoices { choices });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 跳转到场景
    ($scene:ident, jump to $target:ident $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::Jump {
            scene: stringify!($target).to_string(),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
    
    // 结束游戏 - 使用字符串字面量
    ($scene:ident, end with $ending:literal $($rest:tt)*) => {
        $scene.add_command($crate::raven::scene::SceneCommand::EndWith {
            ending: $ending.to_string(),
        });
        $crate::parse_scene_commands!($scene, $($rest)*);
    };
}

// 选择解析宏
#[macro_export]
macro_rules! parse_choices {
    ($choices:ident,) => {};
    
    // 使用字符串字面量避免 expr 后跟 -> 的问题
    ($choices:ident, $text:literal -> $scene:ident, $($rest:tt)*) => {
        $choices.push($crate::raven::scene::Choice::new(
            $text.to_string(),
            stringify!($scene).to_string()
        ));
        $crate::parse_choices!($choices, $($rest)*);
    };
    
    // 处理最后一个选择（没有逗号）
    ($choices:ident, $text:literal -> $scene:ident $($rest:tt)*) => {
        $choices.push($crate::raven::scene::Choice::new(
            $text.to_string(),
            stringify!($scene).to_string()
        ));
        $crate::parse_choices!($choices, $($rest)*);
    };
}

// ============================================================================
// 预导入模块，包含所有常用的导出
// ============================================================================
pub mod prelude {
    // 重新导出所有结构体和枚举
    pub use crate::raven::story::*;
    pub use crate::raven::character::*;
    pub use crate::raven::scene::*;
    pub use crate::raven::background::*;
    pub use crate::raven::game::*;
    
    // 重新导出宏
    pub use crate::story;
    pub use crate::parse_story_items;
    pub use crate::parse_character;
    pub use crate::parse_character_fields;
    pub use crate::parse_background;
    pub use crate::parse_background_fields;
    pub use crate::parse_scene;
    pub use crate::parse_scene_commands;
    pub use crate::parse_choices;
}

// ============================================================================
// 根级别导出
// ============================================================================
pub use story::Story;
pub use character::Character;
pub use scene::{Scene, SceneCommand, Choice};
pub use background::Background;
pub use game::{GameResult, run_raven_game_with_story, get_game_result, end_raven_game, set_game_ending};
```