# Recycling vertices using inidices

The triangle was nice, but what about a lot of triangles? We would need to specify 3 vertices for each triangle. Consider next example:

![Triangles](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-6indices1.jpg?raw=true)

Only 4 out of 6 vertices are unique. So the other 2 are simply a waste of bandwidth to your graphics card! It would be better to define the 4 vertices in an array from 0 to 3, and to define triangle 1 as vertices 1,2 and 3 and triangle 2 as vertices 2,3 and 4. This way, the complex vertex data is not duplicated. This is exactly the idea behind indices. Suppose we would like to draw these 2 triangles :

![Triangles](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-6indices2.jpg?raw=true)

Normally we would have to define 6 vertices, now we will define only 5. So change our SetUpVertices method as follows:

```csharp
 private void SetUpVertices()
 {
     vertices = new VertexPositionColor[5];
 
     vertices[0].Position = new Vector3(0f, 0f, 0f);
     vertices[0].Color = Color.White;
     vertices[1].Position = new Vector3(5f, 0f, 0f);
     vertices[1].Color = Color.White;
     vertices[2].Position = new Vector3(10f, 0f, 0f);
     vertices[2].Color = Color.White;
     vertices[3].Position = new Vector3(5f, 0f, -5f);
     vertices[3].Color = Color.White;
     vertices[4].Position = new Vector3(10f, 0f, -5f);
     vertices[4].Color = Color.White;
 }
```

Vertices 0 to 2 are positioned on the positive X axis. Vertices 3 and 4 have a negative Z component, as XNA considers the negative Z axis to be ‘Forward’. As your vertices are defined along the Right and the Forward directions, the resulting triangles will be lying flat on the ground.

Next, we will create a list of indices. As discussed above, the indices refer to vertices in our array of vertices. The indices define the triangles, so for 2 triangles we will need to define 6 indices. Start by defining the array at the top of your class. Since indices are integer numbers, you will define an array capable of storing ints:

```csharp
 int[] indices;
```

Let’s create another small method that fills the array of indices.

```csharp
 private void SetUpIndices()
 {
     indices = new int[6];
 
     indices[0] = 3;
     indices[1] = 1;
     indices[2] = 0;
     indices[3] = 4;
     indices[4] = 2;
     indices[5] = 1;
 }
```

As you can see, this method defines 6 indices, defining 2 triangles. Vertex number 1 is referred to twice, which was our initial goal as you can see in the image above. In this case, the profit is rather small, but in bigger applications (as you will see soon ;) this is the way to go. Also note that the triangles have been defined in a clockwise order again, so XNA will see them as facing the camera and will not cull them away.

Make sure to call this method from our LoadContent method :

```csharp
 SetUpIndices();
```

All that's left for this chapter is to draw the triangles from our buffer! Change the following line in your Draw method:

```csharp
 device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, vertices, 0, vertices.Length, indices, 0, indices.Length / 3, VertexPositionColor.VertexDeclaration);
```

Instead of using the DrawUserPrimitives method, this time we call the DrawUserIndexedPrimitives method. This allows us to specify both an array of vertices and an array of indices. The second-last argument specifies how many triangles are defined by the indices. Since one triangle is defined by 3 indices, we specify the number of indices divided by 3.

Before your try this code, make your triangles stop rotating by resetting their World matrix to the unity matrix. The Identity matrix is the unity matrix, so your original World space coordinates will be used.

```csharp
 Matrix worldMatrix = Matrix.Identity;
```

That's it! When you run the program, however, there will be not too much to see. This is because both your triangles and your camera are positioned on the floor! We’ll have to reposition our camera so the triangles are in sight of the camera. So go to our SetUpCamera method, and change the position of the camera so it is positioned above our (0,0,0) 3D origin:

```csharp
 viewMatrix = Matrix.CreateLookAt(new Vector3(0, 50, 0), new Vector3(0, 0, 0), new Vector3(0, 0, -1));
```

We've positioned our camera 50 units above the (0,0,0) 3D origin, as the Y axis is considered as Up axis by XNA. However, because the camera is looking down, you can no longer specify the (0,1,0) Up vector as Up vector for the camera! Therefore, you specify the (0,0,-1) Forward vector as Up vector for the camera.

> See Recipe 2-1 for more information about the View matrix.

Now when you run this code, you should see both triangles, but they’re still solid. Try changing this property to your renderstates:

```csharp
 rs.FillMode = FillMode.WireFrame;
```

This will only draw the edges of our triangles, instead of solid triangles.

// Current Status

If this chapter or next chapter doesn’t render any triangles to your window, chances are your graphics card isn’t capable of rendering from many indices. To solve this, store your indices as shorts instead of ints. Define your array like this:

```csharp
 short[] indices;
```

And in your SetUpIndices method, initialize it like this:

```csharp
 indices = new short[6];
```

This should work, even on pcs with lower-end graphics cards.

The benefit of using indices in this example is not very big: we’re passing 5 vertices instead of 6. In the next chapter, however, the benefit will be a lot larger.

You can try these exercises to practice what you've learned:
Try to render the triangle that connects vertices 1, 3 and 4. The only changes you need to make are in the SetUpIndices method.
Use another vector, such as (1,0,0) as Up vector for your camera’s View matrix.
Here's the whole code so far:

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
         int[] indices;
 
         private float angle = 0f;
 
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

            effect = Content.Load<Effect> ("effects");            SetUpCamera();

            SetUpVertices();

             SetUpIndices();
         }
 
         protected override void UnloadContent()
         {
         }
 
         private void SetUpVertices()
         {
             vertices = new VertexPositionColor[5];
 
             vertices[0].Position = new Vector3(0f, 0f, 0f);
             vertices[0].Color = Color.White;
             vertices[1].Position = new Vector3(5f, 0f, 0f);
             vertices[1].Color = Color.White;
             vertices[2].Position = new Vector3(10f, 0f, 0f);
             vertices[2].Color = Color.White;
             vertices[3].Position = new Vector3(5f, 0f, -5f);
             vertices[3].Color = Color.White;
             vertices[4].Position = new Vector3(10f, 0f, -5f);
             vertices[4].Color = Color.White;
         }
 
         private void SetUpIndices()
         {
             indices = new int[6];
 
             indices[0] = 3;
             indices[1] = 1;
             indices[2] = 0;
             indices[3] = 4;
             indices[4] = 2;
             indices[5] = 1;
         }
 
         private void SetUpCamera()
         {
             viewMatrix = Matrix.CreateLookAt(new Vector3(0, 50, 0), new Vector3(0, 0, 0), new Vector3(0, 0, -1));
             projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 300.0f);
         }
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
 
             angle += 0.005f;
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(Color.DarkSlateBlue);
 
             RasterizerState rs = new RasterizerState();
             rs.CullMode = CullMode.None;
             rs.FillMode = FillMode.WireFrame;
             device.RasterizerState = rs;
 
             effect.CurrentTechnique = effect.Techniques["ColoredNoShading"];
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             Matrix worldMatrix = Matrix.Identity;
             effect.Parameters["xWorld"].SetValue(worldMatrix);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, vertices, 0, vertices.Length, indices, 0, indices.Length / 3, VertexPositionColor.VertexDeclaration);
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[Terrain creation basics](Riemers3DXNA1Terrain07terrainbasics)
