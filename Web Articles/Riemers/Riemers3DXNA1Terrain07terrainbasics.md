# Terrain creation basics

At last, we've seen enough topics to start creating our terrain. Let’s start small, by connecting 4x3 specified points. However, we will make our engine dynamic, so next chapter we can easily load a much larger number of points. To do this, we have to create 2 new variables in our class:

```csharp
 private int terrainWidth = 4;
 private int terrainHeight = 3;
```

We will suppose the 4x3 points are equidistant. So the only thing we don't know about our points is the Z coordinate. We will use an array to hold this information, so we'll also add this line to the top of our class as well:

```csharp
 private float[,] heightData;
```

For now, use this method to fill the array :

```csharp
 private void LoadHeightData()
 {
     heightData = new float[4, 3];
     heightData[0, 0] = 0;
     heightData[1, 0] = 0;
     heightData[2, 0] = 0;
     heightData[3, 0] = 0;

     heightData[0, 1] = 0.5f;
     heightData[1, 1] = 0;
     heightData[2, 1] = -1.0f;
     heightData[3, 1] = 0.2f;

     heightData[0, 2] = 1.0f;
     heightData[1, 2] = 1.2f;
     heightData[2, 2] = 0.8f;
     heightData[3, 2] = 0;
 }
```

Don’t forget to call it from within our LoadContent method. Make sure you call it before the SetUpVertices method, as that method will use the contents of the heightData.

```csharp
 LoadHeightData();
```

With our height array filled, we can now create our vertices. Since we have a 4x3 terrain, 12 (=terrainWidth*terrainHeight) vertices will do. The points are equidistant (the distance between them is the same), so we can easily change our SetUpVertices method. To begin with, we will not be using the Z coordinate yet, to see the difference later on in this chapter.

```csharp
 private void SetUpVertices()
 {
     vertices = new VertexPositionColor[terrainWidth * terrainHeight];
     for (int x = 0; x < terrainWidth; x++)
     {
         for (int y = 0; y < terrainHeight; y++)
         {
             vertices[x + y * terrainWidth].Position = new Vector3(x, 0, -y);
             vertices[x + y * terrainWidth].Color = Color.White;
         }
     }
 }
```

Nothing magical going on here, you simply define your 12 points and make them white. Note that the terrain will grow into the positive X direction (Right) and into the negative Z direction (Forward).

Next comes a more difficult part: defining the indices which define the required triangles to connect the 12 vertices. The best way to do this is by creating two sets of vertices:

![Indices](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-7Terrain1.jpg?raw=true)

We'll start by drawing the set of triangles drawn in solid lines. To do this, change our SetUpIndices method like this:

```csharp
 private void SetUpIndices()
 {
     indices = new int[(terrainWidth - 1) * (terrainHeight - 1) * 3];
     int counter = 0;
     for (int y = 0; y < terrainHeight - 1; y++)
     {
         for (int x = 0; x < terrainWidth - 1; x++)
         {
             int lowerLeft = x + y*terrainWidth;
             int lowerRight = (x + 1) + y*terrainWidth;
             int topLeft = x + (y + 1) * terrainWidth;
             int topRight = (x + 1) + (y + 1) * terrainWidth;
 
             indices[counter++] = topLeft;
             indices[counter++] = lowerRight;
             indices[counter++] = lowerLeft;
         }
     }
 }
```

Remember that terrainWidth and terrainHeight are the horizontal and vertical number of vertices in the terrain. We will need 2 rows of 3 triangles, giving 6 triangles. These will require ***3*2 * 3 = 18 indices (=(terrainWidth-1)*(terrainHeight-1)*3)***, hence the first line creates an array capable of storing exactly this amount of integers.

Then, you fill your array with indices. You scan the X and Y coordinates row by row, and this time you create your triangles. During the first row, where y=0, you need to create 6 triangles based on vertices 0 (bottom-left) till 7 (middle-right). Next, y becomes 1 and 1*terrainWidth=4 is added to each index: this time we create our 6 triangles on vertices 4 (middle-left) till 11 (top-right).

To make things easier, I’ve first defined 4 shortcuts for the 4 corner indices of a quad. For each quad you store 3 indices, defining one triangle. Remember culling? It requires us to define the vertices in clockwise order. So first you define the top-left vertex, then the bottom-down vertex and the bottom-left vertex.

The counter variable is an easy way to store vertices to an array, as we increment it each time an index has been added to the array. When the method finishes, the array will contain all indices required to render all bottom-left triangles.

We’ve coded our Draw method in such a way that XNA draws a number of triangles specified by the length of our indices array, so you can immediately run this code!

You should note the triangles look tiny, so try positioning your camera at (0,10,0) and rerun the program. You should see 6 triangles in the right half of your window, every point of every triangle at the same Z coordinate. Now change the height of your points according to your heightData array:

```csharp
 vertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x,y], -y);
```

Running this, you will notice the triangles are no longer positioned in the same plane.

Remember you’re still rendering only the bottom-left triangles. So when you would render the triangles with their solid colors instead of only their wireframes, 50% of your grid would not be covered. To fix this, let’s define some more indices to render the top-right triangles also. We need the same vertices, so the only thing we have to change is the SetUpIndices method:

```csharp
 private void SetUpIndices()
 {
     indices = new int[(terrainWidth - 1) * (terrainHeight - 1) * 6];
     int counter = 0;
     for (int y = 0; y < terrainHeight - 1; y++)
     {
         for (int x = 0; x < terrainWidth - 1; x++)
         {
             int lowerLeft = x + y*terrainWidth;
             int lowerRight = (x + 1) + y*terrainWidth;
             int topLeft = x + (y + 1) * terrainWidth;
             int topRight = (x + 1) + (y + 1) * terrainWidth;
 
             indices[counter++] = topLeft;
             indices[counter++] = lowerRight;
             indices[counter++] = lowerLeft;
 
             indices[counter++] = topLeft;
             indices[counter++] = topRight;
             indices[counter++] = lowerRight;
         }
     }
 }
```

We will be drawing twice as much vertices now, that's why the *3 has been replaced by *6. You see the second set of triangles also has been drawn clockwise relative to the camera: first the top-left corner, then the top-right and finally the bottom-right.

Running this code will give you a better 3 dimensional view. We've especially taken care only to use the variables terrainWidth and terrainHeight, so these are the only things we need change to increase the size of our map, together with the contents of the heightData array. It would be nice to find a mechanism to fill this last one automatically, which we'll do in the next chapter.

// Current Status

If you don't see any triangle when you run the code, see the note at the end of the previous chapter. Your problem should be solved by creating an array that can store shorts instead of ints and by filling it like this:

```csharp
 private void SetUpIndices()
 {
     indices = new short[(terrainWidth - 1) * (terrainHeight - 1) * 6];
     int counter = 0;
     for (int y = 0; y < terrainHeight - 1; y++)
     {
         for (int x = 0; x < terrainWidth - 1; x++)
         {
             short lowerLeft = (short)(x + y * terrainWidth);
             short lowerRight = (short)((x + 1) + y * terrainWidth);
             short topLeft = (short)(x + (y + 1) * terrainWidth);
             short topRight = (short)((x + 1) + (y + 1) * terrainWidth);
 
             indices[counter++] = topLeft;
             indices[counter++] = lowerRight;
             indices[counter++] = lowerLeft;
 
             indices[counter++] = topLeft;
             indices[counter++] = topRight;
             indices[counter++] = lowerRight;
         }
     }
 }
```

> You can try these exercises to practice what you've learned:
>
> - Play around with the values of your heightData array.
> - Try to add an extra row to your grid.

## Our code so far

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
         private int terrainWidth = 4;
         private int terrainHeight = 3;
         private float[,] heightData;
 
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


             LoadHeightData();
             SetUpVertices();
             SetUpIndices();            
         }
 
         protected override void UnloadContent()
         {
         }
 
         private void SetUpVertices()
         {
             vertices = new VertexPositionColor[terrainWidth * terrainHeight];
             for (int x = 0; x < terrainWidth; x++)
             {
                 for (int y = 0; y < terrainHeight; y++)
                 {
                     vertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x, y], -y);
                     vertices[x + y * terrainWidth].Color = Color.White;
                 }
             }
         }
 
         private void SetUpIndices()
         {
             indices = new int[(terrainWidth - 1) * (terrainHeight - 1) * 6];
             int counter = 0;
             for (int y = 0; y < terrainHeight - 1; y++)
             {
                 for (int x = 0; x < terrainWidth - 1; x++)
                 {
                     int lowerLeft = x + y * terrainWidth;
                     int lowerRight = (x + 1) + y * terrainWidth;
                     int topLeft = x + (y + 1) * terrainWidth;
                     int topRight = (x + 1) + (y + 1) * terrainWidth;
 
                     indices[counter++] = topLeft;
                     indices[counter++] = lowerRight;
                     indices[counter++] = lowerLeft;
 
                     indices[counter++] = topLeft;
                     indices[counter++] = topRight;
                     indices[counter++] = lowerRight;
                 }
             }
         }
 
         private void LoadHeightData()
         {
             heightData = new float[4, 3];
             heightData[0, 0] = 0;
             heightData[1, 0] = 0;
             heightData[2, 0] = 0;
             heightData[3, 0] = 0;
 
             heightData[0, 1] = 0.5f;
             heightData[1, 1] = 0;
             heightData[2, 1] = -1.0f;
             heightData[3, 1] = 0.2f;
 
             heightData[0, 2] = 1.0f;
             heightData[1, 2] = 1.2f;
             heightData[2, 2] = 0.8f;
             heightData[3, 2] = 0;
         }
 
         private void SetUpCamera()
         {
             viewMatrix = Matrix.CreateLookAt(new Vector3(0, 10, 0), new Vector3(0, 0, 0), new Vector3(0, 0, -1));
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

[Terrain creation from file](Riemers3DXNA1Terrain08terrainfile)
