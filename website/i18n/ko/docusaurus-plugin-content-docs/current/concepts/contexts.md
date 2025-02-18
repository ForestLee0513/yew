---
title: "컨텍스트"
sidebar_label: 컨텍스트
description: "컨텍스트를 사용해 애플리케이션 내부의 데이터를 전달합니다."
---

Generally data is passed down the component tree using props but that becomes tedious for values such as 
user preferences, authentication information etc. Consider the following example which passes down the 
theme using props:
```rust
use yew::{html, Children, Component, Context, Html, Properties};

#[derive(Clone, PartialEq)]
pub struct Theme {
    foreground: String,
    background: String,
}

#[derive(PartialEq, Properties)]
pub struct NavbarProps {
    theme: Theme,
}

pub struct Navbar;

impl Component for Navbar {
    type Message = ();
    type Properties = NavbarProps;

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        html! {
            <div>
                <Title theme={ctx.props().theme.clone()}>
                    { "App title" }
                </Title>
                <NavButton theme={ctx.props().theme.clone()}>
                    { "Somewhere" }
                </NavButton>
            </div>
        }
    }
}

#[derive(PartialEq, Properties)]
pub struct ThemeProps {
    theme: Theme,
    children: Children,
}

#[yew::function_component(Title)]
fn title(_props: &ThemeProps) -> Html {
    html! {
        // impl
    }
}
#[yew::function_component(NavButton)]
fn nav_button(_props: &ThemeProps) -> Html {
    html! {
        // impl
    }
}

// root
let theme = Theme {
    foreground: "yellow".to_owned(),
    background: "pink".to_owned(),
};

html! {
    <Navbar {theme} />
};
```

Passing down data like this isn't ideal for something like a theme which needs to be available everywhere. 
This is where contexts come in.

Contexts provide a way to share data between components without passing them down explicitly as props.
They make data available to all components in the tree.

## Using Contexts

In order to use contexts, we need a struct which defines what data is to be passed.
For the above use-case, consider the following struct:
```rust
#[derive(Clone, Debug, PartialEq)]
struct Theme {
    foreground: String,
    background: String,
}
```

A context provider is required to consume the context. `ContextProvider<T>`, where `T` is the context struct is used as the provider.
`T` must implement `Clone` and `PartialEq`. `ContextProvider` is the component whose children will have the context available to them.
The children are re-rendered when the context changes.

### Consuming context

#### Struct components

The `Scope::context` method is used to consume contexts in struct components.

##### Example

```rust
use yew::{Callback, html, Component, Context, Html};

#[derive(Clone, Debug, PartialEq)]
struct Theme {
    foreground: String,
    background: String,
}

struct ContextDemo;

impl Component for ContextDemo {
    type Message = ();
    type Properties = ();

    fn create(_ctx: &Context<Self>) -> Self {
        Self
    }

    fn view(&self, ctx: &Context<Self>) -> Html {
        let (theme, _) = ctx
            .link()
            .context::<Theme>(Callback::noop())
            .expect("context to be set");
        html! {
            <button style={format!(
                    "background: {}; color: {};", 
                    theme.background, 
                    theme.foreground
                )}
            >
                { "Click me!" }
            </button>
        }
    }
}
```

#### Function components

`use_context` hook is used to consume contexts in function components. 
See [docs for use_context](function-components/pre-defined-hooks.md#use_context) to learn more.
