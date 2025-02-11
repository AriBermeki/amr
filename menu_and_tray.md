# structs fom python # SubMenu and MenuFrame

```rust


#[allow(dead_code)]
#[derive(Debug,Clone)]
pub struct AboutMetadata {
    pub name: Option<String>,
    pub version: Option<String>,
    pub short_version: Option<String>,
    pub authors: Option<Vec<String>>,
    pub comments: Option<String>,
    pub copyright: Option<String>,
    pub license: Option<String>,
    pub website: Option<String>,
    pub website_label: Option<String>,
    pub credits: Option<String>,
    pub icon: Option<std::path::PathBuf>,
}



#[allow(dead_code)]
#[derive(Debug,Clone)]
pub struct MenuItem {
    pub text: String,
    pub enabled: bool,
    pub modifier:Option<String>,
    pub key: String,
    pub command: Option<String>,
}

#[allow(dead_code)]
#[derive(Debug,Clone)]
pub struct CheckMenuItem {
    pub text: String,
    pub enabled: bool,
    pub checked: bool,
    pub modifier:Option<String>,
    pub key: String,
    pub command: Option<String>,
}

#[allow(dead_code)]
#[derive(Debug,Clone)]
pub struct IconMenuItem {
    pub text: String,
    pub enabled: bool,
    pub icon_path: std::path::PathBuf,
    pub modifier:Option<String>,
    pub key: String,
    pub command: Option<String>,
}




#[allow(dead_code)]
#[derive(Debug,Clone)]
pub struct PredefinedMenuItem {
    pub item_type: String,
    pub text: Option<String>,
    pub metadata: Option<AboutMetadata>,
    pub command: Option<String>,
}





#[allow(dead_code)]
#[derive(Debug,Clone)]
pub struct SystemTray {
    pub title: Option<String>,
    pub icon: Option<std::path::PathBuf>,
    pub is_template: Option<bool>,
    pub menu_on_left_click: Option<bool>,
    pub tooltip: Option<String>,
    pub menu_config: Option<crate::menu::MenuFrame>
}


```


# utils.rs
```rust
use muda::MenuId;
use image::ImageFormat;
use muda;
use std::{fs::read, path::PathBuf};







#[allow(dead_code)]
pub fn create_muda_modifier(modifier: Option<String>) -> Option<muda::accelerator::Modifiers> {
    let modifier = modifier.as_deref()?.to_uppercase(); // Falls None, gibt die Funktion auch None zurück

    match modifier.as_str() {
        "ALT" => Some(muda::accelerator::Modifiers::ALT),
        "ALT_GRAPH" => Some(muda::accelerator::Modifiers::ALT_GRAPH),
        "SCROLL_LOCK" => Some(muda::accelerator::Modifiers::SCROLL_LOCK),
        "SUPER" => Some(muda::accelerator::Modifiers::SUPER),
        "SYMBOL" => Some(muda::accelerator::Modifiers::SYMBOL),
        "SYMBOL_LOCK" => Some(muda::accelerator::Modifiers::SYMBOL_LOCK),
        "FN" => Some(muda::accelerator::Modifiers::FN),
        "FN_LOCK" => Some(muda::accelerator::Modifiers::FN_LOCK),
        "NUM_LOCK" => Some(muda::accelerator::Modifiers::NUM_LOCK),
        "CAPS_LOCK" => Some(muda::accelerator::Modifiers::CAPS_LOCK),
        "CONTROL" => Some(muda::accelerator::Modifiers::CONTROL),
        "HYPER" => Some(muda::accelerator::Modifiers::HYPER),
        "META" => Some(muda::accelerator::Modifiers::META),
        _ => {
            eprintln!("Unsupported modifier: {}", modifier);
            None
        }
    }
}


#[allow(dead_code)]
pub fn create_muda_code(code: String) -> muda::accelerator::Code {
    match code.as_str() {
        "Backquote" => muda::accelerator::Code::Backquote,
        "Backslash" => muda::accelerator::Code::Backslash,
        "BracketLeft" => muda::accelerator::Code::BracketLeft,
        "BracketRight" => muda::accelerator::Code::BracketRight,
        "Comma" => muda::accelerator::Code::Comma,
        "Digit0" => muda::accelerator::Code::Digit0,
        "Digit1" => muda::accelerator::Code::Digit1,
        "Digit2" => muda::accelerator::Code::Digit2,
        "Digit3" => muda::accelerator::Code::Digit3,
        "Digit4" => muda::accelerator::Code::Digit4,
        "Digit5" => muda::accelerator::Code::Digit5,
        "Digit6" => muda::accelerator::Code::Digit6,
        "Digit7" => muda::accelerator::Code::Digit7,
        "Digit8" => muda::accelerator::Code::Digit8,
        "Digit9" => muda::accelerator::Code::Digit9,
        "Equal" => muda::accelerator::Code::Equal,
        "IntlBackslash" => muda::accelerator::Code::IntlBackslash,
        "IntlRo" => muda::accelerator::Code::IntlRo,

        _ => {
            println!("Unsupported Code method: {}", code);
            panic!("Unsupported Code method")
        }
    }
}

#[allow(dead_code)]
pub fn create_muda_native_icon(icon_name: &str) -> muda::NativeIcon {
    match icon_name {
        "Add" => muda::NativeIcon::Add,
        "Advanced" => muda::NativeIcon::Advanced,
        "Bluetooth" => muda::NativeIcon::Bluetooth,
        "Bookmarks" => muda::NativeIcon::Bookmarks,
        "Caution" => muda::NativeIcon::Caution,
        "ColorPanel" => muda::NativeIcon::ColorPanel,
        "ColumnView" => muda::NativeIcon::ColumnView,
        "Computer" => muda::NativeIcon::Computer,
        "EnterFullScreen" => muda::NativeIcon::EnterFullScreen,
        "Everyone" => muda::NativeIcon::Everyone,
    
        _ => {
            println!("Unsupported Code method: {}", icon_name);
            panic!("Unsupported Code method")
        }
    }
}

#[allow(dead_code)]
pub fn create_muda_accelerator(modifier: Option<String>, key: String) ->Option<muda::accelerator::Accelerator >{
    let mods = create_muda_modifier(modifier);
    let code_key = create_muda_code(key);
    let accelerator = muda::accelerator::Accelerator::new(mods, code_key);
    Some(accelerator)
}



pub fn muda_menu_icon(icon_path: PathBuf) -> Option<muda::Icon> {
    let bytes = read(&icon_path).ok()?; // Lies die Datei und gib None zurück, falls ein Fehler auftritt.
    let loaded = image::load_from_memory_with_format(&bytes, ImageFormat::Png).ok()?; // Lade das Bild
    let image_buffer = loaded.to_rgba8();
    let (icon_width, icon_height) = image_buffer.dimensions();
    let icon_rgba = image_buffer.into_raw();
    muda::Icon::from_rgba(icon_rgba, icon_width, icon_height).ok() // Versuche, ein Icon zu erstellen und gib es zurück
}


#[allow(dead_code)]
pub fn create_muda_about_metadata_menu_item(
    metadata: Option<crate::structs::AboutMetadata>,
) -> muda::AboutMetadata {
    let mut about_metadata = muda::AboutMetadata::default();

    if let Some(metadata) = metadata {
        about_metadata.name = metadata.name.clone();
        about_metadata.version = metadata.version.clone();
        about_metadata.website = metadata.website.clone();
        about_metadata.short_version = metadata.short_version.clone();
        about_metadata.authors = metadata.authors.clone();
        about_metadata.comments = metadata.comments.clone();
        about_metadata.copyright = metadata.copyright.clone();
        about_metadata.license = metadata.license.clone();
        about_metadata.website_label = metadata.website_label.clone();
        about_metadata.credits = metadata.credits.clone();

        // Set the icon if it exists and is provided in the metadata
        if let Some(icon) = metadata.icon.clone() {
            about_metadata.icon = muda_menu_icon(icon);
        }
    }

    about_metadata
}



pub fn system_tray_icon(icon_path: PathBuf)-> Option<tray_icon::Icon> {
    let bytes = read(&icon_path).ok()?; // Datei lesen
    let loaded = image::load_from_memory_with_format(&bytes, ImageFormat::Png).ok()?; // Bild laden

    let image_buffer = loaded.to_rgba8();
    let (icon_width, icon_height) = image_buffer.dimensions();
    let icon_rgba = image_buffer.into_raw();

    tray_icon::Icon::from_rgba(icon_rgba, icon_width, icon_height).ok() // Icon erstellen
}


#[allow(dead_code)]
pub fn create_muda_predefined_menu_item(
    item: crate::structs::PredefinedMenuItem,
) -> (MenuId, muda::PredefinedMenuItem, Option<String>) {
    let predefind_metadata = crate::utils::create_muda_about_metadata_menu_item(item.metadata);
    let predefined_item = match item.item_type.as_str() {
        "separator" => muda::PredefinedMenuItem::separator(),
        "about" => muda::PredefinedMenuItem::about(None, Some(predefind_metadata)),
        "close_window" => muda::PredefinedMenuItem::close_window(item.text.as_deref()),
        "copy" => muda::PredefinedMenuItem::copy(item.text.as_deref()),
        "fullscreen" => muda::PredefinedMenuItem::fullscreen(item.text.as_deref()),
        "cut" => muda::PredefinedMenuItem::cut(item.text.as_deref()),
        "hide" => muda::PredefinedMenuItem::hide(item.text.as_deref()),
        "hide_others" => muda::PredefinedMenuItem::hide_others(item.text.as_deref()),
        "maximize" => muda::PredefinedMenuItem::maximize(item.text.as_deref()),
        "minimize" => muda::PredefinedMenuItem::minimize(item.text.as_deref()),
        "paste" => muda::PredefinedMenuItem::paste(item.text.as_deref()),
        "bring_all_to_front" => muda::PredefinedMenuItem::bring_all_to_front(item.text.as_deref()),
        "quit" => muda::PredefinedMenuItem::quit(item.text.as_deref()),
        "redo" => muda::PredefinedMenuItem::redo(item.text.as_deref()),
        "select_all" => muda::PredefinedMenuItem::select_all(item.text.as_deref()),
        "show_all" => muda::PredefinedMenuItem::show_all(item.text.as_deref()),
        "undo" => muda::PredefinedMenuItem::undo(item.text.as_deref()),
        _ => {
            println!("Unsupported menu item type: {}", item.item_type);
            panic!("Unsupported menu item type")
        }
    };
    let py_function = item.command.clone();
    let predefined_item_id = predefined_item.id().clone();
    (predefined_item_id, predefined_item, py_function)
}




```


# core.rs
```rust

use std::collections::HashMap;

use muda::MenuId;

use muda;





pub fn create_check_menuitem(
    item: crate::structs::CheckMenuItem,
) -> (MenuId, muda::CheckMenuItem, Option<String>) {
    let accelerator = crate::utils::create_muda_accelerator(item.modifier.clone(), item.key.clone());
    let check_item = muda::CheckMenuItem::new(
        item.text.clone(),
        item.enabled,
        item.checked,
        accelerator,
    );
    let py_function = item.command.clone();

    let check_item_id = check_item.id().clone();
    (check_item_id, check_item, py_function)
}

pub fn create_icon_menuitem(
    item: crate::structs::IconMenuItem,
) -> (MenuId, muda::IconMenuItem, Option<String>) {
    let menu_icon = crate::utils::muda_menu_icon(item.icon_path.clone());
    let accelerator = crate::utils::create_muda_accelerator(item.modifier.clone(), item.key.clone());
    let icon_item = muda::IconMenuItem::new(
        item.text.clone(),
        item.enabled,
        menu_icon,
        accelerator,
    );
    let py_function = item.command.clone();
    let icon_item_id = icon_item.id().clone();
    (icon_item_id, icon_item, py_function)
}

pub fn create_menuitem(item: crate::structs::MenuItem) -> (MenuId, muda::MenuItem, Option<String>) {
    let accelerator = crate::utils::create_muda_accelerator(item.modifier.clone(), item.key.clone());
    let normal_item = muda::MenuItem::new(item.text.clone(), item.enabled, accelerator);
    let py_function = item.command.clone();
    let normal_item_id = normal_item.id().clone();
    (normal_item_id, normal_item, py_function)
}

pub fn create_submenu(
    item: crate::submenu::Submenu,
    menu_api: &mut HashMap<MenuId, (muda::MenuItemKind, Option<String>)>,
) -> muda::Submenu {
    let submenu = muda::Submenu::new(item.text.clone(), item.enabled);

    if let Some(menu_items) = item.menu_items {
        for menu_item in menu_items {
            let (menu_id, menu_item, py_command) = create_menuitem(menu_item.clone());
            submenu.append(&menu_item).ok();
            menu_api.insert(
                menu_id,
                (muda::MenuItemKind::MenuItem(menu_item), py_command),
            );
        }
    }

    if let Some(icon_menus) = item.icon_menu {
        for icon_menu in icon_menus {
            let (menu_id, menu_item, py_command) = create_icon_menuitem(icon_menu.clone());
            submenu.append(&menu_item).ok();
            menu_api.insert(menu_id, (muda::MenuItemKind::Icon(menu_item), py_command));
        }
    }

    if let Some(check_menus) = item.check_menu {
        for check_menu in check_menus {
            let (menu_id, menu_item, py_command) = create_check_menuitem(check_menu);
            submenu.append(&menu_item).ok();
            menu_api.insert(menu_id, (muda::MenuItemKind::Check(menu_item), py_command));
        }
    }

    if let Some(predefined_menu) = &item.predefined_menu {
        for predefined_menu in predefined_menu {
            let (menu_id, menu_item, py_command) =
            crate::utils::create_muda_predefined_menu_item(predefined_menu.clone());
            submenu.append(&menu_item).ok();
            menu_api.insert(
                menu_id,
                (muda::MenuItemKind::Predefined(menu_item), py_command),
            );
        }
    }

    submenu
}



#[allow(dead_code)]
pub fn build_menu(
    menu: Option<crate::menu::MenuFrame>,
    menu_api:&mut HashMap<muda::MenuId, (muda::MenuItemKind, Option<String>)>,
    win:tao::window::Window
) ->(tao::window::Window,muda::Menu) {
    let menu_bar = muda::Menu::new();
    if  menu.is_some() {
        let menu_system = menu.clone().unwrap();
        // Füge einfache Menüelemente hinzu
        if let Some(menu_items) = menu_system.menu_items {
            for item in menu_items {
                let (menu_id,menu_item, py_command) = create_menuitem(item);
                menu_bar.append(&menu_item).ok();
                menu_api.insert(menu_id, (muda::MenuItemKind::MenuItem(menu_item), py_command));
            }
        }
        
            // Füge Submenüs hinzu
        if let Some(sub_menus) = menu_system.sub_menu {
            for submenu in sub_menus {
                let sub_menu_item = create_submenu(submenu, &mut menu_api.clone());
                menu_bar.append(&sub_menu_item).ok();
            }
        }
        
            // Füge Check-Menüelemente hinzu
        if let Some(check_menus) = menu_system.check_menu {
            for check_item in check_menus {
                let (menu_id,menu_item, py_command) = create_check_menuitem(check_item);
                menu_bar.append(&menu_item).ok();
                menu_api.insert(menu_id, (muda::MenuItemKind::Check(menu_item), py_command));
            }
        }
        
            // Füge Icon-Menüelemente hinzu
        if let Some(icon_menus) = menu_system.icon_menu {
            for icon_item in icon_menus {
                let (menu_id,menu_item, py_command) = create_icon_menuitem(icon_item);
                menu_bar.append(&menu_item).ok();
                menu_api.insert(menu_id, (muda::MenuItemKind::Icon(menu_item), py_command));
        
            }
        }
            // Füge Predefined-Menüelemente hinzu
        if let Some(predefined_menu) = menu_system.predefined_menu {
            for predefined_menu in predefined_menu {
                let (menu_id,menu_item, py_command) = crate::utils::create_muda_predefined_menu_item(predefined_menu);
                menu_bar.append(&menu_item).ok();
                menu_api.insert(menu_id, (muda::MenuItemKind::Predefined(menu_item), py_command));        
            }
        }
        
    }
        
    
    #[cfg(target_os = "windows")]
    unsafe {
        use tao::platform::windows::WindowExtWindows;
        menu_bar.init_for_hwnd(win.hwnd() as _).unwrap();
    }
    #[cfg(target_os = "linux")]
    {
        menu_bar
            .init_for_gtk_window(win.gtk_window(), win.default_vbox())
            .unwrap();
    }
    #[cfg(target_os = "macos")]
    {
        menu_bar.init_for_nsapp();
        win.set_as_windows_menu_for_nsapp();
    }
    
    (win, menu_bar)
}
    
    
   
    
#[allow(dead_code)]
pub fn create_system_tray_menu(
    system_tray: Option<crate::structs::SystemTray>,
    tao_menu_bar: Option<muda::Menu>,
    menu_api:&mut HashMap<muda::MenuId, (muda::MenuItemKind, Option<String>)>,
) -> (muda::Menu, tray_icon::TrayIcon) {
    let mut builder = tray_icon::TrayIconBuilder::new();
    let mut default_menu = None;

    if let Some(menu) = system_tray {
        if let Some(title) = menu.title {
            builder = builder.with_title(title);
        }

        if let Some(icon_path) = menu.icon {
            if let Some(icon) = super::utils::system_tray_icon(icon_path) {
                builder = builder.with_icon(icon);
            } else {
                eprintln!("Error: Tray Icon Path not Found");
            }
        }

        if let Some(is_template) = menu.is_template {
            builder = builder.with_icon_as_template(is_template);
        }

        if let Some(menu_on_left_click) = menu.menu_on_left_click {
            builder = builder.with_menu_on_left_click(menu_on_left_click);
        }

        if let Some(tooltip) = menu.tooltip {
            builder = builder.with_tooltip(tooltip);
        }
        if let Some(menu) = menu.menu_config {
            default_menu = Some(menu);
            println!("default: {:?}", default_menu);
        }
    }
    
 
    // Falls `tao_menu_bar` None ist, erstelle ein **neues lokales** `muda::Menu`
    let menu = tao_menu_bar.unwrap_or(cloned_menu);
    
    builder = builder.with_menu(Box::new(menu.clone()));

    // Fehlerbehandlung für `build()`, um Panics zu vermeiden
    let tray = builder.build().expect("Failed to create system tray");

    (menu, tray)
}



```



# menu.rs
```rust

#[allow(dead_code)]
#[derive(Debug, Clone)]
pub struct MenuFrame {
    pub menu_items: Option<Vec<crate::structs::MenuItem>>,
    pub sub_menu: Option<Vec<crate::submenu::Submenu>>,
    pub check_menu: Option<Vec<crate::structs::CheckMenuItem>>,
    pub icon_menu: Option<Vec<crate::structs::IconMenuItem>>,
    pub predefined_menu: Option<Vec<crate::structs::PredefinedMenuItem>>,
}

impl MenuFrame {
    #[allow(dead_code)]
    pub fn new() -> Self {
        Self {
            menu_items: None,
            sub_menu: None,
            check_menu: None,
            icon_menu: None,
            predefined_menu: None,
        }
    }
    #[allow(dead_code)]
    pub fn add_menu_item(&mut self, text: String, enabled: bool, modifier: Option<String>, key: String, command: Option<String>) {
        if self.menu_items.is_none() {
            self.menu_items = Some(Vec::new());
        }
        self.menu_items.as_mut().unwrap().push(crate::structs::MenuItem {
            text,
            enabled,
            modifier,
            key,
            command,
        });
    }
    #[allow(dead_code)]
    pub fn add_submenu(&mut self, submenu: crate::submenu::Submenu) {
        if self.sub_menu.is_none() {
            self.sub_menu = Some(Vec::new());
        }
        self.sub_menu.as_mut().unwrap().push(submenu);
    }
    #[allow(dead_code)]
    pub fn add_check_menu_item(
        &mut self,
        text: String,
        enabled: bool,
        checked: bool,
        modifier: Option<String>,
        key: String,
        command: Option<String>,
    ) {
        if self.check_menu.is_none() {
            self.check_menu = Some(Vec::new());
        }
        self.check_menu.as_mut().unwrap().push(crate::structs::CheckMenuItem {
            text,
            enabled,
            checked,
            modifier,
            key,
            command,
        });
    }
    #[allow(dead_code)]
    pub fn add_icon_menu_item(
        &mut self, 
        text: String,
        enabled: bool,
        icon_path: std::path::PathBuf,
        modifier: Option<String>,
        key: String,
        command: Option<String>,
    ) {
        if self.icon_menu.is_none() {
            self.icon_menu = Some(Vec::new());
        }
        self.icon_menu.as_mut().unwrap().push(crate::structs::IconMenuItem {
            text,
            enabled,
            icon_path,
            modifier,
            key,
            command,
        });
    }
    #[allow(dead_code)]
    pub fn add_predefined_menu_item(&mut self, item_type: String, text: Option<String>, metadata: Option<crate::structs::AboutMetadata>, command: Option<String>) {
        if self.predefined_menu.is_none() {
            self.predefined_menu = Some(Vec::new());
        }
        self.predefined_menu.as_mut().unwrap().push(crate::structs::PredefinedMenuItem {
            item_type,
            text,
            metadata,
            command,
        });
    }
}

```



# submenu.rs
```rust

#[allow(dead_code)]
#[derive(Debug, Clone)]
pub struct Submenu {
    pub text: String,
    pub enabled: bool,
    pub menu_items: Option<Vec<crate::structs::MenuItem>>,
    pub check_menu: Option<Vec<crate::structs::CheckMenuItem>>,
    pub icon_menu: Option<Vec<crate::structs::IconMenuItem>>,
    pub predefined_menu: Option<Vec<crate::structs::PredefinedMenuItem>>,
}

impl Submenu {
    #[allow(dead_code)]
    pub fn new(text: &str, enabled: bool) -> Self {
        Self {
            text: text.to_string(),
            enabled,
            menu_items: None,
            check_menu: None,
            icon_menu: None,
            predefined_menu: None,
        }
    }
    #[allow(dead_code)]
    pub fn add_menu_item(&mut self, text: String, enabled: bool, modifier: Option<String>, key: String, command: Option<String>) {
        if self.menu_items.is_none() {
            self.menu_items = Some(Vec::new());
        }
        self.menu_items.as_mut().unwrap().push(crate::structs::MenuItem {
            text,
            enabled,
            modifier,
            key,
            command,
        });
    }
    #[allow(dead_code)]
    pub fn add_check_menu_item(
        &mut self,
        text: String,
        enabled: bool,
        checked: bool,
        modifier: Option<String>,
        key: String,
        command: Option<String>,
    ) {
        if self.check_menu.is_none() {
            self.check_menu = Some(Vec::new());
        }
        self.check_menu.as_mut().unwrap().push(crate::structs::CheckMenuItem {
            text,
            enabled,
            checked,
            modifier,
            key,
            command,
        });
    }
    #[allow(dead_code)]
    pub fn add_icon_menu_item(
        &mut self, 
        text: String,
        enabled: bool,
        icon_path: std::path::PathBuf,
        modifier: Option<String>,
        key: String,
        command: Option<String>,
    ) {
        if self.icon_menu.is_none() {
            self.icon_menu = Some(Vec::new());
        }
        self.icon_menu.as_mut().unwrap().push(crate::structs::IconMenuItem {
            text,
            enabled,
            icon_path,
            modifier,
            key,
            command,
        });
    }
    #[allow(dead_code)]
    pub fn add_predefined_menu_item(&mut self, item_type: String, text: Option<String>, metadata: Option<crate::structs::AboutMetadata>, command: Option<String>) {
        if self.predefined_menu.is_none() {
            self.predefined_menu = Some(Vec::new());
        }
        self.predefined_menu.as_mut().unwrap().push(crate::structs::PredefinedMenuItem {
            item_type,
            text,
            metadata,
            command,
        });
    }
}

```



# main.rs
```rust
mod utils;
mod menu_core;
mod menu;
mod structs;
mod submenu;



use menu::MenuFrame;
use structs::SystemTray;
use submenu::Submenu;
use tao::{
    event::{Event, WindowEvent},
    event_loop::{ControlFlow, EventLoop},
    window::WindowBuilder,
  };
  
fn create_webview(menu:Option<MenuFrame>, sys_tray:Option<SystemTray>){
    let event_loop = EventLoop::new();
    // let mut menu_api = std::collections::HashMap::new();
    let window = WindowBuilder::new()
    .with_title("A fantastic window!")
    .with_inner_size(tao::dpi::LogicalSize::new(800.0, 600.0))
    .build(&event_loop)
    .unwrap();
  


    event_loop.run(move |event, _, control_flow| {
    *control_flow = ControlFlow::Wait;

    if let Event::WindowEvent {
      event: WindowEvent::CloseRequested,
      ..
    } = event
    {
      *control_flow = ControlFlow::Exit;
    }
  });
  }

fn main() {
    let mut menu = MenuFrame::new();
    menu.add_menu_item("Malek".into(), true, None, "Equal".to_string(), Some("Malek".to_string()));

    create_webview(None, None)
} 
```
