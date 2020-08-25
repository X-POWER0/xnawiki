# Creating a floorplan

Now we’ve seen how we can import simple images into our XNA project and have XNA display them on triangles, it’s not that difficult to create a large amount of images. It’s more important to find a way to get the computer define all of the vertices for us.

As a small example, let’s simply create a raster of 3x3 images, with the center image missing. This means 8 images, thus 16 triangles, and 48 vertices. Instead of defining all these vertices manually, let’s create a “floorPlan” array, at the top of your code. This array will later contain where we want to have buildings in our 3D city:

```csharp
 int[,] floorPlan;

 VertexBuffer cityVertexBuffer;
```

Since we will need to define the vertices only once, we will store them on the RAM of our graphics card by storing them in a VertexBuffer (see Series 1) rather than in a simple array as done in the previous chapter.

We’ll first create a small method that fills the floorPlan array with data:

```csharp
 private void LoadFloorPlan()
 {
     floorPlan = new int[,]
     {
         {0,0,0},
         {0,1,0},
         {0,0,0},
     };
 }
```

In this data, a 0 means ‘draw a floor texture’ and a 1 means to leave that tile open. Later in this series, a 1 will indicate a building. This method contains all the flexibility our program needs: simply changing a 0 to a 1 will result in an extra building drawn in our 3D city! Load this method from within the Initalize method:

```csharp
 LoadFloorPlan();
```

Now we’ll update our SetUpVertices method, so it reads the data inside the array and automatically creates the corresponding vertices. In the last chapter you’ve learned how to cover triangles with images. This time, we’re going to load 1 texture image file, which is composed of several images next to each other. The leftmost part of the texture will be the floor tile, followed by a wall and a roofing image for each different type of building. You can download my texturefile (link) to see what I mean. It is in short one image that contains multiple images.

You can delete the ‘riemerstexture’ asset from your Solution Explorer by right-clicking on it and selecting Delete. Next, add the image you just downloaded to your XNA project, as you’ve done before. You should see it in the Content entry of your Solution Explorer. In the end, we’re going to rename our ‘texture’ variable to ‘sceneryTexture’, because later in the game we’ll be using more than 1 texture. Change the name of the variable at the top of the code

```csharp
 Texture2D sceneryTexture;
```

and make sure you replace the name of the asset in your LoadContent method:

```csharp
sceneryTexture = Content.Load<Texture2D> ("texturemap");
```

We can delete the contents of the SetUpVertices method. Start with this code, which is based on the last chapter of Series 1:

```csharp
 private void SetUpVertices()
 {
     int cityWidth = floorPlan.GetLength(0);
     int cityLength = floorPlan.GetLength(1);

    List<VertexPositionNormalTexture> verticesList = new List<VertexPositionNormalTexture> ();
    for (int x = 0; x < cityWidth; x++)
    {
        for (int z = 0; z < cityLength; z++)
        {
            //if floorPlan contains a 0 for this tile, add 2 triangles
        }
    }

    cityVertexBuffer = new VertexBuffer(device, VertexPositionNormalTexture.VertexDeclaration, verticesList.Count, BufferUsage.WriteOnly);

    cityVertexBuffer.SetData<VertexPositionNormalTexture> (verticesList.ToArray());}
```

This code first retrieves the width and length of our future city, which will be 3x3 in the case of our current floorMap. Next, we create a List capable of storing VertexPositionNormalTextures. The main advantage of a List over an array is that you don’t have to specify the number of elements you’re going to add. You can just add elements, and the List will grow larger automatically.

Next, you scroll through the contents of the floorMap array. Whenever a 0 if found in the floorMap, we want this for loop to add 6 vertices to the List, for 2 triangles. This will be done next, for now imagine that when the for loop finishes the List contains 6 vertices for each 0 found in the floorMap.

To store the vertices in the RAM of the graphics card, you create a VertexBuffer that is exactly large enough (see Series 1). We transform our List to an array, so we can copy all our vertices to the memory on the graphics card using the SetData method.

To finish this method, we need to add this code inside the for-loop:

```csharp
 int imagesInTexture = 11;
 if (floorPlan[x, z] == 0)
 {
 verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(0, 1, 0), new Vector2(0, 1)));
 verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
 verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));

 verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
 verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z-1), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 0)));
 verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));
 }
```

Every time a 0 is encountered, 2 triangles are defined. The normal vectors are pointing upwards (0,1,0) towards the sky, and the correct portion of the texture image is pasted over the image: the rectangle between [0,0] and [1/imagesintexture,1]. In my sample texture map, I have stored 11 images. So the X coordinates for the first image stretches from 0 to 1/11. Have another look at the texture file to fully understand this.

Right now we have defined a lot of vertices, corresponding to 2 triangles for each 0 in our floorMap. What’s more, is that these vertices have been saved in the RAM on the graphics card.

With this method finished, we can move on to the code that renders the triangles. To keep our Draw method clean, we will define a separate method:

```csharp
 private void DrawCity()
 {
 effect.CurrentTechnique = effect.Techniques["Textured"];
 effect.Parameters["xWorld"].SetValue(Matrix.Identity);
 effect.Parameters["xView"].SetValue(viewMatrix);
 effect.Parameters["xProjection"].SetValue(projectionMatrix);
 effect.Parameters["xTexture"].SetValue(sceneryTexture);

 foreach (EffectPass pass in effect.CurrentTechnique.Passes)
 {
 pass.Apply();
 device.SetVertexBuffer(cityVertexBuffer);
 device.DrawPrimitives(PrimitiveType.TriangleList, 0, cityVertexBuffer.VertexCount/3);
 }
 }
```

We will still be using the Textured technique to render our triangles from our vertices. As always, we need to set our World, View and Projection matrices. As we want the graphics card to sample the colors from our texture, we need to pass this texture to the graphics card.

The triangles are rendered from a VertexBuffer, using the code of the last chapter of Series 1. The last argument of the DrawPrimitives method automatically determines how many triangles can be rendered from the VertexBuffer. Since 3 vertices define 1 triangle, you know how many triangles to render!

Don't forget to call this method from within your Draw method:

```csharp
 protected override void Draw(GameTime gameTime)
 {
     ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);

     DrawCity();

     base.Draw(gameTime);
 }
```

This code should be runnable! Although it might be a good idea to reposition our camera a bit:  viewMatrix = Matrix.CreateLookAt(new Vector3(3, 5, 2), new Vector3(2, 0, -1), new Vector3(0, 1, 0));

When running this code, you should see a small square with a hole in the middle, just as you defined in the LoadFloorPlan method.

This should give you the following image:

![Floorplan](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-03Floorplan1.jpg?raw=true)

> You can try these exercises to practice what you've learned:
>
> - Change the contents of the floorPlan array.
> - Change the size of the floorPlan array.
> - Change the texture coordinates of our vertices, so your graphics card uses another image from our texture map to cover the floor.

This chapter we’ve seen how to use a texturemap and how to use the List, a useful feature of C#.

## The code at this point

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
 
 namespace Series3D2
 {
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         GraphicsDeviceManager graphics;
         SpriteBatch spriteBatch;
         GraphicsDevice device;
         Effect effect;
         Texture2D sceneryTexture;
         VertexBuffer cityVertexBuffer;
 
         int[,] floorPlan;
 
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
             Window.Title = "Riemer's XNA Tutorials -- 3D Series 2";
 
             base.Initialize();
         }
 
         protected override void LoadContent()
         {
             spriteBatch = new SpriteBatch(GraphicsDevice);
             
             device = graphics.GraphicsDevice;

            effect = Content.Load<Effect> ("effects");
            sceneryTexture = Content.Load<Texture2D> ("texturemap");
            SetUpCamera();

             LoadFloorPlan();
             SetUpVertices();
         }
 
         private void SetUpCamera()
         {
             viewMatrix = Matrix.CreateLookAt(new Vector3(3, 5, 2), new Vector3(2, 0, -1), new Vector3(0, 1, 0));
             projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 0.2f, 500.0f);
         }
 
         private void LoadFloorPlan()
         {
             floorPlan = new int[,]
             {
                 {0,0,0},
                 {0,1,0},
                 {0,0,0},
             };
         }
 
         private void SetUpVertices()
         {
             int cityWidth = floorPlan.GetLength(0);
             int cityLength = floorPlan.GetLength(1);
 

            List<VertexPositionNormalTexture> verticesList = new List<VertexPositionNormalTexture> ();
            for (int x = 0; x < cityWidth; x++)
            {
                for (int z = 0; z < cityLength; z++)
                {
                    int imagesInTexture = 11;
                    if (floorPlan[x, z] == 0)
                    {
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(0, 1, 0), new Vector2(0, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 1, 0), new Vector2(0, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z-1), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 1, 0), new Vector2(1.0f / imagesInTexture, 1)));
                    }
                }
            }

            cityVertexBuffer = new VertexBuffer(device, VertexPositionNormalTexture.VertexDeclaration, verticesList.Count, BufferUsage.WriteOnly);

            cityVertexBuffer.SetData<VertexPositionNormalTexture> (verticesList.ToArray());
         }
 
         protected override void UnloadContent()
         {
         }
 
         protected override void Update(GameTime gameTime)
         {
             if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                 this.Exit();
 
             base.Update(gameTime);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
 
             DrawCity();
 
             base.Draw(gameTime);
         }
 
         private void DrawCity()
         {
             effect.CurrentTechnique = effect.Techniques["Textured"];
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xTexture"].SetValue(sceneryTexture);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
                 device.SetVertexBuffer(cityVertexBuffer);
                 device.DrawPrimitives(PrimitiveType.TriangleList, 0, cityVertexBuffer.VertexCount/3);
             }
         }
     }
 }
```

## Next Steps

[Drawing the buildings](Riemers3DXNA2flightsim04creatingcity)
