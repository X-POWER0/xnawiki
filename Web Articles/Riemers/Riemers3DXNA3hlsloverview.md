# XNA Tutorial using C# and HLSL Season 3 – Overview

Welcome to this 3rd installment of my XNA Tutorials for C#. This Series has as its main objective to introduce HLSL to you, what it is, what you can do with it and, of course, how to use it. This 3rd Series is written to be a complete hands-on HLSL Tutorial. Here’s a sample screenshot of what you’ll create:

![Objective](https://github.com/simondarksidej/XNAGameStudio/raw/archive/Images/Riemers/3DXNA0-00overview1.jpg?raw=true)

Once again, we’ll start very small, by drawing a simple triangle, and move on to more advanced topics and integrate them into our project. This time, we will no longer be using my effectx.fx file, as we will code our own. As main goal of this Series, we will be rendering a scene, which is being lit from a light sources.

So what’s so difficult about this? You already should have an idea about how to set up a light in a scene, using regular XNA code. In that case, however, all triangles in your scene would be lit by an amount of light that depends of how the triangle is facing your light source.

But you will not see any shadows! This is because XNA doesn’t know if there are any objects between the triangle and the light sources. So I thought having a light casting its shadows would already be nice goal for an introduction Series on HLSL. Have a quick look at the screenshots at the bottom of this page.

Don’t be mistaken – this is already quite an advanced topic, and we’ll move quickly through the first sets of pages of this Series, as you already know how to draw triangles. This Series will put its main focus on HLSL.

So, what do I expect you to know already? You can click on each item to be taken to the page where the concept was introduced.

Required (concepts that will be expanded on)(links to be replaced with XNA links):

* Camera initialization
* Drawing triangles from a vertex buffer (only vertex buffers, no index buffers)
* Adding textures to triangles
* A basic understanding of lighting (dot product)

Optional (code simply copied from previous chapters):

* Effect loading
* Loading a textured model from file

So what are you waiting for? Let’s move on to the first chapter!

During this series you will need to download the following resources:
Textured triangle: streettexture.dds
World transform: lamppost.x
World transform: carmesh.zip

Shaping the light: carlight.jpg

## Next Steps

[Starting point](Riemers3DXNA3hlsl01starting)
