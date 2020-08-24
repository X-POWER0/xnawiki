# The effect file

One of the main differences between DirectX 9 and in XNA we need an effect for everything we draw. So what exactly is an effect?

In 3D programming, all objects are represented using triangles. Even spheres can be represented using triangles, if you use enough of them. An effect is some code that instructs your hardware (the graphics card) how it should display these triangles. An effect file contains one or more techniques, for example technique A and technique B. Drawing triangles using technique A will for example draw them semi-transparent, while drawing them using technique B will for example draw all objects using only blue-gray colors as seen in some horror movies.

Don’t worry too much about this, as this is already more advanced stuff which we will handle in Series 3. However, XNA needs an effect file to draw even the simplest triangles, so I’ve written an effect file that contains some very basic techniques. You can download it here. Right-click on the link, and select “Save As”. You should put the file on your hard drive, for example in the same map as your code files.

Now you have downloaded the effect file to the same map as your code files, we will import the file into our XNA project. In your Solution Explorer on the right site of your XNA window, find the Series3D1Content entry. Simply drag-and-drop you .fx file onto this entry. Alternatively, right-click on Series3D1Content entry. Select Add->Existing Item and browse to your .fx file. After you’ve clicked the OK button, the .fx file should be added to your Content entry as shown below:

// mgcb-editor screenshot

Next, we’ll link this effect file to a variable, so we can actually use it in our code. We will declare a new Effect object, so put this at the beginning of your class:>

```csharp
 Effect effect;
```

In your LoadContent method, add this line to have XNA load the .fx file into the effect variable for you:

```csharp
effect = Content.Load<Effect> ("effects");
```

The “effects” name refers to the part of the file before the .fx extension.

With all the necessary variables loaded, we can concentrate on the Draw method. You’ll notice the first line start with a Clear command. This line clears the buffer of our window to a specified color. Let’s set this to DarkSlateBlue, just for fun:

```csharp
 device.Clear(Color.DarkSlateBlue);
```

XNA uses a buffer to draw to, instead of drawing directly to the window. At the end of the Draw method, the contents of the buffer is drawn on the screen in one time. This way, the screen will not flicker as it would when we would draw each part of our scene separately to the screen.

Running this code will already give you the image you see below, but I would first like to add some additional code. As discussed above, to draw something we first need to specify a technique from an Effect object. We will immediately activate our effect, so next chapter we are ready to render something to the screen! Add this line to your Draw method:

```csharp
 effect.CurrentTechnique = effect.Techniques["Pretransformed"];
```

You see we select the Pretransformed technique from the effects.fx file. This technique will be used and discussed in the next chapter. The last line tells the effect and the graphics card to get ready for some work.

A technique can be made up of multiple passes, so we need to iterate through them. Add this code below the code you just entered:

```csharp
 foreach (EffectPass pass in effect.CurrentTechnique.Passes)
 {
     pass.Apply();
 }
```

All your drawing code must be put after your call to pass.Apply().

Finally, we’re through the initialization part! If you’re not yet 100% clear on effects and techniques, there’s no need to worry as we will discuss them in detail in Series 3. With all of this code set up, we’re finally ready to start drawing things on the screen, which is what we will in do next chapter.

// Current Screenshot

> You can try these exercises to practice what you've learned:
>
> - No homework today.

## Our code so far

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
         GraphicsDeviceManager graphics;
         SpriteBatch spriteBatch;
         GraphicsDevice device;
 
         Effect effect;
 
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

            effect = Content.Load<Effect> ("effects");
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
             device.Clear(Color.DarkSlateBlue);
 
             effect.CurrentTechnique = effect.Techniques["Pretransformed"];
 
             foreach (EffectPass pass in effect.CurrentTechnique.Passes)
             {
                 pass.Apply();
 
             }
 
             base.Draw(gameTime);
         }
     }
 }
```

## Next Steps

[Drawing your first Triangle](Riemers3DXNA1Terrain03triangles)
