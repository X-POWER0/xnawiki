# Point sprites – Billboarding

This chapter we’ll be adding bullets to our game. For these bullets, we could use real 3D spheres, but this would ask a lot from our graphics card. Instead, we’ll be using a very simple 2D image of a fireball, which you can download here (link). For this, we’ll be using a new technique from my effect file. This technique allows us to only specify the central point of the image in 3D space. The technique will do the rest, to render the image so it is always facing the viewer and is scaled to reflect the distance between the viewer and the point in 3D space. This technique is called billboarding.

Billboarding is used a lot in 3D games, for example to add trees in a forest. For a lot of detailed information and samples on billboarding, see Recipes 3-11 and 3-12.

A 2D image is also called a sprite, and since XNA needs only the center point of the image as 3D location, these 2D sprites used in 3D are called point sprites.

When being fired, we want the bullets to move forward continuously. Thus, for every bullet, we’ll need to keep track of the current position and the rotation to calculate the direction of the bullet, just as with our plane. So we’re going to define a new struct, which you should put at the very top of our code, above our variables:

```csharp
 struct Bullet
 {
     public Vector3 position;
     public Quaternion rotation;
 }
```

With this struct defined, we can create objects of the Bullet type. For each object of the Bullet type, we can store a Vector3 and a Quaternion to store the position and rotation, respectively.

We will keep track of a List containing these Bullet objects. Furthermore, to define the firing speed, we need to keep track of the last time a bullet was fired. Finally, we will need a Texture2D variable to hold the image of the bullet. So put these lines among our other variable declarations:

```csharp
 Texture2D bulletTexture;

 List<Bullet> bulletList = new List<Bullet> ();double lastBulletTime = 0;
```

You can download my sample bullet image here (link).

Go ahead and import the image into your solution as you’ve done before. Next, we’ll load the 2D image into our texture variable. This is done by adding this line to the LoadContent method:

```csharp
bulletTexture = Content.Load<Texture2D> ("bullet");
```

Now, every time the user presses the spacebar, we want a new bullet to be created and be added to our bulletList, so add this code at the bottom of our ProcessKeyboardmethod:

```csharp
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
```

When the spacebar is pressed, we compare the current time with the last time our xwing fired a bullet. If our last shot was more than 100 milliseconds ago, we want to fire a new bullet. This comes down to 10 bullets per second.

If it’s OK to fire a new bullet, we create a new Bullet object, and the current position and rotation of the xwing is stored to it. This is of course because we want the bullet to travel in the same direction as our xwing was flying the moment the bullet was fired. The last line effectively adds the newly created Bullet object to the bulletList.

Before we move on to the drawing code, let’s finish this part by making the bullets move forward. We’ve already created a method that does all the calculations: MoveForward. We’re going to create a new method, UpdateSpritePositions, which will scroll through our bulletList and update the position of every bullet. This is the code:

```csharp
 private void UpdateBulletPositions(float moveSpeed)
 {
     for (int i = 0; i < bulletList.Count; i++)
     {
         Bullet currentBullet = bulletList[i];
         MoveForward(ref currentBullet.position, currentBullet.rotation, moveSpeed * 2.0f);
         bulletList[i] = currentBullet;
     }
 }
```

Pretty straightforward. The position of every bullet in our bulletList is updated. This method will also receive the same moveSpeed as our xwing, but since we multiply it with 2.0f the bullets will go twice as fast as our xwing.

Call this method from within the Update method:

```csharp
 UpdateBulletPositions(moveSpeed);
```

Next, we want to draw our bullets. I’ve provided a technique which requires you to only specify the center point of the bullet, and your graphics card will render the bullet at that location. However, since each bullet is a square image, our graphics card will need two triangles to render. Therefore, for each bullet, you’ll need to provide 6 vertices. Luckily, for each vertex you can provide the same position, and the technique will make sure these positions are adjusted correctly.

We will define a new method, DrawSprites, which draws the bullets stored in our bulletList. Let’s start with this part:

```csharp
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
     }
 }
```

For each bullet in our bulletList, this code will add 6 vertices to the bulletVertices array. As you can see, for all vertices we specify the same position: the location of the bullet. The technique will use the texture coordinates to make sure two triangles are rendered around this center position, which is required to display the image.
This technique is called PointSprites, so we need to select it. We’ll immediately set all the parameters required by the effect to work. Add this code at the end of your if-block:

```csharp
 effect.CurrentTechnique = effect.Techniques["PointSprites"];
 effect.Parameters["xWorld"].SetValue(Matrix.Identity);
 effect.Parameters["xProjection"].SetValue(projectionMatrix);
 effect.Parameters["xView"].SetValue(viewMatrix);
 effect.Parameters["xCamPos"].SetValue(cameraPosition);
 effect.Parameters["xTexture"].SetValue(bulletTexture);
 effect.Parameters["xCamUp"].SetValue(cameraUpDirection);
 effect.Parameters["xPointSpriteSize"].SetValue(0.1f);
```

As for all objects rendered into a 3D world, you first need to set the World, View and Projection matrices. Specific to the PointSprites technique, you need to pas the camera position, as well as the camera Up direction, allowing the technique to render the 2 triangles so they’re always facing the camera. Finally, you need to pass the texture, and define how large you want it to be.

Obviously, before this can work, we first need to define both camera variables at the very top of our code:

```csharp
 Vector3 cameraPosition;
 Vector3 cameraUpDirection;
```

We already calculated this in our UpdateCamera method, so add these lines to the end of that method:

```csharp
 cameraPosition = campos;
 cameraUpDirection = camup;
```

Back to our DrawBullets method. With the vertices ready and the effect set up correctly, all we need to do now is render the bullets! Which is done by the following code:

```csharp
 foreach (EffectPass pass in effect.CurrentTechnique.Passes)
 {
    pass.Apply();

    device.DrawUserPrimitives(PrimitiveType.TriangleList, bulletVertices, 0, bulletList.Count * 2);
 }

DrawBullets();
```

In this example, the CPU is calculating the new position of each bullet for each frame, creates an array of vertices from these positions and sends them to the graphics card for each frame. It would be much better and faster to have the GPU calculate all the new positions. See Recipe 3-12 on how this can be done using a particle engine.

This concludes the method; All we have to do is call this method from the bottom of our Draw method:

```csharp
 DrawBullets();
```
Now try to run this code!

You should see a screen as the one below:

![Shooting](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA2-11Shoot1.jpg?raw=true)

We will fine-tune our bullets in the next chapter. This will also include detecting collisions between a bullet and an obstacle. In case of a collision, the bullet will be removed from our bulletList, which will free the memory.

> You can try these exercises to practice what you've learned:
>
> - Change the size of the point sprites.

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

        List<BoundingSphere> targetList = new List<BoundingSphere> ();
         Texture2D bulletTexture;

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
 
                 foreach (EffectPass pass in effect.CurrentTechnique.Passes)
                 {
                     pass.Apply();
                     device.DrawUserPrimitives(PrimitiveType.TriangleList, bulletVertices, 0, bulletList.Count * 2);
                 }
             }
         }
     }
 }
```

## Next Steps

[Alpha blending – Bullet collision](Riemers3DXNA2flightsim12alphablending)
