# Improving performance by using VertexBuffers and IndexBuffers

Your terrain is fully working, Each frame again, all vertices and indices are being sent over to our graphics card. This means each frame we are sending over exactly the same data. Obviously, this should be optimized.

We want to send the data over to the graphics card only once, after which the graphics card should store it in it’s own superfast memory. This can be done by storing our vertices in a VertexBuffer, and our indices in an IndexBuffer.

? Recipes 5-4 and 5-5 give explain these buffers in a lot more detail.

Start by declaring these 2 variables at the top of your code:

```csharp
 VertexBuffer myVertexBuffer;
 IndexBuffer myIndexBuffer;
```

We will initialize and fill the VertexBuffer and IndexBuffer in a new method, CopyToBuffers. This code does the trick for the VertexBuffer:

```csharp
 private void CopyToBuffers()
 {
     myVertexBuffer = new VertexBuffer(device, VertexPositionColorNormal.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
     myVertexBuffer.SetData(vertices);
 }
```

The first line creates the VertexBuffer, which comes down to allocating a piece of memory on the graphics card that is large enough to store all our vertices. Therefore, you need to specify how many bytes we need. This is done by specifying the number of vertices in our array, as well the VertexDeclaration (which contains the size in bytes for one vertex, as we’ll see in Series 3).

The second line actually copies the data from our local vertices array into the memory on our graphics card.

We need to do the same for our indices, so put this code at the end of the method:

```csharp
 myIndexBuffer = new IndexBuffer(device, typeof(int), indices.Length, BufferUsage.WriteOnly);
 myIndexBuffer.SetData(indices);
```

To find out how many bytes to allocate, we pass in the type of each index as well as how many of them we want to store. The second line copies the indices over to the graphics card.

Don’t forget to call this method at the end of our LoadContent method:

```csharp
 CopyToBuffers();
```

With that being done, you only need to instruct your graphics card to fetch the vertex and index data from its own memory using the DrawIndexedPrimitives method instead of the DrawUserIndexedPrimitives method. Before our call this method, we need to let your graphics card know it should read from the buffers stored in its own memory:

```csharp
 device.Indices = myIndexBuffer;
 device.SetVertexBuffer(myVertexBuffer);
 device.DrawIndexedPrimitives(PrimitiveType.TriangleList, 0, 0, vertices.Length, 0, indices.Length / 3);
```

We indicate where the graphics card should get its indices and vertices from and render the triangles.

Running this code will give you the same result as in last chapter. This time however, all vertex and index data is transferred only once to your graphics card!

// Current Status

> You can try these exercises to practice what you've learned:
>
> - In this code, we’re still keeping the vertices and indices arrays as global variables, only because we need to know their lengths in the DrawIndexesPrimitives call. So if you remember their lengths you can get rid of them and free some memory.

## The final code

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
         VertexBuffer myVertexBuffer;
         IndexBuffer myIndexBuffer;
 
         Effect effect;
         VertexPositionColorNormal[] vertices;
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
            CalculateNormals();

             CopyToBuffers();
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
 
             vertices = new VertexPositionColorNormal[terrainWidth * terrainHeight];
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
 
         private void CalculateNormals()
         {
             for (int i = 0; i < vertices.Length; i++)
                 vertices[i].Normal = new Vector3(0, 0, 0);
 
             for (int i = 0; i < indices.Length / 3; i++)
             {
                 int index1 = indices[i * 3];
                 int index2 = indices[i * 3 + 1];
                 int index3 = indices[i * 3 + 2];
 
                 Vector3 side1 = vertices[index1].Position - vertices[index3].Position;
                 Vector3 side2 = vertices[index1].Position - vertices[index2].Position;
                 Vector3 normal = Vector3.Cross(side1, side2);
 
                 vertices[index1].Normal += normal;
                 vertices[index2].Normal += normal;
                 vertices[index3].Normal += normal;
             }
 
             for (int i = 0; i < vertices.Length; i++)
                 vertices[i].Normal.Normalize();
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
 
         private void CopyToBuffers()
         {
             myVertexBuffer = new VertexBuffer(device, VertexPositionColorNormal.VertexDeclaration, vertices.Length, BufferUsage.WriteOnly);
             myVertexBuffer.SetData(vertices);
 
             myIndexBuffer = new IndexBuffer(device, typeof(int), indices.Length, BufferUsage.WriteOnly);
             myIndexBuffer.SetData(indices);
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
             effect.CurrentTechnique = effect.Techniques["Colored"];
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xWorld"].SetValue(worldMatrix);
             Vector3 lightDirection = new Vector3(1.0f, -1.0f, -1.0f);
             lightDirection.Normalize();
             effect.Parameters["xLightDirection"].SetValue(lightDirection);
             effect.Parameters["xAmbient"].SetValue(0.1f);
             effect.Parameters["xEnableLighting"].SetValue(true);            
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.Indices = myIndexBuffer;
                 device.SetVertexBuffer(myVertexBuffer);
                 device.DrawIndexedPrimitives(PrimitiveType.TriangleList, 0, 0, vertices.Length, 0, indices.Length / 3);                
             }
 
             base.Draw(gameTime);
         }
     }
 }
```
