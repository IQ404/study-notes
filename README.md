# OpenGL

## Contents

I chose not to mannually write a "contents" since the "Outline" button on the top right corner of a github markdown page has already rendered a beautiful navigator.

The basic structure of this note I'm planning is as follows: in the first part we will try to get familiar with the OpenGL context mainly from the C++ end; in the second part I will try to introduce how to write shaders; then in the third part, we will try to dive into some rendering techniques.

### TODOs

- Integrate CSC8502 into this note
- The current GLSL stuffs are messy and incomprehensive, polish them!
- Perhaps merge those stuffs in the second part that is duplicated from the first part.

## Related Projects

[Please see this repository](https://github.com/IQ404/learning-opengl).

## Useful Links

[dos.GL](https://docs.gl/) is a mostly useful (but sometimes confusing) website to check the information about OpenGL functions.

## Currently Unclassified Notes

- In OpenGL, `0` often represents the ID for non-existent object. So, we often use `0` for detaching.

- ❓ Deeply explore the asynchrony between CPU and GPU in OpenGL.

  A short note on this topic:

  Asynchronous CPU-GPU interactions is key for achieving high performance in graphics applicationsfor, as it allows both the CPU and GPU to work in parallel.

  When a CPU thread calls an OpenGL function to perform a GPU operation, it issues OpenGL commands from the CPU, these commands are queued for execution on the GPU (OpenGL commands are placed in a command buffer on GPU, and the GPU executes them in order). The CPU does not necessarily wait for the GPU to finish the operation before it continues executing, it can continue executing subsequent instructions, and can queue many commands rapidly without waiting for the GPU to catch up.

  There are certain points (called synchronization point) where synchronization between the CPU and GPU is required. Generally, when encountering a synchronization point, CPU does not need to wait for all queued commands in GPU to finish before it continues executing. For examples:

  - When swapping buffers (often at the end of rendering a frame, e.g., `glfwSwapBuffers()` in GLFW), the CPU waits for the GPU to finish rendering the current frame before the buffers are swapped. This wait is usually for the completion of the frame rendering commands only, not all commands in the queue. (❓ Is this true? Especially when using ImGui.)
 
  - When reading data back from the GPU (e.g., `glReadPixels`), the CPU will wait for the GPU to finish all operations that affect the data being read back. For example, if you're reading from a framebuffer, the CPU will (only) wait for all rendering commands that update that framebuffer to complete.
 
  - OpenGL provides more fine-grained synchronization mechanisms like `glFenceSync`. A fence sync object can be inserted into the command stream, and the CPU can later wait for just the commands issued before the fence to complete. This allows for more targeted synchronization instead of waiting for all commands to finish.
 
  - When `glFinish()` is called, the CPU will wait until all previously issued OpenGL commands have been fully executed and completed by the GPU.
 
    As a supplement, `glFlush()` ensures that all previously issued OpenGL commands are pushed to the GPU for execution, but it does not wait for their completion. The CPU can continue executing subsequent instructions without waiting for the GPU to finish these commands.

- ❓ Deeply explore double-buffering.

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

❓ Why doesn't the image (e.g., say we have some texture) show on the window (I tried this using Visual Studio's break points) immediately after `glfwSwapBuffers`? (note that if we put sufficient stuffs in between `glfwSwapBuffers` and `glfwPollEvents`, the texture will appear on the window before `glfwPollEvents`)

❓ Explain in detail what is `glfwPollEvents` doing.

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

  - In my current understanding:
  
    If the data type of the vertex attribute is an unsigned type (like `GL_UNSIGNED_BYTE`), normalization scales the integer data to the range `[0, 1]`. For example, if the vertex attribute data is an array of `GL_UNSIGNED_BYTE` (values from `0` to `255`), with normalization enabled, these values are scaled to the range `0.0` to `1.0`.
 
    If the data type is a signed type (like `GL_BYTE`), normalization scales the data to the range `[-1, 1]`. So, if you have an array of `GL_BYTE` (values from `-128` to `127`), with normalization enabled, these values are scaled to the range `-1.0` to `1.0`.

    The normalization process does not change the actual storage format of the data in the buffer. Instead, the normalization is performed at the time the data is read and used by the GPU.

    When vertex attribute data is normalized and presented to the vertex shader in OpenGL, it is typically in single-precision floating-point format (`float`) rather than double-precision floating-point format (`double`).

  `sizeof(float) * 2` there indicates the stride: how many bytes are there between each vertex.

  `0` there is a pointer (it is implicitly `(const void*)0`) indicating the offset (in bytes) of the targeted attribute into the vertex.

  ❓ Elaborate more on things like `(const void*)8`. Why can `0` be implicitly casted to `void*` while other number cannot? Why do we need `const` there?

## Compiling/Linking Shaders

Callings of shader programs happen after issuing GPU an OpenGL draw call: shaders within an OpenGL rendering pipeline specify some of the necessary deatils on how to draw what the corresponding draw call is ultimately aimed to draw.

In its simplest form, vertex shader will be called once for each vertex somewhere after issuing GPU an OpenGL draw call, and fragment shader will be called once for each pixel that needs to be shaded somewhere after the vertex shader stage.

The ultimate purpose of a vertex shader is to determine where those vertex in screen space will be. It can take in the data in the binded vertex buffer, and it can also pass data down into the next shader stage of the shader program pipeline.

The ultimate purpose of a fragment shader is to determine and output the color for the pixel it is shading.

Normally fragment shader will be called much more times than that will be called for a vertex shader. Hence, computations in fragment shader is much more expensive than in vertex shader.

```cpp
static unsigned int CompileShader(unsigned int type, const std::string& source_code)
{
    unsigned int shader_id = glCreateShader(type);
    const char* src = source_code.c_str();  // .c_str returns a pointer to the head of a null-terminated character array.
    glShaderSource(shader_id, 1, &src, nullptr);
    glCompileShader(shader_id);
    
    /*** Error handling: ***/
    int compile_status;
    glGetShaderiv(shader_id, GL_COMPILE_STATUS, &compile_status);
    if (compile_status == GL_FALSE)
    {
        int log_length;
        glGetShaderiv(shader_id, GL_INFO_LOG_LENGTH, &log_length);
        char* log = (char*)alloca(log_length * sizeof(char));
        glGetShaderInfoLog(shader_id, log_length, &log_length, log);
        std::cout << "Failed to compile ";
        switch (type)
        {
        case GL_VERTEX_SHADER:
            std::cout << "vertex shader" << std::endl;
            break;
        case GL_FRAGMENT_SHADER:
            std::cout << "fragment shader" << std::endl;
            break;
        default:
            break;
        }
        std::cout << log << std::endl;
        glDeleteShader(shader_id);
        return 0;   // A value of 0 for shader will be silently ignored by further OpenGL calls.
    }
    /*** End of error handling ***/

    return shader_id;
}
```

- `CompileShader` is declared as `static` just to have internal linkage.

- `glCreateShader` creates a shader object on GPU.

  The type of the shader created will be the type provided (see [here](https://docs.gl/gl4/glCreateShader) for the options).

  It returns the ID representing the shader object that is created.

- `.c_str` returns a pointer to the head of a null-terminated character array.

- `glShaderSource` sets (actually, replaces, i.e., overwrites) the shading source code stored in the shader object with ID `shader_id`.

  `1` there means there is only one source code we want to set. ❓ How does a shader with multiple parts of source code works?

  Since it can have multiple source code, it then receives a pointer to an array of pointers each pointing to a character array (in our case this is `&src`).

  It then receives a pointer to an array of `int` each specifying the length of its corresponding character array (here if the `int` is negative, it means that the crresponding source code character array is null-terminated). In our case we pass `nullptr` (equivalent to `NULL`), which means all the source code character arrays are null-terminated (in our case there is only one source code).

- `glCompileShader` compiles the shader with provided ID and returns `void`.

- `glGetShaderiv` is used to query various kinds of information about a shader object:

  - It first takes in the shader's ID.
 
  - It then takes a macro (which in this case is an `int`) indicating the kind of information to be queried (see [here](https://docs.gl/gl4/glGetShader) for the details of the options).
 
    Note that the length returned by `GL_INFO_LOG_LENGTH` including an extra `char` of the null terminator.
 
  - It then takes a pointer to an `int`, storing the queried information in the `int`.

  - `iv` stands for integer vector, in my current understanding, this indicates that the function outputs integer array (vector) and thus we pass the pointer to `int` to it.

- `char* log = (char*)alloca(log_length * sizeof(char));` is a way to dynamically allocate memory on stack.

- `glGetShaderInfoLog` first takes in the shader's ID.

  It then takes in the length of the character array for storing the returned information log.

  It then takes a pointer to an `int`, putting the length of the returned information log (without the extra `char` of the null terminator) into the `int`.

  It then takes a pointer to a `char` array, putting the information log into it.

- `glDeleteShader` flags the shader object with the provided ID for deletion. The shader object will then only be deleted when it is not attached to any shader program object.

  `glDeleteShader` employs inclusio for `glCreateShader`.

```cpp
static unsigned int CreateShaderProgram(const std::string& vertexShader, const std::string& fragmentShader)
// returns the ID of the shader program in OpenGL
{
    unsigned int shader_program_id = glCreateProgram();

    unsigned int vertex_shader_id = CompileShader(GL_VERTEX_SHADER, vertexShader);
    unsigned int fragment_shader_id = CompileShader(GL_FRAGMENT_SHADER, fragmentShader);

    glAttachShader(shader_program_id, vertex_shader_id);
    glAttachShader(shader_program_id, fragment_shader_id);

    glLinkProgram(shader_program_id);
    glValidateProgram(shader_program_id);

    glDeleteShader(vertex_shader_id);
    glDeleteShader(fragment_shader_id);

    //glDetachShader(shader_program_id, vertex_shader_id);
    //glDetachShader(shader_program_id, fragment_shader_id);

    return shader_program_id;
}
```

- `CreateShaderProgram` is declared as `static` just to have internal linkage.

- `glCreateProgram` creates a program object on GPU and returns its ID.

- With provided IDs, `glAttachShader` attaches the shader object to the program object.

  Note that, since shader objects are created with their type specified, attaching order does not need to match the order in the rendering pipeline.

- `glLinkProgram` generates executables (that will run on the corresponding programmable processor) for the compiled shaders attaching to the program object of providing ID.

- `glValidateProgram` leaves message about "Given how everything is currently set up in OpenGL (the current state), can the shader program with provided ID be executed without any errors?" in the program's information log. Note that this is just for debug purpose, and it's not compulsory after a call of `glLinkProgram`.

- `glDetachShader` undoes the effect of `glAttachShader`: with the provided IDs, it detaches the shader object from the program object.

## Writing Your First Shader

```cpp
std::string vertexShader =
    "#version 450 core\n"
    "\n"
    "layout (location = 0) in vec4 position;\n"
    "\n"
    "void main()\n"
    "{\n"
    "   gl_Position = position;\n"
    "}\n"
    ;

std::string fragmentShader =
    "#version 450 core\n"
    "\n"
    "layout (location = 0) out vec4 color;\n"
    "\n"
    "void main()\n"
    "{\n"
    "   color = vec4(1.0f, 0.0f, 0.0f, 1.0f);\n"
    "}\n"
    ;

unsigned int shader_program_id = CreateShaderProgram(vertexShader, fragmentShader);
glUseProgram(shader_program_id);
// ...
glDeleteProgram(shader_program_id);
```

- `#version 450` specifies to use GLSL 4.5, which is corresponding to OpenGL 4.5

  `core` means that any deprecated functions from earlier versions are not allowed to use.

- `in` means that the following data is inputting from the binded vertex buffer.

  `out` means that the following data is outputting from the fragment shader to the associated framebuffer (❓ Elaborate on the framebuffer associating with fragment shader).

- In vertex shader, `layout (location = 0)` indicates that, for each vertex, `vec4 position` matches the first (0th) attribute of that vertex coming from the vertex buffer (for our previous code, it matches the first `0` in `glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, 0);`).

  An important note here is that, currently location 0 actually feeds in 2 floats: It is predefined that OpenGL will set the 1st and 2nd components of the vec4 to these two floats respectively, and set the 3rd and 4th components of the vec4 to 0.0f (This is based on the assumption that if Z is not provided, the vertex is meant to be in a plane) and 1.0f (implies a homogeneous position in space as opposed to a homogeneous direction vector, which would have set to 0.0f) respectively.

  The fragment shader outputs data to some framebuffer for latter use in the rendering pipeline. The `layout (location = 0)` in the fragment shader is saying that `vec4 color` is outputted to the `0`th color attachment in the framebuffer (❓ Elaborate on the color attachments in the framebuffer).

- `gl_Position` stores the [clip space](https://en.wikipedia.org/wiki/Clip_coordinates) position of the vertex.

  // NEED TO BE REWRITE ACCORDING TO COMPUTER GRAPHICS!!!

  Clip space is the space after transforming by the projection matrix. After the vertex shader, the clip space coordinates in gl_Position are automatically divided by their W component (perspective division) by OpenGL. This step converts clip space coordinates to normalized device coordinates (NDC), which range from -1 to 1 in each axis. The NDC are transformed into screen space coordinates through the viewport transformation. This step maps the NDC to the actual coordinates on the screen (or the framebuffer) and is handled by OpenGL as part of the fixed-function pipeline (❓ Need deeper understanding).

- `glUseProgram` makes the program object with the provided ID in use for current rendering state.

- `glDeleteProgram` flags the program object with the provided ID for deletion. The program object will then only be deleted when it is not in use as part of current rendering state.

## Draw Call without Index Buffer

```cpp
glDrawArrays(GL_TRIANGLES, 0, 3);
```

- `glDrawArrays` is a draw call to the currently binded vertex buffer (this function is often used when index buffer is not in use).

  `GL_TRIANGLES` means we are drawing triangles.

  `0` there means we are starting to render trangles from the first vertex in the binded buffer.

  `3` there means, from (and include) the starting vertex we just specified onwards, how many vertices we are going to render.

## Unifying Shaders Source Code into One File

```cpp
// Standard:
#include <iostream>
#include <fstream>
#include <string>
#include <sstream>
// Externel:
#include <GL/glew.h>        // include this before include gl.h
#include <GLFW/glfw3.h>

struct ShaderProgramSourceCode
{
    std::string VertexShaderSourceCode;
    std::string FragmentShaderSourceCode;
};

static ShaderProgramSourceCode ParseUnifiedShader(const std::string& filepath)
{
    std::ifstream input_stream{ filepath };
    enum class ShaderType
    {
        NONE = -1,
        VERTEX = 0,
        FRAGMENT = 1
    };
    std::string current_line;
    std::stringstream shader_source_code_buffer[2];
    ShaderType current_shader_type = ShaderType::NONE;
    while (std::getline(input_stream, current_line))
    {
        if (current_line.find("#shader") != std::string::npos) // std::string::npos is often implemented as -1
        {
            if (current_line.find("vertex") != std::string::npos)
            {
                current_shader_type = ShaderType::VERTEX;
            }
            else if (current_line.find("fragment") != std::string::npos)
            {
                current_shader_type = ShaderType::FRAGMENT;
            }
            else
            {
                std::cout << "Syntax Error: unspecified shader type in unified shader file." << std::endl;
            }
        }
        else
        {
            shader_source_code_buffer[(int)current_shader_type] << current_line << '\n';
        }
    }
    return { shader_source_code_buffer[0].str(),shader_source_code_buffer[1].str() };
}

// ...

ShaderProgramSourceCode shader_program_source_code = ParseUnifiedShader("res/shaders/Basic.shader");
unsigned int shader_program_id = CreateShaderProgram(shader_program_source_code.VertexShaderSourceCode, shader_program_source_code.FragmentShaderSourceCode);
```

Note that `std::string::find` returns the position of the first character of the first match if match is found, and returns `string::npos` if no matches were found.

`Basic.shader` contains:

```cpp
#shader vertex
#version 460 core

layout (location = 0) in vec4 position;

void main()
{
   gl_Position = position;
}

#shader fragment
#version 460 core

layout (location = 0) out vec4 color;

void main()
{
   color = vec4(0.2f, 0.3f, 0.8f, 1.0f);
}
```

## Index Buffer

Previously, we used `glDrawArrays` to draw primitives by drawing the vertices in the vertex buffer one-by-one along the vertex array directly.

Instead, we can specify an index for each vertex in the vertex buffer, and draw vertices according to an indices array. In doing so we can eliminate the VRAM consumption from duplicated vertices.

In OpenGL this can be done as follows:

```cpp
// ...

float vertices[] =
{
    -0.5f,-0.5f,    // vertex 1
     0.5f,-0.5f,    // vertex 2
     0.5f, 0.5f,    // vertex 3
    -0.5f, 0.5f     // vertex 4
};

unsigned int indices[] =    // OpenGL requires index buffer to store unsigned data!!!
{
    0,1,2,  // lower-half triangle for the rectangle
    0,2,3   // upper-half triangle for the rectangle
};

unsigned int vertex_buffer_id;
glGenBuffers(1, &vertex_buffer_id);
glBindBuffer(GL_ARRAY_BUFFER, vertex_buffer_id);
glBufferData(GL_ARRAY_BUFFER, 4 * 2 * sizeof(float), vertices, GL_STATIC_DRAW);
glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, 0);

unsigned int index_buffer_id;
glGenBuffers(1, &index_buffer_id);
glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, index_buffer_id);
glBufferData(GL_ELEMENT_ARRAY_BUFFER, 3 * 2 * sizeof(unsigned int), indices, GL_STATIC_DRAW);

// ...

// glDrawArrays(GL_TRIANGLES, 0, 6);    // use glDrawArrays when index buffer is not in use
glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr);

// ...
```

- <ins>Note in particular that</ins>, in OpenGL, when we bind a vertex buffer with `glBindBuffer(GL_ARRAY_BUFFER, vertex_buffer_id)` and then bind an index buffer with `glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, index_buffer_id)`, both buffers remain bound to the current rendering state. This is because the first parameter in the `glBindBuffer` function specifies the target to which the buffer is bound, and `GL_ARRAY_BUFFER` and `GL_ELEMENT_ARRAY_BUFFER` are different targets.

- Note that the unsigned integers in the index buffer corresponds to the indices in the array of vertices (i.e. the order in vertex buffer needs to match the order in index buffer).

- `glDrawElements` is recommended to use when index buffer is binded.

  `6` corresponds to the number of indices we are drawing, not the number of vertices in vertex buffer we are drawing!

  `GL_UNSIGNED_INT` indicates the type of data in the index buffer.

  Now, if index buffer is binded (as we did, <ins>which is te recommended way of using index buffer</ins>), the last parameter expecting a pointer sets the offset (<ins>in bytes</ins>) into the index buffer. Here we pass in `nullptr` (which is equivalent to `(void*)0`) means that we are drawing `6` indices from the beginning of the index buffer. If we passed in, say, `glDrawElements(GL_TRIANGLES, 3, GL_UNSIGNED_INT, (void*)(3*sizeof(unsigned int)))`, it will just draw the upper triangle.

  If index buffer is not binded, this last pointer parameter is treated as a pointer to the index data itself (which means the index data is sit on the memory of the CPU side). <ins>This is not recommended</ins>, because then it will send the index data from the CPU to the GPU each time glDrawElements is called.

## Basic Error Report

By basic I mean we are implementing error reporting using the OpenGL function `glGetError`. The value of error flags that can be returned by this function can be found [here](https://docs.gl/gl4/glGetError).

- In my current understanding, if we call some OpenGL function and some error occurs, a global error flag will be set to some `GLenum` representing the type of the error occurred. There may only be one such error flag but it is likely to exist multiple of such error flag in most implementations. Each error flag can only record one error, and if multiple errors occurred, other errors are either recorded by other error flags or ignored (and discarded). `glGetError` checks all the flags, returns and clears the value of one of the error flags (chosen arbitrarily if there are multiple errors), resetting it to `GL_NO_ERROR`. Hence, to relate error to its initiator, we still need to call `glGetError` in a loop.

  ❓ Explore the possible situation(s) where newly occurred error is discarded even when there is available error flag(s).

- One important note is that, in many implementations, `glGetError` will return an error code when it is called without a valid OpenGL context (e.g. after `glfwTerminate();` is called), which means checking `glGetError()` in a loop may result in an infinite loop if we somehow do that after `glfwTerminate();` (e.g. destructor of stack variable may invoke `glGetError` after `glfwTerminate();` which stays inside the closing curly bracket). However, in my current understanding, there is no guarantee for `glGetError` to return error code if there is no valid OpenGL context (it is an undefined behavior) - so, make sure that we do NOT call `glGetError` after `glfwTerminate`.

One way to implement `glGetError` error reporting, using macros in MSVC, is as follows:

```cpp
// Macros:

/*
ASSERT_DebugBreak_MSVC(b): break at the current line if b evaluates to false
    - It is currently MSVC-specific.
    - Add `;` at the end when using it, as its current definition does not end with `;`.
*/
#define ASSERT_DebugBreak_MSVC(b) if (!(b)) __debugbreak()  // __debugbreak is MSVC-specific

/*
GLCall(s): calling OpenGL function with error reporting
    - It is currently MSVC-specific.
    - It will clear all the previously set OpenGL error flags.
    - Add `;` at the end when using GLCall(s), as its current definition does not end with `;`.
    - Don't write one-line statement using this macro, because its current definition body isn't enclosed with {}.
*/
#define GLCall(s)\
        GLClearErrors();\
        s;\
        ASSERT_DebugBreak_MSVC(!(GLErrorLog(#s, __LINE__, __FILE__)))

static void GLClearErrors()
{
    while (glGetError() != GL_NO_ERROR);    // GL_NO_ERROR is guaranteed to be 0
}

static bool GLErrorLog(const char* function_called, int line_calling_from, const char* filepath_calling_from)
// returns true if there is error; false there isn't.
{
    if (GLenum error = glGetError())    // enters if block as long as error != 0
    {
        std::cout << "------[OpenGL Error]------" << '\n'
            << "Error GLenum: " << error << '\n'
            << "By executing: " << function_called << '\n'
            << "At line: " << line_calling_from << '\n'
            << "In file: " << filepath_calling_from << '\n'
            << "--------------------------" << std::endl;

        return true;
    }
    return false;
}

// ...

GLCall(glDrawElements(GL_TRIANGLES, 6, GL_INT, nullptr));  // currently this line is intentionally incorrectly written (GL_INT should be GL_UNSIGNED_INT)

// ...
```

❓ Further explore [replacing text macros](https://en.cppreference.com/w/cpp/preprocessor/replace).

## Uniform

Uniform allows us to send data from CPU to a shader program object on the GPU (each uniform is per shader program object) to be used in the shaders i the shader program object.

When the shader program object is created, each uniform in the program object (if not compiled away) will be assigned with a unique ID.

To retrieve that ID, we need to find bind the shader program object.

If `glGetUniformLocation` can't find that uniform, it will return `-1`. Note that this can happen when the uniform we are retrieving is actually written in the shader code but is compiled away (e.g., if this uniform is not used throughout the shader program).

Note in particular that:

- Uniform values are maintained per shader program object. You can write uniform with the same name and type in multiple shaders and it will refer to one single uniform.

  ❓ When single uniform is used by multiple threads, is there a performance overhead caused by locking and, if so, how does the GPU address that?
  
- Once you set a uniform value in a shader program, that value remains set in that specific program until you explicitly change it or until the program is deleted.
- You need to set the uniform value separately for each shader program object.

Example:

```cpp
// ...
GLCall(glUseProgram(shader_program_id));

GLCall(int u_color_id = glGetUniformLocation(shader_program_id, "u_color"));
//ASSERT_DebugBreak_MSVC(u_color_id != -1);

float r = 0.0f;
float increment = 0.0f;

/* Loop until the user closes the window */
while (!glfwWindowShouldClose(window))
{
    /* Render here */
    GLCall(glClear(GL_COLOR_BUFFER_BIT));

    if (r <= 0.0f)
    {
        increment = 0.05f;
    }
    if (r >= 1.0f)
    {
        increment = -0.05f;
    }
    r += increment;

    GLCall(glUniform4f(u_color_id, r, 0.3f, 0.8f, 1.0f));

    // glDrawArrays(GL_TRIANGLES, 0, 6);    // use glDrawArrays when index buffer is not in use
    GLCall(glDrawElements(GL_TRIANGLES, 6, GL_UNSIGNED_INT, nullptr));

    /* Swap front and back buffers */
    glfwSwapBuffers(window);

    /* Poll for and process events */
    glfwPollEvents();
}
// ...
```

where the used shaders are:

```cpp
#shader vertex
#version 460 core

layout (location = 0) in vec4 position;

void main()
{
   gl_Position = position;
}

#shader fragment
#version 460 core

uniform vec4 u_color;	// the u prefix is a naming convention for uniform variables
layout (location = 0) out vec4 color;

void main()
{
   color = u_color;
}
```

## V-Sync

```cpp
// After we have created OpenGL rendering context
glfwSwapInterval(1);
```

- `glfwSwapBuffers` sets the number of refresh cycles of the monitor to wait from the time `glfwSwapBuffers` was called before swapping the buffers.

  `1` means that, after the execution has enter `glfwSwapBuffers`, it will need to pause and wait to swap the front/back buffers until the current refresh cycle of the monitor has completed. This is to enable V-Sync.

  Setting `0` instead means that the execution can swap the front/back buffers whenever it wants. This is to disable V-Sync.

- In my current understanding, swapping front/back buffers just involves changing the pointers to the front and back buffers (rather than physically copying the data from one buffer to the other), and thus is done really fast.

  When V-Sync is enabled, the graphics driver of the monitor will be timed to ensure that the swapping of the buffers will be done after the current refresh cycle finishes and before the next refresh cycle begins.

- In my current understanding, some monitor updates its display line by line, some reading all the data from the buffer in a single operation.

  When V-Sync is disabled, if a monitor updates its display line by line, screen tearing occurs when the buffer swap happens partway through this process, resulting in different parts of the screen showing data from different frames; if the monitor updates the entire screen at once, screen tearing is less likely to occur, but it can still happen.

- Enabling V-Sync can cause frame rate to drop. Here is an example when monitor's refresh rate is larger than the rendering rate without V-Sync:

  If my monitor is refreshing every 2 ms (500 fps), and if my rendering loop (if v-sync is disabled) reaches `glfwSwapbuffers` every 3 ms (333 fps. In my current understanding, if v-sync is off, the effective frame rate will approximately be the same as this number), then, say at time `t=0` the buffers are swapped and a new refresh cycle starts, the next refreshing cycle will start at `t=2`, but the back buffer will be ready at `t=3`, so the next frame on the monitor will be displaying what the current frame is displaying. Then, in the frame after next frame on the monitor (at `t=4`), data that was ready at `t=3` will be displayed. From `t=3` to `t=4`, the execution for my rendering will be waiting inside `glfwSwapbuffers`, and from `t=4`, the same thing happens as from `t=0`. This means that the monitor will now display new data every 4 ms (250 fps).

  I think the point to take away here is that, given a misalignment between the rate of swapping the buffers and the rate of refreshing display, with V-Sync, the rendering execution, once it is ready for swapping the buffers, will always need to wait for the alignment, while without V-Sync, the execution can use this duration (and all the subsequent duration) of waiting for alignment to render more frames.

## Vertex Array Object

VAO is kinda OpenGL-special: there is no such concept in e.g. DirectX.

VAO is mandatory in OpenGL: in compatibility mode (for older version of OpenGL), the OpenGL context will implicitly create a default VAO and assign its ID with `0`; in core mode, we need to explicitly create/manage VAO(s), and doing `glBindVertexArray(0)` leads no VAO to bound.

```cpp
// ...

unsigned int vao_id;
glGenVertexArrays(1, &vao_id);
glBindVertexArray(vao_id);

// ...

glEnableVertexAttribArray(0);
glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, 0);
```

We can think of a VAO as a constrct which has many slots (this number of slots isn't infinite, it is a fixed number. In my current system, this number is 16. One single VAO in OpenGL cannot hold more attributes than this number).

Each of those slots is assigned with an index (e.g. `0` to `15`).

`glEnableVertexAttribArray(0);` enables the slot `0` on the currently bounded VAO.

Once enabled, we can link an attribute in the currently bounded VBO to that slot e.g. using `glVertexAttribPointer(0, 2, GL_FLOAT, GL_FALSE, sizeof(float) * 2, 0);` as above.

Once a VBO is linked to a VAO and the VAO is active, we don't need to further bind the VBO to the current state to issue draw call.

VAO itself stores the specifications of the layouts of the linked attributes in the associated VBO(s). On the other end, the currently bounded VAO also links to the vertex shader in the currently bounded shader program. The integer `0` specified in e.g. `layout (location = 0) in vec4 position;` matches the slot index on the currently bounded VAO, meaning that the attribute data coming from the VBO linked with slot `0`, extracted according to the corresponding specification stored in the VAO, will be sent into the variable `vec4 position`.

Note that a VAO does NOT govern which shader program is in use.

## OpenGL context version/profile

```cpp
glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);
glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
//glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_COMPAT_PROFILE);
```

`glfwWindowHint` sets hints for the next call to `glfwCreateWindow`.

`glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4); glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);` means the OpenGL context to be created uses OpenGL version `4.6`.

❓ What happens when the OpenGL version/profile specified in `glfwWindowHint` mismatches the `#version` specified in the shader in use?

❓ Explain why, for my system, the OpenGL version specified in `glfwWindowHint` does not affect the OpenGL version in use (returned by `glGetString(GL_VERSION)`) when I use `glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_COMPAT_PROFILE);`.

## Data Model for VBO, VAO, Index Buffer and Vertex Shader

- In my current understanding, the mental picture for the data model of these constructs is as follows:

  - VAO has many slots, each to be linked to (an attribute of) a VBO.
  - VAO itself stores the specifications of the layouts of the linked attributes in the associated VBO(s).
  - VAO can also store the information of an index buffer.
  - VAO is also connected to vertex shader such that each slot is associated to the corresponding vertex shader variable.
  - For each slot, VAO interprets the data in the VBO(s) according to the specifications stored in the VAO. VAO then extracts the data in the VBO(s) according to the linked index buffer, and sends the data to the corresponding vertex shader variables in the active vertex shader.

- In my current understanding, it's crucial to distinguish the index buffer bound to the global state and the index buffer bound to the VAO state:

  While a VAO is bound (i.e. active), it will capture the index buffer that is subsequently bound.

  Once a VAO captures an index buffer, OpenGL will automatically use the captured index buffer for any subsequent drawing calls that use indices (like `glDrawElements`) whenever we bind the VAO.

  We can bind an index buffer to the global state before binding any VAO. Such bound index buffer isn't associated with any subsequently bound VAO. Order matters here: VAO does not retroactively capture an index buffer that was bound before the VAO was bound. In OpenGL, state capture for VAOs is based on the commands issued while the VAO is bound. This means that the VAO only stores the state of the index buffer if the index buffer is bound after the VAO itself is bound.

  When you call `glDrawElements`, OpenGL uses the index buffer that is associated with the currently bound VAO to determine how to fetch vertex data from the vertex buffers (VBOs). The recent global index buffer binding state is independent of the recent VAO's state. Once an index buffer is associated with a VAO, you don't need to keep the index buffer bound globally for `glDrawElements` to use it. The key is the VAO's binding, not the global index buffer binding.

  But it is important to note that `glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0)` DOES break the binding of the index buffer to the active VAO! ([reference](https://stackoverflow.com/a/72760508/17172007))

## Separating out the debug tools

I chose to create a `.cpp` file and a `.h` file specifically for our previously created debug tools: I found that mixing tools with constructs will be highly likely to result in miserable situations for headers inclusion (e.g. caused by very subtle circular inclusion of headers). Only include the minimal of what we need helps to avoid such circular inclusions. Plus, I personally avoid forward declaration where possible, because I think the necessity of forward declaration in a project is highly likely to be an indicator of a either globally or locally bad design of solving the problem.

`DebugTools.h`:

```cpp
#ifndef DEBUGTOOLS_H
#define DEBUGTOOLS_H

#include <GL/glew.h>        // include this before include gl.h

// Macros:

/*
ASSERT_DebugBreak_MSVC(b): break at the current line if b evaluates to false
    - It is currently MSVC-specific.
    - Add `;` at the end when using it, as its current definition does not end with `;`.
*/
#define ASSERT_DebugBreak_MSVC(b) if (!(b)) __debugbreak()  // __debugbreak is MSVC-specific

/*
GLCall(s): calling OpenGL function with error reporting
    - It is currently MSVC-specific.
    - It will clear all the previously set OpenGL error flags.
    - Add `;` at the end when using it, as its current definition does not end with `;`.
    - Don't write one-line statement using this macro, because its current definition body isn't enclosed with {}.
*/
#define GLCall(s)\
        GLClearErrors();\
        s;\
        ASSERT_DebugBreak_MSVC(!(GLErrorLog(#s, __LINE__, __FILE__)))

void GLClearErrors();

bool GLErrorLog(const char* function_called, int line_calling_from, const char* filepath_calling_from);
// returns true if there is error; false there isn't.

#endif // !DEBUGTOOLS_H
```

`DebugTools.cpp`:

```cpp
#include "DebugTools.h"

#include <iostream>

void GLClearErrors()
{
    while (glGetError() != GL_NO_ERROR);    // GL_NO_ERROR is guaranteed to be 0
}

bool GLErrorLog(const char* function_called, int line_calling_from, const char* filepath_calling_from)
// returns true if there is error; false there isn't.
{
    if (GLenum error = glGetError())    // enters if block as long as error != 0
    {
        std::cout << "------[OpenGL Error]------" << '\n'
            << "Error GLenum: " << error << '\n'
            << "By executing: " << function_called << '\n'
            << "At line: " << line_calling_from << '\n'
            << "In file: " << filepath_calling_from << '\n'
            << "--------------------------" << std::endl;

        return true;
    }
    return false;
}
```

## Basic abstraction of VBO

`VBO.h`:

```cpp
#ifndef VBO_H
#define VBO_H

class VBO
{
	unsigned int m_VBOID = 0;

public:

	VBO(const void* data, unsigned int size);	// size in bytes; bind after initialization
	
	~VBO();

	void Bind() const;
	
	void Unbind() const;
};

#endif // !VBO_H
```

`VBO.cpp`:

```cpp
#include "VBO.h"
#include "DebugTools.h"

VBO::VBO(const void* data, unsigned int size)   // size in bytes; bind after initialization
{
    GLCall(glGenBuffers(1, &m_VBOID));
    GLCall(glBindBuffer(GL_ARRAY_BUFFER, m_VBOID));
    GLCall(glBufferData(GL_ARRAY_BUFFER, size, data, GL_STATIC_DRAW));
}

VBO::~VBO()
{
    GLCall(glDeleteBuffers(1, &m_VBOID));
}

void VBO::Bind() const
{
    GLCall(glBindBuffer(GL_ARRAY_BUFFER, m_VBOID));
}

void VBO::Unbind() const
{
    GLCall(glBindBuffer(GL_ARRAY_BUFFER, 0));
}
```

- Note that if a buffer object that is currently bound is deleted, the binding reverts to 0 (the absence of any buffer object).

## Basic abstraction of the layout inside a VBO

`VBOLayout.h`:

```cpp
#ifndef VBOLAYOUT_H
#define VBOLAYOUT_H

#include <iostream>
#include <vector>
#include "DebugTools.h"

struct VBOAttribute
{
	unsigned int opengl_type;
	unsigned int components_count;
	unsigned char normalized;

	static unsigned int SizeOfOpenGLType(unsigned int opengl_type)
	{
		switch (opengl_type)
		{
		case GL_FLOAT:
			return 4;
		case GL_UNSIGNED_INT:
			return 4;
		case GL_UNSIGNED_BYTE:
			return 1;
		default:
			std::cout << "Error: SizeOfType(unsigned int type) - the provided type is currently not supported." << std::endl;
			ASSERT_DebugBreak_MSVC(false);
			return 0;
		}
	}
};

class VBOLayout
{
	unsigned int m_Stride = 0;
	std::vector<VBOAttribute> m_Attributes;

public:

	VBOLayout()
	{

	}

	const std::vector<VBOAttribute> GetAttributes() const
	{
		return m_Attributes;
	}

	unsigned int GetStride() const
	{
		return m_Stride;
	}

	template<typename T>
	void AddAttribute(unsigned int components_count)
	{
		//static_assert(false, "Adding data type that is not implemented for VBOLayout");
		// TODO: how to use static_assert here without always triggering compiler error?
		
		std::cout << "Error: Adding data type that is not implemented for VBOLayout" << std::endl;
		ASSERT_DebugBreak_MSVC(false);
	}

	template<>
	void AddAttribute<float>(unsigned int components_count)
	{
		m_Attributes.push_back({ GL_FLOAT, components_count, GL_FALSE });	// TODO: right now normalized is hard-coded to false
		m_Stride += VBOAttribute::SizeOfOpenGLType(GL_FLOAT) * components_count;
	}

	template<>
	void AddAttribute<unsigned int>(unsigned int components_count)
	{
		m_Attributes.push_back({ GL_UNSIGNED_INT, components_count, GL_FALSE });	// TODO: right now normalized is hard-coded to false
		m_Stride += VBOAttribute::SizeOfOpenGLType(GL_UNSIGNED_INT) * components_count;
	}

	template<>
	void AddAttribute<unsigned char>(unsigned int components_count)
	{
		m_Attributes.push_back({ GL_UNSIGNED_BYTE, components_count, GL_TRUE });	// TODO: right now normalized is hard-coded to true
		m_Stride += VBOAttribute::SizeOfOpenGLType(GL_UNSIGNED_BYTE) * components_count;
	}

};

#endif // !VBOLAYOUT_H
```

## Basic abstraction of index buffer

`IndexBuffer.h`:

```cpp
#ifndef INDEXBUFFER_H
#define INDEXBUFFER_H

class IndexBuffer
{
	unsigned int m_IBOID = 0;
	unsigned int m_IndicesCount = 0;

public:

	/*
	Currently we assume all index buffers contains data of type unsigned int
	*/
	IndexBuffer(const unsigned int* data, unsigned int count);	// count is the number of indices; bind after initialization
	
	~IndexBuffer();

	void Bind() const;
	
	void Unbind() const;

	unsigned int GetIndicesCount() const;
};

#endif // !INDEXBUFFER_H
```

`IndexBuffer.cpp`:

```cpp
#include "IndexBuffer.h"
#include "DebugTools.h"

IndexBuffer::IndexBuffer(const unsigned int* data, unsigned int count)	// count is the number of indices; bind after initialization
    : m_IndicesCount{ count }
{
    ASSERT_DebugBreak_MSVC(sizeof(unsigned int) == sizeof(GLuint));

    GLCall(glGenBuffers(1, &m_IBOID));
    GLCall(glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_IBOID));
    GLCall(glBufferData(GL_ELEMENT_ARRAY_BUFFER, m_IndicesCount * sizeof(unsigned int), data, GL_STATIC_DRAW));
}

IndexBuffer::~IndexBuffer()
{
    GLCall(glDeleteBuffers(1, &m_IBOID));
}

void IndexBuffer::Bind() const
{
    GLCall(glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, m_IBOID));
}

void IndexBuffer::Unbind() const
{
    GLCall(glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, 0));
}

unsigned int IndexBuffer::GetIndicesCount() const
{
    return m_IndicesCount;
}
```

## Basic abstraction of VAO

`VAO.h`:

```cpp
#ifndef VAO_H
#define VAO_H

#include "VBO.h"
#include "VBOLayout.h"

class VAO
{
	unsigned int m_VAOID;

public:

	VAO();
	
	~VAO();

	void Bind() const;

	void Unbind() const;

	// TODO: currently our VAO does not support multiple VBOs
	void LinkVertexBuffer(const VBO& vbo, const VBOLayout& layout);
};

#endif // !VAO_H
```

`VAO.cpp`:

```cpp
#include "VAO.h"
#include "DebugTools.h"

VAO::VAO()
{
	GLCall(glGenVertexArrays(1, &m_VAOID));
	//GLCall(glBindVertexArray(m_VAOID));
}

VAO::~VAO()
{
	GLCall(glDeleteVertexArrays(1, &m_VAOID));
}

void VAO::Bind() const
{
	GLCall(glBindVertexArray(m_VAOID));
}

void VAO::Unbind() const
{
	GLCall(glBindVertexArray(0));
}

void VAO::LinkVertexBuffer(const VBO& vbo, const VBOLayout& layout)
{
	Bind();

	vbo.Bind();
	
	const auto& attributes = layout.GetAttributes();

	unsigned int offset = 0;

	for (unsigned int i = 0; i < attributes.size(); i++)
	{
		const auto& attribute = attributes[i];
		GLCall(glEnableVertexAttribArray(i));
		GLCall(glVertexAttribPointer(i, attribute.components_count, attribute.opengl_type, attribute.normalized, layout.GetStride(), (const void*)offset));  // this line links the vbo to the vao at slot index i
		offset += attribute.components_count * VBOAttribute::SizeOfOpenGLType(attribute.opengl_type);
	}
}
```

## Basic abstraction of shader with Caching Uniform IDs on CPU

`Shader.h`:

```cpp
#ifndef SHADER_H
#define SHADER_H

#include <string>
#include <unordered_map>
#include "glm/glm.hpp"

struct ShaderProgramSourceCode
{
    std::string VertexShaderSourceCode;
    std::string FragmentShaderSourceCode;
};

class Shader
{
    std::string m_FilePath;
    unsigned int m_RendererID = 0;
    mutable std::unordered_map<std::string, int> m_UniformLocationCache;  // a member function marked as const is allowed to modify mutable data members

public:

    Shader(const std::string& filepath);

    ~Shader();

    void Bind() const;

    void Unbind() const;

    void SetUniform_1int(const std::string& u_name, int i1);

    void SetUniform_4floats(const std::string& u_name, float f1, float f2, float f3, float f4);

    void SetUniform_float_matrix_4_4(const std::string& u_name, glm::mat4 matrix);

private:

    ShaderProgramSourceCode ParseUnifiedShader(const std::string& filepath);

    unsigned int CompileShader(unsigned int type, const std::string& source_code);

    unsigned int CreateShaderProgram(const std::string& vertexShader, const std::string& fragmentShader);

    int GetUniformLocation(const std::string& u_name) const;
};

#endif // !SHADER_H
```

The code for `ParseUnifiedShader`, `CompileShader` and `CreateShaderProgram` is the same as what we have covered in previous sections.

`Shader.cpp`:

```cpp
#include "Shader.h"
#include "DebugTools.h"

#include <iostream>
#include <fstream>
#include <sstream>

Shader::Shader(const std::string& filepath)
	: m_FilePath{ filepath }
{
    ShaderProgramSourceCode shader_program_source_code = ParseUnifiedShader(m_FilePath);

    m_RendererID = CreateShaderProgram(shader_program_source_code.VertexShaderSourceCode, shader_program_source_code.FragmentShaderSourceCode);
}

Shader::~Shader()
{
    Unbind();	// Note that a program object is in use as part of current rendering state, it will be flagged for deletion, but it will not be deleted until it is no longer part of current state for any rendering context.
    GLCall(glDeleteProgram(m_RendererID));
}

void Shader::Bind() const
{
    GLCall(glUseProgram(m_RendererID));
}

void Shader::Unbind() const
{
    GLCall(glUseProgram(0));
}

int Shader::GetUniformLocation(const std::string& u_name) const
{
    if (m_UniformLocationCache.find(u_name) != m_UniformLocationCache.end())
    // the uniform's id is in our cache:
    {
        return m_UniformLocationCache[u_name];
    }
    // the uniform's id is not in our cache:
    GLCall(int u_id = glGetUniformLocation(m_RendererID, u_name.c_str()));
    //ASSERT_DebugBreak_MSVC(u_id != -1);
    if (u_id == -1)
    {
        std::cout << "Warning: uniform '" << u_name << "' doesn't exist in the shader program." << std::endl;
    }
    // cache the uniform's id before returning:
    m_UniformLocationCache[u_name] = u_id;
    // Note: here we assume that the existence of uniform will not change unless the shader is recompiled.
    return u_id;
}

void Shader::SetUniform_1int(const std::string& u_name, int i1)
{
    GLCall(glUniform1i(GetUniformLocation(u_name), i1));
}

void Shader::SetUniform_4floats(const std::string& u_name, float f1, float f2, float f3, float f4)
{
    GLCall(glUniform4f(GetUniformLocation(u_name), f1, f2, f3, f4));
}

void Shader::SetUniform_float_matrix_4_4(const std::string& u_name, glm::mat4 matrix)
{
    GLCall(glUniformMatrix4fv(GetUniformLocation(u_name), 1, GL_FALSE, &matrix[0][0]));
}

ShaderProgramSourceCode Shader::ParseUnifiedShader(const std::string& filepath)
{
    std::ifstream input_stream{ filepath };
    enum class ShaderType
    {
        NONE = -1,
        VERTEX = 0,
        FRAGMENT = 1
    };
    std::string current_line;
    std::stringstream shader_source_code_buffer[2];
    ShaderType current_shader_type = ShaderType::NONE;
    while (std::getline(input_stream, current_line))
    {
        if (current_line.find("#shader") != std::string::npos) // std::string::npos is often implemented as -1
        {
            if (current_line.find("vertex") != std::string::npos)
            {
                current_shader_type = ShaderType::VERTEX;
            }
            else if (current_line.find("fragment") != std::string::npos)
            {
                current_shader_type = ShaderType::FRAGMENT;
            }
            else
            {
                std::cout << "Syntax Error: unspecified shader type in unified shader file." << std::endl;
            }
        }
        else
        {
            shader_source_code_buffer[(int)current_shader_type] << current_line << '\n';
        }
    }
    return { shader_source_code_buffer[0].str(),shader_source_code_buffer[1].str() };
}

unsigned int Shader::CompileShader(unsigned int type, const std::string& source_code)
// returns the OpenGL ID of the shader object
{
    GLCall(unsigned int shader_id = glCreateShader(type));
    const char* const src = source_code.c_str();
    GLCall(glShaderSource(shader_id, 1, &src, nullptr));
    GLCall(glCompileShader(shader_id));

    /*** Error handling: ***/
    int compile_status;
    GLCall(glGetShaderiv(shader_id, GL_COMPILE_STATUS, &compile_status));
    if (compile_status == GL_FALSE)
    {
        int log_length;
        GLCall(glGetShaderiv(shader_id, GL_INFO_LOG_LENGTH, &log_length));
        char* log = (char*)alloca(log_length * sizeof(char));   // stack-allocated memory
        GLCall(glGetShaderInfoLog(shader_id, log_length, &log_length, log));
        std::cout << "Failed to compile ";
        switch (type)
        {
        case GL_VERTEX_SHADER:
            std::cout << "vertex shader" << std::endl;
            break;
        case GL_FRAGMENT_SHADER:
            std::cout << "fragment shader" << std::endl;
            break;
        default:
            break;
        }
        std::cout << log << std::endl;
        GLCall(glDeleteShader(shader_id));
        return 0;   // A value of 0 for shader will be silently ignored by further OpenGL calls.
    }
    /*** End of error handling ***/

    return shader_id;
}

unsigned int Shader::CreateShaderProgram(const std::string& vertexShader, const std::string& fragmentShader)
// returns the OpenGL ID of the shader program
{
    GLCall(unsigned int shader_program_id = glCreateProgram());

    unsigned int vertex_shader_id = CompileShader(GL_VERTEX_SHADER, vertexShader);
    unsigned int fragment_shader_id = CompileShader(GL_FRAGMENT_SHADER, fragmentShader);

    GLCall(glAttachShader(shader_program_id, vertex_shader_id));
    GLCall(glAttachShader(shader_program_id, fragment_shader_id));

    GLCall(glLinkProgram(shader_program_id));
    GLCall(glValidateProgram(shader_program_id));

    GLCall(glDeleteShader(vertex_shader_id));
    GLCall(glDeleteShader(fragment_shader_id));

    GLCall(glDetachShader(shader_program_id, vertex_shader_id));
    GLCall(glDetachShader(shader_program_id, fragment_shader_id));

    return shader_program_id;
}
```

- `std::unordered_map` is a container, implemented using hash table (❓ More on hash table are needed), that contains key-value pairs with unique keys.

  Here our keys are (of type) `std::string` and our values are (of type) `int`.

  In my current understanding, when we search for a key in a `std::unordered_map`, we cannot naively lookup what `m_UniformLocationCache[u_name]` returns because `m_UniformLocationCache[u_name]` will create a key using `u_name` and assign its associated value to `0` (and return a reference to the value) if `u_name` does not exist in the `std::unordered_map`.

  We can use `.find(u_name)` to search if the key `u_name` exists. If not, `.find(u_name)` will return `.end()` (❓ More on `.begin()` and `.end()` are needed).

  If not found, it means that the Id of the uniform we are looking for hasn't been stored in our cache yet. We then use `m_UniformLocationCache[u_name] = u_id;` to create the key and assign it with the value.

  Note that we choose to store key-value pairs for uniforms which aren't actually exist on the GPU (e.g. if the `u_name` provided is out of nowhere, or if the uniform has been optimized away).

  ❓ Performance comparison between `std::unordered_map` and `std::vector<std::pair>`.

- For `glUniformMatrix4fv`:

  - The `v` in an OpenGL function stands for "vector". In this context, "vector" doesn't mean a mathematical or geometric vector; instead, it indicates that the function can take an array of values as input.
  - `1` means that the uniform only contains 1 such matrix.
  - `GL_FALSE` there refers to whether the matrix (data) needs to be transposed. OpenGL reads matrix in the column-major order. If the data provided has layout of row-major order, the action of transpose will fit it with the reading order of OpenGL.
  - The last parameter should be provided with a pointer to the first `float` of the data array.

- `glUniform1i` sets the value of the uniform with the provided one `int`. It is worth noting that, in our case, this is used to set the value for `uniform sampler2D u_Texture;` in the fragment shader. `uniform sampler2D` takes an `int` corresponding to the texture unit the 2D sampler will sample from.

## Basic abstraction of renderer

`Renderer.h`:

```cpp
#ifndef RENDERER_H
#define RENDERER_H

#include "VAO.h"
#include "IndexBuffer.h"
#include "Shader.h"

class Renderer
{

public:

    void Clear() const;  // Set the back buffer to black

    void Draw(const VAO& vao, const IndexBuffer& index_buffer, const Shader& shader_program) const;
};

#endif // !RENDERER_H
```

`Renderer.cpp`:

```cpp
#include "Renderer.h"

#include <iostream>
#include "DebugTools.h"

void Renderer::Clear() const
{
    GLCall(glClearColor(0.0f, 0.0f, 0.0f, 1.0f));
    GLCall(glClear(GL_COLOR_BUFFER_BIT));
}

void Renderer::Draw(const VAO& vao, const IndexBuffer& index_buffer, const Shader& shader_program) const
{
    shader_program.Bind();
    vao.Bind();
    index_buffer.Bind();    // this isn't necessary as long as index_buffer has bound to vao somewhere (and without unbind) before this call

    /*
    Currently:
        - we assume all index buffers contains data of type unsigned int
        - we assume the only primitive we are drawing is triangle
    */
    GLCall(glDrawElements(GL_TRIANGLES, index_buffer.GetIndicesCount(), GL_UNSIGNED_INT, nullptr)); 
}
```

## Add the `stb` library into the project

We will be using the `stb` library to load PNG files into our project.

Go to the [stb repository](https://github.com/nothings/stb). Find the [stb_image.h file](https://github.com/nothings/stb/blob/master/stb_image.h). Click `Raw`. Then, use `CTRL + A` and then `CTRL + C` to copy all the source code. Next, add a new header file in your project, delete everything in that file (so that it is an empty file), and paste the source code into the header file (maybe name this header file as `stb_image.h`). Now, create a new `.cpp` flie with the content as follows:

```cpp
#define STB_IMAGE_IMPLEMENTATION
#include "stb_image.h"
```

Then, hit `CTRL + F7` to compile. Your project should now contain the `stb` library.

## Texture in OpenGL

Texture is represented by texture object on GPU.

In my current understanding, the GPU implements OpenGL with many slots (called texture units) to link with texture objects.

On my laptop, there are 32 texture units available.

In my current understanding, there are many targets on each texture unit. A texture object is linked to a texture unit by linking to one of the targets on the texture unit. The first time a texture object binds to a target defines the type of the texture object, and this type cannot change in the following lifetime of this texture object. It is also my current understanding that, on the other hand, the same texture object can be simultaneously bound to multiple texture units, as long as it's bound to the same type of target for each texture unit.

There is an active texture unit for the current rendering state, and the default active texture unit is `0`.

Note that there is also a default texture object. When calling `glBindTexture(GL_TEXTURE_2D, 0)`, it actually associates the default texture object with (the `GL_TEXTURE_2D` target on) the active texture unit.

❓ I think the default texture object (with ID `0`) is a diffent kind of texture object (compared to those texture objects created by `glGenTextures`) in that it can link to different type of targets. I'm not sure if it is linked to all the targets on all texture units that were bound with it (and without binding to other texture objects, yet) simultaneously, or if binding it to target A then binding it to target B will leave target A unbinded to any texture object.

Texture object stores states representing the texture parameters associated with it. Note that those states are NOT stored in texture units. (possible [reference](https://computergraphics.stackexchange.com/a/7846))

## Basic abstraction of texture

`Texture.h`:

```cpp
#ifndef TEXTURE_H
#define TEXTURE_H

#include <string>

class Texture
{
	unsigned int m_TextureObjectID = 0;
	std::string m_FilePath;
	unsigned char* m_CPUBuffer = nullptr;
	int m_Width = 0;
	int m_Height = 0;
	int m_BytesPerPixel = 0;

public:

	Texture(const std::string& file_path);
	
	~Texture();

	void Bind(unsigned int slot = 0) const;

	void Unbind() const;

	int GetWidth() const
	{
		return m_Width;
	}

	int GetHeight() const
	{
		return m_Height;
	}
};


#endif // !TEXTURE_H
```

`Texture.cpp`:

```cpp
#include "Texture.h"
#include "DebugTools.h"
#include "stb_image.h"

#include <iostream>

Texture::Texture(const std::string& file_path)
	: m_FilePath{ file_path }
{
	stbi_set_flip_vertically_on_load(1);	// this is due to that we are loading PNG file (origin at top left) and OpenGL texture's origin is at bottom left

	m_CPUBuffer = stbi_load(m_FilePath.c_str(), &m_Width, &m_Height, &m_BytesPerPixel, 4);	// 4 for rgba

	GLCall(glGenTextures(1, &m_TextureObjectID));
	// Here we associate the texture to the target GL_TEXTURE_2D of the default texture unit (slot): 0
	GLCall(glBindTexture(GL_TEXTURE_2D, m_TextureObjectID));
	// Set up the necessary settings for the bound texture (those are states of the texture object, not the texture unit):
	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR));
	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR));
	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE));	// S is the X (horizontal) for texture
	GLCall(glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE));	// T is the Y (vertical) for texture
	// Send the texture data from CPU to GPU:
	GLCall(glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, m_Width, m_Height, 0, GL_RGBA, GL_UNSIGNED_BYTE, m_CPUBuffer));

	GLCall(glBindTexture(GL_TEXTURE_2D, 0));	// After this line, target GL_TEXTURE_2D on texture unit 0 is associated with the default texture object with ID 0.

	// If we don't want to retain a copy of the pixel data of the texture:
	if (m_CPUBuffer)
	{
		stbi_image_free(m_CPUBuffer);
		m_CPUBuffer = nullptr;	// m_CPUBuffer != nullptr after the above line.
	}
}

Texture::~Texture()
{
	GLCall(glDeleteTextures(1, &m_TextureObjectID));	// relese the memory storing the texture data on the GPU
}

void Texture::Bind(unsigned int slot /* = 0 */) const
// Associate the GL_TEXTURE_2D target of the provided texture unit (slot) to the texture object represented by this Texture
{
	GLCall(glActiveTexture(GL_TEXTURE0 + slot));
	GLCall(glBindTexture(GL_TEXTURE_2D, m_TextureObjectID));
}

void Texture::Unbind() const
// Associate the GL_TEXTURE_2D target of the active texture unit to the default texture object with ID 0
{
	GLCall(glBindTexture(GL_TEXTURE_2D, 0));
}
```

- `stbi_load` returns a pointer to an unsigned char being the head of an array storing the pixels data channel by channel.

  - `stbi_load` takes in a C-style characters array and gives back the width and the height of the PNG texture (in number of pixels, not in `unsigned char`!) as well as how many bytes is used to represent a pixel by the PNG file. The last argument `4` indicates how many channels we want, here we want exactly RGBA.
 
    ❓ When we provide the last argument as `1` or `2`, the 1st component will be "grey" according to the document in [stb_image.h source code](https://github.com/nothings/stb/blob/master/stb_image.h). What does this "grey" means?
  
  - When setting `stbi_set_flip_vertically_on_load(1);`, the array is arranged in a bottom-to-top line-by-line style with the head element being the 1st channel of the bottom-left pixel of the PNG texture.

    Otherwise (the default case), the array is arranged in a top-to-bottom line-by-line style with the head element being the 1st channel of the top-left pixel of the PNG texture.

- `glGenTextures(1, &m_TextureObjectID)` generates 1 texture object on GPU and assigns its ID to the `unsigned int` variable `m_TextureObjectID`.

- In my current understanding, each texture unit holds multiple targets. `glBindTexture(GL_TEXTURE_2D, m_TextureObjectID)` links the texture object referred by `m_TextureObjectID` to the `GL_TEXTURE_2D` target on the currently active texture unit. Each target can only link to a single texture object, but each texture unit can link to multiple texture objects, each links to a different targets on the texture unit.

  It is also my current understanding that a single texture object cannot be bound to multiple targets in OpenGL. When you create a texture with `glGenTextures` and then bind it for the first time with `glBindTexture`, the texture object gets its target type, and this target type is fixed for the lifetime of the texture object. This means that if you first bind a texture object to `GL_TEXTURE_2D`, it will always be a `GL_TEXTURE_2D` texture, and attempting to bind it to a different target (like `GL_TEXTURE_3D`) will result in an error.

- In my current understanding, for the above uses of `glTexParameteri`:

  - `i` in `glTexParameteri` means the (value of the) parameter we are setting is of type `GLint`.

  - `GL_TEXTURE_2D` tells OpenGL that this `glTexParameteri` function is modifying the texture parameter on the texture object linked to the `GL_TEXTURE_2D` target on the currently active texture unit.
 
  - `GL_TEXTURE_MIN_FILTER` specifies which parameter we are setting. In this case, `GL_TEXTURE_MIN_FILTER` controls how the texture will be resampled down if the texture needs to be rendered in a lower resolution (i.e. with a smaller number of pixels).

    `GL_TEXTURE_MAG_FILTER` controls how the texture will be resampled up if the texture needs to be rendered in a higher resolution (i.e. with a larger number of pixels).

    ❓ Explore the cases where resampling up and down happens simultaneously.
 
  - `GL_LINEAR`, in my current understanding, means scaling will be done linearly (i.e. in a linearly interpolation manner). ❓ Currently I don't think this understanding is correct, see later section on filtering.
 
    ❓ Explore the other options.
 
  - `GL_TEXTURE_WRAP_S` controls how to sample if the sampling point is outside of the texture in horizontal direction (x-axis).
 
    `GL_TEXTURE_WRAP_T` controls how to sample if the sampling point is outside of the texture in vertical direction (y-axis).

  - `GL_CLAMP_TO_EDGE` means taking the nearest (and thus, on edge of the texture) row (for T) / column (for S).
 
    On the other hand, `GL_REPEAT` is like taking a $mod$ operation along row (for S) / column (for T).

- Note that we should always set up those texture parameters before rendering the texture.

- `glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA8, m_Width, m_Height, 0, GL_RGBA, GL_UNSIGNED_BYTE, m_CPUBuffer)`

  - In my current understanding, the 1st parameter tells OpenGL what to do with the interpretation of the data in the first place. ❓ Understand the cases when we are not using `GL_TEXTURE_2D` here.
  - `m_CPUBuffer` provides the data. ❓ If this is provided as `nullptr`, how to provide the data in a later stage?
  - `GL_UNSIGNED_BYTE` tells how to interpret the data being sent (i.e. the data in `m_CPUBuffer`).
  - `GL_RGBA` specifies how those unsigned bytes in `m_CPUBuffer` are grouped (here 4 unsigned bytes for each group).
  - `GL_RGBA8` tells OpenGL how to store the sent data on GPU.
  - The 1st `0` means this is not a multi-level texture. ❓ What is a multi-level texture?
  - For more details, you may see the [gl docs](https://docs.gl/gl4/glTexImage2D).
  - ❓ The gl docs says `border` (the 6th parameter) must be `0`. Why?

- `glDeleteTextures` deletes `n` (here is `1`) texture(s) whose IDs are specified in the array (here provided by `&m_TextureObjectID`).

- The currently active texture unit can be changed using `glActiveTexture`. Note that this function receives a GL macro, NOT an integer literal!

## Basic Blending in OpenGL

In my current understanding, blending specifies the mathematical way of how to combine the color we output from the fragment shader with the color that is already in the frame buffer the fragment shader is drawing to.

The blending function, defined in OpenGL by `glBlendFunc`, is in the following form:

$$
\vec{c}_b = f_s \vec{c}_s \cdot f_d \vec{c}_d
$$

where $f$ denotes "RGBA factor", $\vec{c}$ is a 4-components vector representing the color data in RGBA format, $s$ means the color we output from the fragment shader, $d$ means the color that is already in the frame buffer the fragment shader is drawing to, $b$ means the color in the frame buffer after this blending, and the operator $\cdot$ is an operation that can be user-defined by `glBlendEquation`.

By default, OpenGL disables blending. The following code shows how to enable blending and an example setup for blending:

```cpp
glEnable(GL_BLEND);
glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
glBlendEquation(GL_FUNC_ADD);

// rendering...

glDisable(GL_BLEND);
```

- `GL_SRC_ALPHA` sets $f_s$ to the $A$-component (of the RGBA format) of $\vec{c}_s$.

- `GL_ONE_MINUS_SRC_ALPHA` set $f_d$ to $1-A$ where $A$ is the $A$-component (of the RGBA format) of $\vec{c}_s$.

- `GL_FUNC_ADD` sets the operator $\cdot$ to $+$ (i.e. normal addition).

- It's good to disable blending (as we did with `glDisable(GL_BLEND);`) when it's no longer needed, as leaving it enabled when drawing opaque objects can unnecessarily hurt performance.

❓ Say, the output color has alpha being `0.5` and the color in the frame buffer has alpha being `1`. Such blending will result in the final color in the frame buffer to have alpha being `0.75`. Does this matter (since it seems that we never use alpha in the frame buffer for this kind of blending)? If so, what does this imply?

❓ If we treat alpha as transparency/translucency, how physically is this way of blending?

❓ How to control which outputting source and destination buffer the blending is setting for?

Note that disabling blending (so that the fragment shader will simply be overfwriting the target frame buffer) is effectively the same as enabling blending with the following settings:

```cpp
glEnable(GL_BLEND);
glBlendFunc(GL_ONE, GL_ZERO);
glBlendEquation(GL_FUNC_ADD);
```

## Adding `glm` math library into the project

`glm` is another header-only library, which is wonderful.

Go to the [repository](https://github.com/g-truc/glm), click `Releases`, download the `.zip` file in the `Assets` section of the latest version (I believe I am using `0.9.9.8` here). This `.zip` shall only have one folder (named `glm`). Inside this `glm` folder, there is another `glm` folder. Copy this sub `glm` folder into your project. Under the `Show All Files` view of the `Solution Exploer` of Visual Studio, right-click on the `glm` folder we just pasted in, and then click `Include In Project`. (For glm versions older than the version I am using, there may be a `dummy.cpp` file which contains an `int main()` function which will interfere with your own main function, and thus should be `Exclude From Project`. But I believe at least from the version I am using, `glm` does not contain this `dummy.cpp` anymore)

Now, make sure that in `Properties -> Configuration Properties -> C/C++ -> General -> Additional Include Directories` of your C++ project in Visual Studio, you have included the folder you pasted the `glm` folder in. This is because the glm library's source code uses paths relative to the directory containing this `glm` folder.

Once all these are done, your project should compile.

Normally in a C++ OpenGL project, we want to `#include "glm/glm.hpp"` and `#include "glm/gtc/matrix_transform.hpp"` in files where we use the glm library.

- Note that the GLM library is a C++ mathematics library designed to mirror GLSL's syntax and functionalities to ease the development of OpenGL applications in C++. Shader code in OpenGL uses GLSL (OpenGL Shading Language), which is a separate language from C++. So, even if we are not including the glm library, we can still use the identifiers like `vec2`, `vec3`, `vec4`, `mat4`, etc. in our shaders.

- Note that, in `glm` (as in OpenGL), matrices are [column-majored](https://en.wikipedia.org/wiki/Row-_and_column-major_order).

  Thus, we don't need to transpose the matrix memory layout when sending the matrices created by glm library as uniforms to OpenGL shaders.

## Space transformations in OpenGL

This is a perfect example of how OpenGL makes a simple concept f***ing complicated.

- In my current understanding (possible references: [1](https://stackoverflow.com/a/47857256), [2](https://stackoverflow.com/a/72336611)), by default (❓ TODO: how to change default setting?):

  - Object space (the space before model transformation), world space, and view space, in OpenGL, are represented in right-handed coordinates (i.e. y-axis up, x-axis towards right, z-axis outwards).

  - All the spaces after transformed by projection matrix, in OpenGL, are represented in left-handed coordinates (i.e. y-axis up, x-axis towards right, z-axis inwards).

  - Projection matrices created by `glm` do this coordinates flipping for us.

  - The parameters representing near and far planes in function `glm::perspective` are expecting absolute distance from the camera to the plane in front of the camera.

    Note in particular that in GLM version `0.9.9.8` (and possibly any further version),  `glm::perspective` expects `fovy` in <ins>radians</ins>.

  - The parameters representing near and far planes in function `glm::ortho` are expecting values on z-axis in a left-handed coordinates, with `near < far`.

## Multiple draw calls

I think simply call the draw function multiple times will let OpenGL run the pipeline on GPU multiple times (though, this may not be optimal for performance in many cases).

E.g.

```cpp
// Inside the render loop:

shader.Bind();
shader.SetUniform_4floats("u_Color", r, 0.3f, 0.8f, 1.0f);

// model matrix is placing inside the render loop because the location of the object we are rendering can change over frames.
{
    glm::mat4 model_matrix = glm::translate(glm::mat4{ 1.0f }, modelA_world_coordinates);
    glm::mat4 mvp_matrix = projection_matrix * view_matrix * model_matrix;
    shader.SetUniform_float_matrix_4_4("u_MVP", mvp_matrix);
    renderer.Draw(vao, index_buffer, shader);

}
// Note that model B will be on top of model A because the order we draw to the frame buffer.
{
    glm::mat4 model_matrix = glm::translate(glm::mat4{ 1.0f }, modelB_world_coordinates);
    glm::mat4 mvp_matrix = projection_matrix * view_matrix * model_matrix;
    shader.SetUniform_float_matrix_4_4("u_MVP", mvp_matrix);
    renderer.Draw(vao, index_buffer, shader);
}
```

## Integrating `ImGui`

Currently this section will only show how to get ImGui working within a C++ OpenGL project. <ins>I need a much more rigorous learning on ImGui before I can understand what are actually happening underneath ImGui.</ins>

The ImGui version that I am integrating is `1.90.1`. I will probably stick to it and not moving to an updated version unless there is any specific benefit.

First, go to the [repository](https://github.com/ocornut/imgui), click `Releases`. Under the `Assets` section, download `Source Code (zip)`.

Maybe create a folder named `imgui` in your project, copy and paste all the `.h` and `.cpp` files listed in the `imgui-1.90.1` folder (which should be the only folder in `Source Code (zip)`).

In the `imgui-1.90.1` folder, there is a folder named `backends`, open that, there should be 5 files named `imgui_impl_glfw.h`, `imgui_impl_glfw.cpp`, `imgui_impl_opengl3.h`, `imgui_impl_opengl3.cpp`, and `imgui_impl_opengl3_loader.h` respectively. Copy these 5 files into your `imgui` folder as well.

In the `imgui-1.90.1` folder, there is a folder named `examples`, in that, open the folder named `example_glfw_opengl3`. Copy the `main.cpp` file in that folder into your `imgui` folder. MAKE SURE that you exclude this flie from your project. This file just serves as an example so that we can copy and paste the scaffolding source code into our files.

Now, we should have all the files we need for integrating ImGui.

Right-click your `imgui` folder under the `Show All Files` view in the `Solution Explorer` in the Visual Studio. Select `Include In Project`. Then, right-click on the `main.cpp` in your `imgui` folder and select `Exclude From Project`.

In your main file, the scaffolding looks something like this:

```cpp
// include necessary stuffs... (e.g. glew and glfw)

#include "imgui/imgui.h"
#include "imgui/imgui_impl_glfw.h"
#include "imgui/imgui_impl_opengl3.h"

// maybe other inclusions...

int main()
{

    // initialize glfw and other stuffs...

    {
        ImGui::CreateContext();
        ImGui_ImplGlfw_InitForOpenGL(window, true);  // GLFWwindow* window
        ImGui::StyleColorsDark();
        const char* glsl_version = "#version 130";
        ImGui_ImplOpenGL3_Init(glsl_version);

        // Other initial setups...

        while (!glfwWindowShouldClose(window))
        {
            // Maybe clear color buffer here...

            ImGui_ImplOpenGL3_NewFrame();
            ImGui_ImplGlfw_NewFrame();
            ImGui::NewFrame();

            // Other rendering mechanisms...

            ImGui::Render();
            ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());

            // swap buffers and poll events...
        }

        // Any heap clean-up...
    }

    ImGui_ImplOpenGL3_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImGui::DestroyContext();

    // terminate glfw...
}
```

- In my current understanding: (may need to be more accurate)

  - The operating system continuously receives data from devices connected to the computer.
 
  - The operating system continuously sending data relevant to the window of our application. This communication is set up during the initialization of glfw in our application.
 
  - There are mainly two types of data: continuous states (e.g. the position of the mouse) and events (e.g. mouse click). Those "discrete" actions that are categorized as events, when happen, are sent by the operating system into a events queue - a data structure managed by glfw. The former, on the hand, can be queried from the operating system through functions provided by glfw.
 
  - `glfwPollEvents` takes all the events in the events queue (probably clear the events queue) and calls the corresponding callback functions which have been set up for glfw. Each callback function assocates with a kind of event.
 
  - When initialize ImGui with glfw, ImGui can set up callback functions. Thus, when `glfwPollEvents` calls callback functions, ImGui can be informed with the events data.
 
  - The NewFrame functions called at the beginning of the rendering loop query glfw for continuous data.
 
  - Now, ImGui should have all the current state information for the new frame.
 
  - Subsequent calls of ImGui widgets functions will result in ImGui drawing the widgets in accordance with the current states.

❓ Understand why the ImGui window isn't showed up on the canvas at the end of the first iteration of the render loop.

❓ In the above code we supply `glsl_version` as `#version 130`. Here it also seems to be fine to supply `#version 460`. What's the relationship between the version number supplied here, the version number supplied in `glfwWindowHint` and the version number specified in shaders?

## Separating Feature implementations into Tests

`TestBase.h`:

```cpp
#ifndef TESTBASE_H
#define TESTBASE_H

#include <string>

namespace Test
{

	class TestBase
	{
		std::string m_test_name = "Test";

	protected:

		TestBase(const std::string& test_name)
		{
			m_test_name = test_name;
		}

	public:

		TestBase() = delete;	// we don't want the user to directly construct TestBase

		virtual ~TestBase()
		{

		}

		virtual void OnUpdate(float dt)
		{

		}

		virtual void OnRender()
		{

		}

		virtual void OnImGuiRender()
		{

		}

		std::string GetTestName()	// TODO: should this be virtual?
		{
			return m_test_name;
		}

	};

}

#endif // !TESTBASE_H
```

## Basic Menu System for Tests

`TestMenu.h`:

```cpp
#ifndef TESTMENU_H
#define TESTMENU_H

#include "TestBase.h"
#include <vector>
#include <functional>
#include <string>

namespace Test
{

	class TestMenu : public TestBase
	{
		std::vector< std::pair< std::string, std::function< TestBase* () > > > m_Tests;
		TestBase*& m_CurrentTest;  // we want to directly modify an external pointer

	public:

		TestMenu(TestBase*& currentTestPointer);

		// We currently aren't overriding the virtual destructor because all data members are on stack

		void OnImGuiRender() override;

		template <typename T>	// T must be a kind of TestBase
		void AddTest(const std::string& test_name)
		{
			// Note that we captured test_name by value: the parameter dies when AddTest returns
			m_Tests.push_back(std::make_pair(test_name, [test_name]() { return new T{ test_name }; }));  // Note that we are NOT creating the test here!
			// TODO: is this using templated copy constructor or is this using any sort of move semantic?
			// TODO: how many mandatory copy elision happens here?
		}

	};
}

#endif // !TESTMENU_H
```

`TestMenu.cpp`:

```cpp
#include "TestMenu.h"
#include "imgui/imgui.h"

namespace Test
{
	TestMenu::TestMenu(TestBase*& currentTestPointer)
		: m_CurrentTest{ currentTestPointer },
		TestBase("Tests Menu")
	{

	}

	void TestMenu::OnImGuiRender()
	{
		bool test_created = false;	// to avoid potential memory leak from creating multiple tests in one frame
		for (const auto& test : m_Tests)
		{
			if (ImGui::Button(test.first.c_str()) && (!test_created))
			/*
			Note that we cannot have two tests with the same name:
			in that case, the return value of the ImGui::Button will reveal the state for the first button with the name (although that ImGui::Button draws for the second botton)
			in which I think this reveals a sloppy implementation of ImGui...
			*/
			{
				m_CurrentTest = test.second();
				test_created = true;
			}
		}
	}
	
}
```

Now, our complete `App.cpp` scaffolding looks like this:

```cpp
// Standard:
#include <iostream>

// Externel:
#include <GL/glew.h>        // include this before include gl.h
#include <GLFW/glfw3.h>
#include "imgui/imgui.h"
#include "imgui/imgui_impl_glfw.h"
#include "imgui/imgui_impl_opengl3.h"

// Internel:
#include "tests/TestMenu.h"
#include "tests/TestClearColor.h"
#include "tests/TestTexture2D.h"

int main(void)
{
    /* Initialize the library */
    if (!glfwInit())
    {
        std::cout << "Error: glfwInit() failed to initialize" << std::endl;
        return -1;
    }

    glfwWindowHint(GLFW_CONTEXT_VERSION_MAJOR, 4);
    glfwWindowHint(GLFW_CONTEXT_VERSION_MINOR, 6);
    glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_CORE_PROFILE);
    //glfwWindowHint(GLFW_OPENGL_PROFILE, GLFW_OPENGL_COMPAT_PROFILE);  // TODO: why context version not working with compatibility mode?
        
    /* Create a windowed mode window and its OpenGL context */
    GLFWwindow* window = glfwCreateWindow(640, 480, "Learning OpenGL", NULL, NULL);

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
        //return -1;
        // TODO: should we return -1 here?
    }

    /* V-Sync */
    glfwSwapInterval(1);

    //---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

    GLCall(std::cout << "OpenGL version: " << glGetString(GL_VERSION) << '\n' << std::endl);

    /*GLint maxVertexAttribs;
    glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &maxVertexAttribs);
    std::cout << maxVertexAttribs << std::endl;*/

    //---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
    
    {
        Renderer global_renderer;

        // ImGui Initialization:
        ImGui::CreateContext();
        ImGui_ImplGlfw_InitForOpenGL(window, true);
        ImGui::StyleColorsDark();
        const char* glsl_version = "#version 130";  // pointing directly to the read-only section of the memory where the compiler allocates storage for string literals
        ImGui_ImplOpenGL3_Init(glsl_version);

        Test::TestBase* current_test = nullptr;
        Test::TestMenu tests_menu{ current_test };
        current_test = &tests_menu;

        tests_menu.AddTest<Test::TestClearColor>("Background Color");
        tests_menu.AddTest<Test::TestTexture2D>("2D Texture");

        /* Loop until the user closes the window */
        while (!glfwWindowShouldClose(window))
        {
            /* Render here */

            global_renderer.Clear();

            // ImGui refresh:
            ImGui_ImplOpenGL3_NewFrame();
            ImGui_ImplGlfw_NewFrame();
            ImGui::NewFrame();

            if (current_test)   // TODO: do we need to test for nullptr here?
            {
                current_test->OnUpdate(0.0f);
                current_test->OnRender();

                ImGui::Begin(current_test->GetTestName().c_str());
                
                if ((current_test != (&tests_menu)) && (ImGui::Button("<-")))   // Note that we need to put ImGui::Button within the scope between ImGui::Begin and ImGui::End
                {
                    delete current_test;
                    current_test = &tests_menu;
                }
                current_test->OnImGuiRender();
                
                ImGui::End();

                
            }

            ImGui::Render();
            ImGui_ImplOpenGL3_RenderDrawData(ImGui::GetDrawData());
            
            /* Swap front and back buffers */
            glfwSwapBuffers(window);

            /* Poll for and process events */
            glfwPollEvents();  // should we put this at the beginning of the rendering loop?
            // TODO: understand why the ImGui window isn't showed up on the canvas at this point in the first iteration of the render loop.
        }

        if (current_test != (&tests_menu))
        {
            std::cout << "Closed during testing, deleting the current test..." << std::endl;
            delete current_test;
        }

    }

    ImGui_ImplOpenGL3_Shutdown();
    ImGui_ImplGlfw_Shutdown();
    ImGui::DestroyContext();

    glfwTerminate();
    return 0;
}
```

where the specific tests will be presented in the following sections.

- An observation: the name provided to `ImGui::Begin` is remembered by ImGui and ImGui associates a specific panel in which the name is used as the key to invoke that panel. For example, each panel has its own position recorded by ImGui.

## A Basic Background Color Test

`TestClearColor.h`:

```cpp
#ifndef TESTCLEARCOLOR_H
#define TESTCLEARCOLOR_H

#include "TestBase.h"

namespace Test
{
	class TestClearColor : public TestBase
	{
	public:
		TestClearColor(const std::string& test_name);
		~TestClearColor() override;

		void OnUpdate(float dt) override;
		void OnRender() override;
		void OnImGuiRender() override;

	private:
		float m_BackgroundColor[4];
	};

	
}

#endif // !TESTCLEARCOLOR_H
```

`TestClearColor.cpp`:

```cpp
#include "TestClearColor.h"
#include "DebugTools.h"
#include "imgui/imgui.h"

namespace Test
{
	TestClearColor::TestClearColor(const std::string& test_name)
		: m_BackgroundColor{ 0.2f,0.3f,0.8f,1.0f },
		TestBase(test_name)
	{

	}

	TestClearColor::~TestClearColor()
	{

	}

	void TestClearColor::OnUpdate(float dt)
	{

	}
	void TestClearColor::OnRender()
	{
		GLCall(glClearColor(m_BackgroundColor[0], m_BackgroundColor[1], m_BackgroundColor[2], m_BackgroundColor[3]));
		GLCall(glClear(GL_COLOR_BUFFER_BIT));
	}

	void TestClearColor::OnImGuiRender()
	{
		ImGui::ColorEdit4("Background Color", m_BackgroundColor);
	}
}
```

## A Basic Texture Test

`TestTexture2D.h`:

```cpp
#ifndef TESTTEXTURE2D_H
#define TESTTEXTURE2D_H

#include "TestBase.h"
#include "glm/glm.hpp"
#include "glm/gtc/matrix_transform.hpp"
#include "VAO.h"
#include "IndexBuffer.h"
#include "Shader.h"
#include "Texture.h"
#include "Renderer.h"

#include <memory>

namespace Test
{
	class TestTexture2D : public TestBase
	{
		Renderer m_renderer;
		glm::vec3 m_modelA_world_coordinates;
		glm::vec3 m_modelB_world_coordinates;
		glm::mat4 m_model_matrix{1};
		glm::mat4 m_view_matrix;
		glm::mat4 m_projection_matrix;
		glm::mat4 m_mvp_matrix{1};
		std::unique_ptr<VBO> m_vbo;
		std::unique_ptr<VAO> m_vao;
		std::unique_ptr<IndexBuffer> m_index_buffer;
		std::unique_ptr<Shader> m_shader;
		std::unique_ptr<Texture> m_Texture;

	public:
		TestTexture2D();
		~TestTexture2D() override;

		void OnUpdate(float dt) override;
		void OnRender() override;
		void OnImGuiRender() override;
	};
}

#endif // !TESTTEXTURE2D_H
```

`TestTexture2D.cpp`:

```cpp
#include "TestTexture2D.h"
#include "DebugTools.h"
#include "imgui/imgui.h"

namespace Test
{
	TestTexture2D::TestTexture2D()
        : m_modelA_world_coordinates{ 320.0f,240.0f,0.0f }, m_modelB_world_coordinates{ 120.0f,40.0f,0.0f }
        TestBase(test_name)
	{
        GLCall(glEnable(GL_BLEND));
        GLCall(glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA));
        GLCall(glBlendEquation(GL_FUNC_ADD));

        float vertices[] =
        {
            //  position:         texCoords:
                -50.0f,-50.0f,    0.0f,0.0f,    // vertex 1
                 50.0f,-50.0f,    1.0f,0.0f,    // vertex 2
                 50.0f, 50.0f,    1.0f,1.0f,    // vertex 3
                -50.0f, 50.0f,    0.0f,1.0f     // vertex 4
        };

        unsigned int indices[] =    // OpenGL requires index buffer to store unsigned data!!!
        {
            0,1,2,  // lower-half triangle for the rectangle
            0,2,3   // upper-half triangle for the rectangle
        };

        m_vao = std::make_unique<VAO>();

        m_vbo = std::make_unique<VBO>(vertices, 4 * 4 * sizeof(float));

        VBOLayout vbo_layout;
        // position:
        vbo_layout.AddAttribute<float>(2);
        // texture coordinates:
        vbo_layout.AddAttribute<float>(2);

        m_vao->LinkVertexBuffer(*m_vbo, vbo_layout);

        m_index_buffer = std::make_unique<IndexBuffer>(indices, 6);

        // view and projection matrix is effectively placing outside the render loop because the location of our camera and what it sees stay unchange for all frames.
        m_view_matrix = glm::translate(glm::mat4{ 1.0f }, glm::vec3{ -320.0f,-240.0f,0.0f });
        m_projection_matrix = glm::ortho(-320.0f, 320.0f, -240.0f, 240.0f, -100.0f, 100.0f); // aspect ratio = 4:3, as we are rendering 640 by 480

        m_shader = std::make_unique<Shader>("res/shaders/Basic.shader");

        m_Texture = std::make_unique<Texture>("res/textures/brush.png");
        m_Texture->Bind(0);

        m_shader->Bind();
        m_shader->SetUniform_1int("u_Texture", 0); // uniform sampler2D is 1 int corresponding to the texture unit (slot)

        // Careful: if we write m_Texture.Unbind(); here, u_Texture will be linked to the default texture object with ID 0!
	}

	TestTexture2D::~TestTexture2D()
	{
	    /*
	    Remember to disable blending with glDisable(GL_BLEND) when it's no longer needed, as leaving it enabled when drawing opaque objects
	    can unnecessarily hurt performance.
	    */
	    GLCall(glDisable(GL_BLEND));
	}

	void TestTexture2D::OnUpdate(float dt)
	{

	}

	void TestTexture2D::OnRender()
	{
        // model matrix is effectively placing inside the render loop because the location of the object we are rendering can change over frames.
        {
            m_model_matrix = glm::translate(glm::mat4{ 1.0f }, m_modelA_world_coordinates);
            m_mvp_matrix = m_projection_matrix * m_view_matrix * m_model_matrix;
            m_shader->SetUniform_float_matrix_4_4("u_MVP", m_mvp_matrix);
            m_renderer.Draw(*m_vao, *m_index_buffer, *m_shader);

        }
        // Note that model B will be on top of model A because the order we draw to the frame buffer.
        {
            m_model_matrix = glm::translate(glm::mat4{ 1.0f }, m_modelB_world_coordinates);
            m_mvp_matrix = m_projection_matrix * m_view_matrix * m_model_matrix;
            m_shader->SetUniform_float_matrix_4_4("u_MVP", m_mvp_matrix);
            m_renderer.Draw(*m_vao, *m_index_buffer, *m_shader);
        }
	}

	void TestTexture2D::OnImGuiRender()
	{
        ImGui::SliderFloat3("Model A's World Coordinates", &m_modelA_world_coordinates.x, 0.0f, 640.0f); // Note: glm::vec3's x,y,z are contiguous in memory // TODO: understand its implementation
        ImGui::SliderFloat3("Model B's World Coordinates", &m_modelB_world_coordinates.x, 0.0f, 640.0f);

        ImGui::Text("Application average %.3f ms/frame (%.1f FPS)", 1000.0f / ImGui::GetIO().Framerate, ImGui::GetIO().Framerate);
	}
}
```

With all our setup up to this point, our unified `.shader` file now looks like this:

```cpp
#shader vertex
#version 460 core

layout (location = 0) in vec4 position;	// right now this position is the position in local (object) space in right-handed coordinates.

/*
Note that currently location 0 actually feeds in 2 floats.
It is predefined that OpenGL will set the 1st and 2nd components of the vec4 to these two floats respectively,
and set the 3rd and 4th components of the vec4 to 0.0f (This is based on the assumption that if Z is not provided, the vertex is meant to be in a plane) and
1.0f (implies a homogeneous position in space as opposed to a homogeneous direction vector, which would have set to 0.0f) respectively.
*/

layout (location = 1) in vec2 texCoord;

uniform mat4 u_MVP;

out vec2 v_TexCoord;	// v for varying, as OpenGL is interpolating this after vertex shader and before fragment shader.

void main()
{
    gl_Position = u_MVP * position;	// Note that position is right-handed, gl_Position is left-handed
    v_TexCoord = texCoord;
}

#shader fragment
#version 460 core

uniform vec4 u_Color;	// the u prefix is a naming convention for uniform variables
uniform sampler2D u_Texture;

in vec2 v_TexCoord;

layout (location = 0) out vec4 color;

void main()
{
	vec4 texColor = texture(u_Texture, v_TexCoord);
	color = texColor;
}
```

- Note that `sampler2D` takes an integer correspond to the texture unit, we aren't providing which target to sample from! In my current understanding, this is because `sampler2D` will ONLY sample from the target `GL_TEXTURE_2D`.

## Batch Rendering // TODO

## What is GLSL

As I probably have mentioned, shaders are programs that run on the GPU.

GLSL (OpenGL Shading Language) is the language to write source code for OpenGL shaders.

GLSL has C-style syntax.

An OpenGL shader program abstrcts a rendering pipeline based on rasterization.

## Shader Basics Revisit

An OpenGL shader program (should at least) contains a vertex shader (runs on per-vertex basis) and a fragment shader (runs on per-pixel basis).

The responsibilities of a vertex shader are:

- Output the clip space position of the vertex into `gl_Position`.
- Supply varying values down the pipeline (e.g. for fragment shader).

❓ Understand the exact maths of how vertex varyings are interpolated to get fragment varying.

The responsibility of a vertex shader is:

- Calculate the final color of the <ins>rasterized</ins> pixel (and then output the final color to e.g. frame buffer).

Every vertex or fragment shader has to have a `void main() { ... }` function, acting as an entry point.

```cpp
void main()
{
	some_color = vec4(1.0, 0.0, 0.0, 1.0);
}
```

You can define your own function:

```cpp
vec4 getColor()
{
	return vec4(1.0);  // 1.0 is used for all components
}

void main()
{
	some_color = getColor();
}
```

GLSL's function parameter resembles "pass-by-reference" with `out`:

```cpp
void GetColorOut(in vec4 color, out vec4 final)
{
	final = color * vec4(0.5);
}

void main()
{
	vec4 final;
	GetColorOut(vec4(1.0), final);
	some_color = final;
}
```

by this we can "return" multiple values from one function.

❓ Do GLSL function parameters must be declared with `in`/`out`? (no, note also that there is `inout`)

Types that are mainly used in GLSL are: `float`, `bool` and `int`.

There are also built-in vector and matrix types in GLSL: `vec2`, `vec3`, `vec4`; `mat2`, `mat3`, `mat4`.

Common mathematical operations: `+`, `-`, `*`, `/`, `=`, `+=`, `-=`, `*=`, `/=`.

In GLSL, vector operating on vector/scalar and vice versa is component-wise; matrix operating on scalar and vice versa is component-wise.

Matrix operating on matrix follows linear algebra.

Plus, we can do component-wise multiplication between matrices as follows:

```cpp
// m1, m2, m3 are glsl matrices
m1 = matrixCompMult(m2, m3);
```

❓ What if I do division between matrices?

I believe matrix `*` vector follows linear algebra.

❓ What does vector `*` matrix means? (see [here](https://en.wikibooks.org/wiki/GLSL_Programming/Vector_and_Matrix_Operations))

We can compose vector from scalars or other vectors:

```cpp
vec4 q = vec4(1.0);  // 1.0, 1.0, 1.0, 1.0
vec3 p = vec3(1.0, 0.0, 0.0);
vec4 hp = vec4(p, 1.0);
vec3 same_as_p = vec3(hp);
```

On the other hand:

```cpp
mat4 idm = mat4(1.0);  // Identity matrix
```

Note that the predefined variable `gl_FragColor` is removed in modern GLSL. You need to define your own `out` variable for the output of the fragment shader. You can use any name you want.

In GLSL, type can be preceded with "type qualifier":

```cpp
uniform vec4 var1;
attribute vec4 var2;	// removed in modern GLSL
varying vec4 var3;	// removed in modern GLSL
```

The type qualifier `uniform` means `var1` is a global real-only (within shader) variable (global to the shader program i.e. the data behind is a singleton and is shared to both vertex shader and fragment shader). The word "uniform" means it is not changing in the shader program, it holds the same value no matter it is access by the vertex shader or by the fragment shader. E.g.

```cpp
// part of any kind of shader:
uniform vec4 u_Color;  // set and then sent from CPU side.
```

~~Variables declared with `varying` are to be outputted from the vertex shader, and then to be interpolated (which could then be received by the fragment shader). (❓ Deeper understanding needed. E.g., how is `varying` variables differ from the normal `out` variables?)~~

The use of `varying` qualifier is replaced by the use of an `out` qualifier at vertex shader's end and an `in` qualifier at fragment shader's end. E.g.

```cpp
// assume the shader program is linked only with vertex shader + fragment shader.

// part of vertex shader:
out vec2 v_TexCoord;

// part of fragment shader:
in vec2 v_TexCoord;  // this has to have the same type and same name as in the vertex shader.
```

~~Variable declared with the `attribute` type qualifier is per-vertex data in vertex shader. It is the data coming from mesh itself, defined at each vertex. (❓ Deeper understanding needed. E.g., how do I control which kind of attribute is input into the variable?)~~

The use of `attribute` qualifier is replaced by the use of an `in` qualifier in vertex shaders. E.g.

```cpp
// part of vertex shader:
layout (location = 2) in vec2 texColor;
```

An extremely high-level pseudo-code of how a shader program works:

```cpp
function DrawWithShader(mesh, uniforms)
{
	for (face in mesh.faces)	// assume each face is a triangle
	{
		varyings = [
			callVertexShader(face.vertex1.attributes, uniforms),
			callVertexShader(face.vertex2.attributes, uniforms),
			callVertexShader(face.vertex3.attributes, uniforms)
		]
		// Rasterization to obtain pixels_covering_face...
		for (pixel in pixels_covering_face)
		{
			fragment_varying = InterpolateTriangleVaryings(varyings, pixel)
			callFragmentShader(fragment_varying, uniforms)
		}
	}
}
```

GLSL supports standard `for` loop, `while` loop and `do-while` loop (with the same syntax as in C++).

The keywords `break`, `continue` (for loops) and `return` (for functions) are also supported by GLSL.

I believe `if` and `if-else` statements in GLSL also work the same as in C++.

❓ Does GLSL `switch` statement must be switching on integer types (like in C++)? (yes, and the cases must be integer constant, as in C++)

❓ Can we have multiple `return` statements in a GLSL function? (yes)

❓ Can I write statement without curly braces like in C++ (e.g. in a long if-else-if block)? (yes)

## Texture Revisit

GLSL typically deals with colors by floating point numbers in range `[0.0, 1.0]`.

In my current understanding, if the color provided to output from the fragment shader is out of this range (e.g., `vec4(-10.0, -10.0, 5.0, 1.0)`), OpenGL will clamp the values to `[0.0, 1.0]`.

In the 2D texture space (i.e. on a 2D texture), at least in OpenGL, the origin `(0, 0)` is at bottom-left. the first component lies on the U-axis toward right, the second component lies on the V-axis toward top.

A texel is a color value of a texture at a given texure coordinates.

A texture object holding the data of a 2D texture on GPU linked to the `GL_TEXTURE_2D` target of a texture unit can be sampled by a 2D sampler. Such a sampler is represented in a fragment shader using the `sampler2D` keyword in which the variable holds an `int` representing which texture unit to sample from. And together with a texture coordinates represented by `vec2`, the sampler can supply the sampled texel to the fragment shader via the built-in GLSL function `texture` (which returns a `vec4` representing the texel). E.g.

```cpp
// part of a fragment shader:

uniform sampler2D u_Texture;
in vec2 v_TexCoord;

layout (location = 0) out vec4 color;

void main()
{
	color = texture(u_Texture, v_TexCoord);
}
```

Note that `texture2D`, if not completely replaced by `texture`, is strongly deprecated (see [here](https://stackoverflow.com/questions/12307278/texture-vs-texture2d-in-glsl)).

❓ Is the sampler per-variable or per-fragment-shader or per-texture-unit or per-texture-unit-target or per-shader-program? Is that possible if two sampling processes happens at the same time? If so, is that possible if a single sampler is queried by both processes?

### Multiplicative Blending (also called modulate/modulation blending)

It just means `Color1 * Color2`

E.g. the following code will filter the texture with only red left:

```cpp
// part of a fragment shader:

uniform sampler2D u_Texture;
in vec2 v_TexCoord;

layout (location = 0) out vec4 color;

void main()
{
	vec4 red_channel = vec4(1.0, 0.0, 0.0, 1.0);
	vec4 texColor = texture(u_Texture, v_TexCoord);
	color = texColor * red_channel;
}
```

If we were to write:

```cpp
color = vec4(texColor.r);
```

The resulting image holds the similar meaning. It will be a black-and-white image indicating the distribution of red, where brighter region (more white) indicates more red.

### Blend textures with Transparency within Fragment shader

An Example:

```cpp
// part of a fragment shader:

uniform sampler2D u_TextureAtBottom;
uniform sampler2D u_TextureOnTop;
in vec2 v_TexCoord;
layout (location = 0) out vec4 color;

void main()
{
	vec4 texColorAtBottom = texture(u_TextureAtBottom, v_TexCoord);
	vec4 texColorOnTop = texture(u_TextureOnTop, v_TexCoord);
	color = mix(texColorAtBottom, texColorOnTop, texColorOnTop.a);  // amounts to (1 - texColorOnTop.a) * texColorAtBottom + texColorOnTop.a * texColorOnTop
}
```

❓ To sample two textures in a single fragment shader, are there any ways other than using multiple texture units?

### Basic Addressing

Addressing mode is just another name for wrap mode (recall the texture parameters we set previously for wrap modes `GL_TEXTURE_WRAP_S` and `GL_TEXTURE_WRAP_T`, when initializing a Texture in our project).

Apart from the previously mentioned `GL_CLAMP_TO_EDGE` and `GL_REPEAT`, another probably frequently-used parameter value is `GL_MIRRORED_REPEAT`. Its effect is exactly as its name suggests.

### Basic Filtering

Filtering modes are the texture parameter we set previously for `GL_TEXTURE_MIN_FILTER` and `GL_TEXTURE_MAG_FILTER`.

I think the most frequently-used two kinds of filters we can choose from in OpenGL are the Nearest filtering and the Linear filtering.

Let's first talk about MAG filter (when you zoom in and the pixels to render is more than the texels): under such circumstances, nearest filtering roughly means that just give me the value of the texel where my sampling point hits (❓ Not sure what if the sampling point is on edges of texels). On the other hand, in the setting of Linear filtering, each time GPU will sample the four closest texels and then blend the values using bilinar filtering (❓ Understand bilinear filtering).

Nearest filtering is faster than Linear filtering, since GPU only needs to get one texel value.

For MIN filter (when you zoom out and the pixels to render is less than the texels), we have an extra option to use mipmaps.

Mipmaps is a set of textures, each one of them is half the size of the last until $1 \times 1$.

I think, if the mipmaps is created rather than being read in, the larest texture in a mipmaps is the original texture being read in. (❓ Verify this)

❓ How can we create or read in mipmaps? Does it have to happen at the point when we create the texture?

When sampling with mipmaps, GPU can decide which texture in the mipmaps chain to read from. (❓ How exactly does this result in acceleration on the GPU?)

Trilinear Filtering: If you choose to use mipmaps, you can further choose from "only sampling from the nearest mipmap level (I think level here just means the one texture chosen from the set)" or "sampling 2 nearest (upper and lower) mipmap levels and then blend them together". Trilinear filtering means you choose the latter, and on each mipmap level, bilinar filtering is applied.

❓ How to actually set up those settings in OpenGL? 

❓ What happens in situations like, say, the texture is mapped onto the geometry such that we need to zoom in one direction and zoom out in another direction (e.g. a square texture on a rectangular mesh).

## Common Functions in GLSL

- `abs` stands for absolute value

  ```cpp
  vec2 v = vec2(-1.5, 2.3);
  vec2 result = abs(v);
  // result will be vec2(1.5, 2.3)
  ```

- `min(a, b)` returns the smaller value between `a` and `b` (componentwisely)

  ```cpp
  vec2 a = vec2(1.0, 3.0);
  vec2 b = vec2(2.0, 2.0);
  vec2 result = min(a, b); // result is vec2(1.0, 2.0)
  ```

- `max(a, b)` returns the larger value between `a` and `b` (componentwisely)

  ```cpp
  vec2 a = vec2(1.0, 3.0);
  vec2 b = vec2(2.0, 2.0);
  vec2 result = max(a, b); // result is vec2(2.0, 3.0)
  ```

- `clamp(value, lower, upper)` returns the cloest value to `value` that is within the range `[lower, upper]` (componentwisely)

  ```cpp
  vec2 value = vec2(0.5, 2.5);
  vec2 minVal = vec2(0.0, 1.0);
  vec2 maxVal = vec2(1.0, 2.0);
  
  vec2 result = clamp(value, minVal, maxVal);
  // result will be vec2(0.5, 2.0)
  ```

- `saturate(value)` or just `sat(value)` is NOT a built-in function in GLSL, but it's very common to see in shader code. It is just `clamp(value, 0.0, 1.0)`.

- `smoothstep(lower, upper, value)` is implemented similarly as follows:

  ```cpp
  t = clamp((value - lower)/(upper - lower), 0.0, 1.0);
  return t * t * (3.0 - 2.0 * t);  // cubic Hermite interpolation
  ```

  It is implemented such that it can also perform componentwisely:

  ```cpp
  vec2 edge0 = vec2(0.2, 0.3);
  vec2 edge1 = vec2(0.8, 0.9);
  vec2 x = vec2(0.5, 0.85);
  
  vec2 result = smoothstep(edge0, edge1, x);  // 0.5, 0.9803...
  ```
  
  For details, see [here](https://en.wikipedia.org/wiki/Smoothstep).
  
  Currently, I just think of `smoothstep` as normalizing the number between a range to `[0, 1]` and then redistributing them towards the polars.

  ❓ Understand (cubic) Hermite interpolation.

  I implement a photograph-filter using `smoothstep` in our project. See the related repository of this note.

- `step(threshold, value)` is implemented similarly as follows:

  ```cpp
  if (value < threshold) return 0.0;
  return 1.0;
  ```

  It is implemented such that it can also perform componentwisely:

  ```cpp
  vec2 edge = vec2(0.5, 0.5);  // this can also be float edge = 0.5;
  vec2 x = vec2(0.3, 0.6);
  
  vec2 result = step(edge, x);
  // result will be vec2(0.0, 1.0)
  ```

- `mix(a,b,t)` returns `a + t * (b - a)` where `t` is a percentage between `0` and `1`.

  It is implemented such that it can also perform componentwisely:

  ```cpp
  vec2 x = vec2(1.0, 2.0);
  vec2 y = vec2(3.0, 4.0);
  vec2 a = vec2(0.5, 0.25);
  float b = 0.5;
  
  vec2 result1 = mix(x, y, a);  // 2.0, 2.5
  vec2 result1 = mix(x, y, b);  // 2.0, 3.0
  ```

  `mix` sometimes (e.g. in HLSL) is also called `lerp`, stands for linear interpolation.

Using the abovementioned functions, one can implement graph of functions purely within fragment shader. See the related repository of this note.

- `InverseLerp(value, min, max)` is NOT a built-in function in GLSL, but it's very common to see in shader code. It's a rearrange of `mix(a,b,t)` to get `t` from `a`, `b` and `mix(a,b,t)`:

  ```cpp
  return (value - min)/(max - min)
  ```

  Comparing to `smoothstep`, `InverseLerp` is like "`linearstep`".

- `Remap(value, inMin, inMax, outMin, outMax)` is NOT a built-in function in GLSL, but it's very common to see in shader code. It linearly remaps the `value` between `inMin` and `inMax` to a value between `outMin` and `outMax`:
  
  ```cpp
  float t = InverseLerp(value, inMin, inMax);
  return mix(outMin, outMax, t);
  ```

- `floor(value)` returns the value of the nearest integer that is less than or equal to `value`.

  Note that this definition is strictly implemented for negative number as well, which means, e.g., `floor(-1.6)` returns `-2`.

- `ceil(value)` is the exact opposite of `floor(value)`.

  Note that this means, e.g., `ceil(-1.6)` returns `-1`.

- `round(value)` returns the value of the nearest integer to `value`.

  ❓ How does `round` deal with `.5`?

- `fract(value)` returns the fractional part of `value`.

  Beware that `fract` is implemented base on `floor`, which means, e.g., `fract(-0.6)` returns `0.4`.

- `mod(x, y)` returns the floating point version of $x$ $\text{mod}$ $y$.

Using the abovementioned functions, one can implement graph of functions on a grid purely within fragment shader. See the related repository of this note.

- Screen-space Derivative (only available in fragment shader)

  - `dFdx(value)` returns the difference between `value` calculated in the current fragment shader and the `value` calculated in the fragment shader invoked for a horizontal neighbour pixel.

  - `dFdy(value)` returns the difference between `value` calculated in the current fragment shader and the `value` calculated in the fragment shader invoked for a vertical neighbour pixel.

  ❓ Any extra overhead these function would occur in any situations that is worth mentioning?

  ❓ Fully explain how does `dF/dx` and `dF/dy` can be implemented, especially how does the GPU works to accomplish the implementation.

  In my current very limited understanding, I think the GPU groups the data of $2 \times 2$ fragment shader invocations (invoked for the $2 \times 2$ adjacent pixels on screen) into a block (also called quad) such that the data communication is allowed between fragment shader invocations within the block. Every fragment shader invocation lives within one of such blocks. The difference calculated in `dFdx` is t2 - t1, where t2 belongs to a right-hand side fragment in the block and t1 belongs to a left-hand side fragment in the block, and the difference calculated in `dFdy` is t4 - t3, where t4 belongs to an upper fragment in the block and t3 belongs to a lower fragment in the block (so fragment shader invocations of the same row returns the same value for `dFdx`, fragment shader invocations of the same column returns the same value for `dFdy`). I think we typically do not have explicit control or knowledge about the specific position of the fragment within its 2x2 block. And I think blocks cannot have overlap. The quads themselves are adjacent to each other, covering the render target in a tiled fashion. The borders of one quad are immediately next to the borders of another, but without overlapping pixels.

  ❓ How does the GPU adress the situation at edge of the rasterized region where there is not enough pixels to form block?

  ❓ Can a block consist of fragments of different primitives (e.g., a block covering the edge connecting two triangles)? (In my current understanding, direct computation of `dFdx` and `dFdy` in GLSL within fragment shaders is restricted to individual primitives)

  For example, `normalize(cross(dFdx(world_pos), dFdy(world_pos)))` can give the world normal.

  In this example, note that, in my current understanding, the mathematical formula for a cross product is the same no matter we are using a left or a right-handed coordinates. The handedness affects the resulting vector (and whether we can obtain its direction using left or right hand rule). If `world_pos` is in right handed coordinates, the `cross` can then be interpreted using right hand rule.

## Built-in Math Functions in GLSL

- `sin(x)` expects `x` to be a value in radians and returns the sine of it.
- `cos(x)` expects `x` to be a value in radians and returns the cosine of it.

Using trigonometric function, our project implemented an old style TV scan lines effect in fragment shader. See the related project of this note.

- `length`

- `normalize`

- `dot`

- `cross`

- `reflect`

- `refract`

  Note that the `eta` value you provide to `refract` should be the index of refraction of the medium from which the incident ray comes, divided by the index of refraction of the medium into which the ray is refracted.

### Swizzling

```cpp

```

## Adding OBJ Loader into the project

- [Repository](https://github.com/Bly7/OBJ-Loader)

OBJ Loader is a single-header-only `.obj` file loader for C++ projects. It is written purely in C++ and it only uses `std`. That makes it extremely easy to include:

> All you need to to is copy over the `OBJ_Loader.h` header file, include it in your solution, and you are good to go.

- At the current point, I haven't dived deep into how OBJ Loader gathers data into its classes.

  By far, I think, for each mesh loaded, `Vertices` is a `std::vector` that contains all the vertices to draw the mesh such that every three consecutive vertices in `Vertices` form a `f`ace (triangle) specified in the `.obj` file.

  ❓ This right now can contain duplicated vertices. Are there any ways to obtain a set of unique vertices and a corresponding set of indices for drawing the mesh?

One plausible scaffolding is as follows:

```cpp
// ...
#include <OBJ_Loader.h>

// ...

objl::Loader Robert_Smith_Loader;
Robert_Smith_Loader.LoadFile("res/meshes/suzanne.obj");
assert(Robert_Smith_Loader.LoadedMeshes.size() == 1);
objl::Mesh loaded_mesh = Robert_Smith_Loader.LoadedMeshes[0];
m_count = loaded_mesh.Vertices.size();
float* arr = new float[m_count * (3 + 2 + 3) * sizeof(float)];
const auto& v = loaded_mesh.Vertices;
for (unsigned int i = 0; i < m_count; i++)
{
    // position:
    arr[i * (3 + 2 + 3) + 0] = v[i].Position.X;
    arr[i * (3 + 2 + 3) + 1] = v[i].Position.Y;
    arr[i * (3 + 2 + 3) + 2] = v[i].Position.Z;
    // texture coordinates:
    arr[i * (3 + 2 + 3) + 3] = v[i].TextureCoordinate.X;
    arr[i * (3 + 2 + 3) + 4] = v[i].TextureCoordinate.Y;
    // normal:
    arr[i * (3 + 2 + 3) + 5] = v[i].Normal.X;
    arr[i * (3 + 2 + 3) + 6] = v[i].Normal.Y;
    arr[i * (3 + 2 + 3) + 7] = v[i].Normal.Z;
}

m_vao = std::make_unique<VAO>();

m_vbo = std::make_unique<VBO>(arr, m_count * (3 + 2 + 3) * sizeof(float));

VBOLayout vbo_layout;
// position:
vbo_layout.AddAttribute<float>(3);
// texture coordinates:
vbo_layout.AddAttribute<float>(2);
// normal:
vbo_layout.AddAttribute<float>(3);

m_vao->LinkVertexBuffer(*m_vbo, vbo_layout);

// ...

delete[] arr;
```

- A brief explanation of `.obj` file can be found [here](https://www.opengl-tutorial.org/beginners-tutorials/tutorial-7-model-loading/).

## A Quick Setup of the Default Depth Buffer attached to the Default Framebuffer

We need depth buffer when we start to draw 3D models.

## Lighting Models

### Ambient Lighting

### Hemisphere Lighting

### Lambertian Lighting

### sRGB Color Space vs. Linear Color Space

❗ Color spaces are really important topics, I DEFINITELY need to conduct further studies on colors.

For now, in my understanding:

- There is a differentiation between rgb color (to display) and what I currently call "linear intensity" (for lighting calculation).

- If we define, say, `vec3 object_color = vec3(0.5);` where we are thinking of an exact mid-grey object (i.e. the values are defined in sRGB color space), and we want to use it (as albedo?) in lighting calculation, we need to first transform it into the linear color space, then conduct lighting calculation, then transform the result back to sRGB color space, then output for display.

- Similarly, if we define, say, `vec3 light = vec3(0.5);` and use it in lighting calculation, `light` is then the linear intensity (in linear color space) of the represented light. The perceptual color of the light is actually brighter then mid-grey.

This may be a very rudimentary idea, but sometimes I think of color values in this way:

- For light, I think of each component of a `vec3(x,y,z)` where $x,y,z \in [0,1]$ in linear color space as the intensity ("rate" of photons) of the corresponding channel. $1$ here means the maximum intensity the display can emit for the corresponding channel. Its counterpart in sRGB color space is the perceptual color of the light.

- For material albedo, I think of each component of a `vec3(x,y,z)` where $x,y,z \in [0,1]$ in linear color space as the proportion of the intensity ("rate" of photons) of the corresponding channel being reflected. Its counterpart in sRGB color space is the perceptual color of the material when it is illuminated by `[1,1,1]` light.

### A Quick Setup of a Skybox using Cube Map

### Toon/Cel Shading

## Vertex Shader Revisit

## Signed Distance Function (SDF) for Simple Shapes

## Noise // TODO

## Post-processing // TODO

## Deferred Rendering // TODO
