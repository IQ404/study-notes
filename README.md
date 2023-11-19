# OpenGL

[dos.GL](https://docs.gl/) is a useful website to check the information about any OpenGL function.

- OpenGL, in itself, is a specification (function declarations) acting as an API to control GPU.

  The implementation (function definitions) is in the driver of the controlled GPU (written by the GPU's manufacturer, which means the implementations of OpenGL are most often not open sourced).

  OpenGL is cross platform: same OpenGL code can run on multiple platforms.

- Shader: a program to run on GPU.

- GLFW:  // NEED ELABORATION

  A library written in C to create and manage OpenGL context and its associated window specific to the local OS in our program.

  ❓ What does "creating an OpenGL context" really mean?

- GLEW/GLAD:

  Libraries to find the extended OpenGL functions implemented in the GPU driver at runtime, to manage the pointers to those functions and to provide interfaces (function declarations) for our program to call those extended OpenGL functions.

To write modern OpenGL (i.e. use those extended OpenGL functions), we often want to link GLFW and GLEW/GLAD libraries to a C++ program.

[Here](https://www.youtube.com/watch?v=OR4fNpBjmq8&list=PLlrATfBNZ98foTJPJ_Ev03o2oq3-GGOS2&index=2) is how Cherno set up GLFW + GLEW in a C++ project in visual studio.

## Vertex Buffer

- It is a block of memory in GPU (in VRAM) to be processed by shader.

We can create a vertex buffer as follows:

```cpp
unsigned int buffer_id;
glGenBuffers(1, &id);
```

`1` specifies that we want to create 1 buffer. `glGenBuffers` assigns the id of the created buffer into buffer_id (which must be `unsigned int`).
