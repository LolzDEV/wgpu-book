# Wgpu initialization

First, we need to add some dependencies to our project

```toml
[dependencies]
winit = "0.26"
wgpu = "0.13"
pollster = "0.2"
```

As the [wgpu docs](https://docs.rs/wgpu/latest/wgpu/) suggest the first thing we need to do to interract with the wgpu crate is to crate an `Instance`.
Before doing so I'd like to better organize the code so that it's easier to understand what's going on by packing all the rendering related stuff into a structure.

```rust
struct Renderer {
    surface: wgpu::Surface,
    surface_config: wgpu::SurfaceConfiguration,
    device: wgpu::Device,
    queue: wgpu::Queue,
}
```

We're going to explain what those structs are more in detail later. Now we need a way to create our renderer.

```rust
impl Renderer {
    fn new(window: &Window) -> Self {
        let instance = Instance::new(Backends::all());

        ...
    }
}
```

And now we're ready to interact with `wgpu`!

As you might think, since we want to draw stuff on our window we need a _surface_ to draw on, linked to our window. That's exactly what we're going to create now

```rust
impl Renderer {
    fn new(window: &Window) -> Self {
        let instance = Instance::new(Backends::all());

        let surface = unsafe { instance.create_surface(window) };

        ...
    }
}
```

And we got our surface! As you can see the method used to create the surface is unsafe, that's because when using metal (macOS or iOS) if this is called outside the main thread it will crash.

Next we need to get access to our GPU since we want to use it to render stuff on our `Surface`

```rust
impl Renderer {
    fn new(window: &Window) -> Self {
        let instance = Instance::new(Backends::all());

        let surface = unsafe { instance.create_surface(window) };
        let adapter = pollster::block_on(instance.request_adapter(&wgpu::RequestAdapterOptions {
            power_preference: PowerPreference::default(),
            force_fallback_adapter: false,
            compatible_surface: Some(&surface),
        }))
        .unwrap();

        let (device, queue) = pollster::block_on(adapter.request_device(
            &wgpu::DeviceDescriptor {
                label: Some("Device"),
                features: Features::empty(),
                limits: Limits::default(),
            },
            None,
        )).unwrap();

        ...
    }
}
```

I know, that's a lot of code, so I'm going to explain what it does. First we are requesting the `Adapter` which is an handle for our physical GPU, the actual driver, then we are using the `Adapter` to request a `Device` which is the software representation of the GPU and a `Queue`. When we're telling the GPU to do something we are sending _commands_ to the GPU, so we use the `Queue` to store the commands and then send all of them to the GPU once we need to draw the frame. You also might notice that we are using the `pollster::block_on` function, this is because the adapter and device creation are _asyncronous_ so since we are in a syncronous contex we need a way to wait for them to complete.

So, what's next? Now we need to tell the surface how it should look like before start doing something on the GPU. We can easily to that with the `SurfaceConfiguration` structure.

```rust
impl Renderer {
    fn new(window: &Window) -> Self {
        ...

        let size = window.inner_size();
        let surface_config = SurfaceConfiguration {
            usage: TextureUsages::RENDER_ATTACHMENT,
            format: surface.get_supported_formats(&adapter)[0],
            width: size.width,
            height: size.height,
            present_mode: wgpu::PresentMode::Fifo,
        };

        surface.configure(&device, &surface_config);
        ...
    }
}
```

Here we go, first we are getting the window inner size (excluding window borders) in pixels and we pass the size into the surface configuration so that the surface is correctly resized, then we also tell the surface to use the first available format from the `Adapter` since as the docs say, the first format is the preferred one. We also specify that the usage for our texture (surface) is `RENDER_ATTACHMENT` that's because with this flag we are allowing our texture to be the output of a `RenderPass`, we will get into what the `RenderPass` is in the next chapter. Last, we define which mode the surface should use to present our frames, we choose `Fifo` (VSync). After our configuration is created we call `surface.configure` to update our surface with the new configuration.

Finally we can construct our `Renderer`

```rust
impl Renderer {
    fn new(window: &Window) -> Self {
        ...

        Self {
            surface,
            surface_config,
            device,
            queue,
        }
    }
}
```
