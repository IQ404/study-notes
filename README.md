# OpenGL

## Contents

## Useful Links

[dos.GL](https://docs.gl/) is a mostly useful (but sometimes confusing) website to check the information about OpenGL functions.

## Currently Unclassified Notes

- In OpenGL, `0` often represents the ID for non-existent object. So, we often use `0` for detaching.

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

  The fragment shader outputs data to some framebuffer for latter use in the rendering pipeline. The `layout (location = 0)` in the fragment shader is saying that `vec4 color` is outputted to the `0`th color attachment in the framebuffer (❓ Elaborate on the color attachments in the framebuffer).

- `gl_Position` stores the <ins>clip space</ins> position of the vertex.

  Clip space is the space after transforming by the projection matrix. After the vertex shader, the clip space coordinates in gl_Position are automatically divided by their W component (perspective division) by OpenGL. This step converts clip space coordinates to normalized device coordinates (NDC), which range from -1 to 1 in each axis. The NDC are transformed into screen space coordinates through the viewport transformation. This step maps the NDC to the actual coordinates on the screen (or the framebuffer) and is handled by OpenGL as part of the fixed-function pipeline (❓ Need deeper understanding).

- `glUseProgram` makes the program object with the provided ID in use for current rendering state.

- `glDeleteProgram` flags the program object with the provided ID for deletion. The program object will then only be deleted when it is not in use as part of current rendering state.

## Draw Calls

```cpp
glDrawArrays(GL_TRIANGLES, 0, 3);
```

- `glDrawArrays` is a draw call to the currently binded vertex buffer (❓ when index buffer is not used).

  `GL_TRIANGLES` means we are drawing triangles.

  `0` there means we are starting to render trangles from the first vertex in the binded buffer.

  `3` there means how many vertices we are going to render.

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

where `Basic.shader` contains:

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
