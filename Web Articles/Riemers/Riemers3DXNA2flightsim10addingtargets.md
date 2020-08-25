# Adding targets

With our scene set up and our xwing flying through it, it’s time to add some targets to shoot at. In this example, we’ll be using simple spheres. As done before with our xwing, we will load this model from a file.

You can download an example target model here: link. It is a simple red sphere, of which the size is exactly 1. Next, add this .x file to the Content entry of your solution, as done before.

Next, we’ll define a variable at the top of our code that indicates the maximal number of targets we want to have in our city, as well as a Model variable that will hold the structure of our target:

```csharp
 const int maxTargets = 50;
 Model targetModel;

List<BoundingSphere> targetList = new List<BoundingSphere> ();
```

We’ll use the last variable to hold track of our target. A List is exceptionally useful here, as we will need to be able add and remove target from the list while our program is running. For each target we need to store 2 things:

- its position and
- its size

Therefore, a BoundingSphere works perfectly to store a target! Even more, later on we can use the built-in functionality of the BoundingSphere object to test for collisions.

First, add this line to our LoadContent method, to load the .x file into the targetModel variable:

```csharp
 targetModel = LoadModel("target");
```

Which calls the LoadModel method that fills our targetModel variable, and replaces all effects in that model with copies of our own effect.

We will code a new method, AddTargets, which will create our targets and add them to our List:

```csharp
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

         BoundingSphere newTarget = new BoundingSphere(new Vector3(x,y,z), radius);

         if (CheckCollision(newTarget) == CollisionType.None)
             targetList.Add(newTarget);
     }
 }
```

This method starts by looking up the size of our city and by creating a new randomizer, from which we will draw random numbers to generate random positions and sizes for our targets.

Each iteration of the while loop, we will create a new target, see if it doesn’t collide with a building of our city and add it to the list.

This is done by first generating random X,Y,Z coordinates and a random size for our new target. These are used to create a new BoundingSphere. As we have a BoundingSphere, we can simply pass this to our CheckCollision method to check if it doesn’t collide with any buildings of our city! Later on in this chapter, you’ll extend the CheckCollision method so it also checks for collisions with our targets, so by calling this method here you also make sure 2 targets will never collide.

If the newly created target doesn’t collide with the buildings of the city or with any existing targets, the target is added to the List.

Call this method from the end of our LoadContent method:

```csharp
 AddTargets();
```

Running this code shouldn’t give any problems, but you won’t notice a difference either as we’re not yet drawing the targets. Add this method to the bottom of our code:

```csharp
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
```

This method does exactly the same as our DrawModel method, except that it’s code is iterated for each target in our targetList.

The World matrix that is created for each target sets the position and size of the target. Next, the target is rendered using the Colored technique, as the vertices in this particular .x file contains only Colored and Normal information.

Make sure you call this method from within our Draw method:

```csharp
 DrawTargets();
```

Running this code will already render the targets in your 3D city!

All we have to do now is detect when the xwing crashes against a target. Even more, when this happens, the target should be removed from the targetList. So add this code to the CheckCollision method:

```csharp
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
```

For each target in your targetList, you check if it collides with the BoundingSpere passed to the method. If it does, you remove it from the list and call AddTargets to add a new one at a random position. Don’t forget to return the type of collision to the calling code!

2 remarks:

- When you remove an entry from a List, you should decrement the counter, hence the i—line. Otherwise, the following object in the list will not be checked!
- This detects when the xwing crashes into a target, but also makes sure 2 targets will never be added to the same positions. This is because we also call this method from the AddTargets method.

That’s it! When you run this code, the targets should be drawn in the city. When you crash your xwing against them, your xwing should be reset to its original position.

![Targets](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-10Targets1.jpg?raw=true)

What good are targets when you can’t shoot at them? Next chapter we’ll add some bullets.

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
         enum CollisionType { None, Building, Boundary, Target }
 
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

        List<BoundingSphere> targetList = new List<BoundingSphere> ();
 
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
 
         protected override void Draw(GameTime gameTime)
         {
             device.Clear(ClearOptions.Target | ClearOptions.DepthBuffer, Color.DarkSlateBlue, 1.0f, 0);
 
             DrawCity();
             DrawModel();
             DrawTargets();
 
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
     }
 }
```

## Next Steps

[Point sprites – Billboarding](Riemers3DXNA2flightsim11pointsprites)
