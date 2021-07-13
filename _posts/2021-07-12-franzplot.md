---
layout: post
title:  "Franzplot: a teaching software (re)written in Rust"
author: Francesco Cattoglio
categories: [ tutorial ]
---
# Franzplot: a teaching software (re)written in Rust

## A bit of background
My name is Francesco Cattoglio and I have been a research assistant at Politecnico di Milano for a few years. Since 2018 I have taken up the role of teaching assistant for a class titled "Curves and Surfaces for Design". The goal of this class is to explain some 3D math concepts to Design students. As with everything related to mathematics, grasping some abstract concepts without practical examples can be tricky, so we spend about 15-20 hours doing computer-based exercises. As soon as I started my lessons I saw the students struggling with the tool we were using, so I proposed coding a new tool from scratch. And we did! In just four months of part-time working I managed to scrap together the first version of the new teaching software. Since my nickname at the office was Franz, my supervisor kept referring to it as "FranzPlot". In the end, that was the name which stuck, and she still apologizes to me every time I show the software to my students for the first time because of how ridiculous it sounds!

## First version
In short, the user creates a graph of nodes which contain some equations to create a given set of geometrical objects (parametric curves and surfaces) and apply some transformation matrices to them. Then, at scene creation time, the CPU computes those shapes and the user can inspect the scene to see what those objects look like.
The first version was written in C++, and used OpenGL via the Magnum library, which is a really amazing piece of work considering the very small team that put it together. I would really like to thank Mosra for making the first version of the Franzplot possible! Imgui was used to create the node editor interface which, as far as teaching goes, is the most important part of the whole project.

Node graph editor, screenshot from the latest version
![Node graph editing](https://github.com/francesco-cattoglio/stories/raw/main/assets/img/franzplot_nodes.png)

Scene rendering, screenshot from the last version. Nothing fancy, this is just a triangle mesh with a matcap shader multiplied by a black/white texture
![Node graph editing](https://github.com/francesco-cattoglio/stories/raw/main/assets/img/franzplot_scene.png)

## RIIR! (Rewrite It In Rust)
If the first version was C++, why was the second version built in Rust?  All the people involved saw the potential and decided to turn this side project into a "real" software. However, to take things to the next level many things needed addressing. First and foremost, the amount of technical debt was rather high. Second, OpenGL has already been deprecated by Apple and we could not afford the risk of having a third of our students being unable to run the software in the future if Apple decided to remove support altogether. Third, we wanted to make the tool more powerful, flexible and available as a web app as well.

To enhance the interactivity of the software we needed to be able to recompute *everything* at least 30 times per second, so the formula interpreter that we were using would not cut it anymore. After realizing that even a cheap laptop iGPU has a huge amount of flops, I decided to go for a compute-based solution, but how can one have compute shaders on a webpage? Enters WebGPU.
I first read of wgpu around March 2020: it was clear that everything was still a work in progress, but I already knew the amazing work made by the gfx-rs team and I was confident that wgpu-rs would have been the right bet. More than a year later I firmly stand by that assertion.
Up until that point I had very little Rust experience, but I was already in love with the language. During a meeting I explained my supervisors that it wouldn't have been an easy nor a quick job, but I was sure that the rust+wgpu combo would have been the right choice going forward: a modern language and a modern gpu API would have solved all our issues.

After many months of learning, opening github issues, getting help and advice from Kvark, Cwfitzgerald and many others from the community, we finally went live with the new version. The core of the software is rust, but the UI is still powered by imgui, using cxx as a bridge to use the imnodes library to implement the node graph editor. Most of the puzzle pieces fell right into their place, only requiring a minimal nudge to work together.
The new version was a success: even though some bugs needed ironing, the final product has been very solid so far. There are even a couple students that are unable to run the *old* OpenGL version on their laptops, but the new one runs just fine!

## Technical details
Franzplot uses wgpu for both compute and visualization purposes. Every time the "Generate Scene" button is clicked, a few things happen:
- first, the software checks that the graph does not contain any error nor cycles, and nodes are sorted according to their dependency
- then, for each node, the memory requirements for its output are computed and a wgpu buffer gets allocated
- finally, the expressions written by the user are turned into a compute shader that reads the input buffer and writes to the output one
The whole process takes very little time, less then one second for a reasonably-sized node graph. In a sense, FranzPlot is "just" a compiler: it turns mathematical formulas into GLSL. Everything else is just some glue code with a super simple 3D scene visualization on top.

When the user switches to the scene visualization all the shaders are run each frame, and this updates all the buffers that make up the meshes which are displayed to the user. The user can then change the value of a few defined global variables (stored in a uniform buffer) so that the scene can be updated in real time.

## Current state and future developments
The software is currently closed source, since we are still trying to figure out what the next steps should be. It might become open source in the future, but there are many things that we need to consider first.
W.r.t. the actual code, there are still many things to be done in the future: even though the new code was written from scratch, I still feel like some technical debt crept in and there are a few changes I would like to make to the internals. Adding more features will be fun, and I would *love* to ditch GLSL completely and move to WGSL, since it has matured a lot and that would allow me to get rid of the huge shaderc dependency.
I really hope that we manage to advertise the software a bit and find other universities with similar classes and need an easy-to-use tool for their students, possibly expanding its features.
Finally, even though I enjoyed working with imgui-rs & imnodes, I would like to find a rust-only solution for the UI. Right now I am investigating egui as a possible replacement for imgui, but this is kind of low-priority. 
