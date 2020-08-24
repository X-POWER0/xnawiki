# Terrain creation from file

It's time to finally create a nice looking landscape. Instead of manually entering the data into our heightData array, we are going to fill it from a file. To do this, we are going to load a grayscale image of 128x128 pixel. We are going to use the 'white value' of every pixel as the height coordinate for the corresponding pixel! You can download my example file here (link).

Once you’ve downloaded the file, add it to your project as you’ve done for your .fx file. You can also do this by selecting the file in Windows Explorer, and dragging it onto the Content entry of your XNA Project. If everything is OK, you should see the heightmap.bmp file added in your Content entry.

The image file should be loaded in the LoadContent method. The .fx file was loaded into an Effect variable, an image should be loaded into a Texture2D variable:

```csharp
Texture2D heightMap = Content.Load<Texture2D> ("heightmap");LoadHeightData(heightMap);
```

By using the default Content Pipeline, it doesn’t matter whether you’re using a .bmp, .jpg or .pgn file. The second line calls the LoadHeightData, which we will adjust so it receives this texture. This is the new LoadHeightData method:

```csharp
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
```

As you can see, this method receives the image as argument. Instead of using a predefined width and height for your terrain, you should use the resolution of your image. The first 2 lines read the width and height of the image, and store them as width and height for the rest of our program. This will cause the rest of our code to automatically generate enough vertices and indices, and to render enough triangles.

Because you want to access the color values of the pixels of the image, you create an array of Color objects, into which you store the color of each pixel of the image. This is done in 2 easy lines.

Your next task would be to reshape this 1D array of Colors into a 2D array of floats. First you create a 2D matrix capable of storing just enough floats. Next, you browse through all colors and select the Red value, which is a value between 0 (no red) and 255 (fully red). Before you store this value inside the 2D array, you scale it a bit down, as otherwise your terrain would be too steep.

At this point, you have a 2D array containing the height for each point of your terrain. Your SetUpVertices method will generate a vertex for each point of the array. Your SetUpIndices method will generate 3 indices for each triangle that needs to be drawn to completely cover your terrain. Finally, your Draw method will render many triangles as your indices array allows. So when you run this code, you should see your terrain!

Bad luck, again. But the solution is simple, again. This corner of the terrain is positioned above your camera! So if you would increase the height of your camera to 40, you should see your terrain.

When you run your program, you will only see one corner of a huge terrain. You want to move your terrain, so its center is shifted to the (0,0,0) 3D origin point. This can be done in the Draw method:

```csharp
 Matrix worldMatrix = Matrix.CreateTranslation(-terrainWidth / 2.0f, 0, terrainHeight / 2.0f);
 effect.CurrentTechnique = effect.Techniques["ColoredNoShading
 "];
 effect.Parameters["xView"].SetValue(viewMatrix);
 effect.Parameters["xProjection"].SetValue(projectionMatrix);
 effect.Parameters["xWorld"].SetValue(worldMatrix);
 This will bring your terrain to the center of your screen.
```

> See Recipe 4-2 for an explanation of World matrices

As the camera is still looking straight on the terrain, we’ll get a nicer look by repositioning the camera a bit:

```csharp
 viewMatrix = Matrix.CreateLookAt(new Vector3(60, 80, -80), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
```

When you set your clearing color to Color.Black, you should get the image shown below.

// Current Status

> You can try these exercises to practice what you've learned:
>
> - This code scales the Red color value down by 5. Try out some other scaling values.
> - Open the heightmap.bmp image in Paint, and adjust some pixel values. Make some pixels pure red, others pure blue. Load this into your program.
> - Find another image file and load it into your program. Look for images with a small resolution, as this approach will not allow you to render huge terrains.

## Here's the total code

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
 
             angle += 0.005f;
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(Color.Black);
 
             RasterizerState rs = new RasterizerState();
             rs.CullMode = CullMode.None;
             rs.FillMode = FillMode.WireFrame;
             device.RasterizerState = rs;
     
     
             Matrix worldMatrix = Matrix.CreateTranslation(-terrainWidth / 2.0f, 0, terrainHeight / 2.0f);
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

[Rotate your terrain using the keyboard](Riemers3DXNA1Terrain09keyboard)
