```rust

use tao::{
    event::{Event, WindowEvent},
    event_loop::{ControlFlow, EventLoop},
    window::{Window, WindowBuilder},
};

#[allow(dead_code)]
#[derive(Debug, Clone)]
pub struct MenuItem {
    pub text: String,
    pub enabled: bool,
    pub command: Option<String>,
}

#[allow(dead_code)]
#[derive(Debug, Clone)]
pub struct MenuFrame {
    pub item: MenuItem,
}

#[allow(dead_code)]
#[derive(Debug, Clone)]
pub struct SystemTray {
    pub title: Option<String>,
    pub icon: Option<std::path::PathBuf>,
    pub is_template: Option<bool>,
    pub menu_on_left_click: Option<bool>,
    pub tooltip: Option<String>,
    pub menu_config: Option<MenuFrame>,
}

fn create_menu(win: Window, menu_conf: Option<MenuFrame>) -> (Window, Option<muda::Menu>) {
    if let Some(conf) = menu_conf {
        let menu = muda::Menu::new();
        let code = muda::accelerator::Code::AltLeft;
        let accelerator = muda::accelerator::Accelerator::new(None, code);
        let item = muda::MenuItem::new(conf.item.text, conf.item.enabled, Some(accelerator));
        let _ = menu.append(&item);

        #[cfg(target_os = "windows")]
        unsafe {
            use tao::platform::windows::WindowExtWindows;
            menu.init_for_hwnd(win.hwnd() as _).unwrap();
        }
        #[cfg(target_os = "linux")]
        {
            menu.init_for_gtk_window(win.gtk_window(), win.default_vbox())
                .unwrap();
        }
        #[cfg(target_os = "macos")]
        {
            menu.init_for_nsapp();
            win.set_as_windows_menu_for_nsapp();
        }

        (win, Some(menu))
    } else {
        (win, None)
    }
}

fn create_system_tray(muda_menu: Option<muda::Menu>, tray_conf: Option<SystemTray>) -> Option<tray_icon::TrayIcon> {
    if let Some(conf) = tray_conf {
        let mut tray_sys = tray_icon::TrayIconBuilder::new();

        if let Some(title) = conf.title {
            tray_sys = tray_sys.with_title(title);
        }

        if let Some(menu) = muda_menu {
            tray_sys = tray_sys.with_menu(Box::new(menu));
        }

        // Fehlerbehandlung für `build()`
        return Some(tray_sys.build().expect("Failed to create system tray"));
    }
    None
}

fn create_webview(menu_conf: Option<MenuFrame>, tray_conf: Option<SystemTray>) {
    let event_loop = EventLoop::new();

    let window = WindowBuilder::new()
        .with_title("A fantastic window!")
        .with_inner_size(tao::dpi::LogicalSize::new(800.0, 600.0))
        .build(&event_loop)
        .unwrap();

    let (window, menu) = create_menu(window, menu_conf);
    let _tray_icon = create_system_tray(menu.clone(), tray_conf);

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
 
    
    // Case 1: System tray only

    // create_webview(None, Some(SystemTray { title: Some("Tray".into()), icon: None, is_template: None, menu_on_left_click: None, tooltip: None, menu_config: None }));

    // Case 2: only Menü
    // create_webview(Some(MenuFrame { item: MenuItem { text: "Option".into(), enabled: true, command: None } }), None);

    // Case 3: Both (Menu & System Tray)
    // create_webview(Some(MenuFrame { item: MenuItem { text: "Option".into(), enabled: true, command: None } }), Some(SystemTray { title: Some("Tray".into()), icon: None, is_template: None, menu_on_left_click: None, tooltip: None, menu_config: None }));

    // Case 4: Neither menu nor system tray
    // create_webview(None, None);
}

```
