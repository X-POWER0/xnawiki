# Adding some color and the Z-Buffer

You might already have a rotating terrain, but it definitely would be better looking filled with some colors instead of just plain white lines. One idea to do this, is to use natural colors, like the ones that we find in the mountains. At the bottom we have blue lakes, then the green trees, the brown mountain and finally snow topped peaks. This means we will have to extend our SetUpVertices method a bit, so it stores the correct colors in each vertex.

You can’t however expect every image to have a lake at height 0, and a mountain peak at height 255 (the maximum value for a .bmp pixel). Imagine an image with height values only between 50 and 200, this image would then probably produce a terrain without any lakes or snow topped peaks.

To remain as general as possible, we first have to detect the minimum and maximum heights in our image. We will store these in the minHeight and maxHeight variables, which we find by putting this code at the top of our SetUpVertices method:

```csharp
 float minHeight = float.MaxValue;
 float maxHeight = float.MinValue;
 for (int x = 0; x < terrainWidth; x++)
 {
     for (int y = 0; y < terrainHeight; y++)
     {
         if (heightData[x, y] < minHeight)
             minHeight = heightData[x, y];
         if (heightData[x, y] > maxHeight)
             maxHeight = heightData[x, y];
     }
 }
```

You check for each point of our grid if the current point’s height is below the current minHeight or above the current maxHeight. If it is, store the current height in the corresponding variable.

With these variables filled, you can specify the 4 regions of your colors:

![Color by Height](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA1-10TerrainColor1.jpg?raw=true)

Now when you declare your vertices and their colors, you are going to define the desired colors to the correct height regions as follows:

```csharp
 vertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x, y], -y);
 
 if (heightData[x, y] < minHeight + (maxHeight - minHeight) / 4)
     vertices[x + y * terrainWidth].Color = Color.Blue;
 else if (heightData[x, y] < minHeight + (maxHeight - minHeight) * 2 / 4)
     vertices[x + y * terrainWidth].Color = Color.Green;
 else if (heightData[x, y] < minHeight + (maxHeight - minHeight) * 3 / 4)
     vertices[x + y * terrainWidth].Color = Color.Brown;
 else
     vertices[x + y * terrainWidth].Color = Color.White;
```

When your run this code, you will indeed see a nicely colored network of lines. When we want to see the whole colored terrain, we just have to remove this line (or set it to FillMode.Solid):

```csharp
 rs.FillMode = FillMode.WireFrame;
```

When you execute this, take a few moments to rotate the terrain a couple of times. On some computers, you will see that sometimes the middle peaks get overdrawn by the ‘invisible’ lake behind it. This is because we have not yet defined a ‘Z-buffer’! This Z buffer is nothing more than an array where your video card keeps track of the depth coordinate of every pixel that should be drawn on your screen (so in our case, a 500x500 matrix!). Every time your card receives a triangle to draw, it checks whether the triangle’s pixels are closer to the screen than the pixels already present in the Z-buffer. If they are closer, the Z-buffer’s contents is updated with these pixels for that region.

Of course, this whole process if fully automated. All we have to do, is to initialize our Z buffer with the largest possible distance to start with. So in fact, we have to first fill our buffer with ones. To do this automatically every update of our screen, change this line in the Draw method:

```csharp
 device.Clear(ClearOptions.Target|ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
```

(The | is a bitwise OR operator, in this case it means both the Target (the colors) as well as the DepthBuffer have to be cleared) Now everyone should see the terrain rotating as expected.

// Current Status

Although now we’ve got some colors in our terrain, it doesn’t really look any better than it did last chapter..

> You can try these exercises to practice what you've learned:
>
> - Lighting is required to obtain a 3D effect. In the image above, you get a little 3D feeling because you're using multiple colors. Change the colors again back to one single color, and see that the 3D feeling is completely gone!

## The code for this chapter

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


            Texture2D heightMap = Content.Load<Texture2D> ("heightmap");            LoadHeightData(heightMap);
            SetUpVertices();
            SetUpIndices();            
        }

        protected override void UnloadContent()
        {
        }

        private void SetUpVertices()
        {

             float minHeight = float.MaxValue;
             float maxHeight = float.MinValue;
             for (int x = 0; x < terrainWidth; x++)
             {
                 for (int y = 0; y < terrainHeight; y++)
                 {
                     if (heightData[x, y] < minHeight)
                         minHeight = heightData[x, y];
                     if (heightData[x, y] > maxHeight)
                         maxHeight = heightData[x, y];
                 }
             }
 
             vertices = new VertexPositionColor[terrainWidth * terrainHeight];
             for (int x = 0; x < terrainWidth; x++)
             {
                 for (int y = 0; y < terrainHeight; y++)
                 {
                     vertices[x + y * terrainWidth].Position = new Vector3(x, heightData[x, y], -y);
 
                     if (heightData[x, y] < minHeight + (maxHeight - minHeight) / 4)
                         vertices[x + y * terrainWidth].Color = Color.Blue;
                     else if (heightData[x, y] < minHeight + (maxHeight - minHeight) * 2 / 4)
                         vertices[x + y * terrainWidth].Color = Color.Green;
                     else if (heightData[x, y] < minHeight + (maxHeight - minHeight) * 3 / 4)
                         vertices[x + y * terrainWidth].Color = Color.Brown;
                     else
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
 
         private void LoadHeightData(Texture2D heightMap)
         {
             terrainWidth = heightMap.Width;
             terrainHeight = heightMap.Height;
 
             Color[] heightMapColors = new Color[terrainWidth * terrainHeight];
             heightMap.GetData(heightMapColors);
 
             heightData = new float[terrainWidth, terrainHeight];
             for (int x = 0; x < terrainWidth; x++)
                 for (int y = 0; y < terrainHeight; y++)
                     heightData[x, y] = heightMapColors[x + y * terrainWidth].R / 5.0f;
         }
 
         private void SetUpCamera()
         {
             viewMatrix = Matrix.CreateLookAt(new Vector3(60, 80, -80), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
             projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 1.0f, 300.0f);
         }
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
 
             KeyboardState keyState = Keyboard.GetState();
             if (keyState.IsKeyDown(Keys.Delete))
                 angle += 0.05f;
             if (keyState.IsKeyDown(Keys.PageDown))
                 angle -= 0.05f;
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.Black, 1.0f, 0);
 
             RasterizerState rs = new RasterizerState();
             rs.CullMode = CullMode.None;
             device.RasterizerState = rs;
 
             Matrix worldMatrix = Matrix.CreateTranslation(-terrainWidth / 2.0f, 0, terrainHeight / 2.0f) * Matrix.CreateRotationY(angle);
             effect.CurrentTechnique = effect.Techniques["ColoredNoShading"];
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
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

[Experimenting with Lights in XNA](Riemers3DXNA1Terrain11lighting)
