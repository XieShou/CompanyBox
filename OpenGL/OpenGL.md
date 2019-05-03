# 渲染管线

接收一组3D坐标，然后把它们转变为你屏幕上的有色2D像素输出。

### 1. *顶点着色器（Vertex Shader）*阶段

它把一个单独的顶点作为输出，允许对顶点属性进行一些基本处理。主要目的是实现顶点的空间坐标转换。

### 2. *图元装配（Primitive Assembly）*阶段

将顶点装配成图元。

### 3. *几何着色器（Geometry Shader）*

把图元形式的一系列顶点的集合作为输入，它可以通过产生新的（或是其他的）图元来生成其它形状。

### 4. *光栅化（Rasterization Stage）*阶段

把图元映射为最终屏幕上相应的像素，生成供*片段着色器（Fragment Shader）*使用的*片段（Fragment）*。在*片段着色器*运行之前会对超出视图以外的所有像素执行*裁切（Clipping）*。

### 5. *片段着色器（Fragment Shader）*

主要目的是计算一个像素的最终颜色，也是所有OpenGL高级效果产生的地方。

### 6. *Alpha测试和混合（Blending）*阶段



## 定义一个着色器

将文本字符串编译为`shader`，绑定数据源给`shader`使用

```cpp
const char* vertexShaderSource =  "#version 330 core\n"
"layout (location = 0) in vec3 aPos;\n"
"void main()\n"
"{\n"
"   gl_Position = vec4(aPos.x, aPos.y, aPos.z, 1.0);\n"
"}\0";

//定义四个顶点
float vertices[] = {
		 0.5f,  0.5f, 0.0f,  // top right
		 0.5f, -0.5f, 0.0f,  // bottom right
		-0.5f, -0.5f, 0.0f,  // bottom left
		-0.5f,  0.5f, 0.0f   // top left 
	};

unsigned int vertexShader;//声明一个着色器，使用id来引用

vertexShader = glCreateShader(GL_VERTEX_SHADER);//创建一个顶点着色器

//着色器对象、传递的源码字符串数量、着色器源码
glShaderSource(vertexShader, 1, &vertexShaderSource, NULL);
glCompileShader(vertexShader);

//检测编译
int  success;
char infoLog[512];
glGetShaderiv(vertexShader, GL_COMPILE_STATUS, &success);

if(!success)
{
    glGetShaderInfoLog(vertexShader, 512, NULL, infoLog);
    std::cout << "ERROR::SHADER::VERTEX::COMPILATION_FAILED\n" << infoLog << std::endl;
}
```


