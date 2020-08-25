# Textures

Up till now, the only way we’ve seen to add some color to our scene is to declare separate vertices for every different color. Of course, this is not the way today’s great games are being made. XNA supports a very efficient way of adding color and images to the scene: you can simply put an image on a triangle. Such images are called textures.

As a first example, we’re going to draw 1 simple triangle, and cover it with a texture. You can find a sample texture here (link) (you can download the image as a file by right-clicking on the image, and selecting Save Image). Import the image into the Content entry of your XNA Solution Explorer, just as you’ve done for your effect file.

When you click on the file in the Content entry of your Solution Explorer, you can see in the property box at the bottom-right of your screen that this asset has the name ‘riemerstexture’. You can change this to your liking, but leave it like this for now.

In our code, we are going to add a new variable to hold this texture image. Add this line to the top of your code:

```csharp
 Texture2D texture;
```

Now find the LoadContent method in your code. Add this line under the line where you load your effect file:

```csharp
texture = Content.Load<Texture2D> ("riemerstexture");
```

This line binds the asset we just loaded in our project to the texture variable!

With our texture loaded into our XNA project, it’s time to define 3 vertices, which we’ll be storing in an array. As our vertices will need to be able to store both a 3D position and a texture coordinate (explained below), the vertex format will be VertexPositionTexture, so declare this variable at the top of your code:

```csharp
 VertexPositionTexture[] vertices;
```

Next we’ll be defining the 3 vertices of our triangle in our SetUpVertices method we’ll create:

```csharp
 private void SetUpVertices()
 {
     vertices = new VertexPositionTexture[3];
 
     vertices[0].Position = new Vector3(-10f, 10f, 0f);
     vertices[0].TextureCoordinate.X = 0;
     vertices[0].TextureCoordinate.Y = 0;
 
     vertices[1].Position = new Vector3(10f, -10f, 0f);
     vertices[1].TextureCoordinate.X = 1;
     vertices[1].TextureCoordinate.Y = 1;
 
     vertices[2].Position = new Vector3(-10f, -10f, 0f);
     vertices[2].TextureCoordinate.X = 0;
     vertices[2].TextureCoordinate.Y = 1;
 
      texturedVertexDeclaration = new VertexDeclaration(device, VertexPositionTexture.VertexElements);
 }
```

As you see, for every vertex we first define its position in 3D space. Notice again that we have defined our vertices in a clockwise way, so XNA will not cull them away (see the “World Space” chapter in series 1).

The next 2 settings are very important, as they define which point in our texture image we want to correspond with the vertex. These coordinates are simply the X and Y coordinates of the texture, with the (0,0) texture coordinate being the top left point of the texture image, the (1,0) texture coordinate the top-right point and the (1,1) texture coordinate the bottom-right point of the texture.

Don’t forget to call the SetUpVertices method from your LoadContent method:

```csharp
 SetUpVertices ();
```

OK, we have our vertices set up, and our texture image loaded into a variable. Let’s draw the triangle!

Go to our Draw method, and add this code after our call to the Clear method:

```csharp
 Matrix worldMatrix = Matrix.Identity;
 effect.CurrentTechnique = effect.Techniques["TexturedNoShading"];
 effect.Parameters["xWorld"].SetValue(worldMatrix);
 effect.Parameters["xView"].SetValue(viewMatrix);
 effect.Parameters["xProjection"].SetValue(projectionMatrix);
 effect.Parameters["xTexture"].SetValue(texture);
 effect.Begin();
 foreach (EffectPass pass in effect.CurrentTechnique.Passes)
 {
     pass.Begin();

     device.VertexDeclaration = texturedVertexDeclaration;
     device.DrawUserPrimitives(PrimitiveType.TriangleList, vertices, 0, 1);

     pass.End();
 }
 effect.End();
```

As usual, we indicate which technique we want the graphics card to use to render the triangle to the screen. We need to instruct our graphics card to sample the color of every pixel from the texture image. This is exactly what the TexturedNoShading technique of my effect file does, so we set it as active technique. As we didn’t specify any normals for our vectors, we cannot expect the effect to do any meaningful shading calculations.

As explained in Series 1, we need to set the World matrix to identity so the triangles will be rendered where we defined them, and View and Projection matrices so the graphics card can map the 3D positions to 2D screen coordinates.

Finally, we pass our texture to the technique. Then we actually draw our triangle from our vertices array, as done before in the first series.

Running this should already give you a textured triangle, displaying half of the texture image! To display the whole image, we simply have to expand our SetUpVertices method by adding the second triangle:

```csharp
 private void SetUpVertices()
 {
      vertices = new VertexPositionTexture[6];

      vertices[0].Position = new Vector3(-10f, 10f, 0f);
      vertices[0].TextureCoordinate.X = 0;
      vertices[0].TextureCoordinate.Y = 0;

      vertices[1].Position = new Vector3(10f, -10f, 0f);
      vertices[1].TextureCoordinate.X = 1;
      vertices[1].TextureCoordinate.Y = 1;

      vertices[2].Position = new Vector3(-10f, -10f, 0f);
      vertices[2].TextureCoordinate.X = 0;
      vertices[2].TextureCoordinate.Y = 1;

      vertices[3].Position = new Vector3(10.1f, -9.9f, 0f);
      vertices[3].TextureCoordinate.X = 1;
      vertices[3].TextureCoordinate.Y = 1;

      vertices[4].Position = new Vector3(-9.9f, 10.1f, 0f);
      vertices[4].TextureCoordinate.X = 0;
      vertices[4].TextureCoordinate.Y = 0;

      vertices[5].Position = new Vector3(10.1f, 10.1f, 0f);
      vertices[5].TextureCoordinate.X = 1;
      vertices[5].TextureCoordinate.Y = 0;
 }
```

We simply added another set of 3 vertices for a second triangle, to complete the texture image. Don’t forget to adjust your Draw method so you render 2 triangles instead of only 1:

```csharp
 device.DrawUserPrimitives(PrimitiveType.TriangleList, vertices, 0, 2, VertexPositionTexture.VertexDeclaration);
```

Now run this code, and you should see the whole texture image, displayed by 2 triangles!

![Status](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-02Textures1.jpg?raw=true)

You’ll notice the small gap between both triangles... This is of course because we defined the positions of the vertices that way, so you can actually see the image is made out of two separate triangles.

> You can try these exercises to practice what you've learned:
>
> - Try to remove the gap between the triangles.
> - Play around with the texture coordinates in the SetUpVertices method, it’s worth it!! You can choose any value between 0 and 1.

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
 
 namespace Series3D2
 {
     public class Game1 : Microsoft.Xna.Framework.Game
     {
         GraphicsDeviceManager graphics;
         SpriteBatch spriteBatch;
         GraphicsDevice device;
         Effect effect;
         Texture2D texture;
 
         VertexPositionTexture[] vertices;
 
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
            texture = Content.Load<Texture2D> ("riemerstexture");
            SetUpCamera();

             SetUpVertices();
         }
 
         private void SetUpCamera()
         {
             viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, 30), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
             projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 0.2f, 500.0f);
         }
 
         private void SetUpVertices()
         {
             vertices = new VertexPositionTexture[6];
 
             vertices[0].Position = new Vector3(-10f, 10f, 0f);
             vertices[0].TextureCoordinate.X = 0;
             vertices[0].TextureCoordinate.Y = 0;
 
             vertices[1].Position = new Vector3(10f, -10f, 0f);
             vertices[1].TextureCoordinate.X = 1;
             vertices[1].TextureCoordinate.Y = 1;
 
             vertices[2].Position = new Vector3(-10f, -10f, 0f);
             vertices[2].TextureCoordinate.X = 0;
             vertices[2].TextureCoordinate.Y = 1;
 
             vertices[3].Position = new Vector3(10.1f, -9.9f, 0f);
             vertices[3].TextureCoordinate.X = 1;
             vertices[3].TextureCoordinate.Y = 1;
 
             vertices[4].Position = new Vector3(-9.9f, 10.1f, 0f);
             vertices[4].TextureCoordinate.X = 0;
             vertices[4].TextureCoordinate.Y = 0;
 
             vertices[5].Position = new Vector3(10.1f, 10.1f, 0f);
             vertices[5].TextureCoordinate.X = 1;
             vertices[5].TextureCoordinate.Y = 0;
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
 
             Matrix worldMatrix = Matrix.Identity;
             effect.CurrentTechnique = effect.Techniques["TexturedNoShading"];
             effect.Parameters["xWorld"].SetValue(worldMatrix);
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xTexture"].SetValue(texture);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
                 device.DrawUserPrimitives(PrimitiveType.TriangleList, vertices, 0, 2, VertexPositionTexture.VertexDeclaration);
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[Creating a floorplan](Riemers3DXNA2flightsim03floorplan)
