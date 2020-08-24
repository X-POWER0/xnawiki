# Drawing your first Triangle

This chapter will cover the basics of drawing. First a few things you should know about.

Every object drawn in 3D is drawn using triangles. Even spheres can be represented using triangles, if you use enough of them. Surprisingly enough, a triangle is defined by 3 points. Every point is defined by a vector, specifying the X, Y and Z coordinates of the point. However, just knowing the coordinates of a point might not be enough. For example, you might want to define a color for the given point as well. This is where a vertex (pl. vertices) comes in: it is the list of properties of a given point, including the position, color and so on.

XNA has a structure that fits perfectly to hold our vertex information: the VertexPositionColor struct. A vertex of this type can hold a position and a color, which is perfect to begin with. To define a triangle, we’ll need 3 of those vertices, which we will store in an array. So let’s declare this variable at the top of our class:

 VertexPositionColor[] vertices;

Next, we will add a small method to our code, SetUpVertices, which will fill this array with 3 vertices:

 private void SetUpVertices()
 {
     vertices = new VertexPositionColor[3];
 
     vertices[0].Position = new Vector3(-0.5f, -0.5f, 0f);
     vertices[0].Color = Color.Red;
     vertices[1].Position = new Vector3(0, 0.5f, 0f);
     vertices[1].Color = Color.Green;
     vertices[2].Position = new Vector3(0.5f, -0.5f, 0f);
     vertices[2].Color = Color.Yellow;
 }

The array is initialized to hold 3 vertices, after which it is filled. For now, we’re using coordinates that are relative to the screen: the (0,0,0) point would be the middle of our screen, the (-1, -1, 0) point bottom-left and the (1, 1, 0) point top-right. So in the example above, the first point is halfway to the bottom left of the window, and the second point is halfway to the top in the middle of our screen. (As they are not really 3D coordinates, they don’t need to be transformed to 2D coords. Hence, the name of the technique: ‘Pretransformed’)

The 'f' behind some of the numbers indicates the values are floats, the format of preference when working with XNA. We set each of the vertices to different colors.
We still need to call this SetUpVertices method. As it uses the device, call it at the end of the LoadContent method:

```csharp
 SetUpVertices();
```

All we have to do now is tell the device to draw the triangle! Go to our Draw method, where we should draw the triangle after the call to pass.Apply:

```csharp
 device.DrawUserPrimitives(PrimitiveType.TriangleList, vertices, 0, 1, VertexPositionColor.VertexDeclaration);
```

This line actually tells our graphics card to draw the triangle: we want to draw 1 triangle from the vertices array, starting at vertex 0. TriangleList means that our vertices array contains a list of triangles (in our case, a list of only 1 triangle). If you would want to draw 4 triangles, you would need an array of 12 vertices. Another possibility is to use a TriangleStrip, which can perform a lot faster, but is only useful to draw triangles that are connected to each other. More on all kinds of primitives in Recipe 5-1.

The last moment specifies the VertexDeclaration, which is quite important. We have stored the position and color data for 3 vertices inside an array. When we instruct XNA to render a triangle based on this data, XNA will put all this data in a byte stream and send it over to the graphics card. Our graphics card receives the byte stream, but doesn’t know what’s in there! That’s where the VertexDeclaration comes in: this object tells the graphics device what kind of vertices it can expect.

The VertexDeclaration is very important, as you will always need it before you can render any triangle to the screen. By specifying the VertexPositionColor.VertexDeclaration, we inform the graphics card that there are vertices coming its way that contain Position and Color data. We’ll see how we can create our own vertex types later on in this series.

Running this code should give you this window:

// Current Status

That's all there is to it! Running this code will already give you a colorful triangle on a blue background. Feel free to experiment with the colors and the coordinates.

> You can try these exercises to practice what you've learned:
>
> - Make the bottom-left corner of the triangle collide with the bottom-left corner of your window.
> - Render 2 triangles, covering your whole window.

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
         GraphicsDeviceManager graphics;
         SpriteBatch spriteBatch;
         GraphicsDevice device;
 
         Effect effect;
         VertexPositionColor[] vertices;
 
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
 
             SetUpVertices();
         }
 
         protected override void UnloadContent()
         {
         }
 
         private void SetUpVertices()
         {
             vertices = new VertexPositionColor[3];
 
             vertices[0].Position = new Vector3(-0.5f, -0.5f, 0f);
             vertices[0].Color = Color.Red;
             vertices[1].Position = new Vector3(0, 0.5f, 0f);
             vertices[1].Color = Color.Green;
             vertices[2].Position = new Vector3(0.5f, -0.5f, 0f);
             vertices[2].Color = Color.Yellow;
         }
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(Color.DarkSlateBlue);
 
             effect.CurrentTechnique = effect.Techniques["Pretransformed"];
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.DrawUserPrimitives(PrimitiveType.TriangleList, vertices, 0, 1, VertexPositionColor.VertexDeclaration);
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[World Space coordinates and the camera](Riemers3DXNA1Terrain04worldspace)
