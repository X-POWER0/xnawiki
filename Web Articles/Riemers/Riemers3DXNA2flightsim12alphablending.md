# Alpha blending – Bullet collision

As a difficult audience, the result of last chapter probably didn’t fully please you. My guess is the black borders around our fireballs didn’t seem too normal to you. And you’re completely right, the borders should be blended with whatever is behind them. Welcome to this chapter on alpha blending.

Up till now, if 2 objects shared the same pixel, the pixel takes the color of the object closest to the camera. In the simplest form of alpha blending, the colors of both objects are added together, and the pixel takes the ‘sum’ of their colors. This is called “Additive alpha blending”.

For example. Imagine having a completely blue background. Adding a red triangle in front of it would now, with the current settings, simply display a red triangle. With additive alpha blending turned on, the pixel of the triangle would contain blue+red, thus the whole triangle would be purple.

Black is a special case. In terms of XNA, black isn’t a color. It’s nothing. So black+blue gives simply blue. In general, black + a color results in this color. This is why I’ve drawn the borders of the fireball black, because the more black the image gets, the more of the background will be let through.

When do we need to turn on alpha blending? We need our bullets to be blended with the background, but all the rest of our scene mustn’t change. So we need to turn on alpha blending before drawing our bullets, and turn it off again afterwards.

Add the following code to our DrawBullets method, before the foreach loop that actually draws the bullets:

```csharp
 device.BlendState = BlendState.Additive;
```

In short, when 2 objects compete for one pixel and alpha blending is turned on, this is how the final color is calculated:

```csharp
    FinalColor = Color1*SourceBlend + Color2*DestinationBlend
```

Which gives in the case of additive blending:

```csharp
    FinalColor = Color1*1 + Color2*1

    FinalColor = Color1 + Color2
```

Told you, it’s simply the sum. It’s VERY important to disable alpha blending after we’ve drawn our bullets (otherwise, the next frame our entire scene would be rendered with additive alpha blending!), so put this line at the end of the method:

```csharp
 device.BlendState = BlendState.Opaque;
```

Which basically switches off alpha blending by setting DestinationBlend to 0.

Running this should give you a MUCH nicer effect on your bullets!

We have nice-looking bullets, but what good are bullets if they don’t destroy our targets? Luckily, we already have a method that can check if 2 objects collide. So go to your UpdateBulletPositions method, and add this code to the end of the for-loop that scrolls through our bulletList:

```csharp
 BoundingSphere bulletSphere = new BoundingSphere(currentBullet.position, 0.05f);
 CollisionType colType = CheckCollision(bulletSphere);
 if (colType != CollisionType.None)
 {
     bulletList.RemoveAt(i);
     i--;

     if (colType == CollisionType.Target)
         gameSpeed *= 1.05f;
 }
```

This code starts by creating a BoundingSphere object for each of our bullets. Next, you pass this to the CheckCollision method, which checks for possible collisions between this sphere and the buildings and targets in our city.

If there is a collision, the bullet is removed from the bulletList, making sure XNA never draws this bullet anymore. More important, if the bullet collided with a target, we increase the speed of our game with a notch. The CheckCollision method automatically takes care of the removal of the target.

![Alpha Blending](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-12AlphaBlending1.jpg?raw=true)

That will be all! With a 3D scene, our plane and some targets we can aim at and destroy, this is starting to look as a game. Let’s do something about that background.

You can try these exercises to practice what you've learned:
Remove the line that turns off alpha blending, and see what’s going wrong.
Our code:

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
     }
 }
```

## Next Steps

[Drawing the skybox](Riemers3DXNA2flightsim13skybox)
