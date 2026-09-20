![ratatui-hypertile demo](assets/showcase.gif)

[![CI](https://github.com/nikolic-milos/ratatui-hypertile/actions/workflows/ci.yml/badge.svg)](https://github.com/nikolic-milos/ratatui-hypertile/actions/workflows/ci.yml)
[![Crates.io](https://img.shields.io/crates/v/ratatui-hypertile.svg)](https://crates.io/crates/ratatui-hypertile)
[![Docs.rs](https://docs.rs/ratatui-hypertile/badge.svg)](https://docs.rs/ratatui-hypertile)

Cook up delicious terminal interfaces with Hyprland-style tiling for [Ratatui](https://github.com/ratatui/ratatui). Tile your panes, switch between tabs, drag borders and whole panes with the mouse, and watch them glide into place. Save the layout for when you want it back.

## Two crates to tile them all

[`ratatui-hypertile`](https://crates.io/crates/ratatui-hypertile) is the core engine. You give it an area, it gives you rectangles. It tracks the tree, focus, and movement, and otherwise stays out of your way. Reach for this when you want full control.

[`ratatui-hypertile-extras`](https://crates.io/crates/ratatui-hypertile-extras) wraps the core in a ready-to-go runtime: plugins, vim keymaps, a command palette, workspace tabs, and pane-move animations. Implement `HypertilePlugin` and you're set.

## Try it out

From the repo root:

```sh
cargo run -p ratatui-hypertile-extras --example basic
cargo run --example core_only
```

## Quickstart

Add one (or both) to your `Cargo.toml`:

```toml
ratatui-hypertile = "0.4"
ratatui-hypertile-extras = "0.4"
```

```rust
use ratatui::layout::{Direction, Rect};
use ratatui_hypertile::Hypertile;

let mut layout = Hypertile::new();
layout.split_focused(Direction::Horizontal)?;
layout.compute_layout(Rect::new(0, 0, 80, 24));

for pane in layout.panes_iter() {
    draw_pane(pane.rect, pane.is_focused);
}
```

## Application-owned palette choices

The extras runtime normally offers every registered plugin except its built-in
placeholder, then splits or replaces a pane when you confirm. Applications with
singleton panes or hidden plugin types can opt into a curated palette:

```rust
use ratatui_hypertile_extras::{HypertileRuntime, PaletteBehavior, PaletteConfig};

let mut runtime = HypertileRuntime::builder()
    .with_palette_config(PaletteConfig {
        allowed_plugins: Some(vec!["logs".into(), "inspector".into()]),
        behavior: PaletteBehavior::EmitSelection,
    })
    .build();
```

After registering those plugins, call `open_palette()`. While `is_palette_open()`
is true, route input through the runtime before your application's shortcuts.
After handling input, `take_palette_selection()` returns the confirmed plugin
name once, letting your app focus an existing pane or create one itself. No pane
is created by `open_palette()` or by confirming in `EmitSelection` mode.

Split shortcuts using `PromptPalette` still create a placeholder first; the
selection's `target_pane` identifies it. Your app decides what to do with that
pane. `close_palette()` and Escape dismiss the palette without removing it.
Reopening the palette or calling `set_palette_config()` clears any unclaimed
selection. An empty allowlist disables the palette, and unknown names are ignored.

`render()` includes the palette. If your app draws content outside the plugin
registry afterward, call `render_palette(area, buffer)` last to keep the palette
on top. It does nothing while closed.

## FAQ

**Why not just use tmux or Zellij?**

They solve a different problem. tmux and Zellij are multiplexers: they tile whole programs in your terminal. ratatui-hypertile is a library that adds tiling inside a single Ratatui app, so the panes your app draws can split, focus, and resize. The two are not mutually exclusive. You can run a hypertile app inside tmux or Zellij just fine.

## License

This project is licensed under the [MIT License](LICENSE).
