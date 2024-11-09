```rust

use std::{collections::HashMap, sync::Arc};
use tao::{event::{Event, WindowEvent}, event_loop::{ControlFlow, EventLoop, EventLoopBuilder, EventLoopProxy}, window::{Window, WindowBuilder, WindowId}};
use wry::{WebView, WebViewBuilder};
use pyo3::prelude::*;

use crate::{executer::FunctionInfo, ipc::eventlistner};

#[macro_export]
macro_rules! unsafe_impl_sync_send {
    ($type:ty) => {
        unsafe impl Send for $type {}
        unsafe impl Sync for $type {}
    };
}


#[allow(dead_code)]
#[derive(Debug, Clone)]
pub enum PythonEventAPI {
    UserEvent(String, String),
    NoneEvent(String),
}


pub(crate) fn connection_event_emitter(event: &str, pyload:&str) -> PythonEventAPI {
    match event {
        "ari_event" => PythonEventAPI::UserEvent(((&event)).to_string(), ((&pyload)).to_string()),
        _ => PythonEventAPI::NoneEvent(format!("Not Spourted Event: {}", event)),  // Rückgabe von None, wenn das Event nicht übereinstimmt
    }
}

#[derive(Debug, Clone)]
pub(crate) struct PyProxyEventEmitter {
    emitter: EventLoopProxy<PythonEventAPI>,
}

impl PyProxyEventEmitter {
    pub fn new(emitter: EventLoopProxy<PythonEventAPI>) -> Self {
        Self {
            emitter,
        }
    }

    pub(crate) fn send_event(&mut self, event: &str, payload: &str) {
        let main_proxy_event = connection_event_emitter(event, payload); 
        let _ = self.emitter.send_event(main_proxy_event);
    }
}




#[pyclass]
pub struct RustAPI {
    event_loop:EventLoop<PythonEventAPI>,
    emitter: PyProxyEventEmitter
}


#[pymethods]
impl RustAPI {
    #[new]
    pub fn new()->Self{
        let event_loop: EventLoop<PythonEventAPI> = EventLoopBuilder::<PythonEventAPI>::with_user_event().build();
        println!("create_eventloop{event_loop:?}");
        let proxy = event_loop.create_proxy();
        println!("create_proxy{proxy:?}");
        let emitter = PyProxyEventEmitter::new(proxy);
        println!("create_emitter{emitter:?}");

        Self{
            event_loop,
            emitter
        }
    }


    pub fn send_proxy_event(&mut self, event: &str, pyload: &str)->PyResult<()>{
       println!("send_proxy_event input data: {event}{pyload}");
        self.emitter.send_event(event, pyload); // to be able to send the data outside the box immediately to the event loop
        Ok(())
    }





    
    pub fn start(
        &mut self, 
        handler: Option<FunctionInfo>, 
        initial_script: Option<Vec<String>>,
        frontend: Option<String>,
    ) {
        println!("start main thread with the handler: {handler:?}");
        let mut webview_runtime_api: HashMap<WindowId, (Window, WebView)> = HashMap::new();
        let event_loop = std::mem::replace(
            &mut self.event_loop, 
            EventLoopBuilder::with_user_event().build()
        );
    
        let window = WindowBuilder::new().build(&event_loop).unwrap();
    
        let web_view = {
            let mut builder = WebViewBuilder::new();
            
            // Conditionally add the IPC handler
            if let Some(func) = handler.clone() {
                let func = Arc::new(func);  // Wrap FunctionInfo in Arc
                let handler = eventlistner(window.id(), Some(func));
                builder = builder.with_ipc_handler(handler);
            }
            


            if let Some(scripts) = initial_script.clone() {
                for script in scripts {
                    builder = builder.with_initialization_script(&script);
                }
                
            }


            if let Some(script) = frontend.clone() {
                if script.starts_with("http") || script.starts_with("https") {
                    builder = builder.with_url(&script);
                } else {
                    builder = builder.with_html(&script);
                }
                
            }
            // Platform-specific build logic
            #[cfg(any(
                target_os = "windows",
                target_os = "macos",
                target_os = "ios",
                target_os = "android"
            ))]
            {
                builder.build(&window).unwrap()
            }
            
            #[cfg(not(any(
                target_os = "windows",
                target_os = "macos",
                target_os = "ios",
                target_os = "android"
            )))]
            {
                use tao::platform::unix::WindowExtUnix;
                use wry::WebViewBuilderExtUnix;
                let vbox = window.default_vbox().unwrap();
                builder.build_gtk(vbox)?
            }
        };
    
        webview_runtime_api.insert(window.id(), (window, web_view));
    
        let mut webviews_api = webview_runtime_api;
    
        event_loop.run(move |event, _target_loop, control_flow| {
            *control_flow = ControlFlow::Wait;
    
            match event {
                Event::WindowEvent {
                    event: WindowEvent::CloseRequested,
                    window_id,
                    ..
                } => handle_close_requested(window_id, &mut webviews_api, control_flow),
    
                Event::UserEvent(ref user_event) => {
                    match user_event {
                        PythonEventAPI::UserEvent(event, pyload) => {
                            println!(
                                "Some Event from Python [ event Type: {}, payload: {}]",
                                event, pyload
                            );
                        }
                        PythonEventAPI::NoneEvent(data) => {
                            println!(
                                "Not Supported Event from Python: {}",
                                data
                            );
                        }
                    }
                },
                _ => (),
            }
        });
    }

}




pub fn handle_close_requested(
    window_id: WindowId,
    webviews: &mut HashMap<WindowId, (Window, WebView)>,
    control_flow: &mut ControlFlow,
) {
    webviews.remove(&window_id);
    if webviews.is_empty() {
        *control_flow = ControlFlow::Exit;
    }
}
unsafe_impl_sync_send!(RustAPI);




```




```rust

use std::sync::Arc;

use pyo3::{prelude::*, types::PyDict};
use muda::Menu;

// /  The FunctionInfo object contains information about a Python callback for the menu item, 
/// in case a callback handler or event is required to be fired.
#[pyclass]
#[derive(Debug, Clone)]
pub struct FunctionInfo {
    #[pyo3(get, set)]
    pub handler: Py<PyAny>,
    #[pyo3(get, set)]
    pub is_async: bool,
    #[pyo3(get, set)]
    pub number_of_params: u8,
    #[pyo3(get, set)]
    pub args: Py<PyDict>,
    #[pyo3(get, set)]
    pub kwargs: Py<PyDict>,
}

#[pymethods]
impl FunctionInfo {
    #[new]
    pub fn new(
        handler: Py<PyAny>,
        is_async: bool,
        number_of_params: u8,
        args: Py<PyDict>,
        kwargs: Py<PyDict>,
    ) -> Self {
        Self {
            handler,
            is_async,
            number_of_params,
            args,
            kwargs,
        }
    }
}


// The PyFrameMenuItem object contains information about a PyFrameMenuItem.
#[pyclass]
#[derive(Debug, Clone)]
pub struct PyFrameMenuItem {
    #[pyo3(get, set)]
    pub menu_type:Option<String>,
    #[pyo3(get, set)]
    pub handler:Option<FunctionInfo>,
}

#[pymethods]
impl PyFrameMenuItem {
    #[new]
    pub fn new(
        menu_type:Option<String>,
        handler:Option<FunctionInfo>,
    ) -> Self {
        Self {
            menu_type,
            handler
        }
    }
}


/// The PyFrameSystemMenu object contains all the information about the menu itself and its items, 
/// which is passed to the create_pyframe_menu function on the Rust side to create the menu structure.

#[pyclass]
#[derive(Debug, Clone)]
pub struct PyFrameSystemMenu {
    #[pyo3(get, set)]
    menus:Vec<PyFrameMenuItem>
}

#[pymethods]
impl PyFrameSystemMenu {
    #[new]
    pub fn new(
        menus:Vec<PyFrameMenuItem>
    ) -> Self {
        Self {
            menus
        }
    }
}



/// has the task to create the menu, with all information from python
pub fn create_pyframe_menu(menu: Option<Arc<Vec<PyFrameMenuItem>>>,)->Menu{
    let menu = Menu::new();
    menu
}



```
