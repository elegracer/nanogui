# nanogui-fork

This is a fork of <https://github.com/wjakob/nanogui>.

# Why?

In order to adapt to my superbuild framework.

Removed git submodules because they're a large pile of shit.

# Documentation

## Creating widgets

NanoGUI makes it easy to instantiate widgets, set layout constraints, and
register event callbacks using high-level C++11 code. For instance, the
following two lines from the included example application add a new button to
an existing window `window` and register an event callback.

```cpp
Button *b = new Button(window, "Plain button");
b->setCallback([] { cout << "pushed!" << endl; });
```

The following lines from the example application create the coupled
slider and text box on the bottom of the second window (see the screenshot).

```cpp
/* Create an empty panel with a horizontal layout */
Widget *panel = new Widget(window);
panel->setLayout(new BoxLayout(BoxLayout::Horizontal, BoxLayout::Middle, 0, 20));

/* Add a slider and set defaults */
Slider *slider = new Slider(panel);
slider->setValue(0.5f);
slider->setFixedWidth(80);

/* Add a textbox and set defaults */
TextBox *textBox = new TextBox(panel);
textBox->setFixedSize(Vector2i(60, 25));
textBox->setValue("50");
textBox->setUnits("%");

/* Propagate slider changes to the text box */
slider->setCallback([textBox](float value) {
    textBox->setValue(std::to_string((int) (value * 100)));
});
```

## Simple mode

Christian Schüller contributed a convenience class that makes it possible to
create AntTweakBar-style variable manipulators using just a few lines of code.
For instance, the source code below was used to create the following example
application.

```cpp
/// dvar, bar, strvar, etc. are double/bool/string/.. variables

FormHelper *gui = new FormHelper(screen);
ref<Window> window = gui->addWindow(Eigen::Vector2i(10, 10), "Form helper example");
gui->addGroup("Basic types");
gui->addVariable("bool", bvar);
gui->addVariable("string", strvar);

gui->addGroup("Validating fields");
gui->addVariable("int", ivar);
gui->addVariable("float", fvar);
gui->addVariable("double", dvar);

gui->addGroup("Complex types");
gui->addVariable("Enumeration", enumval, enabled)
    ->setItems({"Item 1", "Item 2", "Item 3"});
gui->addVariable("Color", colval);

gui->addGroup("Other widgets");
gui->addButton("A button", [](){ std::cout << "Button pressed." << std::endl; });

screen->setVisible(true);
screen->performLayout();
window->center();
```

## Compiling

Just use cmake >= 3.13.

# License

NanoGUI is provided under a BSD-style license that can be found in the LICENSE_
file. By using, distributing, or contributing to this project, you agree to the
terms and conditions of this license.

NanoGUI uses Daniel Bruce's `Entypo+ <http://www.entypo.com/>`_ font for the
icons used on various widgets.  This work is licensed under a
`CC BY-SA 4.0 <https://creativecommons.org/licenses/by-sa/4.0/>`_ license.
Commercial entities using NanoGUI should consult the proper legal counsel for
how to best adhere to the attribution clause of the license.
