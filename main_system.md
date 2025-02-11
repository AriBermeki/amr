# normal menu build
```rust


#[allow(dead_code)]
pub fn build_menu(
    menu: Option<crate::structs::pystructs::MenuFrame>,
    menu_api:&mut HashMap<muda::MenuId, (muda::MenuItemKind, Option<crate::structs::executors::FunctionInfo>)>,
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
                let (menu_id,menu_item, py_command) = create_muda_predefined_menu_item(predefined_menu);
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

```
# system_tray menu build

```rust
#[allow(dead_code)]
pub fn default_tray_menu(
    menu: Option<crate::structs::pystructs::MenuFrame>,
    menu_api:&mut HashMap<muda::MenuId, (muda::MenuItemKind, Option<crate::structs::executors::FunctionInfo>)>,
) ->muda::Menu{
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
                let (menu_id,menu_item, py_command) = create_muda_predefined_menu_item(predefined_menu);
                menu_bar.append(&menu_item).ok();
                menu_api.insert(menu_id, (muda::MenuItemKind::Predefined(menu_item), py_command));        
            }
        }
        
    }
        

    
    menu_bar
}

```


# system_tray  build
```rust
    
#[allow(dead_code)]
pub fn create_system_tray_menu(
    system_tray: Option<crate::structs::pystructs::SystemTray>,
    tao_menu_bar: Option<muda::Menu>,
    menu_api:&mut HashMap<muda::MenuId, (muda::MenuItemKind, Option<crate::structs::executors::FunctionInfo>)>,
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
    
    let menu = tao_menu_bar.unwrap_or_else(|| default_tray_menu(default_menu, menu_api));

    
    builder = builder.with_menu(Box::new(menu.clone()));

    // Fehlerbehandlung für `build()`, um Panics zu vermeiden
    let tray = builder.build().expect("Failed to create system tray");

    (menu, tray)
}






```

# python entry  point

```rust
#[pyfunction]
#[pyo3(signature = (window_config, control, webview_config, command_config, menu_config=None, tray_config=None))]
pub fn crate_webview(
    window_config:crate::structs::pystructs::WindowConfig,
    control: crate::structs::state_control::WindowControlCenter,
    webview_config:crate::structs::pystructs::WebViewConfig,
    command_config:crate::structs::pystructs::WebViewCommands,
    menu_config:Option<structs::pystructs::MenuFrame>,
    tray_config:Option<structs::pystructs::SystemTray>,
) -> utils::PyFrameResult<()> {


    let mut menu_tray = std::collections::HashMap::new();
    let mut webviews = std::collections::HashMap::new();
    let mut menu_command_api = std::collections::HashMap::new();

    let event_loop = EventLoopBuilder::with_user_event().build();

    let menu_proxy = event_loop.create_proxy();
    let main_proxy = event_loop.create_proxy();

    let (win,web)= manager::core_window(
        &event_loop,
        main_proxy,
        window_config,
        &control,
        webview_config,
        command_config
    )?;
    let menu_tray_win_id = win.id();
    let webviewa_win_id = win.id();
    let loop_win_id = win.id();



    let (win, menu_bar)= context::build_menu(menu_config, &mut menu_command_api, win);
    let (menu_bar,system_tray) = context::create_system_tray_menu(tray_config, Some(menu_bar), &mut menu_command_api);

    menu_tray.insert(menu_tray_win_id, (menu_bar, system_tray));
    webviews.insert(webviewa_win_id, (win, web));



    let menu_proxy_clone = menu_proxy.clone();

    event_loop.run(move |event, target, control_flow| {
        #[cfg(target_os = "macos")]
        unsafe {
            use objc2_core_foundation::{CFRunLoopGetMain, CFRunLoopWakeUp};

            let rl = CFRunLoopGetMain().unwrap();
            CFRunLoopWakeUp(&rl);
        }
        runtime::handle_pyframe_events(event, target, control_flow, loop_win_id, &control, &mut webviews);
        runtime::menu::handle_menu_eventloop(loop_win_id, &mut menu_api, menu_proxy_clone.clone());
    });

}

```
