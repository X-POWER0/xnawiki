# Using quaternions for rotations - dynamically positioning the camera behind our xwing

This chapter we're going to make our xwing fly around the city. Of course, this is not as easy as telling the thing to fly. We will need to reposition it ourselves, and to make the camera track the xwing.

Imagine our xwing would be flying through our 3D city. We would like our camera to always follow our xwing, so the camera would always be positioned behind the xwing, no matter which rotation of translation our xwing has.

First we're going to introduce 2 new variables, xwingPosition and xwingRotationg. These variables will determine the position of our camera, as the position of the camera needs to change as our xwing moves/rotates.

We could store the rotation of our xwing as separate rotations around the X, Y and Z axis. However, in practice this is very tricky. The problem is mainly the same as with matrices: a rotation around X followed by a rotation around Y does NOT yield the same as a rotation around Y followed by a rotation around X. Recipe 4-2 explains the cause of these problems.

In addition, deriving the total rotation from 3 separate values requires some trigonometric formulae (sin, cos, tan,…). Sometimes, these suddenly switch signs, what would make our xwing to fly backwards.

To solve both problems, we’ll be using a Quaternion to store the rotation of our xwing. A Quaternion is much like a Matrix, but can only store a rotation. Don’t let the name of this thing scare you away: although it’s very hard to understand mathematically, a quaternion is VERY easy to use. Go ahead and declare these 2 variables at the top of your code:

```csharp
 Vector3 xwingPosition = new Vector3(8, 1, -3);
 Quaternion xwingRotation = Quaternion.Identity;
```

As you can see, we’ve already initialized a starting position and rotation for our xwing. Next chapter we’ll see how we can adjust the quaternion based on user input, for now use the unity quaternion.

To accomplish the goal of this chapter, we will draw the 3D city, then draw the xwing at its correct position and rotation, and then reposition the camera immediately behind our airplane.

In our DrawModel method, we can already change the world matrix for our xwing, so it’s drawn at the correct location, and with the correct rotation:

```csharp
 Matrix worldMatrix = Matrix.CreateScale(0.0005f, 0.0005f, 0.0005f) * Matrix.CreateRotationY(MathHelper.Pi) * Matrix.CreateFromQuaternion(xwingRotation) * Matrix.CreateTranslation(xwingPosition);
```

This line looks complex, but it’s quite easy: first the mesh is translated (=moved) to its correct position. Next, the xwing is rotated among the rotation stored in the xwingRotation quaternion. After this, it is rotated along 180 degrees to compensate for the opposite direction stored inside the model. And finally, the model is scaled down so it fits nicely in our scene.

Here you can already notice how easy it is to retrieve the rotation matrix corresponding to a quaternion.

Now we have our xwing at the correct position and correctly rotated, it’s time to position the camera behind the xwing. We’re going to create a method to do this. So our method will create a new viewMatrix and projectionMatrix that depend on the current position and rotation of the xwing. Let’s start with this:

```csharp
 private void UpdateCamera()
 {
     Vector3 campos = new Vector3(0, 0.1f, 0.6f);
 }
```

This vector will define where we want our camera to be, relative to the position of the xwing: we want to position the camera a bit behind (Z=+0.6f)and above (Y=+0.1f) our xwing, so the vector only has a Y and Z component. This vector still needs to be transformed: it needs to be translated (=moved) to the position of our mesh, and then it needs to be rotated with the rotation of the xwing, so it end up nicely behind the xwing.

You can find a detailed explanation about this approach in Recipe 2-4.

Let’s first process the rotation transform:

```csharp
 campos = Vector3.Transform(campos, Matrix.CreateFromQuaternion(xwingRotation));
```

Now the campos variable holds the vector that will always be behind (and a bit above) our xwing no matter what its rotation is, IF the xwing is in the (0,0,0) position. Because the position of our xwing will constantly change, we need to move (=translate) our campos vector to the position of our xwing, which is done using the following line:

```csharp
 campos += xwingPosition;
```

OK, so now we have the vector that will always be a bit behind and a bit above our xwing, no matter which rotation and/or translation our xwing has!

Remember, when we create a viewMatrix, we not only need the position and target of our camera (which we both know at this moment), but also the vector that indicates the ‘up’-position of the xwing. This is found exactly the same way: we start with the vector that points up, and rotate it with the xwing rotation matrix:

```csharp
 Vector3 camup = new Vector3(0, 1, 0);
 camup = Vector3.Transform(camup, Matrix.CreateFromQuaternion(xwingRotation));
```

Now we have everything to create our camera matrices: the position, target and up-vector, so we can create our new matrices:

```csharp
 viewMatrix = Matrix.CreateLookAt(campos, xwingPosition, camup);
 projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 0.2f, 500.0f);
```

Only the first line contains the newly created vectors, the second line has remained the same.

Quite a method, let’s make sure we call it from the Update method. This will cause both camera matrices to be updated according to the latest position or rotation of the xwing!

```csharp
 protected override void Update(GameTime gameTime)
 {
     UpdateCamera();

     base.Update(gameTime);
 }
```

That’s it! When you run this code, you’ll see the camera has been positioned behind and above your xwing, as on the image below:

![Camera View](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-07Camera1.jpg?raw=true)

Now we have our code ready so the camera is adjusted automatically to the xwingPosition and xwingRotation variables, it’s time to read out the keyboard. Anyway, we now have a dynamic camera that follows our xwing, no matter what its position and rotation is.

> You can try these exercises to practice what you've learned:
>
> - Change the position of the xwing, you’ll see that the camera follows!

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
         Texture2D sceneryTexture;
         Model xwingModel;
         VertexBuffer cityVertexBuffer;
 
         int[,] floorPlan;
         int[] buildingHeights = new int[] { 0, 2, 2, 6, 5, 4 };
         Vector3 lightDirection = new Vector3(3, -2, 5);
         Vector3 xwingPosition = new Vector3(8, 1, -3);
         Quaternion xwingRotation = Quaternion.Identity;
 
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
 
             lightDirection.Normalize();
 
             base.Initialize();
         }
 
         protected override void LoadContent()
         {
             spriteBatch = new SpriteBatch(GraphicsDevice);
             
             device = graphics.GraphicsDevice;

            effect = Content.Load<Effect> ("effects");
            sceneryTexture = Content.Load<Texture2D> ("texturemap");
             xwingModel = LoadModel("xwing");
 
             LoadFloorPlan();
             SetUpVertices();
         }
 
         private void LoadFloorPlan()
         {
             floorPlan = new int[,]
             {
                 {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
                 {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,0,1,1,0,0,0,1,1,0,0,1,0,1},
                 {1,0,0,1,1,0,0,0,1,0,0,0,1,0,1},
                 {1,0,0,0,1,1,0,1,1,0,0,0,0,0,1},
                 {1,0,0,0,0,0,0,0,0,0,0,1,0,0,1},
                 {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,1,1,0,0,0,1,0,0,0,0,0,0,1},
                 {1,0,1,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,0,0,0,0,0,0,0,0,0,0,0,0,1},
                 {1,0,0,0,0,1,0,0,0,0,0,0,0,0,1},
                 {1,0,0,0,0,1,0,0,0,1,0,0,0,0,1},
                 {1,0,1,0,0,0,0,0,0,1,0,0,0,0,1},
                 {1,0,1,1,0,0,0,0,1,1,0,0,0,1,1},
                 {1,0,0,0,0,0,0,0,1,1,0,0,0,1,1},
                 {1,1,1,1,1,1,1,1,1,1,1,1,1,1,1},
             };
 
             Random random = new Random();
             int differentBuildings = buildingHeights.Length - 1;
             for (int x = 0; x < floorPlan.GetLength(0); x++)
                 for (int y = 0; y < floorPlan.GetLength(1); y++)
                     if (floorPlan[x, y] == 1)
                         floorPlan[x, y] = random.Next(differentBuildings) + 1;
         }
 
         private void SetUpVertices()
         {
             int differentBuildings = buildingHeights.Length - 1;
             float imagesInTexture = 1 + differentBuildings * 2;
 
             int cityWidth = floorPlan.GetLength(0);
             int cityLength = floorPlan.GetLength(1);
 

            List<VertexPositionNormalTexture> verticesList = new List<VertexPositionNormalTexture> ();
            for (int x = 0; x < cityWidth; x++)
            {
                for (int z = 0; z < cityLength; z++)
                {
                    int currentbuilding = floorPlan[x, z];

                    //floor or ceiling
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z), new Vector3(0, 1, 0), new Vector2(currentbuilding * 2 / imagesInTexture, 1)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z - 1), new Vector3(0, 1, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 0)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z), new Vector3(0, 1, 0), new Vector2((currentbuilding * 2 + 1) / imagesInTexture, 1)));

                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z - 1), new Vector3(0, 1, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 0)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z - 1), new Vector3(0, 1, 0), new Vector2((currentbuilding * 2 + 1) / imagesInTexture, 0)));
                    verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z), new Vector3(0, 1, 0), new Vector2((currentbuilding * 2 + 1) / imagesInTexture, 1)));

                    if (currentbuilding != 0)
                    {
                        //front wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z - 1), new Vector3(0, 0, -1), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z - 1), new Vector3(0, 0, -1), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(0, 0, -1), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 1)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z - 1), new Vector3(0, 0, -1), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z - 1), new Vector3(0, 0, -1), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z - 1), new Vector3(0, 0, -1), new Vector2((currentbuilding * 2) / imagesInTexture, 0)));

                        //back wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 0, 1), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(0, 0, 1), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z), new Vector3(0, 0, 1), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z), new Vector3(0, 0, 1), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z), new Vector3(0, 0, 1), new Vector2((currentbuilding * 2) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(0, 0, 1), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));

                        //left wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(-1, 0, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z - 1), new Vector3(-1, 0, 0), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z - 1), new Vector3(-1, 0, 0), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z - 1), new Vector3(-1, 0, 0), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, buildingHeights[currentbuilding], -z), new Vector3(-1, 0, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x, 0, -z), new Vector3(-1, 0, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));

                        //right wall
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(1, 0, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z - 1), new Vector3(1, 0, 0), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z - 1), new Vector3(1, 0, 0), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 1)));

                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z - 1), new Vector3(1, 0, 0), new Vector2((currentbuilding * 2 - 1) / imagesInTexture, 0)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, 0, -z), new Vector3(1, 0, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 1)));
                        verticesList.Add(new VertexPositionNormalTexture(new Vector3(x + 1, buildingHeights[currentbuilding], -z), new Vector3(1, 0, 0), new Vector2((currentbuilding * 2) / imagesInTexture, 0)));
                    }
                }
            }

            cityVertexBuffer = new VertexBuffer(device, VertexPositionNormalTexture.VertexDeclaration, verticesList.Count, BufferUsage.WriteOnly);

            cityVertexBuffer.SetData<VertexPositionNormalTexture> (verticesList.ToArray());        }

        private Model LoadModel(string assetName)
        {

            Model newModel = Content.Load<Model> (assetName);            foreach (ModelMesh mesh in newModel.Meshes)
                foreach (ModelMeshPart meshPart in mesh.MeshParts)
                    meshPart.Effect = effect.Clone();
            return newModel;
        }

        protected override void UnloadContent()
        {
        }

        protected override void Update(GameTime gameTime)
        {
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                this.Exit();


             UpdateCamera();
 
             base.Update(gameTime);
         }
 
         private void UpdateCamera()
         {
             Vector3 campos = new Vector3(0, 0.1f, 0.6f);
             campos = Vector3.Transform(campos, Matrix.CreateFromQuaternion(xwingRotation));
             campos += xwingPosition;
 
             Vector3 camup = new Vector3(0, 1, 0);
             camup = Vector3.Transform(camup, Matrix.CreateFromQuaternion(xwingRotation));
 
             viewMatrix = Matrix.CreateLookAt(campos, xwingPosition, camup);
             projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 0.2f, 500.0f);
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
 
             DrawCity();
             DrawModel();
 
             base.Draw(gameTime);
         }
 
         private void DrawCity()
         {
             effect.CurrentTechnique = effect.Techniques["Textured"];
             effect.Parameters["xWorld"].SetValue(Matrix.Identity);
             effect.Parameters["xView"].SetValue(viewMatrix);
             effect.Parameters["xProjection"].SetValue(projectionMatrix);
             effect.Parameters["xTexture"].SetValue(sceneryTexture);
             effect.Parameters["xEnableLighting"].SetValue(true);
             effect.Parameters["xLightDirection"].SetValue(lightDirection);
             effect.Parameters["xAmbient"].SetValue(0.5f);
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
                 device.SetVertexBuffer(cityVertexBuffer);
                 device.DrawPrimitives(PrimitiveType.TriangleList, 0, cityVertexBuffer.VertexCount/3);
             }
         }
 
         private void DrawModel()
         {
             Matrix worldMatrix = Matrix.CreateScale(0.0005f, 0.0005f, 0.0005f) * Matrix.CreateRotationY(MathHelper.Pi) * Matrix.CreateFromQuaternion(xwingRotation) * Matrix.CreateTranslation(xwingPosition);
 
             Matrix[] xwingTransforms = new Matrix[xwingModel.Bones.Count];
             xwingModel.CopyAbsoluteBoneTransformsTo(xwingTransforms);
             foreach (ModelMesh mesh in xwingModel.Meshes)
             {
                 foreach (Effect currentEffect in mesh.Effects)
                 {
                     currentEffect.CurrentTechnique = currentEffect.Techniques["Colored"];
                     currentEffect.Parameters["xWorld"].SetValue(xwingTransforms[mesh.ParentBone.Index] * worldMatrix);
                     currentEffect.Parameters["xView"].SetValue(viewMatrix);
                     currentEffect.Parameters["xProjection"].SetValue(projectionMatrix);
                     currentEffect.Parameters["xEnableLighting"].SetValue(true);
                     currentEffect.Parameters["xLightDirection"].SetValue(lightDirection);
                     currentEffect.Parameters["xAmbient"].SetValue(0.5f);
                 }
                 mesh.Draw();
             }
         }
     }
 }
```

## Next Steps

[Quaternions: Flight kinematics](Riemers3DXNA2flightsim08flightkinematics)
