# Adding normals to our Terrain – Intuitive approach

We’ll be adding normal data to all vertices of our terrain, so our graphics card can perform some lighting calculations on it. As seen in the previous chapter, we will need to add a normal to each of our vertices.

> The remainder of this chapter is explained in much more detail in Recipe 5-7

This normal should be perpendicular to the triangle of the vertex. In cases where the vertex is shared among multiple triangles (as in our terrain), you should find the normal of all triangles that use the vertex, and store the average of those normals in the vertex.
First we have to reload our code from the ‘Adding colors’ chapter, after which we need to add the struct (which allows normal data to be added to our vectors) to the top of our class:

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

Don’t forget to update our vertices declaration:

```csharp
 VertexPositionColorNormal[] vertices;
```

During the instantiation of this array:

```csharp
 vertices = new VertexPositionColorNormal[terrainWidth * terrainHeight];
```

As well as the line that actually renders your triangles in our Draw method:

```csharp
 device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, vertices, 0, vertices.Length, indices, 0, indices.Length / 3, VertexPositionColorNormal.VertexDeclaration);
```

Now we’re ready to calculate the normals, which will be done in the CalculateNormals method. Start by clearing the normals in each of your vertices:

```csharp
 private void CalculateNormals()
 {
     for (int i = 0; i < vertices.Length; i++)
         vertices[i].Normal = new Vector3(0, 0, 0);
 }
```

Next, you’re going to scroll through each of your triangles. For each triangle, you will calculate its normal. Finally, you will add this normal to the normal each of the triangle’s 3 vertices:

```csharp
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
```

You look up the indices for the 3 vertices of the triangle. If you know 2 sides of a triangle, you can find its normal by taking the cross product of these 2 sides. Given any 2 vectors, their cross product gives you’re the vector that is perpendicular to both vectors.

So first you find 2 sides of the triangle, by subtracting the position from one corner from the position of another. Once you know the normal for the triangle, you add it to the normal of each of the 3 vertices.

At this point, all vertices contain huge normal vectors, while they need to be of unity length. So end by normalizing all of them:

```csharp
 for (int i = 0; i < vertices.Length; i++)
     vertices[i].Normal.Normalize();
```

That’s it for the normals! Don’t forget to call this method at the end of our LoadContent method:

```csharp
 CalculateNormals();
```

In your Draw method, tell your graphics card to use the correct effect:

```csharp
 effect.CurrentTechnique = effect.Techniques["Colored"];
```

And turn on the light:

```csharp
 Vector3 lightDirection = new Vector3(1.0f, -1.0f, -1.0f);
 lightDirection.Normalize();
 effect.Parameters["xLightDirection"].SetValue(lightDirection);
 effect.Parameters["xAmbient"].SetValue(0.1f); effect.Parameters["xEnableLighting"].SetValue(true);            
```

The last line turns on ambient lighting, so even the parts of the terrain that are not lit at all by the directional light still receive some lighting. When you run this code, you should see the image below:

// Current Status

As you can see, adding correct lighting dramatically adds more 3D feeling to your scene.

> Chapter 6 is completely dedicated to lighting a 3D scene.

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
 
                 device.DrawUserIndexedPrimitives(PrimitiveType.TriangleList, vertices, 0, vertices.Length, indices, 0, indices.Length / 3, VertexPositionColorNormal.VertexDeclaration);
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[Improving performance by using VertexBuffers and IndexBuffers](Riemers3DXNA1Terrain13buffers)
