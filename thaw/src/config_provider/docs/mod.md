# ConfigProvider

The `<ConfigProvider />` component allows you to apply a consistent
configuration to all the thaw-ui components nested within it. Think of it as a
centralized control panel for the look and feel of your app.

It uses Leptos's context system to provide this configuration down the component
tree. This means you don't have to manually configure every single button,
input, or other thaw-ui component. You set the configuration once on the
ConfigProvider, and all the child components will inherit it.

By wrapping your router and other providers with `<ConfigProvider>`, you are
ensuring that all the components in your application (like pages and their
sub-components) will use the configuration you provide.

Currently, the most common properties you'll use with `<ConfigProvider />` are:

* `theme`: This allows you to set a theme (e.g. light or dark) for all
    components. You can pass it a signal
    ([RwSignal](https://docs.rs/leptos/latest/leptos/prelude/struct.RwSignal.html))
    to dynamically change the theme.
* `dir`: This sets the text and layout direction.


### Theme demo

```rust demo
let theme = RwSignal::new(Theme::light());

view! {
    <ConfigProvider theme>
        <Card>
            <Space>
                <Button on_click=move |_| theme.set(Theme::light())>"Light"</Button>
                <Button on_click=move |_| theme.set(Theme::dark())>"Dark"</Button>
            </Space>
        </Card>
    </ConfigProvider>
}
```

### Customize Theme

```rust demo
let theme = RwSignal::new(Theme::light());
let on_customize_theme = move |_| {
    theme.update(|theme| {
        theme.color.set_color_brand_background("#f5222d".to_string());
        theme.color.set_color_brand_background_hover("#ff4d4f".to_string());
        theme.color.set_color_brand_background_pressed("#cf1322".to_string());
    });
};

view! {
    <ConfigProvider theme>
        <Card>
            <Space>
                <Button appearance=ButtonAppearance::Primary on_click=move |_| theme.set(Theme::light())>"Light"</Button>
                <Button appearance=ButtonAppearance::Primary on_click=on_customize_theme>"Customize Theme"</Button>
            </Space>
        </Card>
    </ConfigProvider>
}
```

### ConfigProvider Props

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| class | `MaybeProp<String>` | `Default::default()` |  |
| theme | `Option<RwSignal<Theme>>` | `None` | Sets the theme used in a scope. |
| theme_id | `Option<String>` | `None` | Theme id. |
| dir | `RwSignal<Option<ConfigDirection>>` |  | Sets the direction of text & generated styles. |
| locale | `Option<RwSignal<LocaleConfig>>` | | Set the locale used for some component. | 
| children | `Children` |  |  |
