# Creating the shader

First, we need to add the pipeline as a member of our `Renderer`

```rust
struct Renderer {
    surface: wgpu::Surface,
    surface_config: wgpu::SurfaceConfiguration,
    device: wgpu::Device,
    queue: wgpu::Queue,
    render_pipeline: wgpu::RenderPipeline, // NEW
}
```

Next we need to create the shader module for our pipeline, let's create a new file

_src/shader.wgsl_

```wgsl
struct VertexInput {
    @location(0) pos: vec3<f32>,
}
```

Here we're defining the input for our vertex stage, with the `@location(0)` attribute we're telling the shader where this position should be taken from, and since we are using this as input for our vertex stage the value will be taken from the first position (0) in the vertex input (we will define the input in rust later on).

We also need to define an output for our stage

_src/shader.wgsl_

```wgsl
...

struct VertexOutput {
    @builtin(position) pos: vec4<f32>
}
```

Now we're using another attribte, `@builtin`. This attribute is used to speicfy that the value is one of the builtin values of wgsl.

_src/shader.wgsl_
```wgsl
...

@vertex
fn vertex_main(in: VertexInput) -> VertexOutput {
    var out: VertexOutput;

    out.pos = vec4<f32>(in.pos, 1.0);

    return out;
}
```

And finally we have our vertex stage defined, it will just take the position from the input and copy it in the output. With the `@vertex` attribute we're specifying that this part of the shader is in the vertex stage.

Now that we have our vertex shader we need to create a **fragment shader**. As we said in the previous chapter, the fragment shader is needed to give a color to the pixels that compose the triangle we are drawing. In this example we're creating a fragment shader which has a fixed red color for every pixel we draw.

_src/shader.wgsl_
```wgsl
...

@fragment
fn fragment_main(in: VertexOutput) -> @location(0) vec4<f32> {
    return vec4<f32>(1.0, 0.0, 0.0, 1.0);
}
```

This is pretty easy, we're taking the output from the previous stage (vertex) as input for the fragment stage and we're returning the color red in RGBA.