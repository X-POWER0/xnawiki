# Drawing the skybox

This chapter, we’ll do something about the solid background color surrounding our city. This will be done using a skybox.

What exactly is a skybox? In fact, it’s nothing more than a cube with 6 nice landscape images, drawn around the camera. This gives the player the impression of being in an outdoor environment, while in fact, he’s enclosed in a box!

Using the methods we’ve created thus far, we could expect this to be quite easy, as a skybox is nothing more than a mesh. Only this time, we need to load some texture files to accompany the mesh.

You can download the mesh file itself (link), as well as the texure files (link). I’ve put the texture files in a zip file, if you’ve got problems opening it, you can also find the files here (link). The meshfile itself used to be supplied with the DirectX SDK. You can find lots of skybox texturefiles on the internet.

This time, we will need 2 variables: one to store the model, and an extra one to store the textures (because each subset of the model can have a different texture). So create these variables at the top of your code:

```csharp
 Texture2D[] skyboxTextures;
 Model skyboxModel;
```

Normally, we would use the LoadModel method to load our skybox, but we cannot do this since that method doesn’t load the textures. We’ll fix this by overloading that method: we’ll create a completely new version of the method, with exactly the same name, but with different arguments!

```csharp
 private Model LoadModel(string assetName, out Texture2D[] textures)
 {

    Model newModel = Content.Load<Model> (assetName);
    textures = new Texture2D[newModel.Meshes.Count];
    int i = 0;
    foreach (ModelMesh mesh in newModel.Meshes)
        foreach (BasicEffect currentEffect in mesh.Effects)
            textures[i++] = currentEffect.Texture;

    foreach (ModelMesh mesh in newModel.Meshes)
        foreach (ModelMeshPart meshPart in mesh.MeshParts)
            meshPart.Effect = effect.Clone();

    return newModel;
}
```

> Note that the signature of this method differs from the first LoadModel method in that it has 2 arguments.

The first line and 3 last lines are taken directly from our first LoadModel method. Only the middle part is new.

Remember all parts of the model can hold a different effect? These effects also hold the name of the texture, corresponding to its part of the model. So we cycle through each effect in our model, and save all the textures in the textures array!

Call this method from within your LoadContent method:

```csharp
 skyboxModel = LoadModel("skybox", out skyboxTextures);
```

So far for loading the skybox. All we have to do now, is draw it. What makes a skybox special? When your plane is moving, the city has to move relative to it. Not so for your skybox, which has always to be at a constant distance from your airplane. This will make our skybox look like it’s infinitely far away! So when our airplanes moves, we move the skybox with it, so our xwing will always be in the middle of it.

Add this method at the bottom of your code:

```csharp
 private void DrawSkybox()
 {
     SamplerState ss = new SamplerState();
     ss.AddressU = TextureAddressMode.Clamp;
     ss.AddressV = TextureAddressMode.Clamp;
     device.SamplerStates[0] = ss;

     DepthStencilState dss = new DepthStencilState();
     dss.DepthBufferEnable = false;
     device.DepthStencilState = dss;

     Matrix[] skyboxTransforms = new Matrix[skyboxModel.Bones.Count];
     skyboxModel.CopyAbsoluteBoneTransformsTo(skyboxTransforms);
     int i = 0;
     foreach (ModelMesh mesh in skyboxModel.Meshes)
     {
         foreach (Effect currentEffect in mesh.Effects)
         {
             Matrix worldMatrix = skyboxTransforms[mesh.ParentBone.Index] * Matrix.CreateTranslation(xwingPosition);
             currentEffect.CurrentTechnique = currentEffect.Techniques["Textured"];
             currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
             currentEffect.Parameters["xView"].SetValue(viewMatrix);
             currentEffect.Parameters["xProjection"].SetValue(projectionMatrix);
             currentEffect.Parameters["xTexture"].SetValue(skyboxTextures[i++]);
         }
         mesh.Draw();
     }

     dss = new DepthStencilState();
     dss.DepthBufferEnable = true;
     device.DepthStencilState = dss;
 }
```

This method is explained in detail in Recipe 2-8. The texture addressing mode is set to Clamp, to get rid of the texture seams you would otherwise see at the edges of the box. We need to disable the Zbuffer so we don’t have to specify a size for the skybox. The skybox is rendered around our xwing, as its positions is used to create the World matrix.

As discussed in Recipe 2-8, you need to call this method as the first line in your Draw method after clearing the screen:

```csharp
 DrawSkybox();
```

That’s it for this chapter! Running the code should give you an image as displayed below:

![Skybox](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-13Skybox1.jpg?raw=true)

## The code thus far

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
         enum CollisionType { None, Building, Boundary, Target }
 
         struct Bullet
         {
             public Vector3 position;
             public Quaternion rotation;
         }
 
         GraphicsDeviceManager graphics;
         SpriteBatch spriteBatch;
         GraphicsDevice device;
         Effect effect;
         Texture2D sceneryTexture;
         Model xwingModel;
         VertexBuffer cityVertexBuffer;
         Texture2D[] skyboxTextures;
         Model skyboxModel;

         int[,] floorPlan;
         int[] buildingHeights = new int[] { 0, 2, 2, 6, 5, 4 };
         Vector3 lightDirection = new Vector3(3, -2, 5);
         Vector3 xwingPosition = new Vector3(8, 1, -3);
         Quaternion xwingRotation = Quaternion.Identity;
         float gameSpeed = 1.0f;
         BoundingBox[] buildingBoundingBoxes;
         BoundingBox completeCityBox;
         const int maxTargets = 50;
         Model targetModel;

        List<BoundingSphere> targetList = new List<BoundingSphere> ();        Texture2D bulletTexture;

        List<Bullet> bulletList = new List<Bullet> ();        double lastBulletTime = 0;
        Vector3 cameraPosition;
        Vector3 cameraUpDirection;

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
            sceneryTexture = Content.Load<Texture2D> ("texturemap");            xwingModel = LoadModel("xwing");
            targetModel = LoadModel("target");

            bulletTexture = Content.Load<Texture2D> ("bullet");
             skyboxModel = LoadModel("skybox", out skyboxTextures);
 
             LoadFloorPlan();
             SetUpVertices();
             SetUpBoundingBoxes();
             AddTargets();
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
 
         private void SetUpBoundingBoxes()
         {
             int cityWidth = floorPlan.GetLength(0);
             int cityLength = floorPlan.GetLength(1);
 

            List<BoundingBox> bbList = new List<BoundingBox> ();            for (int x = 0; x < cityWidth; x++)
            {
                for (int z = 0; z < cityLength; z++)
                {
                    int buildingType = floorPlan[x, z];
                    if (buildingType != 0)
                    {
                        int buildingHeight = buildingHeights[buildingType];
                        Vector3[] buildingPoints = new Vector3[2];
                        buildingPoints[0] = new Vector3(x, 0, -z);
                        buildingPoints[1] = new Vector3(x + 1, buildingHeight, -z - 1);
                        BoundingBox buildingBox = BoundingBox.CreateFromPoints(buildingPoints);
                        bbList.Add(buildingBox);
                    }
                }
            }
            buildingBoundingBoxes = bbList.ToArray();

            Vector3[] boundaryPoints = new Vector3[2];
            boundaryPoints[0] = new Vector3(0, 0, 0);
            boundaryPoints[1] = new Vector3(cityWidth, 20, -cityLength);
            completeCityBox = BoundingBox.CreateFromPoints(boundaryPoints);
        }

        private void AddTargets()
        {
            int cityWidth = floorPlan.GetLength(0);
            int cityLength = floorPlan.GetLength(1);

            Random random = new Random();

            while (targetList.Count < maxTargets)
            {
                int x = random.Next(cityWidth);
                int z = -random.Next(cityLength);
                float y = (float)random.Next(2000) / 1000f + 1;
                float radius = (float)random.Next(1000) / 1000f * 0.2f + 0.01f;

                BoundingSphere newTarget = new BoundingSphere(new Vector3(x, y, z), radius);

                if (CheckCollision(newTarget) == CollisionType.None)
                    targetList.Add(newTarget);
            }
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


         private Model LoadModel(string assetName, out Texture2D[] textures)
         {

            Model newModel = Content.Load<Model> (assetName);
            textures = new Texture2D[newModel.Meshes.Count];
            int i = 0;
            foreach (ModelMesh mesh in newModel.Meshes)
                foreach (BasicEffect currentEffect in mesh.Effects)
                    textures[i++] = currentEffect.Texture;

            foreach (ModelMesh mesh in newModel.Meshes)
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
 
             ProcessKeyboard(gameTime);
             float moveSpeed = gameTime.ElapsedGameTime.Milliseconds / 500.0f * gameSpeed;
             MoveForward(ref xwingPosition, xwingRotation, moveSpeed);
 
             BoundingSphere xwingSpere = new BoundingSphere(xwingPosition, 0.04f);
             if (CheckCollision(xwingSpere) != CollisionType.None)
             {
                 xwingPosition = new Vector3(8, 1, -3);
                 xwingRotation = Quaternion.Identity;
                 gameSpeed /= 1.1f;
             }
 
             UpdateCamera();
             UpdateBulletPositions(moveSpeed);
 
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
 
             cameraPosition = campos;
             cameraUpDirection = camup;
         }
 
         private void ProcessKeyboard(GameTime gameTime)
         {
             float leftRightRot = 0;
 
             float turningSpeed = (float)gameTime.ElapsedGameTime.TotalMilliseconds / 1000.0f;
             turningSpeed *= 1.6f * gameSpeed;
             KeyboardState keys = Keyboard.GetState();
             if (keys.IsKeyDown(Keys.Right))
                 leftRightRot += turningSpeed;
             if (keys.IsKeyDown(Keys.Left))
                 leftRightRot -= turningSpeed;
 
             float upDownRot = 0;
             if (keys.IsKeyDown(Keys.Down))
                 upDownRot += turningSpeed;
             if (keys.IsKeyDown(Keys.Up))
                 upDownRot -= turningSpeed;
 
             Quaternion additionalRot = Quaternion.CreateFromAxisAngle(new Vector3(0, 0, -1), leftRightRot) * Quaternion.CreateFromAxisAngle(new Vector3(1, 0, 0), upDownRot);
             xwingRotation *= additionalRot;
 
             if (keys.IsKeyDown(Keys.Space))
             {
                 double currentTime = gameTime.TotalGameTime.TotalMilliseconds;
                 if (currentTime - lastBulletTime > 100)
                 {
                     Bullet newBullet = new Bullet();
                     newBullet.position = xwingPosition;
                     newBullet.rotation = xwingRotation;
                     bulletList.Add(newBullet);
 
                     lastBulletTime = currentTime;
                 }
             }
         }
 
         private void MoveForward(ref Vector3 position, Quaternion rotationQuat, float speed)
         {
             Vector3 addVector = Vector3.Transform(new Vector3(0, 0, -1), rotationQuat);
             position += addVector * speed;
         }
 
         private CollisionType CheckCollision(BoundingSphere sphere)
         {
             for (int i = 0; i < buildingBoundingBoxes.Length; i++)
                 if (buildingBoundingBoxes[i].Contains(sphere) != ContainmentType.Disjoint)
                     return CollisionType.Building;
 
             if (completeCityBox.Contains(sphere) != ContainmentType.Contains)
                 return CollisionType.Boundary;
 
             for (int i = 0; i < targetList.Count; i++)
             {
                 if (targetList[i].Contains(sphere) != ContainmentType.Disjoint)
                 {
                     targetList.RemoveAt(i);
                     i--;
                     AddTargets();
 
                     return CollisionType.Target;
                 }
             }
 
             return CollisionType.None;
         }
 
         private void UpdateBulletPositions(float moveSpeed)
         {
             for (int i = 0; i < bulletList.Count; i++)
             {
                 Bullet currentBullet = bulletList[i];
                 MoveForward(ref currentBullet.position, currentBullet.rotation, moveSpeed * 2.0f);
                 bulletList[i] = currentBullet;
 
                 BoundingSphere bulletSphere = new BoundingSphere(currentBullet.position, 0.05f);
                 CollisionType colType = CheckCollision(bulletSphere);
                 if (colType != CollisionType.None)
                 {
                     bulletList.RemoveAt(i);
                     i--;
 
                     if (colType == CollisionType.Target)
                         gameSpeed *= 1.05f;
                 }
             }
         }
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
 
             DrawSkybox();
             DrawCity();
             DrawModel();
             DrawTargets();
             DrawBullets();
 
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
 
         private void DrawTargets()
         {
             for (int i = 0; i < targetList.Count; i++)
             {
                 Matrix worldMatrix = Matrix.CreateScale(targetList[i].Radius) * Matrix.CreateTranslation(targetList[i].Center);
 
                 Matrix[] targetTransforms = new Matrix[targetModel.Bones.Count];
                 targetModel.CopyAbsoluteBoneTransformsTo(targetTransforms);
                 foreach (ModelMesh modmesh in targetModel.Meshes)
                 {
                     foreach (Effect currentEffect in modmesh.Effects)
                     {
                         currentEffect.CurrentTechnique = currentEffect.Techniques["Colored"];
                         currentEffect.Parameters["xWorld"].SetValue(targetTransforms[modmesh.ParentBone.Index] * worldMatrix);
                         currentEffect.Parameters["xView"].SetValue(viewMatrix);
                         currentEffect.Parameters["xProjection"].SetValue(projectionMatrix);
                         currentEffect.Parameters["xEnableLighting"].SetValue(true);
                         currentEffect.Parameters["xLightDirection"].SetValue(lightDirection);
                         currentEffect.Parameters["xAmbient"].SetValue(0.5f);
                     }
                     modmesh.Draw();
                 }
             }
         }
 
         private void DrawBullets()
         {
             if (bulletList.Count > 0)
             {
                 VertexPositionTexture[] bulletVertices = new VertexPositionTexture[bulletList.Count * 6];
                 int i = 0;
                 foreach (Bullet currentBullet in bulletList)
                 {
                     Vector3 center = currentBullet.position;
 
                     bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 1));
                     bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 0));
                     bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 0));
 
                     bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(1, 1));
                     bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 1));
                     bulletVertices[i++] = new VertexPositionTexture(center, new Vector2(0, 0));
                 }
 
                 effect.CurrentTechnique = effect.Techniques["PointSprites"];
                 effect.Parameters["xWorld"].SetValue(Matrix.Identity);
                 effect.Parameters["xProjection"].SetValue(projectionMatrix);
                 effect.Parameters["xView"].SetValue(viewMatrix);
                 effect.Parameters["xCamPos"].SetValue(cameraPosition);
                 effect.Parameters["xTexture"].SetValue(bulletTexture);
                 effect.Parameters["xCamUp"].SetValue(cameraUpDirection);
                 effect.Parameters["xPointSpriteSize"].SetValue(0.1f);
 
                 device.BlendState = BlendState.Additive;
 
                 foreach (EffectPass pass in effect.CurrentTechnique.Passes)
                 {
                     pass.Apply();
                     device.DrawUserPrimitives(PrimitiveType.TriangleList, bulletVertices, 0, bulletList.Count * 2);
                 }
 
                 device.BlendState = BlendState.Opaque;
             }
         }
 
         private void DrawSkybox()
         {
             SamplerState ss = new SamplerState();
             ss.AddressU = TextureAddressMode.Clamp;
             ss.AddressV = TextureAddressMode.Clamp;
             device.SamplerStates[0] = ss;
 
             DepthStencilState dss = new DepthStencilState();
             dss.DepthBufferEnable = false;
             device.DepthStencilState = dss;
 
             Matrix[] skyboxTransforms = new Matrix[skyboxModel.Bones.Count];
             skyboxModel.CopyAbsoluteBoneTransformsTo(skyboxTransforms);
             int i = 0;
             foreach (ModelMesh mesh in skyboxModel.Meshes)
             {
                 foreach (Effect currentEffect in mesh.Effects)
                 {
                     Matrix worldMatrix = skyboxTransforms[mesh.ParentBone.Index] * Matrix.CreateTranslation(xwingPosition);
                     currentEffect.CurrentTechnique = currentEffect.Techniques["Textured"];
                     currentEffect.Parameters["xWorld"].SetValue(worldMatrix);
                     currentEffect.Parameters["xView"].SetValue(viewMatrix);
                     currentEffect.Parameters["xProjection"].SetValue(projectionMatrix);
                     currentEffect.Parameters["xTexture"].SetValue(skyboxTextures[i++]);
                 }
                 mesh.Draw();
             }
 
             dss = new DepthStencilState();
             dss.DepthBufferEnable = true;
             device.DepthStencilState = dss;
         }
     }
 }
```

## Next Steps

[Adding camera delay](Riemers3DXNA2flightsim14cameradelay)
