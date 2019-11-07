# Graphics 2D
## Requires
- Visual Studio 2010
## License
- Apache License, Version 2.0
## Technologies
- Animation
- Graphics
- Windows Phone
## Topics
- Animation
- Graphics
- Windows Phone
## Updated
- 02/11/2014
## Description

<h1>Introduction</h1>
<p><em>Graphics 2D is a sample of animation from Engineering and Computer Works utilising the template from Microsoft for Graphics2DSample3. The animation features printed characters, moving asteroid and spacs ships and a running green man in space. Touch the
 screen to further the simple animation.</em></p>
<h1><span>Building the Sample</span></h1>
<p><em>Built on Visual Studio 2010. </em></p>
<p><span style="font-size:20px; font-weight:bold">Description</span></p>
<p><em>Graphics 2D is a sample of animation from Engineering and Computer Works utillising the template from Microsoft for Graphics2D Sample3. The animation features printged characters, moving asteriod and space ships and a running green man in space. Touch
 the screen and another screen with two asteoids and two spacde ships will appear. The asteroids and space ships are moving about the screens.</em></p>
<p>&nbsp;</p>
<div class="scriptcode">
<div class="pluginEditHolder" pluginCommand="mceScriptCode">
<div class="title"><span>Visual Basic</span></div>
<div class="pluginLinkHolder"><span class="pluginEditHolderLink">Edit</span>|<span class="pluginRemoveHolderLink">Remove</span></div>
<span class="hidden">vb</span>
<pre class="hidden">#region File Information
//-----------------------------------------------------------------------------
// Graphics2DSample.cs
//
// Microsoft XNA Community Game Platform
// Copyright (C) Microsoft Corporation. All rights reserved.
//-----------------------------------------------------------------------------
#endregion

#region Using Statements
using System;
using System.Collections.Generic;
using System.Linq;
using Microsoft.Xna.Framework;
using Microsoft.Xna.Framework.Audio;
using Microsoft.Xna.Framework.Content;
using Microsoft.Xna.Framework.GamerServices;
using Microsoft.Xna.Framework.Graphics;
using Microsoft.Xna.Framework.Input;
using Microsoft.Xna.Framework.Input.Touch;
using Microsoft.Xna.Framework.Media;
using System.Globalization;

#endregion

namespace Graphics2DSample3
{
    /// &lt;summary&gt;
    /// This is the main type for your game
    /// &lt;/summary&gt;
    public class Graphics2DSampleGame : Microsoft.Xna.Framework.Game
    {
        #region Fields
        GraphicsDeviceManager graphics;
        SpriteBatch spriteBatch;

        // Font Variables
        SpriteFont calibri; // For SpriteFont usage sample
        SpriteFont buxton; // For BitmapFont as SpriteFont usage sample
        SpriteFont quadrantFont; // SpriteFont for in-quadrant descriptions

        // Texture Variables
        Texture2D background; // Screen background texture
        Texture2D frame; // Screen frame to make final UI looks like four-quadrant layout
        Texture2D ufo;
        Texture2D asteroid;
        Texture2D ground;

        // Gameplay variables
        Vector2 ufoPosition;
        Vector2 asteroidPosition;
        Vector2 groundPosition;
        Vector2 ufoVelocity;
        float asteroidRotation;
        float asteroidScale;
        bool isGrowing;
        Random random;

        // Animation variables
        Vector2 characterPosition;
        Animation animation;
        #endregion

        #region Initialization
        public Graphics2DSampleGame()
        {
            graphics = new GraphicsDeviceManager(this);
            Content.RootDirectory = &quot;Content&quot;;

            // Frame rate is 30 fps by default for Windows Phone.
            TargetElapsedTime = TimeSpan.FromTicks(333333);

            graphics.IsFullScreen = true; // Hide system tray for better gameplay experience
        }

        /// &lt;summary&gt;
        /// Allows the game to perform any initialization it needs to before starting to run.
        /// This is where it can query for any required services and load any non-graphic
        /// related content.  Calling base.Initialize will enumerate through any components
        /// and initialize them as well.
        /// &lt;/summary&gt;
        protected override void Initialize()
        {
            // Initialize initial gameplay state and variables
            random = new Random();
            asteroidPosition = new Vector2(600, 100);
            ufoPosition = new Vector2(180, 280);
            asteroidRotation = 0;
            asteroidScale = 1.0f;
            isGrowing = true;
            while (ufoVelocity.X == 0 || ufoVelocity.Y == 0)
                ufoVelocity = new Vector2(random.Next(-1, 2), random.Next(-1, 2));

            base.Initialize();
        }
        #endregion

        #region Load and Unload
        /// &lt;summary&gt;
        /// LoadContent will be called once per game and is the place to load
        /// all of your content.
        /// &lt;/summary&gt;
        protected override void LoadContent()
        {
            // Create a new SpriteBatch, which can be used to draw textures.
            spriteBatch = new SpriteBatch(GraphicsDevice);

            // Load game content here
            calibri = Content.Load&lt;SpriteFont&gt;(&quot;PericlesSpriteFont&quot;);
            buxton = Content.Load&lt;SpriteFont&gt;(&quot;BuxtonSketchBitmapFont&quot;);
            quadrantFont = Content.Load&lt;SpriteFont&gt;(&quot;QuadrantFont&quot;);
            background = Content.Load&lt;Texture2D&gt;(&quot;spaceBG&quot;);
            ufo = Content.Load&lt;Texture2D&gt;(&quot;UFO&quot;);
            asteroid = Content.Load&lt;Texture2D&gt;(&quot;asteroid&quot;);
            ground = Content.Load&lt;Texture2D&gt;(&quot;ground&quot;);
            frame = Content.Load&lt;Texture2D&gt;(&quot;frame&quot;);

            int borderWidth = 1; // A width of the screen border
            groundPosition = new Vector2(GraphicsDevice.Viewport.Width &#43; ground.Width &#43; borderWidth,
                                         GraphicsDevice.Viewport.Height &#43; ground.Height &#43; borderWidth);

            // Load multiple animations form XML definition
            System.Xml.Linq.XDocument doc = System.Xml.Linq.XDocument.Load(&quot;Content/AnimationDef.xml&quot;);
         //   System.Xml.Linq.XDocument a = System.Xml.Linq.XDocument.Load(&quot;Content/AnimationDef1.xml&quot;);
           // System.Xml.Linq.XDocument doc = System.Xml.Linq.XDocument.Load(&quot;Content/AnimationDef1.xml&quot;);
            // Get the first (and only in this case) animation from the XML definition
            var definition = doc.Root.Element(&quot;Definition&quot;);
           // var def = a.Root.Element(&quot;Definition&quot;);
            Texture2D texture = Content.Load&lt;Texture2D&gt;(definition.Attribute(&quot;SheetName&quot;).Value);
          //  Texture2D texture1 = Content.Load&lt;Texture2D&gt;(def.Attribute(&quot;SheetName&quot;).Value);
            Point frameSize = new Point();
            frameSize.X = int.Parse(definition.Attribute(&quot;FrameWidth&quot;).Value, NumberStyles.Integer);
            frameSize.Y = int.Parse(definition.Attribute(&quot;FrameHeight&quot;).Value, NumberStyles.Integer);
         //   frameSize.X = int.Parse(def.Attribute(&quot;FrameWidth&quot;).Value, NumberStyles.Integer);
         //   frameSize.Y = int.Parse(def.Attribute(&quot;FrameHeight&quot;).Value, NumberStyles.Integer);
            Point sheetSize = new Point();
            sheetSize.X = int.Parse(definition.Attribute(&quot;SheetColumns&quot;).Value, NumberStyles.Integer);
            sheetSize.Y = int.Parse(definition.Attribute(&quot;SheetRows&quot;).Value, NumberStyles.Integer);
        //    sheetSize.X = int.Parse(def.Attribute(&quot;SheetColumns&quot;).Value, NumberStyles.Integer);
        //    sheetSize.Y = int.Parse(def.Attribute(&quot;SheetRows&quot;).Value, NumberStyles.Integer);
            TimeSpan frameInterval = TimeSpan.FromSeconds(1.0f / int.Parse(definition.Attribute(&quot;Speed&quot;).Value, NumberStyles.Integer));
        //    TimeSpan frameInterval = TimeSpan.FromSeconds(1.0f / int.Parse(def.Attribute(&quot;Speed&quot;).Value, NumberStyles.Integer));

            // Define animation position at the bottom of the screen
            characterPosition = new Vector2(GraphicsDevice.Viewport.Width / 2, GraphicsDevice.Viewport.Height - frameSize.Y);

            // Define a new Animation instance
            animation = new Animation(texture, frameSize, sheetSize, frameInterval);
        }

        /// &lt;summary&gt;
        /// UnloadContent will be called once per game and is the place to unload
        /// all content.
        /// &lt;/summary&gt;
        protected override void UnloadContent()
        {
            // TODO: Unload any non ContentManager content here
        }
        #endregion

        #region Update
        /// &lt;summary&gt;
        /// Allows the game to run logic such as updating the world,
        /// checking for collisions, gathering input, and playing audio.
        /// &lt;/summary&gt;
        /// &lt;param name=&quot;gameTime&quot;&gt;Provides a snapshot of timing values.&lt;/param&gt;
        protected override void Update(GameTime gameTime)
        {
            // Allows the game to exit
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed)
                this.Exit();

            // TODO: Add your update logic here
            float elapsed = (float)gameTime.ElapsedGameTime.TotalSeconds;

            // Calculate UFO movement
            UpdateUFO(elapsed);

            // Scale and rotate the asteroid
            UpdateAsteroid();

            // Progress the animation
            bool isProgressed = animation.Update(gameTime);

            // If animation progressed to the next frame change the position accordingly
            if (isProgressed)
            {
                characterPosition.X &#43;= animation.frameSize.X / 2;
                if (characterPosition.X &gt; GraphicsDevice.Viewport.Width)
                    characterPosition.X = GraphicsDevice.Viewport.Width / 2;
            }

            base.Update(gameTime);
        }

        /// &lt;summary&gt;
        /// Update the asteroid position
        /// &lt;/summary&gt;
        private void UpdateAsteroid()
        {
            asteroidRotation &#43;= 10;
            if (asteroidRotation &gt;= 360)
                asteroidRotation = 0;

            asteroidScale = asteroidScale &#43; (isGrowing ? 0.01f : -0.01f);
            if (asteroidScale &gt; 2.0f)
                isGrowing = false;
            else if (asteroidScale &lt; 0.5f)
                isGrowing = true;
        }

        /// &lt;summary&gt;
        /// Update the UFO position
        /// &lt;/summary&gt;
        /// &lt;param name=&quot;elapsed&quot;&gt;&lt;/param&gt;
        private void UpdateUFO(float elapsed)
        {
            ufoPosition &#43;= ufoVelocity * 64.0f * elapsed;
            ufoPosition.X = MathHelper.Clamp(ufoPosition.X, 0, GraphicsDevice.Viewport.Width / 2 - ufo.Width);
            if (ufoPosition.X == 0 || ufoPosition.X == GraphicsDevice.Viewport.Width / 2 - ufo.Width)
                ufoVelocity.X = random.Next(-1, 2);

            if (ufoVelocity.X == 0)
                ufoVelocity.X = 1;

            ufoPosition.Y = MathHelper.Clamp(ufoPosition.Y, GraphicsDevice.Viewport.Height / 2, GraphicsDevice.Viewport.Height - ufo.Height);
            if (ufoPosition.Y == GraphicsDevice.Viewport.Height / 2 || ufoPosition.Y == GraphicsDevice.Viewport.Height - ufo.Height)
                ufoVelocity.Y = random.Next(-1, 2);

            if (ufoVelocity.Y == 0)
                ufoVelocity.Y = 1;
        }
        #endregion

        #region Render
        /// &lt;summary&gt;
        /// This is called when the game should draw itself.
        /// &lt;/summary&gt;
        /// &lt;param name=&quot;gameTime&quot;&gt;Provides a snapshot of timing values.&lt;/param&gt;
        protected override void Draw(GameTime gameTime)
        {
            GraphicsDevice.Clear(Color.CornflowerBlue);

            // Draw the background texture
            spriteBatch.Begin();
            // Draw a background picture 
            spriteBatch.Draw(background, new Vector2(0, 0), Color.White);

            //Draw asteroid picture - rotated
            DrawTopRightQuadrant();
          //  DrawBottomRightQuadrant();
            // Draw the text
            DrawTopLeftQuadrant();
        //    DrawBottomRightQuadrant();
            // Draw UFO picture
            DrawBottomLeftQuadrant();
         //   DrawBottomRightQuadrant();
            DrawBottomRightQuadrant();

            // Draw the frame
            spriteBatch.Draw(frame, new Vector2(0, 0), Color.Gray);

            spriteBatch.End(); //Don't forget to close it at the end.

            base.Draw(gameTime);
        }

        private void DrawBottomRightQuadrant()
        {//Animated Sprite
            Vector2 textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 &#43; 10,
                                               GraphicsDevice.Viewport.Height / 2 &#43; 10);
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White);

            // Draw the ground
            spriteBatch.Draw(ground, groundPosition, Color.White);

            // Produce animation
            animation.Draw(spriteBatch, characterPosition, SpriteEffects.None);
        }

        private void DrawBottomLeftQuadrant()
        {
            Vector2 textPosition = new Vector2(10, GraphicsDevice.Viewport.Height / 2 &#43; 10);
           /// spriteBatch.DrawString(quadrantFont, &quot;Translation&quot;, textPosition, Color.White);

            // Draw the UFO at current location
            spriteBatch.Draw(ufo, ufoPosition, Color.White);
            spriteBatch.Draw(ufo, ufoPosition,
null, Color.White, MathHelper.ToRadians(asteroidRotation),
new Vector2(ufo.Width, ufo.Height), asteroidScale,
SpriteEffects.None, 0);

         //   Copy
 // Process touch events
TouchCollection touchCollection = TouchPanel.GetState();
foreach (TouchLocation tl in touchCollection)
{
    if ((tl.State == TouchLocationState.Pressed)
            || (tl.State == TouchLocationState.Moved))
    {
        spriteBatch.Draw(ufo, ufoPosition,
  null, Color.White, MathHelper.ToRadians(asteroidRotation),
  new Vector2(ufo.Width, ufo.Height), asteroidScale,
  SpriteEffects.None, 0);
        // Draw asteroid
        spriteBatch.Draw(asteroid, asteroidPosition,
          null, Color.White, MathHelper.ToRadians(asteroidRotation),
          new Vector2(asteroid.Width / 2, asteroid.Height / 2), asteroidScale,
          SpriteEffects.None, 0);

        // add sparkles based on the touch location
       // sparkles.Add(new Sparkle(tl.Position.X,
        //         tl.Position.Y, ttms));

    }
}


   
         //   Vector2 textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 &#43; 10,
//                            GraphicsDevice.Viewport.Height / 2 &#43; 10);Animated Sprite
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White);

            // Draw the ground
          //  spriteBatch.Draw(ground, groundPosition, Color.White);

            // Produce animation
           // animation.Draw(spriteBatch, characterPosition, SpriteEffects.None);
        
        }

        private void DrawTopLeftQuadrant()
        {//Sprite and Bitmap Fonts
            Vector2 textPosition = new Vector2(10, 10);
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White);
             
            // Draw text using Sprite Font
            string text = &quot;&quot;;//&quot;Sprite Font&quot;;Junt Hoong Chan
            Vector2 size = calibri.MeasureString(text);
            textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 - size.X / 2, 40);
            spriteBatch.DrawString(calibri, text, textPosition, Color.Blue);

            // Draw text using Bitmap Font with primitive shadow effect
            text = &quot;Engineering and Computer Works&quot;;//&quot;Bitmap Font&quot;;
            size = buxton.MeasureString(text);
            textPosition = new Vector2((GraphicsDevice.Viewport.Width / 2 &#43; 2) - size.X / 2, 104);
            spriteBatch.DrawString(buxton, text, textPosition, Color.Gray);
            spriteBatch.DrawString(buxton, text, textPosition - new Vector2(4, 4), Color.CadetBlue);
        }

        private void DrawTopRightQuadrant()
        {//Scale and Rotate
            Vector2 textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 &#43; 10, 10);
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White);

            // Draw asteroid
            spriteBatch.Draw(asteroid, asteroidPosition,
              null, Color.White, MathHelper.ToRadians(asteroidRotation),
             new Vector2(asteroid.Width, asteroid.Height), asteroidScale,
              SpriteEffects.None, 0);
        ///      

            // Draw asteroid
            spriteBatch.Draw(asteroid, asteroidPosition,
              null, Color.White, MathHelper.ToRadians(asteroidRotation),
              new Vector2(asteroid.Width / 2, asteroid.Height / 2), asteroidScale,
              SpriteEffects.None, 0);
        }

        
        #endregion
    }
}

#region File Information 
//----------------------------------------------------------------------------- 
// Graphics2DSample.cs 
// 
// Microsoft XNA Community Game Platform 
// Copyright (C) Microsoft Corporation. All rights reserved. 
//----------------------------------------------------------------------------- 
#endregion 
 
#region Using Statements 
using System; 
using System.Collections.Generic; 
using System.Linq; 
using Microsoft.Xna.Framework; 
using Microsoft.Xna.Framework.Audio; 
using Microsoft.Xna.Framework.Content; 
using Microsoft.Xna.Framework.GamerServices; 
using Microsoft.Xna.Framework.Graphics; 
using Microsoft.Xna.Framework.Input; 
using Microsoft.Xna.Framework.Input.Touch; 
using Microsoft.Xna.Framework.Media; 
using System.Globalization; 
 
#endregion 
 
namespace Graphics2DSample3 
{ 
    /// &lt;summary&gt; 
    /// This is the main type for your game 
    /// &lt;/summary&gt; 
    public class Graphics2DSampleGame : Microsoft.Xna.Framework.Game 
    { 
        #region Fields 
        GraphicsDeviceManager graphics; 
        SpriteBatch spriteBatch; 
 
        // Font Variables 
        SpriteFont calibri; // For SpriteFont usage sample 
        SpriteFont buxton; // For BitmapFont as SpriteFont usage sample 
        SpriteFont quadrantFont; // SpriteFont for in-quadrant descriptions 
 
        // Texture Variables 
        Texture2D background; // Screen background texture 
        Texture2D frame; // Screen frame to make final UI looks like four-quadrant layout 
        Texture2D ufo; 
        Texture2D asteroid; 
        Texture2D ground; 
 
        // Gameplay variables 
        Vector2 ufoPosition; 
        Vector2 asteroidPosition; 
        Vector2 groundPosition; 
        Vector2 ufoVelocity; 
        float asteroidRotation; 
        float asteroidScale; 
        bool isGrowing; 
        Random random; 
 
        // Animation variables 
        Vector2 characterPosition; 
        Animation animation; 
        #endregion 
 
        #region Initialization 
        public Graphics2DSampleGame() 
        { 
            graphics = new GraphicsDeviceManager(this); 
            Content.RootDirectory = &quot;Content&quot;; 
 
            // Frame rate is 30 fps by default for Windows Phone. 
            TargetElapsedTime = TimeSpan.FromTicks(333333); 
 
            graphics.IsFullScreen = true; // Hide system tray for better gameplay experience 
        } 
 
        /// &lt;summary&gt; 
        /// Allows the game to perform any initialization it needs to before starting to run. 
        /// This is where it can query for any required services and load any non-graphic 
        /// related content.  Calling base.Initialize will enumerate through any components 
        /// and initialize them as well. 
        /// &lt;/summary&gt; 
        protected override void Initialize() 
        { 
            // Initialize initial gameplay state and variables 
            random = new Random(); 
            asteroidPosition = new Vector2(600, 100); 
            ufoPosition = new Vector2(180, 280); 
            asteroidRotation = 0; 
            asteroidScale = 1.0f; 
            isGrowing = true; 
            while (ufoVelocity.X == 0 || ufoVelocity.Y == 0) 
                ufoVelocity = new Vector2(random.Next(-1, 2), random.Next(-1, 2)); 
 
            base.Initialize(); 
        } 
        #endregion 
 
        #region Load and Unload 
        /// &lt;summary&gt; 
        /// LoadContent will be called once per game and is the place to load 
        /// all of your content. 
        /// &lt;/summary&gt; 
        protected override void LoadContent() 
        { 
            // Create a new SpriteBatch, which can be used to draw textures. 
            spriteBatch = new SpriteBatch(GraphicsDevice); 
 
            // Load game content here 
            calibri = Content.Load&lt;SpriteFont&gt;(&quot;PericlesSpriteFont&quot;); 
            buxton = Content.Load&lt;SpriteFont&gt;(&quot;BuxtonSketchBitmapFont&quot;); 
            quadrantFont = Content.Load&lt;SpriteFont&gt;(&quot;QuadrantFont&quot;); 
            background = Content.Load&lt;Texture2D&gt;(&quot;spaceBG&quot;); 
            ufo = Content.Load&lt;Texture2D&gt;(&quot;UFO&quot;); 
            asteroid = Content.Load&lt;Texture2D&gt;(&quot;asteroid&quot;); 
            ground = Content.Load&lt;Texture2D&gt;(&quot;ground&quot;); 
            frame = Content.Load&lt;Texture2D&gt;(&quot;frame&quot;); 
 
            int borderWidth = 1; // A width of the screen border 
            groundPosition = new Vector2(GraphicsDevice.Viewport.Width &#43; ground.Width &#43; borderWidth, 
                                         GraphicsDevice.Viewport.Height &#43; ground.Height &#43; borderWidth); 
 
            // Load multiple animations form XML definition 
            System.Xml.Linq.XDocument doc = System.Xml.Linq.XDocument.Load(&quot;Content/AnimationDef.xml&quot;); 
         //   System.Xml.Linq.XDocument a = System.Xml.Linq.XDocument.Load(&quot;Content/AnimationDef1.xml&quot;); 
           // System.Xml.Linq.XDocument doc = System.Xml.Linq.XDocument.Load(&quot;Content/AnimationDef1.xml&quot;); 
            // Get the first (and only in this case) animation from the XML definition 
            var definition = doc.Root.Element(&quot;Definition&quot;); 
           // var def = a.Root.Element(&quot;Definition&quot;); 
            Texture2D texture = Content.Load&lt;Texture2D&gt;(definition.Attribute(&quot;SheetName&quot;).Value); 
          //  Texture2D texture1 = Content.Load&lt;Texture2D&gt;(def.Attribute(&quot;SheetName&quot;).Value); 
            Point frameSize = new Point(); 
            frameSize.X = int.Parse(definition.Attribute(&quot;FrameWidth&quot;).Value, NumberStyles.Integer); 
            frameSize.Y = int.Parse(definition.Attribute(&quot;FrameHeight&quot;).Value, NumberStyles.Integer); 
         //   frameSize.X = int.Parse(def.Attribute(&quot;FrameWidth&quot;).Value, NumberStyles.Integer); 
         //   frameSize.Y = int.Parse(def.Attribute(&quot;FrameHeight&quot;).Value, NumberStyles.Integer); 
            Point sheetSize = new Point(); 
            sheetSize.X = int.Parse(definition.Attribute(&quot;SheetColumns&quot;).Value, NumberStyles.Integer); 
            sheetSize.Y = int.Parse(definition.Attribute(&quot;SheetRows&quot;).Value, NumberStyles.Integer); 
        //    sheetSize.X = int.Parse(def.Attribute(&quot;SheetColumns&quot;).Value, NumberStyles.Integer); 
        //    sheetSize.Y = int.Parse(def.Attribute(&quot;SheetRows&quot;).Value, NumberStyles.Integer); 
            TimeSpan frameInterval = TimeSpan.FromSeconds(1.0f / int.Parse(definition.Attribute(&quot;Speed&quot;).Value, NumberStyles.Integer)); 
        //    TimeSpan frameInterval = TimeSpan.FromSeconds(1.0f / int.Parse(def.Attribute(&quot;Speed&quot;).Value, NumberStyles.Integer)); 
 
            // Define animation position at the bottom of the screen 
            characterPosition = new Vector2(GraphicsDevice.Viewport.Width / 2, GraphicsDevice.Viewport.Height - frameSize.Y); 
 
            // Define a new Animation instance 
            animation = new Animation(texture, frameSize, sheetSize, frameInterval); 
        } 
 
        /// &lt;summary&gt; 
        /// UnloadContent will be called once per game and is the place to unload 
        /// all content. 
        /// &lt;/summary&gt; 
        protected override void UnloadContent() 
        { 
            // TODO: Unload any non ContentManager content here 
        } 
        #endregion 
 
        #region Update 
        /// &lt;summary&gt; 
        /// Allows the game to run logic such as updating the world, 
        /// checking for collisions, gathering input, and playing audio. 
        /// &lt;/summary&gt; 
        /// &lt;param name=&quot;gameTime&quot;&gt;Provides a snapshot of timing values.&lt;/param&gt; 
        protected override void Update(GameTime gameTime) 
        { 
            // Allows the game to exit 
            if (GamePad.GetState(PlayerIndex.One).Buttons.Back == ButtonState.Pressed) 
                this.Exit(); 
 
            // TODO: Add your update logic here 
            float elapsed = (float)gameTime.ElapsedGameTime.TotalSeconds; 
 
            // Calculate UFO movement 
            UpdateUFO(elapsed); 
 
            // Scale and rotate the asteroid 
            UpdateAsteroid(); 
 
            // Progress the animation 
            bool isProgressed = animation.Update(gameTime); 
 
            // If animation progressed to the next frame change the position accordingly 
            if (isProgressed) 
            { 
                characterPosition.X &#43;= animation.frameSize.X / 2; 
                if (characterPosition.X &gt; GraphicsDevice.Viewport.Width) 
                    characterPosition.X = GraphicsDevice.Viewport.Width / 2; 
            } 
 
            base.Update(gameTime); 
        } 
 
        /// &lt;summary&gt; 
        /// Update the asteroid position 
        /// &lt;/summary&gt; 
        private void UpdateAsteroid() 
        { 
            asteroidRotation &#43;= 10; 
            if (asteroidRotation &gt;= 360) 
                asteroidRotation = 0; 
 
            asteroidScale = asteroidScale &#43; (isGrowing ? 0.01f : -0.01f); 
            if (asteroidScale &gt; 2.0f) 
                isGrowing = false; 
            else if (asteroidScale &lt; 0.5f) 
                isGrowing = true; 
        } 
 
        /// &lt;summary&gt; 
        /// Update the UFO position 
        /// &lt;/summary&gt; 
        /// &lt;param name=&quot;elapsed&quot;&gt;&lt;/param&gt; 
        private void UpdateUFO(float elapsed) 
        { 
            ufoPosition &#43;= ufoVelocity * 64.0f * elapsed; 
            ufoPosition.X = MathHelper.Clamp(ufoPosition.X, 0, GraphicsDevice.Viewport.Width / 2 - ufo.Width); 
            if (ufoPosition.X == 0 || ufoPosition.X == GraphicsDevice.Viewport.Width / 2 - ufo.Width) 
                ufoVelocity.X = random.Next(-1, 2); 
 
            if (ufoVelocity.X == 0) 
                ufoVelocity.X = 1; 
 
            ufoPosition.Y = MathHelper.Clamp(ufoPosition.Y, GraphicsDevice.Viewport.Height / 2, GraphicsDevice.Viewport.Height - ufo.Height); 
            if (ufoPosition.Y == GraphicsDevice.Viewport.Height / 2 || ufoPosition.Y == GraphicsDevice.Viewport.Height - ufo.Height) 
                ufoVelocity.Y = random.Next(-1, 2); 
 
            if (ufoVelocity.Y == 0) 
                ufoVelocity.Y = 1; 
        } 
        #endregion 
 
        #region Render 
        /// &lt;summary&gt; 
        /// This is called when the game should draw itself. 
        /// &lt;/summary&gt; 
        /// &lt;param name=&quot;gameTime&quot;&gt;Provides a snapshot of timing values.&lt;/param&gt; 
        protected override void Draw(GameTime gameTime) 
        { 
            GraphicsDevice.Clear(Color.CornflowerBlue); 
 
            // Draw the background texture 
            spriteBatch.Begin(); 
            // Draw a background picture  
            spriteBatch.Draw(background, new Vector2(0, 0), Color.White); 
 
            //Draw asteroid picture - rotated 
            DrawTopRightQuadrant(); 
          //  DrawBottomRightQuadrant(); 
            // Draw the text 
            DrawTopLeftQuadrant(); 
        //    DrawBottomRightQuadrant(); 
            // Draw UFO picture 
            DrawBottomLeftQuadrant(); 
         //   DrawBottomRightQuadrant(); 
            DrawBottomRightQuadrant(); 
 
            // Draw the frame 
            spriteBatch.Draw(frame, new Vector2(0, 0), Color.Gray); 
 
            spriteBatch.End(); //Don't forget to close it at the end. 
 
            base.Draw(gameTime); 
        } 
 
        private void DrawBottomRightQuadrant() 
        {//Animated Sprite 
            Vector2 textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 &#43; 10, 
                                               GraphicsDevice.Viewport.Height / 2 &#43; 10); 
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White); 
 
            // Draw the ground 
            spriteBatch.Draw(ground, groundPosition, Color.White); 
 
            // Produce animation 
            animation.Draw(spriteBatch, characterPosition, SpriteEffects.None); 
        } 
 
        private void DrawBottomLeftQuadrant() 
        { 
            Vector2 textPosition = new Vector2(10, GraphicsDevice.Viewport.Height / 2 &#43; 10); 
           /// spriteBatch.DrawString(quadrantFont, &quot;Translation&quot;, textPosition, Color.White); 
 
            // Draw the UFO at current location 
            spriteBatch.Draw(ufo, ufoPosition, Color.White); 
            spriteBatch.Draw(ufo, ufoPosition, 
null, Color.White, MathHelper.ToRadians(asteroidRotation), 
new Vector2(ufo.Width, ufo.Height), asteroidScale, 
SpriteEffects.None, 0); 
 
         //   Copy 
 // Process touch events 
TouchCollection touchCollection = TouchPanel.GetState(); 
foreach (TouchLocation tl in touchCollection) 
{ 
    if ((tl.State == TouchLocationState.Pressed) 
            || (tl.State == TouchLocationState.Moved)) 
    { 
        spriteBatch.Draw(ufo, ufoPosition, 
  null, Color.White, MathHelper.ToRadians(asteroidRotation), 
  new Vector2(ufo.Width, ufo.Height), asteroidScale, 
  SpriteEffects.None, 0); 
        // Draw asteroid 
        spriteBatch.Draw(asteroid, asteroidPosition, 
          null, Color.White, MathHelper.ToRadians(asteroidRotation), 
          new Vector2(asteroid.Width / 2, asteroid.Height / 2), asteroidScale, 
          SpriteEffects.None, 0); 
 
        // add sparkles based on the touch location 
       // sparkles.Add(new Sparkle(tl.Position.X, 
        //         tl.Position.Y, ttms)); 
 
    } 
} 
 
 
    
         //   Vector2 textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 &#43; 10, 
//                            GraphicsDevice.Viewport.Height / 2 &#43; 10);Animated Sprite 
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White); 
 
            // Draw the ground 
          //  spriteBatch.Draw(ground, groundPosition, Color.White); 
 
            // Produce animation 
           // animation.Draw(spriteBatch, characterPosition, SpriteEffects.None); 
         
        } 
 
        private void DrawTopLeftQuadrant() 
        {//Sprite and Bitmap Fonts 
            Vector2 textPosition = new Vector2(10, 10); 
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White); 
              
            // Draw text using Sprite Font 
            string text = &quot;&quot;;//&quot;Sprite Font&quot;;Junt Hoong Chan 
            Vector2 size = calibri.MeasureString(text); 
            textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 - size.X / 2, 40); 
            spriteBatch.DrawString(calibri, text, textPosition, Color.Blue); 
 
            // Draw text using Bitmap Font with primitive shadow effect 
            text = &quot;Engineering and Computer Works&quot;;//&quot;Bitmap Font&quot;; 
            size = buxton.MeasureString(text); 
            textPosition = new Vector2((GraphicsDevice.Viewport.Width / 2 &#43; 2) - size.X / 2, 104); 
            spriteBatch.DrawString(buxton, text, textPosition, Color.Gray); 
            spriteBatch.DrawString(buxton, text, textPosition - new Vector2(4, 4), Color.CadetBlue); 
        } 
 
        private void DrawTopRightQuadrant() 
        {//Scale and Rotate 
            Vector2 textPosition = new Vector2(GraphicsDevice.Viewport.Width / 2 &#43; 10, 10); 
            spriteBatch.DrawString(quadrantFont, &quot;&quot;, textPosition, Color.White); 
 
            // Draw asteroid 
            spriteBatch.Draw(asteroid, asteroidPosition, 
              null, Color.White, MathHelper.ToRadians(asteroidRotation), 
             new Vector2(asteroid.Width, asteroid.Height), asteroidScale, 
              SpriteEffects.None, 0); 
        ///       
 
            // Draw asteroid 
            spriteBatch.Draw(asteroid, asteroidPosition, 
              null, Color.White, MathHelper.ToRadians(asteroidRotation), 
              new Vector2(asteroid.Width / 2, asteroid.Height / 2), asteroidScale, 
              SpriteEffects.None, 0); 
        } 
 
         
        #endregion 
    } 
} 
</pre>
<div class="preview">
<pre class="vb"><span class="visualBasic__preproc">#region&nbsp;File&nbsp;Information</span>&nbsp;
//-----------------------------------------------------------------------------&nbsp;
//&nbsp;Graphics2DSample.cs&nbsp;
//&nbsp;
//&nbsp;Microsoft&nbsp;XNA&nbsp;Community&nbsp;Game&nbsp;Platform&nbsp;
//&nbsp;Copyright&nbsp;(C)&nbsp;Microsoft&nbsp;Corporation.&nbsp;All&nbsp;rights&nbsp;reserved.&nbsp;
//-----------------------------------------------------------------------------<span class="visualBasic__preproc">&nbsp;
#endregion</span><span class="visualBasic__preproc">&nbsp;
&nbsp;
#region&nbsp;Using&nbsp;Statements</span>&nbsp;
using&nbsp;System;&nbsp;
using&nbsp;System.Collections.Generic;&nbsp;
using&nbsp;System.Linq;&nbsp;
using&nbsp;Microsoft.Xna.Framework;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Audio;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Content;&nbsp;
using&nbsp;Microsoft.Xna.Framework.GamerServices;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Graphics;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Input;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Input.Touch;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Media;&nbsp;
using&nbsp;System.Globalization;<span class="visualBasic__preproc">&nbsp;
&nbsp;
#endregion</span>&nbsp;
&nbsp;
namespace&nbsp;Graphics2DSample3&nbsp;
{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;This&nbsp;is&nbsp;the&nbsp;main&nbsp;type&nbsp;for&nbsp;your&nbsp;game&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;class&nbsp;Graphics2DSampleGame&nbsp;:&nbsp;Microsoft.Xna.Framework.Game&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Fields</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDeviceManager&nbsp;graphics;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteBatch&nbsp;spriteBatch;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Font&nbsp;Variables&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteFont&nbsp;calibri;&nbsp;//&nbsp;<span class="visualBasic__keyword">For</span>&nbsp;SpriteFont&nbsp;usage&nbsp;sample&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteFont&nbsp;buxton;&nbsp;//&nbsp;<span class="visualBasic__keyword">For</span>&nbsp;BitmapFont&nbsp;as&nbsp;SpriteFont&nbsp;usage&nbsp;sample&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteFont&nbsp;quadrantFont;&nbsp;//&nbsp;SpriteFont&nbsp;for&nbsp;in-quadrant&nbsp;descriptions&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Texture&nbsp;Variables&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;background;&nbsp;//&nbsp;Screen&nbsp;background&nbsp;texture&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;frame;&nbsp;//&nbsp;Screen&nbsp;frame&nbsp;to&nbsp;make&nbsp;final&nbsp;UI&nbsp;looks&nbsp;like&nbsp;four-quadrant&nbsp;layout&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;ufo;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;asteroid;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;ground;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Gameplay&nbsp;variables&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;ufoPosition;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;asteroidPosition;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;groundPosition;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;ufoVelocity;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;asteroidRotation;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;asteroidScale;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool&nbsp;isGrowing;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Random&nbsp;random;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Animation&nbsp;variables&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;characterPosition;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Animation&nbsp;animation;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span><span class="visualBasic__preproc">&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Initialization</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Graphics2DSampleGame()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;graphics&nbsp;=&nbsp;new&nbsp;GraphicsDeviceManager(this);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Content.RootDirectory&nbsp;=&nbsp;<span class="visualBasic__string">&quot;Content&quot;</span>;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Frame&nbsp;rate&nbsp;is&nbsp;<span class="visualBasic__number">30</span>&nbsp;fps&nbsp;by&nbsp;default&nbsp;for&nbsp;Windows&nbsp;Phone.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TargetElapsedTime&nbsp;=&nbsp;TimeSpan.FromTicks(<span class="visualBasic__number">333333</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;graphics.IsFullScreen&nbsp;=&nbsp;true;&nbsp;//&nbsp;Hide&nbsp;system&nbsp;tray&nbsp;for&nbsp;better&nbsp;gameplay&nbsp;experience&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Allows&nbsp;the&nbsp;game&nbsp;to&nbsp;perform&nbsp;any&nbsp;initialization&nbsp;it&nbsp;needs&nbsp;to&nbsp;before&nbsp;starting&nbsp;to&nbsp;run.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;This&nbsp;is&nbsp;where&nbsp;it&nbsp;can&nbsp;query&nbsp;for&nbsp;any&nbsp;required&nbsp;services&nbsp;and&nbsp;load&nbsp;any&nbsp;non-graphic&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;related&nbsp;content.&nbsp;&nbsp;Calling&nbsp;base.Initialize&nbsp;will&nbsp;enumerate&nbsp;through&nbsp;any&nbsp;components&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;and&nbsp;initialize&nbsp;them&nbsp;as&nbsp;well.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;Initialize()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Initialize&nbsp;initial&nbsp;gameplay&nbsp;state&nbsp;and&nbsp;variables&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;random&nbsp;=&nbsp;new&nbsp;Random();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">600</span>,&nbsp;<span class="visualBasic__number">100</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">180</span>,&nbsp;<span class="visualBasic__number">280</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidRotation&nbsp;=&nbsp;<span class="visualBasic__number">0</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidScale&nbsp;=&nbsp;<span class="visualBasic__number">1</span>.0f;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;isGrowing&nbsp;=&nbsp;true;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(ufoVelocity.X&nbsp;==&nbsp;<span class="visualBasic__number">0</span>&nbsp;||&nbsp;ufoVelocity.Y&nbsp;==&nbsp;<span class="visualBasic__number">0</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity&nbsp;=&nbsp;new&nbsp;Vector2(random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>),&nbsp;random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>));&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;base.Initialize();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span><span class="visualBasic__preproc">&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Load&nbsp;and&nbsp;Unload</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;LoadContent&nbsp;will&nbsp;be&nbsp;called&nbsp;once&nbsp;per&nbsp;game&nbsp;and&nbsp;is&nbsp;the&nbsp;place&nbsp;to&nbsp;load&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;all&nbsp;of&nbsp;your&nbsp;content.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;LoadContent()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Create&nbsp;a&nbsp;new&nbsp;SpriteBatch,&nbsp;which&nbsp;can&nbsp;be&nbsp;used&nbsp;to&nbsp;draw&nbsp;textures.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch&nbsp;=&nbsp;new&nbsp;SpriteBatch(GraphicsDevice);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Load&nbsp;game&nbsp;content&nbsp;here&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;calibri&nbsp;=&nbsp;Content.Load&lt;SpriteFont&gt;(<span class="visualBasic__string">&quot;PericlesSpriteFont&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buxton&nbsp;=&nbsp;Content.Load&lt;SpriteFont&gt;(<span class="visualBasic__string">&quot;BuxtonSketchBitmapFont&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;quadrantFont&nbsp;=&nbsp;Content.Load&lt;SpriteFont&gt;(<span class="visualBasic__string">&quot;QuadrantFont&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;background&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;spaceBG&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufo&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;UFO&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroid&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;asteroid&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ground&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;ground&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;frame&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;frame&quot;</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;borderWidth&nbsp;=&nbsp;<span class="visualBasic__number">1</span>;&nbsp;//&nbsp;A&nbsp;width&nbsp;of&nbsp;the&nbsp;screen&nbsp;border&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;groundPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;&#43;&nbsp;ground.Width&nbsp;&#43;&nbsp;borderWidth,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Viewport.Height&nbsp;&#43;&nbsp;ground.Height&nbsp;&#43;&nbsp;borderWidth);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Load&nbsp;multiple&nbsp;animations&nbsp;form&nbsp;XML&nbsp;definition&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.Xml.Linq.XDocument&nbsp;doc&nbsp;=&nbsp;System.Xml.Linq.XDocument.Load(<span class="visualBasic__string">&quot;Content/AnimationDef.xml&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;System.Xml.Linq.XDocument&nbsp;a&nbsp;=&nbsp;System.Xml.Linq.XDocument.Load(<span class="visualBasic__string">&quot;Content/AnimationDef1.xml&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;System.Xml.Linq.XDocument&nbsp;doc&nbsp;=&nbsp;System.Xml.Linq.XDocument.Load(<span class="visualBasic__string">&quot;Content/AnimationDef1.xml&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;<span class="visualBasic__keyword">Get</span>&nbsp;the&nbsp;first&nbsp;(and&nbsp;only&nbsp;in&nbsp;this&nbsp;case)&nbsp;animation&nbsp;from&nbsp;the&nbsp;XML&nbsp;definition&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;var&nbsp;definition&nbsp;=&nbsp;doc.Root.Element(<span class="visualBasic__string">&quot;Definition&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;var&nbsp;def&nbsp;=&nbsp;a.Root.Element(<span class="visualBasic__string">&quot;Definition&quot;</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;texture&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(definition.Attribute(<span class="visualBasic__string">&quot;SheetName&quot;</span>).Value);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;Texture2D&nbsp;texture1&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(def.Attribute(<span class="visualBasic__string">&quot;SheetName&quot;</span>).Value);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Point&nbsp;frameSize&nbsp;=&nbsp;new&nbsp;Point();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;frameSize.X&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;FrameWidth&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;frameSize.Y&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;FrameHeight&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;frameSize.X&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;FrameWidth&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;frameSize.Y&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;FrameHeight&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Point&nbsp;sheetSize&nbsp;=&nbsp;new&nbsp;Point();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.X&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;SheetColumns&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.Y&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;SheetRows&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.X&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;SheetColumns&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.Y&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;SheetRows&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TimeSpan&nbsp;frameInterval&nbsp;=&nbsp;TimeSpan.FromSeconds(<span class="visualBasic__number">1</span>.0f&nbsp;/&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;Speed&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>));&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;TimeSpan&nbsp;frameInterval&nbsp;=&nbsp;TimeSpan.FromSeconds(<span class="visualBasic__number">1</span>.0f&nbsp;/&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;Speed&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>));&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Define&nbsp;animation&nbsp;position&nbsp;at&nbsp;the&nbsp;bottom&nbsp;of&nbsp;the&nbsp;screen&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;characterPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;GraphicsDevice.Viewport.Height&nbsp;-&nbsp;frameSize.Y);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Define&nbsp;a&nbsp;new&nbsp;Animation&nbsp;instance&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;animation&nbsp;=&nbsp;new&nbsp;Animation(texture,&nbsp;frameSize,&nbsp;sheetSize,&nbsp;frameInterval);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;UnloadContent&nbsp;will&nbsp;be&nbsp;called&nbsp;once&nbsp;per&nbsp;game&nbsp;and&nbsp;is&nbsp;the&nbsp;place&nbsp;to&nbsp;unload&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;all&nbsp;content.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;UnloadContent()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;TODO:&nbsp;Unload&nbsp;any&nbsp;non&nbsp;ContentManager&nbsp;content&nbsp;here&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span><span class="visualBasic__preproc">&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Update</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Allows&nbsp;the&nbsp;game&nbsp;to&nbsp;run&nbsp;logic&nbsp;such&nbsp;as&nbsp;updating&nbsp;the&nbsp;world,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;checking&nbsp;for&nbsp;collisions,&nbsp;gathering&nbsp;input,&nbsp;and&nbsp;playing&nbsp;audio.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;param&nbsp;name=<span class="visualBasic__string">&quot;gameTime&quot;</span>&gt;Provides&nbsp;a&nbsp;snapshot&nbsp;of&nbsp;timing&nbsp;values.&lt;/param&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;Update(GameTime&nbsp;gameTime)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Allows&nbsp;the&nbsp;game&nbsp;to&nbsp;exit&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(GamePad.GetState(PlayerIndex.One).Buttons.Back&nbsp;==&nbsp;ButtonState.Pressed)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this.<span class="visualBasic__keyword">Exit</span>();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;TODO:&nbsp;Add&nbsp;your&nbsp;update&nbsp;logic&nbsp;here&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;elapsed&nbsp;=&nbsp;(float)gameTime.ElapsedGameTime.TotalSeconds;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Calculate&nbsp;UFO&nbsp;movement&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UpdateUFO(elapsed);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Scale&nbsp;and&nbsp;rotate&nbsp;the&nbsp;asteroid&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UpdateAsteroid();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Progress&nbsp;the&nbsp;animation&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool&nbsp;isProgressed&nbsp;=&nbsp;animation.Update(gameTime);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;<span class="visualBasic__keyword">If</span>&nbsp;animation&nbsp;progressed&nbsp;to&nbsp;the&nbsp;next&nbsp;frame&nbsp;change&nbsp;the&nbsp;position&nbsp;accordingly&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(isProgressed)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;characterPosition.X&nbsp;&#43;=&nbsp;animation.frameSize.X&nbsp;/&nbsp;<span class="visualBasic__number">2</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(characterPosition.X&nbsp;&gt;&nbsp;GraphicsDevice.Viewport.Width)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;characterPosition.X&nbsp;=&nbsp;GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;base.Update(gameTime);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Update&nbsp;the&nbsp;asteroid&nbsp;position&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;UpdateAsteroid()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidRotation&nbsp;&#43;=&nbsp;<span class="visualBasic__number">10</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(asteroidRotation&nbsp;&gt;=&nbsp;<span class="visualBasic__number">360</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidRotation&nbsp;=&nbsp;<span class="visualBasic__number">0</span>;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidScale&nbsp;=&nbsp;asteroidScale&nbsp;&#43;&nbsp;(isGrowing&nbsp;?&nbsp;<span class="visualBasic__number">0</span>.01f&nbsp;:&nbsp;-<span class="visualBasic__number">0</span>.01f);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(asteroidScale&nbsp;&gt;&nbsp;<span class="visualBasic__number">2</span>.0f)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;isGrowing&nbsp;=&nbsp;false;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else&nbsp;if&nbsp;(asteroidScale&nbsp;&lt;&nbsp;<span class="visualBasic__number">0</span>.5f)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;isGrowing&nbsp;=&nbsp;true;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Update&nbsp;the&nbsp;UFO&nbsp;position&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;param&nbsp;name=<span class="visualBasic__string">&quot;elapsed&quot;</span>&gt;&lt;/param&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;UpdateUFO(float&nbsp;elapsed)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition&nbsp;&#43;=&nbsp;ufoVelocity&nbsp;*&nbsp;<span class="visualBasic__number">64</span>.0f&nbsp;*&nbsp;elapsed;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition.X&nbsp;=&nbsp;MathHelper.Clamp(ufoPosition.X,&nbsp;<span class="visualBasic__number">0</span>,&nbsp;GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;-&nbsp;ufo.Width);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoPosition.X&nbsp;==&nbsp;<span class="visualBasic__number">0</span>&nbsp;||&nbsp;ufoPosition.X&nbsp;==&nbsp;GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;-&nbsp;ufo.Width)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.X&nbsp;=&nbsp;random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoVelocity.X&nbsp;==&nbsp;<span class="visualBasic__number">0</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.X&nbsp;=&nbsp;<span class="visualBasic__number">1</span>;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition.Y&nbsp;=&nbsp;MathHelper.Clamp(ufoPosition.Y,&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;GraphicsDevice.Viewport.Height&nbsp;-&nbsp;ufo.Height);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoPosition.Y&nbsp;==&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;||&nbsp;ufoPosition.Y&nbsp;==&nbsp;GraphicsDevice.Viewport.Height&nbsp;-&nbsp;ufo.Height)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.Y&nbsp;=&nbsp;random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoVelocity.Y&nbsp;==&nbsp;<span class="visualBasic__number">0</span>)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.Y&nbsp;=&nbsp;<span class="visualBasic__number">1</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span><span class="visualBasic__preproc">&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Render</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;This&nbsp;is&nbsp;called&nbsp;when&nbsp;the&nbsp;game&nbsp;should&nbsp;draw&nbsp;itself.&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;param&nbsp;name=<span class="visualBasic__string">&quot;gameTime&quot;</span>&gt;Provides&nbsp;a&nbsp;snapshot&nbsp;of&nbsp;timing&nbsp;values.&lt;/param&gt;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;Draw(GameTime&nbsp;gameTime)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Clear(Color.CornflowerBlue);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;background&nbsp;texture&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Begin();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;a&nbsp;background&nbsp;picture&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(background,&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">0</span>,&nbsp;<span class="visualBasic__number">0</span>),&nbsp;Color.White);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//Draw&nbsp;asteroid&nbsp;picture&nbsp;-&nbsp;rotated&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawTopRightQuadrant();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;text&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawTopLeftQuadrant();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;UFO&nbsp;picture&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawBottomLeftQuadrant();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;frame&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(frame,&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">0</span>,&nbsp;<span class="visualBasic__number">0</span>),&nbsp;Color.Gray);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.<span class="visualBasic__keyword">End</span>();&nbsp;//Don<span class="visualBasic__com">'t&nbsp;forget&nbsp;to&nbsp;close&nbsp;it&nbsp;at&nbsp;the&nbsp;end.</span>&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;base.Draw(gameTime);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawBottomRightQuadrant()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{//Animated&nbsp;Sprite&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;ground&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ground,&nbsp;groundPosition,&nbsp;Color.White);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Produce&nbsp;animation&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;animation.Draw(spriteBatch,&nbsp;characterPosition,&nbsp;SpriteEffects.None);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawBottomLeftQuadrant()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">10</span>,&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;Translation&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;UFO&nbsp;at&nbsp;current&nbsp;location&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ufo,&nbsp;ufoPosition,&nbsp;Color.White);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ufo,&nbsp;ufoPosition,&nbsp;
null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;
new&nbsp;Vector2(ufo.Width,&nbsp;ufo.Height),&nbsp;asteroidScale,&nbsp;
SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;Copy&nbsp;
&nbsp;//&nbsp;Process&nbsp;touch&nbsp;events&nbsp;
TouchCollection&nbsp;touchCollection&nbsp;=&nbsp;TouchPanel.GetState();&nbsp;
foreach&nbsp;(TouchLocation&nbsp;tl&nbsp;in&nbsp;touchCollection)&nbsp;
{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;((tl.State&nbsp;==&nbsp;TouchLocationState.Pressed)&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;||&nbsp;(tl.State&nbsp;==&nbsp;TouchLocationState.Moved))&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ufo,&nbsp;ufoPosition,&nbsp;
&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;
&nbsp;&nbsp;new&nbsp;Vector2(ufo.Width,&nbsp;ufo.Height),&nbsp;asteroidScale,&nbsp;
&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;asteroid&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(asteroid,&nbsp;asteroidPosition,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new&nbsp;Vector2(asteroid.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;asteroid.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>),&nbsp;asteroidScale,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;add&nbsp;sparkles&nbsp;based&nbsp;on&nbsp;the&nbsp;touch&nbsp;location&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;sparkles.Add(new&nbsp;Sparkle(tl.Position.X,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tl.Position.Y,&nbsp;ttms));&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
}&nbsp;
&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>,&nbsp;
//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>);Animated&nbsp;Sprite&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;ground&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;spriteBatch.Draw(ground,&nbsp;groundPosition,&nbsp;Color.White);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Produce&nbsp;animation&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;animation.Draw(spriteBatch,&nbsp;characterPosition,&nbsp;SpriteEffects.None);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawTopLeftQuadrant()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{//Sprite&nbsp;and&nbsp;Bitmap&nbsp;Fonts&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">10</span>,&nbsp;<span class="visualBasic__number">10</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;text&nbsp;using&nbsp;Sprite&nbsp;Font&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;string&nbsp;text&nbsp;=&nbsp;<span class="visualBasic__string">&quot;&quot;</span>;//<span class="visualBasic__string">&quot;Sprite&nbsp;Font&quot;</span>;Junt&nbsp;Hoong&nbsp;Chan&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;size&nbsp;=&nbsp;calibri.MeasureString(text);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;-&nbsp;size.X&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;<span class="visualBasic__number">40</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(calibri,&nbsp;text,&nbsp;textPosition,&nbsp;Color.Blue);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;text&nbsp;using&nbsp;Bitmap&nbsp;Font&nbsp;with&nbsp;primitive&nbsp;shadow&nbsp;effect&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;text&nbsp;=&nbsp;<span class="visualBasic__string">&quot;Engineering&nbsp;and&nbsp;Computer&nbsp;Works&quot;</span>;//<span class="visualBasic__string">&quot;Bitmap&nbsp;Font&quot;</span>;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;size&nbsp;=&nbsp;buxton.MeasureString(text);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2((GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">2</span>)&nbsp;-&nbsp;size.X&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;<span class="visualBasic__number">104</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(buxton,&nbsp;text,&nbsp;textPosition,&nbsp;Color.Gray);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(buxton,&nbsp;text,&nbsp;textPosition&nbsp;-&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">4</span>,&nbsp;<span class="visualBasic__number">4</span>),&nbsp;Color.CadetBlue);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawTopRightQuadrant()&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{//Scale&nbsp;and&nbsp;Rotate&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>,&nbsp;<span class="visualBasic__number">10</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;asteroid&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(asteroid,&nbsp;asteroidPosition,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new&nbsp;Vector2(asteroid.Width,&nbsp;asteroid.Height),&nbsp;asteroidScale,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;asteroid&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(asteroid,&nbsp;asteroidPosition,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new&nbsp;Vector2(asteroid.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;asteroid.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>),&nbsp;asteroidScale,&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}<span class="visualBasic__preproc">&nbsp;
&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span>&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;
}<span class="visualBasic__preproc">&nbsp;
&nbsp;
#region&nbsp;File&nbsp;Information</span>&nbsp;&nbsp;
//-----------------------------------------------------------------------------&nbsp;&nbsp;
//&nbsp;Graphics2DSample.cs&nbsp;&nbsp;
//&nbsp;&nbsp;
//&nbsp;Microsoft&nbsp;XNA&nbsp;Community&nbsp;Game&nbsp;Platform&nbsp;&nbsp;
//&nbsp;Copyright&nbsp;(C)&nbsp;Microsoft&nbsp;Corporation.&nbsp;All&nbsp;rights&nbsp;reserved.&nbsp;&nbsp;
//-----------------------------------------------------------------------------&nbsp;<span class="visualBasic__preproc">&nbsp;
#endregion</span>&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;
#region&nbsp;Using&nbsp;Statements</span>&nbsp;&nbsp;
using&nbsp;System;&nbsp;&nbsp;
using&nbsp;System.Collections.Generic;&nbsp;&nbsp;
using&nbsp;System.Linq;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Audio;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Content;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework.GamerServices;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Graphics;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Input;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Input.Touch;&nbsp;&nbsp;
using&nbsp;Microsoft.Xna.Framework.Media;&nbsp;&nbsp;
using&nbsp;System.Globalization;&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;
#endregion</span>&nbsp;&nbsp;
&nbsp;&nbsp;
namespace&nbsp;Graphics2DSample3&nbsp;&nbsp;
{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;This&nbsp;is&nbsp;the&nbsp;main&nbsp;type&nbsp;for&nbsp;your&nbsp;game&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;class&nbsp;Graphics2DSampleGame&nbsp;:&nbsp;Microsoft.Xna.Framework.Game&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Fields</span>&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDeviceManager&nbsp;graphics;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteBatch&nbsp;spriteBatch;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Font&nbsp;Variables&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteFont&nbsp;calibri;&nbsp;//&nbsp;<span class="visualBasic__keyword">For</span>&nbsp;SpriteFont&nbsp;usage&nbsp;sample&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteFont&nbsp;buxton;&nbsp;//&nbsp;<span class="visualBasic__keyword">For</span>&nbsp;BitmapFont&nbsp;as&nbsp;SpriteFont&nbsp;usage&nbsp;sample&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteFont&nbsp;quadrantFont;&nbsp;//&nbsp;SpriteFont&nbsp;for&nbsp;in-quadrant&nbsp;descriptions&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Texture&nbsp;Variables&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;background;&nbsp;//&nbsp;Screen&nbsp;background&nbsp;texture&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;frame;&nbsp;//&nbsp;Screen&nbsp;frame&nbsp;to&nbsp;make&nbsp;final&nbsp;UI&nbsp;looks&nbsp;like&nbsp;four-quadrant&nbsp;layout&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;ufo;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;asteroid;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;ground;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Gameplay&nbsp;variables&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;ufoPosition;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;asteroidPosition;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;groundPosition;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;ufoVelocity;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;asteroidRotation;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;asteroidScale;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool&nbsp;isGrowing;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Random&nbsp;random;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Animation&nbsp;variables&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;characterPosition;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Animation&nbsp;animation;&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span>&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Initialization</span>&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public&nbsp;Graphics2DSampleGame()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;graphics&nbsp;=&nbsp;new&nbsp;GraphicsDeviceManager(this);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Content.RootDirectory&nbsp;=&nbsp;<span class="visualBasic__string">&quot;Content&quot;</span>;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Frame&nbsp;rate&nbsp;is&nbsp;<span class="visualBasic__number">30</span>&nbsp;fps&nbsp;by&nbsp;default&nbsp;for&nbsp;Windows&nbsp;Phone.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TargetElapsedTime&nbsp;=&nbsp;TimeSpan.FromTicks(<span class="visualBasic__number">333333</span>);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;graphics.IsFullScreen&nbsp;=&nbsp;true;&nbsp;//&nbsp;Hide&nbsp;system&nbsp;tray&nbsp;for&nbsp;better&nbsp;gameplay&nbsp;experience&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Allows&nbsp;the&nbsp;game&nbsp;to&nbsp;perform&nbsp;any&nbsp;initialization&nbsp;it&nbsp;needs&nbsp;to&nbsp;before&nbsp;starting&nbsp;to&nbsp;run.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;This&nbsp;is&nbsp;where&nbsp;it&nbsp;can&nbsp;query&nbsp;for&nbsp;any&nbsp;required&nbsp;services&nbsp;and&nbsp;load&nbsp;any&nbsp;non-graphic&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;related&nbsp;content.&nbsp;&nbsp;Calling&nbsp;base.Initialize&nbsp;will&nbsp;enumerate&nbsp;through&nbsp;any&nbsp;components&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;and&nbsp;initialize&nbsp;them&nbsp;as&nbsp;well.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;Initialize()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Initialize&nbsp;initial&nbsp;gameplay&nbsp;state&nbsp;and&nbsp;variables&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;random&nbsp;=&nbsp;new&nbsp;Random();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">600</span>,&nbsp;<span class="visualBasic__number">100</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">180</span>,&nbsp;<span class="visualBasic__number">280</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidRotation&nbsp;=&nbsp;<span class="visualBasic__number">0</span>;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidScale&nbsp;=&nbsp;<span class="visualBasic__number">1</span>.0f;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;isGrowing&nbsp;=&nbsp;true;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;while&nbsp;(ufoVelocity.X&nbsp;==&nbsp;<span class="visualBasic__number">0</span>&nbsp;||&nbsp;ufoVelocity.Y&nbsp;==&nbsp;<span class="visualBasic__number">0</span>)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity&nbsp;=&nbsp;new&nbsp;Vector2(random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>),&nbsp;random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>));&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;base.Initialize();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span>&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Load&nbsp;and&nbsp;Unload</span>&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;LoadContent&nbsp;will&nbsp;be&nbsp;called&nbsp;once&nbsp;per&nbsp;game&nbsp;and&nbsp;is&nbsp;the&nbsp;place&nbsp;to&nbsp;load&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;all&nbsp;of&nbsp;your&nbsp;content.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;LoadContent()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Create&nbsp;a&nbsp;new&nbsp;SpriteBatch,&nbsp;which&nbsp;can&nbsp;be&nbsp;used&nbsp;to&nbsp;draw&nbsp;textures.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch&nbsp;=&nbsp;new&nbsp;SpriteBatch(GraphicsDevice);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Load&nbsp;game&nbsp;content&nbsp;here&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;calibri&nbsp;=&nbsp;Content.Load&lt;SpriteFont&gt;(<span class="visualBasic__string">&quot;PericlesSpriteFont&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;buxton&nbsp;=&nbsp;Content.Load&lt;SpriteFont&gt;(<span class="visualBasic__string">&quot;BuxtonSketchBitmapFont&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;quadrantFont&nbsp;=&nbsp;Content.Load&lt;SpriteFont&gt;(<span class="visualBasic__string">&quot;QuadrantFont&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;background&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;spaceBG&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufo&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;UFO&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroid&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;asteroid&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ground&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;ground&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;frame&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(<span class="visualBasic__string">&quot;frame&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;int&nbsp;borderWidth&nbsp;=&nbsp;<span class="visualBasic__number">1</span>;&nbsp;//&nbsp;A&nbsp;width&nbsp;of&nbsp;the&nbsp;screen&nbsp;border&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;groundPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;&#43;&nbsp;ground.Width&nbsp;&#43;&nbsp;borderWidth,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Viewport.Height&nbsp;&#43;&nbsp;ground.Height&nbsp;&#43;&nbsp;borderWidth);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Load&nbsp;multiple&nbsp;animations&nbsp;form&nbsp;XML&nbsp;definition&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;System.Xml.Linq.XDocument&nbsp;doc&nbsp;=&nbsp;System.Xml.Linq.XDocument.Load(<span class="visualBasic__string">&quot;Content/AnimationDef.xml&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;System.Xml.Linq.XDocument&nbsp;a&nbsp;=&nbsp;System.Xml.Linq.XDocument.Load(<span class="visualBasic__string">&quot;Content/AnimationDef1.xml&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;System.Xml.Linq.XDocument&nbsp;doc&nbsp;=&nbsp;System.Xml.Linq.XDocument.Load(<span class="visualBasic__string">&quot;Content/AnimationDef1.xml&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;<span class="visualBasic__keyword">Get</span>&nbsp;the&nbsp;first&nbsp;(and&nbsp;only&nbsp;in&nbsp;this&nbsp;case)&nbsp;animation&nbsp;from&nbsp;the&nbsp;XML&nbsp;definition&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;var&nbsp;definition&nbsp;=&nbsp;doc.Root.Element(<span class="visualBasic__string">&quot;Definition&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;var&nbsp;def&nbsp;=&nbsp;a.Root.Element(<span class="visualBasic__string">&quot;Definition&quot;</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Texture2D&nbsp;texture&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(definition.Attribute(<span class="visualBasic__string">&quot;SheetName&quot;</span>).Value);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;Texture2D&nbsp;texture1&nbsp;=&nbsp;Content.Load&lt;Texture2D&gt;(def.Attribute(<span class="visualBasic__string">&quot;SheetName&quot;</span>).Value);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Point&nbsp;frameSize&nbsp;=&nbsp;new&nbsp;Point();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;frameSize.X&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;FrameWidth&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;frameSize.Y&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;FrameHeight&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;frameSize.X&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;FrameWidth&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;frameSize.Y&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;FrameHeight&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Point&nbsp;sheetSize&nbsp;=&nbsp;new&nbsp;Point();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.X&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;SheetColumns&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.Y&nbsp;=&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;SheetRows&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.X&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;SheetColumns&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;sheetSize.Y&nbsp;=&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;SheetRows&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;TimeSpan&nbsp;frameInterval&nbsp;=&nbsp;TimeSpan.FromSeconds(<span class="visualBasic__number">1</span>.0f&nbsp;/&nbsp;int.Parse(definition.Attribute(<span class="visualBasic__string">&quot;Speed&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>));&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;TimeSpan&nbsp;frameInterval&nbsp;=&nbsp;TimeSpan.FromSeconds(<span class="visualBasic__number">1</span>.0f&nbsp;/&nbsp;int.Parse(def.Attribute(<span class="visualBasic__string">&quot;Speed&quot;</span>).Value,&nbsp;NumberStyles.<span class="visualBasic__keyword">Integer</span>));&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Define&nbsp;animation&nbsp;position&nbsp;at&nbsp;the&nbsp;bottom&nbsp;of&nbsp;the&nbsp;screen&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;characterPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;GraphicsDevice.Viewport.Height&nbsp;-&nbsp;frameSize.Y);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Define&nbsp;a&nbsp;new&nbsp;Animation&nbsp;instance&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;animation&nbsp;=&nbsp;new&nbsp;Animation(texture,&nbsp;frameSize,&nbsp;sheetSize,&nbsp;frameInterval);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;UnloadContent&nbsp;will&nbsp;be&nbsp;called&nbsp;once&nbsp;per&nbsp;game&nbsp;and&nbsp;is&nbsp;the&nbsp;place&nbsp;to&nbsp;unload&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;all&nbsp;content.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;UnloadContent()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;TODO:&nbsp;Unload&nbsp;any&nbsp;non&nbsp;ContentManager&nbsp;content&nbsp;here&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span>&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Update</span>&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Allows&nbsp;the&nbsp;game&nbsp;to&nbsp;run&nbsp;logic&nbsp;such&nbsp;as&nbsp;updating&nbsp;the&nbsp;world,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;checking&nbsp;for&nbsp;collisions,&nbsp;gathering&nbsp;input,&nbsp;and&nbsp;playing&nbsp;audio.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;param&nbsp;name=<span class="visualBasic__string">&quot;gameTime&quot;</span>&gt;Provides&nbsp;a&nbsp;snapshot&nbsp;of&nbsp;timing&nbsp;values.&lt;/param&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;Update(GameTime&nbsp;gameTime)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Allows&nbsp;the&nbsp;game&nbsp;to&nbsp;exit&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(GamePad.GetState(PlayerIndex.One).Buttons.Back&nbsp;==&nbsp;ButtonState.Pressed)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;this.<span class="visualBasic__keyword">Exit</span>();&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;TODO:&nbsp;Add&nbsp;your&nbsp;update&nbsp;logic&nbsp;here&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;float&nbsp;elapsed&nbsp;=&nbsp;(float)gameTime.ElapsedGameTime.TotalSeconds;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Calculate&nbsp;UFO&nbsp;movement&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UpdateUFO(elapsed);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Scale&nbsp;and&nbsp;rotate&nbsp;the&nbsp;asteroid&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;UpdateAsteroid();&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Progress&nbsp;the&nbsp;animation&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bool&nbsp;isProgressed&nbsp;=&nbsp;animation.Update(gameTime);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;<span class="visualBasic__keyword">If</span>&nbsp;animation&nbsp;progressed&nbsp;to&nbsp;the&nbsp;next&nbsp;frame&nbsp;change&nbsp;the&nbsp;position&nbsp;accordingly&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(isProgressed)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;characterPosition.X&nbsp;&#43;=&nbsp;animation.frameSize.X&nbsp;/&nbsp;<span class="visualBasic__number">2</span>;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(characterPosition.X&nbsp;&gt;&nbsp;GraphicsDevice.Viewport.Width)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;characterPosition.X&nbsp;=&nbsp;GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;base.Update(gameTime);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Update&nbsp;the&nbsp;asteroid&nbsp;position&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;UpdateAsteroid()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidRotation&nbsp;&#43;=&nbsp;<span class="visualBasic__number">10</span>;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(asteroidRotation&nbsp;&gt;=&nbsp;<span class="visualBasic__number">360</span>)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidRotation&nbsp;=&nbsp;<span class="visualBasic__number">0</span>;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;asteroidScale&nbsp;=&nbsp;asteroidScale&nbsp;&#43;&nbsp;(isGrowing&nbsp;?&nbsp;<span class="visualBasic__number">0</span>.01f&nbsp;:&nbsp;-<span class="visualBasic__number">0</span>.01f);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(asteroidScale&nbsp;&gt;&nbsp;<span class="visualBasic__number">2</span>.0f)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;isGrowing&nbsp;=&nbsp;false;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;else&nbsp;if&nbsp;(asteroidScale&nbsp;&lt;&nbsp;<span class="visualBasic__number">0</span>.5f)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;isGrowing&nbsp;=&nbsp;true;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;Update&nbsp;the&nbsp;UFO&nbsp;position&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;param&nbsp;name=<span class="visualBasic__string">&quot;elapsed&quot;</span>&gt;&lt;/param&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;UpdateUFO(float&nbsp;elapsed)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition&nbsp;&#43;=&nbsp;ufoVelocity&nbsp;*&nbsp;<span class="visualBasic__number">64</span>.0f&nbsp;*&nbsp;elapsed;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition.X&nbsp;=&nbsp;MathHelper.Clamp(ufoPosition.X,&nbsp;<span class="visualBasic__number">0</span>,&nbsp;GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;-&nbsp;ufo.Width);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoPosition.X&nbsp;==&nbsp;<span class="visualBasic__number">0</span>&nbsp;||&nbsp;ufoPosition.X&nbsp;==&nbsp;GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;-&nbsp;ufo.Width)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.X&nbsp;=&nbsp;random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoVelocity.X&nbsp;==&nbsp;<span class="visualBasic__number">0</span>)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.X&nbsp;=&nbsp;<span class="visualBasic__number">1</span>;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoPosition.Y&nbsp;=&nbsp;MathHelper.Clamp(ufoPosition.Y,&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;GraphicsDevice.Viewport.Height&nbsp;-&nbsp;ufo.Height);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoPosition.Y&nbsp;==&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;||&nbsp;ufoPosition.Y&nbsp;==&nbsp;GraphicsDevice.Viewport.Height&nbsp;-&nbsp;ufo.Height)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.Y&nbsp;=&nbsp;random.<span class="visualBasic__keyword">Next</span>(-<span class="visualBasic__number">1</span>,&nbsp;<span class="visualBasic__number">2</span>);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;(ufoVelocity.Y&nbsp;==&nbsp;<span class="visualBasic__number">0</span>)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ufoVelocity.Y&nbsp;=&nbsp;<span class="visualBasic__number">1</span>;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span>&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#region&nbsp;Render</span>&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;This&nbsp;is&nbsp;called&nbsp;when&nbsp;the&nbsp;game&nbsp;should&nbsp;draw&nbsp;itself.&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;/summary&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&lt;param&nbsp;name=<span class="visualBasic__string">&quot;gameTime&quot;</span>&gt;Provides&nbsp;a&nbsp;snapshot&nbsp;of&nbsp;timing&nbsp;values.&lt;/param&gt;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;protected&nbsp;override&nbsp;void&nbsp;Draw(GameTime&nbsp;gameTime)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Clear(Color.CornflowerBlue);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;background&nbsp;texture&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Begin();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;a&nbsp;background&nbsp;picture&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(background,&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">0</span>,&nbsp;<span class="visualBasic__number">0</span>),&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//Draw&nbsp;asteroid&nbsp;picture&nbsp;-&nbsp;rotated&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawTopRightQuadrant();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;text&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawTopLeftQuadrant();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;UFO&nbsp;picture&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawBottomLeftQuadrant();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DrawBottomRightQuadrant();&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;frame&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(frame,&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">0</span>,&nbsp;<span class="visualBasic__number">0</span>),&nbsp;Color.Gray);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.<span class="visualBasic__keyword">End</span>();&nbsp;//Don<span class="visualBasic__com">'t&nbsp;forget&nbsp;to&nbsp;close&nbsp;it&nbsp;at&nbsp;the&nbsp;end.&nbsp;</span>&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;base.Draw(gameTime);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawBottomRightQuadrant()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{//Animated&nbsp;Sprite&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;ground&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ground,&nbsp;groundPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Produce&nbsp;animation&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;animation.Draw(spriteBatch,&nbsp;characterPosition,&nbsp;SpriteEffects.None);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawBottomLeftQuadrant()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">10</span>,&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;Translation&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;UFO&nbsp;at&nbsp;current&nbsp;location&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ufo,&nbsp;ufoPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ufo,&nbsp;ufoPosition,&nbsp;&nbsp;
null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;&nbsp;
new&nbsp;Vector2(ufo.Width,&nbsp;ufo.Height),&nbsp;asteroidScale,&nbsp;&nbsp;
SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;Copy&nbsp;&nbsp;
&nbsp;//&nbsp;Process&nbsp;touch&nbsp;events&nbsp;&nbsp;
TouchCollection&nbsp;touchCollection&nbsp;=&nbsp;TouchPanel.GetState();&nbsp;&nbsp;
foreach&nbsp;(TouchLocation&nbsp;tl&nbsp;in&nbsp;touchCollection)&nbsp;&nbsp;
{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;if&nbsp;((tl.State&nbsp;==&nbsp;TouchLocationState.Pressed)&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;||&nbsp;(tl.State&nbsp;==&nbsp;TouchLocationState.Moved))&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;{&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(ufo,&nbsp;ufoPosition,&nbsp;&nbsp;
&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;&nbsp;
&nbsp;&nbsp;new&nbsp;Vector2(ufo.Width,&nbsp;ufo.Height),&nbsp;asteroidScale,&nbsp;&nbsp;
&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;asteroid&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(asteroid,&nbsp;asteroidPosition,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new&nbsp;Vector2(asteroid.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;asteroid.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>),&nbsp;asteroidScale,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;add&nbsp;sparkles&nbsp;based&nbsp;on&nbsp;the&nbsp;touch&nbsp;location&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;sparkles.Add(new&nbsp;Sparkle(tl.Position.X,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;tl.Position.Y,&nbsp;ttms));&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>,&nbsp;&nbsp;
//&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;GraphicsDevice.Viewport.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>);Animated&nbsp;Sprite&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;the&nbsp;ground&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;&nbsp;spriteBatch.Draw(ground,&nbsp;groundPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Produce&nbsp;animation&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;animation.Draw(spriteBatch,&nbsp;characterPosition,&nbsp;SpriteEffects.None);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawTopLeftQuadrant()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{//Sprite&nbsp;and&nbsp;Bitmap&nbsp;Fonts&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">10</span>,&nbsp;<span class="visualBasic__number">10</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;text&nbsp;using&nbsp;Sprite&nbsp;Font&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;string&nbsp;text&nbsp;=&nbsp;<span class="visualBasic__string">&quot;&quot;</span>;//<span class="visualBasic__string">&quot;Sprite&nbsp;Font&quot;</span>;Junt&nbsp;Hoong&nbsp;Chan&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;size&nbsp;=&nbsp;calibri.MeasureString(text);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;-&nbsp;size.X&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;<span class="visualBasic__number">40</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(calibri,&nbsp;text,&nbsp;textPosition,&nbsp;Color.Blue);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;text&nbsp;using&nbsp;Bitmap&nbsp;Font&nbsp;with&nbsp;primitive&nbsp;shadow&nbsp;effect&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;text&nbsp;=&nbsp;<span class="visualBasic__string">&quot;Engineering&nbsp;and&nbsp;Computer&nbsp;Works&quot;</span>;//<span class="visualBasic__string">&quot;Bitmap&nbsp;Font&quot;</span>;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;size&nbsp;=&nbsp;buxton.MeasureString(text);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2((GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">2</span>)&nbsp;-&nbsp;size.X&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;<span class="visualBasic__number">104</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(buxton,&nbsp;text,&nbsp;textPosition,&nbsp;Color.Gray);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(buxton,&nbsp;text,&nbsp;textPosition&nbsp;-&nbsp;new&nbsp;Vector2(<span class="visualBasic__number">4</span>,&nbsp;<span class="visualBasic__number">4</span>),&nbsp;Color.CadetBlue);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private&nbsp;void&nbsp;DrawTopRightQuadrant()&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;{//Scale&nbsp;and&nbsp;Rotate&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Vector2&nbsp;textPosition&nbsp;=&nbsp;new&nbsp;Vector2(GraphicsDevice.Viewport.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>&nbsp;&#43;&nbsp;<span class="visualBasic__number">10</span>,&nbsp;<span class="visualBasic__number">10</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.DrawString(quadrantFont,&nbsp;<span class="visualBasic__string">&quot;&quot;</span>,&nbsp;textPosition,&nbsp;Color.White);&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;asteroid&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(asteroid,&nbsp;asteroidPosition,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new&nbsp;Vector2(asteroid.Width,&nbsp;asteroid.Height),&nbsp;asteroidScale,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;///&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;//&nbsp;Draw&nbsp;asteroid&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;spriteBatch.Draw(asteroid,&nbsp;asteroidPosition,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;null,&nbsp;Color.White,&nbsp;MathHelper.ToRadians(asteroidRotation),&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;new&nbsp;Vector2(asteroid.Width&nbsp;/&nbsp;<span class="visualBasic__number">2</span>,&nbsp;asteroid.Height&nbsp;/&nbsp;<span class="visualBasic__number">2</span>),&nbsp;asteroidScale,&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;SpriteEffects.None,&nbsp;<span class="visualBasic__number">0</span>);&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;<span class="visualBasic__preproc">&nbsp;
&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;#endregion</span>&nbsp;&nbsp;
&nbsp;&nbsp;&nbsp;&nbsp;}&nbsp;&nbsp;
}&nbsp;&nbsp;
</pre>
</div>
</div>
</div>
<h1><span>Source Code Files</span></h1>
<p>&nbsp;</p>
<p><span>111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111</span></p>
<p><span>111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111</span></p>
<p><span>1111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111111</span></p>
<p><span><br>
</span></p>
<ul>
</ul>
<p style="text-align:left"><em><em>&nbsp;</em></em></p>
<ul>
</ul>
<h1>More Information</h1>
<p><em>For more information, please email <a href="mailto:chanjunthoong@theiet.org">
chanjunthoong@theiet.org</a> or contact EngineeringComputerWorks.com</em></p>