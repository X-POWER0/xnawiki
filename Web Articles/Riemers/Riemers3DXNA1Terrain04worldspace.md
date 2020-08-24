# World Space coordinates and the camera

Last chapter we drew a triangle, using 'pretransformed' coordinates. These coordinates allow you to directly specify their position on the screen. However, you will usually use the untransformed coordinates, the so called World space coordinates, which we specify in 3D coordinates. These allow you to create a whole scene using simple 3D coordinates, and, also very important, to position a camera through which the user will look at the scene.

Let’s start by redefining our triangle coordinates in 3D World space. Replace the code in our SetUpVertices method with this code:

```csharp
 private void SetUpVertices()
 {
     vertices = new VertexPositionColor[3];

     vertices[0].Position = new Vector3(0f, 0f, 0f);
     vertices[0].Color = Color.Red;
     vertices[1].Position = new Vector3(10f, 10f, 0f);
     vertices[1].Color = Color.Yellow;
     vertices[2].Position = new Vector3(10f, 0f, -5f);
     vertices[2].Color = Color.Green;
 }
```

As you can see, from here on we’ll be using the Z-coordinate as well.

Because we are no longer using pretransformed screen coordinates (where x and y coordinate should be in the [-1, 1] region), we need to select a different technique from our effect file. I called the technique ‘ColoredNoShading’, to reflect that we’ll be rendering a Colored 3D image, but did not yet specify any lighting/shading information. So make this adjustment in our Draw method:

```csharp
 effect.CurrentTechnique = effect.Techniques["ColoredNoShading"];
```

Let's run this code.

Very nice, your triangle has disappeared again. Why's that? Easy, because you haven't told XNA yet where to position the camera in your 3D World, and where to look at!

To position our camera, we need to define some matrices. Stop!! matrices?!?

First a small word about matrices. We define our points in 3D space. Because our screen is 2D, it is only logical that our 3D points somehow need to be ‘transformed’ to 2D space. This is done by multiplying our 3D positions with a matrix. So in short, you should see a matrix simply as a mathematical element that holds a transformation. If you multiply a 3D position with such a matrix, you get the transformed position. (If you want to know more about matrices, you can find more info in the Extra Reading section of this site)

Because there are a lot of properties that need to be defined when transforming our points from 3D world space to our 2D screen, this transformation is split in 2 steps, so we get 2 matrices. First add these variables to the top of your class:

```csharp
 Matrix viewMatrix;
 Matrix projectionMatrix;
```

To initialize them ,add this code to our program:

```csharp
 private void SetUpCamera()
 {
     viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, 50), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
     projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 300.0f);
 }
```

These 2 lines seem complicated, but all they do is defining the position and the lens of the camera.

The first line creates a matrix that stores the position and orientation of the camera, through which we look at the scene. The first argument defines the position of the camera. We position it 50 units on the positive Z axis. The next parameter sets the target point the camera is looking at. We will be looking at our (0,0,0) 3D origin. At this point, we have defined the viewing axis of our camera, but we can still rotate our camera around this axe. So we still need to define which vector will be considered as 'up'.

The second line creates a matrix which stores how the camera looks at the scene, much like defining the lens if you will. The first argument sets the view angle, 45 degrees in our case. Then we set the view aspect ratio, the ratio between the width and height of your screen. In our case of a 500x500 window this will equal 1, but this will be different for other resolutions. The last parameters define the view range. Any objects closer to the camera than 1f will not be shown. Any object further away than 300.0f won't be shown either. These distances are called the near and the far clipping planes, since all objects not between these planes will be clipped (=not drawn).

> Recipe 2-1 explains how to create a camera in much more detail.

Now we have these matrices, we need to pass it to our technique, where they will be combined. This is done by the next lines of code, which we need to add to our Draw method:

```csharp
 effect.Parameters["xView"].SetValue(viewMatrix);
 effect.Parameters["xProjection"].SetValue(projectionMatrix);
 effect.Parameters["xWorld"].SetValue(Matrix.Identity);
```

Although the first 2 lines are explained above, they are discussed in much more detail in Series 3. The third line sets another parameter, which is discussed in the next chapter.

Don’t forget to call this method from within the LoadContent method:

```csharp
 SetUpCamera();
```

Now run the code. You should see the image below: a triangle, of which the bottom-right corner is not exactly below the top-right corner. This is because you have assigned the bottom-right corner a negative Z coordinate, positioning it a bit further away from the camera than the other corners.

One important thing you should notice before you start experimenting: you'll see the green corner of the triangle is on the right side of the window, which seems normal because you defined it on the positive x-axis. So, if you would position your camera on the negative z-axis:

```csharp
 viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, -50), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
```

you would expect to see the green point in the left half of the window. Try to run this now.

This might not be exactly what you expected. Something very important has happened. XNA only draws triangles that are facing the camera. XNA specifies that triangles facing the camera should be defined clockwise relative to the camera. If you position the camera on the negative z-axis, the corner points of the triangle in our vertices array will be defined counter-clockwise relative to the camera, and thus will not be drawn!

> See Recipe 5-6 for a more detailed explanation on backface culling and some examples.

Culling can greatly improve performance, as it can reduce the number of triangles to be drawn. However, when designing an application, it’s better to turn culling off by putting these lines of code in the beginning of your Draw method:

```csharp
 RasterizerState rs = new RasterizerState();
 rs.CullMode = CullMode.None;
 device.RasterizerState = rs;
```

This will simply draw all triangles, even those not facing the camera. You should note that this should never be done in a final product, because it slows down the drawing process, as all triangles will be drawn, even those not facing the camera! (Except for some cases where you intend to achieve this effect). Now put the camera back to the positive part of the Z axis.

// Current Status

> You can try these exercises to practice what you've learned:
>
> - Experiment with different positions for your 3D vertices and camera position.

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
             vertices = new VertexPositionColor[3];
 
             vertices[0].Position = new Vector3(0f, 0f, 0f);
             vertices[0].Color = Color.Red;
             vertices[1].Position = new Vector3(10f, 10f, 0f);
             vertices[1].Color = Color.Yellow;
             vertices[2].Position = new Vector3(10f, 0f, -5f);
             vertices[2].Color = Color.Green;
         }
 
         private void SetUpCamera()
         {
             viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, 50), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
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
             device.Clear(Color.DarkSlateBlue);
 
             RasterizerState rs = new RasterizerState();
             rs.CullMode = CullMode.None;
             device.RasterizerState = rs;
 
             effect.CurrentTechnique = effect.Techniques["ColoredNoShading"];
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
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

[Rotations and translations](Riemers3DXNA1Terrain05rotation)
