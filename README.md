# OpenGL

## Useful Links

[dos.GL](https://docs.gl/) is a useful website to check the information about any OpenGL function.

## What is OpenGL

- OpenGL, in itself, is a specification (function declarations) acting as an API to control GPU.

  The implementation (function definitions) is in the driver of the controlled GPU (written by the GPU's manufacturer, which means the implementations of OpenGL are most often not open sourced).

  OpenGL is cross platform: same OpenGL code can run on multiple platforms.

- Shader: a program to run on GPU.

## Dependencies

- GLFW:  // NEED ELABORATION

  A library written in C to create and manage OpenGL context and its associated window specific to the local OS in our program.

❓ What does "creating an OpenGL context" really mean?

In my current understanding, context consists of the metadata of the state machine, if you think of OpenGL as a huge state machine.

- GLEW/GLAD:

  Libraries to find the extended OpenGL functions implemented in the GPU driver at runtime, to manage the pointers to those functions and to provide interfaces (function declarations) for our program to call those extended OpenGL functions.

- `opengl32.lib`:

  In my current understanding, on Windows, `opengl32.lib` provides fundamental functions to access base/modern OpenGL functions. Both GLEW/GLAD and GLFW rely on `opengl32.lib` for the program to call OpenGL functions.

To write modern OpenGL (i.e. use those extended OpenGL functions) on Windows, we often want to link `opengl32.lib`, GLEW/GLAD and GLFW libraries to a C++ program.

[Here](https://www.youtube.com/watch?v=OR4fNpBjmq8&list=PLlrATfBNZ98foTJPJ_Ev03o2oq3-GGOS2&index=2) is how Cherno set up GLFW + `opengl32.lib` + GLEW in a C++ project in visual studio.

A simple scaffolding is as follows:

```cpp
#include <iostream>

#include <GL/glew.h>        // include this before include gl.h
#include <GLFW/glfw3.h>


int main(void)
{
    /* Initialize the library */
    if (!glfwInit())
    {
        std::cout << "Error: glfwInit() failed to initialize" << std::endl;
        return -1;
    }
        
    /* Create a windowed mode window and its OpenGL context */
    GLFWwindow* window = glfwCreateWindow(640, 480, "Hello World", NULL, NULL);
    
    if (!window)
    {
        glfwTerminate();
        std::cout << "Error: glfwCreateWindow() failed to create the window" << std::endl;
        return -1;
    }

    /* Make the window's context current */
    glfwMakeContextCurrent(window);         // create OpenGL rendering context

    if (glewInit() != GLEW_OK)              // Do glewInit() only after we have a valid OpenGL rendering context
    {
        std::cout << "Error: glewInit() != GLEW_OK" << std::endl;
    }

    //-----------------------

    std::cout << "OpenGL version: " << glGetString(GL_VERSION) << std::endl;

    //-----------------------

    /* Loop until the user closes the window */
    while (!glfwWindowShouldClose(window))
    {
        /* Render here */
        glClear(GL_COLOR_BUFFER_BIT);

        glBegin(GL_TRIANGLES);
        glVertex2f(-0.5f, -0.5f);
        glVertex2f( 0.0f,  0.5f);
        glVertex2f( 0.5f, -0.5f);
        glEnd();

        /* Swap front and back buffers */
        glfwSwapBuffers(window);

        /* Poll for and process events */
        glfwPollEvents();
    }

    glfwTerminate();
    return 0;
}
```

Note that the code from `glBegin()` to `glEnd()` above is the legacy way to draw triangle in OpenGL. It is not encouraged to do so in modern OpenGL program. It is written here only to test if OpenGL is loaded correctly, since it is a fast way to render something.

## Vertex Buffer

It is an array of memory on GPU (in VRAM) to be processed by shader.

```cpp
float vertices[] =
{
    -0.5, -0.5f,    // vertex 1
    0.0f, 0.5f,     // vertex 2
    0.5f, -0.5f     // vertex 3
};

unsigned int buffer_id;
glGenBuffers(1, &buffer_id);
glBindBuffer(GL_ARRAY_BUFFER, buffer_id);
glBufferData(GL_ARRAY_BUFFER, 6 * sizeof(float), vertices, GL_STATIC_DRAW);
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, 0);
```

- `glGenBuffers` creates a vertex buffer. `1` specifies that we want to create 1 buffer. `glGenBuffers` assigns the ID of the created buffer into `buffer_id`, which must be (an array of) `unsigned int`.

- `glBindBuffer` lets the OpenGL state machine select the provided buffer so that the following manipulations are done on the buffer associated to `buffer_id`.

  `GL_ARRAY_BUFFER` means the nature of the buffer associated to buffer_id is an array. It is used for all buffer used to store vertex attributes.

- `glBufferData` lets OpenGL set the size for the binded buffer, sends the data that should be in the binded buffer, and gives a hint on how this buffer would be used (see [here](https://docs.gl/gl4/glBufferData) for the details).

- `glEnableVertexAttribArray` enables the attribute, only then will it be used in OpenGL draw calls. The `0` provided indicates the index of the attribute (with respect to the vertex) we are enabling.

- `glVertexAttribPointer` tells OpenGL the layout of the binded buffer specifically for one attribute of the vertex (and thus, `glVertexAttribPointer` should be called once per attribute).

  `0` there indicates the index of the attribute (with respect to the vertex) we are stating for. Here we only have 1 attribute (position) and its index is 0.

  `2` there indicates that this targeted attribute is consisted of two components: we have two floats to represent a position for a vertex.

  `GL_FLOAT` there indicates the type of the components just mentioned is float.

  `GL_FALSE` there indicates that we don't need OpenGL to further normalize the values we provided for the components (<ins>FURTHER ELABORATION NEEDED</ins>: how does this normalization actually work?).

  `sizeof(float) * 2` there indicates the stride: how many bytes are there between each vertex.

  `0` there is a pointer (it is implicitly `(const void*)0`) indicating the offset (in bytes) of the targeted attribute into the vertex.

  ❓ Elaborate more on things like `(const void*)8`.

```cpp
glDrawArrays(GL_TRIANGLES, 0, 3);
```

- `glDrawArrays` is a draw call to the currently binded vertex buffer when index buffer is not used.

  `GL_TRIANGLES` means we are drawing triangles.

  `0` there means we are starting to render trangles from the first vertex in the binded buffer.

  `3` there means how many vertices we are going to render.

## Compiling Shaders

