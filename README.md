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
