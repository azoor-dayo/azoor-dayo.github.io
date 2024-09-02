**Buffer Object**: Contiguous memory blocks on GPU of arbitrary size (not defined yet) 1. Creating VBO: `glCreateBuffers(1, &vbo_hdl);`

> If a buffer object is filled with per-vertex data, it is also known as a **Vertex Buffer Object (VBO)**. If no data store yet, size undefined.

CLEANUP: `glDeleteBuffers(size, *buffers)` Using Per-Vertex data structure!

**Data Store**: Actual data to put into Buffer Objects, defining the size of the Buffer Object too. In this case, write in data for each vertex in an array for per-vertex structure. 1. Creating entire data store: `glNamedBufferStorage(hdlName, size, data, flag);`

> Also allocates the entire available space for this Buffer Object. If there are multiple arrays to copy, don't write data at this step.

2. Copying SubData into Data Store: `glNamedBufferSubData(hdlName, offset, size, data);`

> Start writing in data. SubData does not force a reallocation of GPU memory, but Storage does.

CLEANUP: `glInvalidateBufferData(buffer) // no need to call if glDeleteBuffers is used`

**Vertex Attributes**: Each vertex has position, color and texture data. Each of these are called **Vertex Attributes**. MAX UP TO 4 data (e.g. color (r,g,b,a)) per **attribute** ONLY. Each **vertex** can support up to min 16 attributes, dependent on hardware. Usually, each array of data to be written in is one attribute for all vertices e.g. position vertices for x4 vertices. Then color vertices for x4 vertices. That's 2 attributes for each vertex.
> `glGetIntegerv(GL_MAX_VERTEX_ATTRIBS, &max_vtx_attribs);`

**Vertex Array Object (VAO)**: Acts as connection between the stuff on VBO (vertex data) and the vertex shader. Holds information to be able to pull info from VBO into the shader. Need to configure VAO with the proper info depending on how VBO is set up. **Attribute index** - index 0-15 (On my PC: 16 available). Each attribute is an array of X size that has up to 4 components in each element. (e.g. array of COLORS (rgba) for 42069 pixels) Can be color buffer, depth buffer, vertex positions etc. `GL_MAX_VERTEX_ATTRIBS` **Vertex buffer binding index** - index 0-15 (On my PC: 16 available). Is just a "location" to bind VERTEX BUFFER OBJECTS (VBOs) onto the VAO. `GL_MAX_VERTEX_ATTRIB_BINDINGS`
Note: These 2 indices are different things. 

1. Create default VAO: `glCreateVertexArrays(1, &vaoid);
   > VAO will be used as the main thingy that holds all the necessary data that is DIRECTLY used to render frames. Shaders, framebuffer data all come here.

	CLEANUP: `glDeleteVertexArrays(size, *arrays)` 
	
2. Enable **attribute index** 8: `glEnableVertexArrayAttrib(vaoid, 8); // Value is btwn 0 and GL_MAX_VERTEX_ATTRIBS - 1`
   > Used during VERTEX SHADER. Basically, You're gonna associate this guy with a **vertex buffer binding index** later. For now, just enable this "location". In future use (doing shaders), you'll only touch this **attribute binding**.
   
    CLEANUP: `glDisableVertexArrayAttrib(vaoid, 8) // May not be needed if delete VAO`
 3. Binding VBO data to VAO using **buffer binding index** 3 (can be any number): `glVertexArrayVertexBuffer(vaoid, 3, vbo_hdl, 0, sizeof(glm::vec2));`
    > Binds the VBO to VAO **buffer binding index** 3 so that the VAO can use this data to render shit later. Still need to specify how to use this data, in the following steps. Will only bind up to the SubData store array is defined in. Won't touch other SubData.

	CLEANUP: `glVertexArrayVertexBuffer(vaoid, 3, 0, x, x) // buffer 0 unbinds index 3` 

 4. Provide Format to VAO: `glVertexArrayAttribFormat(vaoid, 8, 2, GL_FLOAT, GL_FALSE, 0);`
    >Tells the VAO how to interpret the VBO data that WILL BE BOUND TO **attribute index** 8. Currently nothing is bound yet. `vaoid` - Target VAO `8` - **Attribute Index** `2` - 2 items per array element (Position data, (x, y)) `GL_FLOAT` - each item is a float `GL_FALSE` - Don't normalize. (?) `0` - Zero Padding
    
 5. Bind **Attribute Index** 8 to **Vertex Buffer Binding Index** 3 `glVertexArrayAttribBinding(vaoid, 8, 3);`
    > Now you can just refer to attribute index 8 in your shaders to refer to POSITION VERTICES. Do the same thing to COLOUR VERTICES using another **Attribute Index** and **Vertex Buffer Binding Index**.

	_Note: In this example, Attribute Index 8 and Buffer Binding Index 3 are RANDOM numbers. It's much more logical and reasonable to use the same numbers._ _e.g. Position: Att. Index 0, Binding Index 0. | Color: Att. Index 1, Binding Index 1. | Texture: Att. Index 2, Binding Index 2 etc._

Congratulations, your VAO is now set up... except it still has no info of in what order to render these vertices to make triangles! Time to do that

