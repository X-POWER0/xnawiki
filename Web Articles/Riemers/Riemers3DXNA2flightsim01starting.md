Initialization of our Project
Hi there! Glad you made it to this second series of 3D XNA Tutorials. We’re going to cover some new XNA features, put them together into one project, and end up with a real flight simulator! Again, the main goal is to show you some principles of XNA, so don’t expect complete realistic flight physics, such as gravity, coriolis and others. This would add too much maths, and draw our attention away from the XNA part.

The sole purpose of this first chapter is to set up our starting code. The code should be very simple to understand if you’ve finished the first series. This is what the starting code does:
    - Loading the effect
    - Positioning the camera
    - Clearing the window and Z buffer in the Draw method
    
So open a new Windows Game (4.0) project as described in chapter one of the first series, I named my project Series3D2. You’re free to give your project a different name, but then you must replace the namespace of my code with your project name. This line is the first line under your using-block in my code.

If you haven’t done this already, you can download my standard effect file here, which you need to import into your XNA project as explained in “The Effect file” in series 1. This effect file contains all techniques we’re going to need in this second series. Remember, you’ll learn everything you need to know about effect files in the third series. Now simply copy-paste the code below into the Game1.cs file.

Compiling and running the code should give you an empty window, cleared to a color of your choice by XNA:

// Current Screenshot

We’re ready to start our second project.

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

            effect = Content.Load<Effect> ("effects");            SetUpCamera();
        }

        private void SetUpCamera()
        {
            viewMatrix = Matrix.CreateLookAt(new Vector3(0, 0, 30), new Vector3(0, 0, 0), new Vector3(0, 1, 0));
            projectionMatrix = Matrix.CreatePerspectiveFieldOfView(MathHelper.PiOver4, device.Viewport.AspectRatio, 0.2f, 500.0f);
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

            base.Draw(gameTime);
        }
    }
}
```

## Next Steps

[Textures](Riemers3DXNA2flightsim02textures)
