---
title: 在C++中使用freetype显示中文的问题
date: 2024-03-31T09:43:10+08:00
categories:
- issue
tags:
- C++
- FreeType
---

# 问题
在使用opengl渲染时，尝试使用freetype来渲染文字，但是在渲染中文汉字的时候遇到了问题，无法正确渲染中文

# 解决方案
直接先说解决方案，在从face中获取glyph时，使用wstring代替string或者char *，并且在渲染查找glyph时，也这么做:

<!-- more -->

```c++

void demo()
{
    std::wstring text = L"HelloWorld中文测试";
    FT_Library ft;
    camera = new OrthographicCamera(800, 600);
    // All functions return a value different than 0 whenever an error occurred
    if (FT_Init_FreeType(&ft))
        std::cout << "ERROR::FREETYPE: Could not init FreeType Library" << std::endl;
    FT_Face face;
    if (FT_New_Face(ft, "C:/Windows/Fonts/SIMLI.TTF", 0, &face))
        std::cout << "ERROR::FREETYPE: Failed to load font" << std::endl;

    FT_Set_Pixel_Sizes(face, 0, 20);

    glPixelStorei(GL_UNPACK_ALIGNMENT, 1);
    for (int x = 0; x < size; x++)
    {
        wchar_t c = text[x];
        // FT_ULong
        GLint ch = static_cast<GLint>(c);
        if (CharacterCache.find(ch) != CharacterCache.end())
        {
            return;
        }
        // Load character glyph
        // FT_Load_Glyph
        // FT_Get_Char_Index(face, text[i])
        if (FT_Load_Glyph(face, FT_Get_Char_Index(face, ch), FT_LOAD_RENDER))
        {
            std::cout << "ERROR::FREETYTPE: Failed to load Glyph" << std::endl;
            continue;
        }
        // Generate texture
        GLuint texture;
        glGenTextures(1, &texture);
        glBindTexture(GL_TEXTURE_2D, texture);
        glTexImage2D(
            GL_TEXTURE_2D,
            0,
            GL_RED,
            face->glyph->bitmap.width,
            face->glyph->bitmap.rows,
            0,
            GL_RED,
            GL_UNSIGNED_BYTE,
            face->glyph->bitmap.buffer);
        // Set texture options
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);
        std::cout << "ch: " << ch << " bind to id: " << texture << std::endl;

        // Now store character for later use
        Character character = {
            texture,
            glm::ivec2(face->glyph->bitmap.width, face->glyph->bitmap.rows),
            glm::ivec2(face->glyph->bitmap_left, face->glyph->bitmap_top),
            face->glyph->advance.x};
        // CharacterCache.insert(std::pair<FT_ULong, Character>(ch, character));
        CharacterCache[ch] = character;
    }
}
```

渲染时的代码如下:
```c++
void demoRender () {
    std::wstring text = L"HelloWorld中文测试";
     // Activate corresponding render state
    glEnable(GL_BLEND);
    glBlendFunc(GL_SRC_ALPHA, GL_ONE_MINUS_SRC_ALPHA);
    program->use();
    glUniform3f(glGetUniformLocation(program->ID, "u_color"), color.x, color.y, color.z);
    program->setUniformMat4("projection", camera->projection);
    glActiveTexture(GL_TEXTURE0);
    glBindVertexArray(VAO);

    for (int i = 0; i < size; i++)
    {
        Character ch = CharacterCache[text[i]];

        GLfloat xpos = x + ch.Bearing.x * scale;
        GLfloat ypos = y - (ch.Size.y - ch.Bearing.y) * scale;

        GLfloat w = ch.Size.x * scale;
        GLfloat h = ch.Size.y * scale;
        // Update VBO for each character
        GLfloat vertices[6][4] = {
            {xpos, ypos + h, 0.0, 0.0},
            {xpos, ypos, 0.0, 1.0},
            {xpos + w, ypos, 1.0, 1.0},

            {xpos, ypos + h, 0.0, 0.0},
            {xpos + w, ypos, 1.0, 1.0},
            {xpos + w, ypos + h, 1.0, 0.0}};
        // Render glyph texture over quad
        glBindTexture(GL_TEXTURE_2D, ch.TextureID);
        // Update content of VBO memory
        glBindBuffer(GL_ARRAY_BUFFER, VBO);
        glBufferSubData(GL_ARRAY_BUFFER, 0, sizeof(vertices), vertices); // Be sure to use glBufferSubData and not glBufferData
        glBindBuffer(GL_ARRAY_BUFFER, 0);
        // Render quad
        glDrawArrays(GL_TRIANGLES, 0, 6);
        // Now advance cursors for next glyph (note that advance is number of 1/64 pixels)
        x += (ch.Advance >> 6) * scale; // Bitshift by 6 to get value in pixels (2^6 = 64 (divide amount of 1/64th pixels by 64 to get amount of pixels))
        // text++;
    }
    glBindVertexArray(0);
    glBindTexture(GL_TEXTURE_2D, 0);
    glDisable(GL_BLEND);
}
```
好了，freetype已经可以正常显示中文了。
![2024-03-31T095450](2024-03-31T095450.png)
# 排查过程
首先，我是按照 [教程](https://learnopengl-cn.github.io/06%20In%20Practice/02%20Text%20Rendering) 来学习freetype渲染文字的。
教程原版是英文的，所以原版教程中的渲染只有基础的128个ascII码。我这边便想到了拓展到实现中文渲染。（因为在libGDX中，也是使用freetype，是可以正常渲染中文的）.

于是我便写了如下的代码:
```c++

void CharacterManager::create(std::string &text)
{
    FT_Library ft;
    camera = new OrthographicCamera(800, 600);
    // All functions return a value different than 0 whenever an error occurred
    if (FT_Init_FreeType(&ft))
        std::cout << "ERROR::FREETYPE: Could not init FreeType Library" << std::endl;
    FT_Face face;
    if (FT_New_Face(ft, "C:/Windows/Fonts/BRLNSDB.TTF", 0, &face))
        std::cout << "ERROR::FREETYPE: Failed to load font" << std::endl;

    // Set size to load glyphs as
    FT_Set_Pixel_Sizes(face, 0, 48);
    glPixelStorei(GL_UNPACK_ALIGNMENT, 1);

    for (int x = 0; x < text.size(); x++)
    {
        char c = text[x];

        // FT_ULong
        loadCharater(c, face);
    }

    FT_Done_Face(face);
    FT_Done_FreeType(ft);
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    glBindVertexArray(VAO);
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, sizeof(GLfloat) * 6 * 4, NULL, GL_DYNAMIC_DRAW);
    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 4, GL_FLOAT, GL_FALSE, 4 * sizeof(GLfloat), 0);
    glBindBuffer(GL_ARRAY_BUFFER, 0);
    glBindVertexArray(0);
}

```
注意，我这里使用的是`std::string`,底层还是char数组.
于是，在render的时候所有的中文全部被变成了**未定义的方框**。
这个时候我比较好奇了，最主要的是控制台是可以正常输出这段字符串char数组的。
于是我先`百度`了c++ freetype 中文 等关键字，刷到了这篇[CSDN文章](https://blog.csdn.net/qq_22655017/article/details/90034431),我意识到了，中文跟一般的英文字符不一样，中文使用的unicode，一个unicode = 2byte。
但是在c++中，char是等同于byte的。所以一个char是无法存储一个中文汉字的。
> 这点跟JAVA不一样，java中，char类型底层就是两个字节，所以可以正常使用中文.可能java底层的c++代码中，char使用的就是wchar

于是，我把代码中的char*换成了wchar_t,但是我首先想到的是先在控制台输出一下wchar_t，以免对项目有不必要的改动。
这一试，就出现了问题，控制台无法打印wchar_t，于是，我感觉可能是出问题了，我就去`stack overflow`搜了wchar 与 char之间的转换，以及如何让系统使用wchar。但是好像并没有特别好的结果。

但最后我突然想到，freetype是查找char使用方法`FT_Get_Char_Index`中,charcode的类型其实是 `FT_ULong`，是一个无符号的long，c++中`long`的范围至少有32位，那么我直接使用wstring中的wchar强转成long来直接放入查找可以吗？
于是我就进行了尝试，结果是OK的。

# 总结
这个问题其实本身并不难，甚至比较基础，但是查询过程中，由于对c++熟悉度不够，还是走了很多弯路。甚至中间曾经出现过使用nullptr的情况，这些跟java有太多不一样了。

但最后，如何在控制台输出wchar还是没有解决。
对于这个问题，我有自己的想法<span style='text-decoration: line-through;'>(猜测)</span>,那就是控制台是windows平台的，windows平台可能对底层的char进行了自己的优化，在展示时会根据char值来判断是否是unicode来进行展示，判断是unicode则使用unicode来展示，所以直接cout输出char数组是可以展示中文的。
但是对于wchar，可能windows底层并没有对其优化，想要展示还需要我们开发做一些其他工作。
至于java为什么即使使用宽字节也可以正常展示中文，我猜测可能是虚拟机（VM）底层对其进行了优化，使其能够将宽字节正确处理成unicode。

最后附上[项目地址](https://github.com/voidvvv/LinkA)