```rust



use tao::{
    event::{Event, WindowEvent},
    event_loop::{ControlFlow, EventLoop},
    window::{Window, WindowBuilder},
  };



  #[allow(dead_code)]
  #[derive(Debug,Clone)]
  pub struct MenuItem {
      pub text: String,
      pub enabled: bool,
      pub command: Option<String>,
  }

  


  #[allow(dead_code)]
  #[derive(Debug,Clone)]
  pub struct MenuFrame {
    pub item:MenuItem
  }
  



  #[allow(dead_code)]
  #[derive(Debug,Clone)]
  pub struct SystemTray {
      pub title: Option<String>,
      pub icon: Option<std::path::PathBuf>,
      pub is_template: Option<bool>,
      pub menu_on_left_click: Option<bool>,
      pub tooltip: Option<String>,
      pub menu_config: Option<MenuFrame>
}
  


fn create_menu(win:Window, menu_conf:Option<MenuFrame>)->(Window, muda::Menu){
    let menu = muda::Menu::new();
    if let Some(conf) =  menu_conf{
        
        let code = muda::accelerator::Code::AltLeft;
        let accelerator = muda::accelerator::Accelerator::new(None, code);
        let item = muda::MenuItem::new(conf.item.text, conf.item.enabled, Some(accelerator));
        menu.append(&item);
    };
    #[cfg(target_os = "windows")]
    unsafe {
        use tao::platform::windows::WindowExtWindows;
        menu.init_for_hwnd(win.hwnd() as _).unwrap();
    }
    #[cfg(target_os = "linux")]
    {
        menu
            .init_for_gtk_window(win.gtk_window(), win.default_vbox())
            .unwrap();
    }
    #[cfg(target_os = "macos")]
    {
        menu.init_for_nsapp();
        win.set_as_windows_menu_for_nsapp();
    }
    
    (win, menu)

}




fn create_system_tray(muda: muda::Menu, tray_conf: Option<SystemTray>) -> (muda::Menu, tray_icon::TrayIcon) {
    let mut tray_sys = tray_icon::TrayIconBuilder::new();
    
    if let Some(menu) = tray_conf {
        if let Some(title) = menu.title {
            tray_sys = tray_sys.with_title(title); // Ownership zurück in tray_sys speichern
        }
    }
    
    let tray_sys = tray_sys.with_menu(Box::new(muda.clone())); // Hier wird `tray_sys` erneut überschrieben

    // Fehlerbehandlung für `build()`, um Panics zu vermeiden
    let tray = tray_sys.build().expect("Failed to create system tray");
    (muda, tray)
}



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

    create_webview(None, None)
} 



```
