# Drawer

A Drawer is a component that slides in from an edge of the screen to show
content. Think of it like a panel or a side menu. In Thaw UI, it's used to
display information or actions without forcing the user to navigate to a new
page.

There are two main types of drawers in this library:

1. `<OverlayDrawer/>`: This drawer slides in over the top of the current page
    content. When it's open, it usually has a dark backdrop behind it, and you
    can't interact with the rest of the page until the drawer is closed.
2. `<InlineDrawer/>`: This drawer is part of the page's layout. When it opens, it
   pushes the other content on the page to make space for itself.

## Using an `<OverlayDrawer/>`

To use an `<OverlayDrawer/>`, you control its visibility with a [boolean
signal](https://docs.rs/reactive_graph/latest/reactive_graph/signal/struct.RwSignal.html).
When the signal is `true`, the drawer is open; when `false`, it's closed.

See the following example:

```rust demo
// Create signals to control the drawer's open/closed state and position
let open = RwSignal::new(false);
let position = RwSignal::new(DrawerPosition::Top);

let open_f = move |new_position: DrawerPosition| {
    // Note: Since `show` changes are made in real time,
    // please put it in front of `show.set(true)` when changing `placement`.
    position.set(new_position);
    open.set(true);
};

view! {
    <ButtonGroup>
        <Button on_click=move |_| open_f(DrawerPosition::Top)>"Top"</Button>
        <Button on_click=move |_| open_f(DrawerPosition::Right)>"Right"</Button>
        <Button on_click=move |_| open_f(DrawerPosition::Bottom)>"Bottom"</Button>
        <Button on_click=move |_| open_f(DrawerPosition::Left)>"Left"</Button>
    </ButtonGroup>
    <OverlayDrawer open position>
        <DrawerHeader>
          <DrawerHeaderTitle>
            <DrawerHeaderTitleAction slot>
                 <Button
                    appearance=ButtonAppearance::Subtle
                    on_click=move |_| open.set(false)
                >
                    "x"
                </Button>
            </DrawerHeaderTitleAction>
            "Default Drawer"
          </DrawerHeaderTitle>
        </DrawerHeader>
        <DrawerBody>
          <p>"Drawer content"</p>
        </DrawerBody>
    </OverlayDrawer>
}
```

In this example:

- Two `RwSignal`s named `open` and `position` are created to manage the drawer's state and position.
- Four `<Button>`'s set the position of the drawer and open it.
- The `<OverlayDrawer>` component takes the `open` and `position` signals as props.
- Inside the drawer, another `<Button>` sets `open` back to false to close it.

### Non-modal `<OverlayDrawer>`

A non-modal `<OverlayDrawer/>` is a variation of the overlay drawer that, when open,
**does not prevent interaction with the rest of the page**. It still slides in over
the top of the page content, but it doesn't have the dimming backdrop, and users
are free to click on and interact with other elements on the page.

This is useful when you want to display supplementary information or tools that
a user might want to access or view alongside the main content, without locking
them out of the main application

See the following example:


```rust demo
let open = RwSignal::new(false);

view! {
    <Button on_click=move |_| open.update(|open| *open = !*open)>"Toggle"</Button>
    <OverlayDrawer open modal_type=DrawerModalType::NonModal>
        <DrawerHeader>
          <DrawerHeaderTitle>
            <DrawerHeaderTitleAction slot>
                 <Button
                    appearance=ButtonAppearance::Subtle
                    on_click=move |_| open.set(false)
                >
                    "x"
                </Button>
            </DrawerHeaderTitleAction>
            "Non-Modal Drawer"
          </DrawerHeaderTitle>
        </DrawerHeader>
        <DrawerBody>
          <p>"Drawer content"</p>
          <p>"You can interact with the rest of the page while this is open."</p>
        </DrawerBody>
    </OverlayDrawer>
}
```

The key difference in the code is the line:

```rust
modal_type=DrawerModalType::NonModal
```

This tells the `<OverlayDrawer>` to render without the backdrop and to allow user
interaction with the page content behind it.


## Using an `<InlineDrawer>`

An `<InlineDrawer>` is a drawer that is embedded **directly within the layout of
the page or a specific container**. Unlike the `<OverlayDrawer>` which floats on top
of all content, the `<InlineDrawer>` is part of the content flow. When it opens, it
*pushes* the adjacent content to make space for itself, rather than covering it.

This type of drawer is ideal when you want to integrate a panel as a permanent
part of a specific layout, such as a collapsible sidebar within a dashboard's
main view. The container of the `<InlineDrawer>` needs to be styled to handle the
drawer’s appearance, typically using CSS Flexbox.


The following example demonstrates a complex layout with three separate
InlineDrawer components opening from the left, right, and bottom

```rust demo
// Three separate signals are created to control the open/closed state of each of the three drawers independently.
let open_left = RwSignal::new(false);
let open_right = RwSignal::new(false);
let open_buttom = RwSignal::new(false);

view! {
    // This is the main container for the entire component.
    // It's a flex container with a column direction, allowing the bottom drawer to appear below the main content area.
    // `overflow: hidden` is important to ensure the drawers don't break the parent layout.
    <div style="display: flex; flex-direction: column; overflow: hidden; height: 400px; background-color: #0078ff88;">
         // This is the container for the top part of the layout, which includes the left and right
         // drawers and the central content.
         // It's a flex container with the default row direction
        <div style="display: flex; overflow: hidden; height: 400px;">
            // This is the first InlineDrawer. It will open from the left by default.
            // Its visibility is tied to the `open_left` signal.
            <InlineDrawer open=open_left>
                <DrawerHeader>
                <DrawerHeaderTitle>
                    // The close button for the left drawer. It sets `open_left` to false.
                    <DrawerHeaderTitleAction slot>
                        <Button
                            appearance=ButtonAppearance::Subtle
                            on_click=move |_| open_left.set(false)
                        >
                            "x"
                        </Button>
                    </DrawerHeaderTitleAction>
                    "Inline Drawer"
                </DrawerHeaderTitle>
                </DrawerHeader>
                <DrawerBody>
                    <p>"Drawer content"</p>
                </DrawerBody>
            </InlineDrawer>
            // This is the central content area that holds the buttons to open the drawers.
            // `flex: 1` allows this div to take up the remaining available space between the left and right drawers.
            <div style="flex: 1">
                <Button on_click=move |_| open_left.set(true)>"Open left"</Button>
                <Button on_click=move |_| open_right.set(true)>"Open right"</Button>
                <Button on_click=move |_| open_buttom.set(true)>"Open buttom"</Button>
            </div>

            // This is the second InlineDrawer. `position=DrawerPosition::Right` makes it open from the right.
            // Its visibility is tied to the `open_right` signal.
            <InlineDrawer open=open_right position=DrawerPosition::Right>
                <DrawerHeader>
                <DrawerHeaderTitle>
                    // The close button for the right drawer.
                    <DrawerHeaderTitleAction slot>
                        <Button
                            appearance=ButtonAppearance::Subtle
                            on_click=move |_| open_right.set(false)
                        >
                            "x"
                        </Button>
                    </DrawerHeaderTitleAction>
                    "Inline Drawer"
                </DrawerHeaderTitle>
                </DrawerHeader>
                <DrawerBody>
                    <p>"Drawer content"</p>
                </DrawerBody>
            </InlineDrawer>
        </div>
        // This is the third InlineDrawer. `position=DrawerPosition::Bottom` makes it open from the bottom.
        // It's placed in the outer flex container, so it will appear below the `div` that contains the other drawers.
        // Its visibility is tied to the `open_buttom` signal.
        <InlineDrawer open=open_buttom position=DrawerPosition::Bottom>
            <DrawerHeader>
            <DrawerHeaderTitle>
                // The close button for the bottom drawer.
                <DrawerHeaderTitleAction slot>
                    <Button
                        appearance=ButtonAppearance::Subtle
                        on_click=move |_| open_buttom.set(false)
                    >
                        "x"
                    </Button>
                </DrawerHeaderTitleAction>
                "Inline Drawer"
            </DrawerHeaderTitle>
            </DrawerHeader>
            <DrawerBody>
                <p>"Drawer content"</p>
            </DrawerBody>
        </InlineDrawer>
    </div>
}
```

This example demonstrates how to compose a complex layout using multiple `<InlineDrawer>` components.

It sets up a parent `<div>` as a vertical flex container (`flex-direction: column`) to house two main sections: a top area and a bottom drawer.

The top area is itself a horizontal flex container (`display: flex`) that holds three elements:

1. An `<InlineDrawer>` on the left (the default position).
2. A central `<div>` that takes up the remaining space (`flex: 1`) and contains
   the buttons to open all three drawers.
3. A second `<InlineDrawer>` on the right, configured with
   `position=DrawerPosition::Right`.

The third `<InlineDrawer>` is placed in the main vertical container and is
configured to open from the bottom with `position=DrawerPosition::Bottom`.

Each of the three drawers is controlled by its own independent `RwSignal<bool>`
(`open_left`, `open_right`, `open_bottom`), allowing them to be opened and closed
without affecting each other. This showcases how `<InlineDrawer>` can be
integrated into a specific part of a UI, pushing content within its flex
container to make space for itself.

### Using an `<OverlayDrawer>` With Scroll

This example demonstrates how the `<OverlayDrawer>` (or `<InlineDrawer>`)
component handles content that is taller than the drawer itself. By default, the
`<DrawerBody>` will automatically provide a scrollbar when its content overflows,
which is the behavior shown here:

```rust demo
let open = RwSignal::new(false);

view! {
    <Button on_click=move |_| open.set(true)>"Open"</Button>
    <OverlayDrawer open>
        <DrawerHeader>
          <DrawerHeaderTitle>
            <DrawerHeaderTitleAction slot>
                 <Button
                    appearance=ButtonAppearance::Subtle
                    on_click=move |_| open.set(false)
                >
                    "x"
                </Button>
            </DrawerHeaderTitleAction>
            "Default Drawer"
          </DrawerHeaderTitle>
        </DrawerHeader>
        // A paragraph with a large amount of text.
        // The `line-height` style is used here to make the content artificially tall.
        // Because this content is taller than the drawer's body, the DrawerBody component
        // will automatically add a vertical scrollbar.
        <DrawerBody>
          <p style="line-height: 40px">r#"This being said, the world is moving in the direction opposite to Clarke's predictions. In 2001: A Space Odyssey, in the year of 2001, which has already passed, human beings have built magnificent cities in space, and established permanent colonies on the moon, and huge nuclear-powered spacecraft have sailed to Saturn. However, today, in 2018, the walk on the moon has become a distant memory.And the farthest reach of our manned space flights is just as long as the two-hour mileage of a high-speed train passing through my city. At the same time, information technology is developing at an unimaginable speed. With the entire world covered by the Internet, people have gradually lost their interest in space, as they find themselves increasingly comfortable in the space created by IT. Instead of an exploration of the real space, which is full of real difficulties, people now just prefer to experience virtual space through VR. Just like someone said, "You promised me an ocean of stars, but you actually gave me Facebook.""#</p>
        </DrawerBody>
    </OverlayDrawer>
}
```

This example effectively shows that you don't need to do any special handling
for oversized content within a Drawer.

The `<DrawerBody>`component is designed to be scrollable by default, which is a
common requirement for side panels that might display long lists, navigation
trees, or extensive details.


### OverlayDrawer Props

| Name            | Type                     | Default                  | Desciption                                  |
| --------------- | ------------------------ | ------------------------ | ------------------------------------------- |
| class           | `MaybeProp<String>`      | `Default::default()`     |                                             |
| container_class | `MaybeProp<String>`      | `Default::default()`     |                                             |
| open            | `Model<bool>`            |                          | Controls the open state of the Drawer.      |
| mask_closeable  | `Signal<bool>`           | `true`                   | Whether to emit hide event when click mask. |
| close_on_esc    | `bool`                   | `false`                  | Whether to close drawer on Esc is pressed.  |
| position        | `Signal<DrawerPosition>` | `DrawerPlacement::Left`  | Position of the drawer.                     |
| size            | `Signal<DrawerSize>`     | `DrawerSize::Small`      | Size of the drawer.                         |
| modal_type      | `DrawerModalType`        | `DrawerModalType::Modal` | Dialog variations.                          |
| children        | `Children`               |                          |                                             |

### InlineDrawer Props

| Name     | Type                     | Default                 | Desciption                             |
| -------- | ------------------------ | ----------------------- | -------------------------------------- |
| class    | `MaybeProp<String>`      | `Default::default()`    |                                        |
| open     | `Model<bool>`            |                         | Controls the open state of the Drawer. |
| position | `Signal<DrawerPosition>` | `DrawerPlacement::Left` | Position of the drawer.                |
| size     | `Signal<DrawerSize>`     | `DrawerSize::Small`     | Size of the drawer.                    |
| children | `Children`               |                         |                                        |

### DrawerHeader Props

| Name     | Type                | Default              | Description |
| -------- | ------------------- | -------------------- | ----------- |
| class    | `MaybeProp<String>` | `Default::default()` |             |
| children | `Children`          |                      |             |

### DrawerHeaderTitle Props

| Name                       | Type                                   | Default              | Description |
| -------------------------- | -------------------------------------- | -------------------- | ----------- |
| class                      | `MaybeProp<String>`                    | `Default::default()` |             |
| drawer_header_title_action | slot `Option<DrawerHeaderTitleAction>` | `None`               |             |
| children                   | `Children`                             |                      |             |

### DrawerHeaderTitleAction Props

| Name     | Type       | Default | Description |
| -------- | ---------- | ------- | ----------- |
| children | `Children` |         |             |

### DrawerBody Props

| Name             | Type                | Default              | Description                                |
| ---------------- | ------------------- | -------------------- | ------------------------------------------ |
| class            | `MaybeProp<String>` | `Default::default()` |                                            |
| native_scrollbar | `bool`              | `false`              | Whether to use native scrollbar on itself. |
| children         | `Children`          |                      |                                            |
