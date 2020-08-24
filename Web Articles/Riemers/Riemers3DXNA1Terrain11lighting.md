# Experimenting with Lights in XNA

Even when using colors and a Z buffer, your terrain seems to miss some depth detail when you turn on the Solid FillMode. By adding some lighting, it will look much better. This chapter we will see the impact of a light on 2 simple triangles, so we can have a better understanding of how lights work in XNA. We will be using the code from the ‘World space’ chapter, so reload that code now.

This chapter, we will be using a directional light. Imagine this as the sunlight: the light will travel in one particular direction. To calculate the effect of light hitting a triangle, XNA needs another input: the 'normal' in every vertex. Consider next figure:

![Normals](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-11Lighting1.jpg?raw=true)

If you have a light source a), and you shine it on the shown 3 surfaces, how is XNA supposed to know that surface 1 should be lit more intensely than surface 3? If you look at the thin red lines in figure b), you'll notice that their length is a nice indication of how much light you would want to be reflected (and thus seen) on every surface. So how can we calculate the length of these lines? Actually, XNA does the job for us. All we have to do is give the blue arrow perpendicular (with an angle of 90 degrees, the thin blue lines) to every surface and XNA does the rest (a simple cosine projection) for us!

This is why we need to add normals (the perpendicular blue lines) to our vertex data. The VertexPositionColor will no longer do as it does not allow us to store a normal for each vertex, and unfortunately, XNA does not offer a structure that can contain a position, a color and a normal. But that’s no problem, we can easily create one of our own. Let’s put this code at the top of our class, immediately above our variable declarations:

```csharp
 public struct VertexPositionColorNormal
 {
     public Vector3 Position;
     public Color Color;
     public Vector3 Normal;
 
     public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
     (
         new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
         new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0),
         new VertexElement(sizeof(float) * 3 + 4, VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
     );
 }
```

This might look complicated, but I’m sure you understand the first 3 lines: this new struct can hold a postion, a color and a normal; exactly what we need! The bottom of the struct is a bit more complex, and we’ll discuss it in its full detail in Series 3. For now, think of it as a manual for the graphics card to understand what kind of data is contained inside each vertex.

This allows us to change our vertex variable declaration:

```csharp
 VertexPositionColorNormal[] vertices;
```

Now we could start defining triangles with normals. But first, let’s have a look at the next picture, where the arrows at the top represent the direction of the light and the color bar below the drawing represents the color of every pixel along our surface:

![Surfaces](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-11Lighting2.jpg?raw=true)

If we simply define the perpendicular vectors, it is easy to see there will be an 'edge' in the lighting (see the bar directly above the ‘a’)). This is because the right surface is lit more than the left surface. So it will be easy to see the surface is made of separate triangles. However, if we place in the shared top vertex a 'normal' as shown in figure b), XNA automatically interpolates the lighting in every point of our surface! This will give a much smoother effect, as you can see in the bar above the b). This vector is the average of the 2 top vectors of a).

As always, the average of 2 vectors can be found by summing them and by dividing them by two.

To demonstrate this example, we will first reset the camera position:

```csharp
 viewMatrix = Matrix.CreateLookAt(new Vector3(0, -40, 100), new Vector3(0, 50, 0), new Vector3(0, 1, 0));
```

Next, we will update our SetUpVertices method with 6 vertices that define the 2 triangles of the example above:

```csharp
 private void SetUpVertices()
 {
     vertices = new VertexPositionColorNormal[6];

     vertices[0].Position = new Vector3(0f, 0f, 50f);
     vertices[0].Color = Color.Blue;
     vertices[0].Normal = new Vector3(1, 0, 1);
     vertices[1].Position = new Vector3(50f, 0f, 00f);
     vertices[1].Color = Color.Blue;
     vertices[1].Normal = new Vector3(1, 0, 1);
     vertices[2].Position = new Vector3(0f, 50f, 50f);
     vertices[2].Color = Color.Blue;
     vertices[2].Normal = new Vector3(1, 0, 1);

     vertices[3].Position = new Vector3(-50f, 0f, 0f);
     vertices[3].Color = Color.Blue;
     vertices[3].Normal = new Vector3(-1, 0, 1);
     vertices[4].Position = new Vector3(0f, 0f, 50f);
     vertices[4].Color = Color.Blue;
     vertices[4].Normal = new Vector3(-1, 0, 1);
     vertices[5].Position = new Vector3(0f, 50f, 50f);
     vertices[5].Color = Color.Blue;
     vertices[5].Normal = new Vector3(-1, 0, 1);

     for (int i = 0; i < vertices.Length; i++)
         vertices[i].Normal.Normalize();
 }
```

This defines the 2 surfaces of the picture above. By adding a Z coordinate (other than 0), the triangles are now 3D. You can notice that I’ve defined the normal vectors perpendicular to the triangles, as to reflect example a) of the image above.

At the end, we ‘normalize our normals’ . These 2 words have absolutely nothing to do with each other. It means we scale our normal vectors so their lengths become exactly one. This is required for correct lighting, as the length of the normals has an impact on the amount of lighting.

All there's left to do is change the Draw method a bit:

```csharp
 device.DrawUserPrimitives(PrimitiveType.TriangleList, vertices, 0, 2, VertexPositionColorNormal.VertexDeclaration);
```

And let the graphics card know that from now on it has to use the Colored technique. This technique works exactly the same as the ColoredNoShading technique, but also adds correct lighting (in case you provided correct normals!):

```csharp
 effect.CurrentTechnique = effect.Techniques["Colored"];
```

When you run this code, you should see an arrow (our 2 triangles), but you don’t see any shading because we haven’t yet defined the light! We can define this by setting these additional parameters of our effect:

```csharp
 effect.Parameters["xEnableLighting"].SetValue(true);
 Vector3 lightDirection = new Vector3(0.5f, 0, -1.0f);
 lightDirection.Normalize();
 effect.Parameters["xLightDirection"].SetValue(lightDirection);
```

This instructs our technique to enable lighting calculations (now the technique needs the normals), and we set the direction of our light. Note again that you need to normalize this vector, so its lengths becomes one (otherwise the length of this vector influences the strength of the shading, while you want the shading to depend solely on the direction of the incoming light). You might also want to change the background color to black, so you get a better view.

Now run this code and you'll see what I mean with 'edged lighting': the light shines brightly on the left panel and the right panel is darker. You can clearly see the difference between the two triangles! This is what was shown in the left part of the example image above.

Now it's time to combine the vectors on the edge that is shared by the 2 triangles from (-1,0,1) and (1,0,1) to (-1+1,0,1+1)/2 = (0,0,1):

```csharp
 vertices[0].Position = new Vector3(0f, 0f, 50f);
 vertices[0].Color = Color.Blue;
 vertices[0].Normal = new Vector3(0, 0, 1);
 vertices[1].Position = new Vector3(50f, 0f, 00f);
 vertices[1].Color = Color.Blue;
 vertices[1].Normal = new Vector3(1, 0, 1);
 vertices[2].Position = new Vector3(0f, 50f, 50f);
 vertices[2].Color = Color.Blue;
 vertices[2].Normal = new Vector3(0, 0, 1);
 
 vertices[3].Position = new Vector3(-50f, 0f, 0f);
 vertices[3].Color = Color.Blue;
 vertices[3].Normal = new Vector3(-1, 0, 1);
 vertices[4].Position = new Vector3(0f, 0f, 50f);
 vertices[4].Color = Color.Blue;
 vertices[4].Normal = new Vector3(0, 0, 1);
 vertices[5].Position = new Vector3(0f, 50f, 50f);
 vertices[5].Color = Color.Blue;
 vertices[5].Normal = new Vector3(0, 0, 1);
```

When you run this code, you'll see that the reflection is nicely distributed from the dark right tip to the brighter left panel. It's not difficult to imagine that this effect will give a much nicer and smoother effect on a large number of triangles, such as our terrain.

// Current Status

Also, you can see that since the middle vertices use the SAME normal, we could again combine the 2x2 shared vertices to 2x1 vertices using an index buffer. Notice however, that in the case you really WANT to create an edge, you need to specify the 2 separate normal vectors.

> You can try these exercises to practice what you've learned:
>
> - Try moving the normals a bit to the left and right in both the separate and shared case.

## Code so far

```csharp
 using System;
 using System.Collections.Generic;
 using System.Linq;
 using Microsoft.Xna.Framework;
 using Microsoft.Xna.Framework.Audio;
 using Microsoft.Xna.Framework.Content;
 using Microsoft.Xna.Framework.GamerServices;
 using Microsoft.Xna.Framework.Graphics;
 using Microsoft.Xna.Framework.Input;
 using Microsoft.Xna.Framework.Media;
 
 namespace Series3D1
 {
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         public struct VertexPositionColorNormal
         {
             public Vector3 Position;
             public Color Color;
             public Vector3 Normal;
 
             public readonly static VertexDeclaration VertexDeclaration = new VertexDeclaration
             (
                 new VertexElement(0, VertexElementFormat.Vector3, VertexElementUsage.Position, 0),
                 new VertexElement(sizeof(float) * 3, VertexElementFormat.Color, VertexElementUsage.Color, 0),
                 new VertexElement(sizeof(float) * 3 + 4, VertexElementFormat.Vector3, VertexElementUsage.Normal, 0)
             );
         }
 
         GraphicsDeviceManager graphics;
         SpriteBatch spriteBatch;
         GraphicsDevice device;
 
         Effect effect;
         VertexPositionColorNormal[] vertices;
 
         Matrix viewMatrix;
         Matrix projectionMatrix;
 
 
         public Game1()
         {
             graphics = new GraphicsDeviceManager(this);
             Content.RootDirectory = "Content";
         }
 
         protected override void Initialize()
         {
             graphics.PreferredBackBufferWidth = 500;
             graphics.PreferredBackBufferHeight = 500;
             graphics.IsFullScreen = false;
             graphics.ApplyChanges();
             Window.Title = "Riemer's XNA Tutorials -- 3D Series 1";
 
             base.Initialize();
         }
 
         protected override void LoadContent()
         {
             spriteBatch = new SpriteBatch(GraphicsDevice);
 
             device = graphics.GraphicsDevice;

            effect = Content.Load<Effect> ("effects");
            SetUpCamera();
            SetUpVertices();
        }

        protected override void UnloadContent()
        {
        }

        private void SetUpVertices()
        {

             vertices = new VertexPositionColorNormal[6];
 
             vertices[0].Position = new Vector3(0f, 0f, 50f);
             vertices[0].Color = Color.Blue;
             vertices[0].Normal = new Vector3(0, 0, 1);
             vertices[1].Position = new Vector3(50f, 0f, 00f);
             vertices[1].Color = Color.Blue;
             vertices[1].Normal = new Vector3(1, 0, 1);
             vertices[2].Position = new Vector3(0f, 50f, 50f);
             vertices[2].Color = Color.Blue;
             vertices[2].Normal = new Vector3(0, 0, 1);
 
             vertices[3].Position = new Vector3(-50f, 0f, 0f);
             vertices[3].Color = Color.Blue;
             vertices[3].Normal = new Vector3(-1, 0, 1);
             vertices[4].Position = new Vector3(0f, 0f, 50f);
             vertices[4].Color = Color.Blue;
             vertices[4].Normal = new Vector3(0, 0, 1);
             vertices[5].Position = new Vector3(0f, 50f, 50f);
             vertices[5].Color = Color.Blue;
             vertices[5].Normal = new Vector3(0, 0, 1);
 
             for (int i = 0; i < vertices.Length; i++)
                 vertices[i].Normal.Normalize();
         }
 
         private void SetUpCamera()
         {
             viewMatrix = Matrix.CreateLookAt(new Vector3(0, -40, 100), new Vector3(0, 50, 0), new Vector3(0, 1, 0));
             projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 300.0f);
         }
 
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(Color.Black);
 
             RasterizerState rs = new RasterizerState();
             rs.CullMode = CullMode.None;
             device.RasterizerState = rs;
 
             effect.CurrentTechnique = effect.Techniques["Colored"];
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
             effect.Parameters["xEnableLighting"].SetValue(true);
             Vector3 lightDirection = new Vector3(0.5f, 0, -1.0f);
             lightDirection.Normalize();
             effect.Parameters["xLightDirection"].SetValue(lightDirection);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.DrawUserPrimitives(PrimitiveType.TriangleList, vertices, 0, 2, VertexPositionColorNormal.VertexDeclaration);
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[Adding normals to our Terrain – Intuitive approach](Riemers3DXNA1Terrain12terrainlighting)
