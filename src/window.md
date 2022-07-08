# Window Creation

As you might know, to draw graphics on the screen we first need a window. Most modern operating systems let programmers use some APIs to interact with their windowing system. Since we want to target the majority of the operating systems out there we'll use a cross-platform crate: [`winit`](https://docs.rs/winit).

First we need to add winit as a dependency for our project:

```toml
[dependencies]
winit = "0.26"
```

Now let's dive into the source code. We need to create the `EventLoop`

```rust
use winit::event_loop::EventLoop;

fn main() {
    let event_loop = EventLoop::new();
}
```

You may be wondering _what is an `EventLoop`?_. Basically, when we create a window the windowing system starts sending events to our program such as when a key is pressed or when the window is resized so that we can handle that easily. So we need to run a loop to receive these events and then choose what to do with them.

After we got our `EventLoop` we need the actual window, we can create the window by using a `WindowBuilder`

```rust
use winit::{event_loop::EventLoop, window::WindowBuilder};

fn main() {
    let event_loop = EventLoop::new();
    let window = WindowBuilder::new().with_title("wgpu").build(&event_loop).unwrap();
}
```

And we're done! Now we can just start the `EventLoop` and handle events for our window!

```rust
use winit::{event_loop::{EventLoop, ControlFlow}, window::WindowBuilder, event::WindowEvent};

fn main() {
    let event_loop = EventLoop::new();
    let window = WindowBuilder::new().with_title("wgpu").build(&event_loop).unwrap();

    event_loop.run(move |event, _, control_flow| {
        *control_flow = ControlFlow::Poll;

        match event {
            winit::event::Event::WindowEvent { window_id, event } => if window_id == window.id() {
                match event {
                    WindowEvent::CloseRequested => *control_flow = ControlFlow::Exit,
                    _ => ()
                }
            },
            _ => ()
        }
    });
}
```

As you can see we're using the `ControlFlow` to tell the loop what to do next, with `ControlFlow::Poll` we're telling the loop to jump to the next iteration just once the current one finishes, and we're also using pattern matching to tell the loop to stop when the window should be closed (for example when the button to close the window is pressed).

>**Note:** On wayland systems you are not going to see a window until there is actual content to be displayed so don't worry if you don't see anyhing yet.
