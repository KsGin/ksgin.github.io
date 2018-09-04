---
title: 【OpenGL】13-Mesh与Model类
date: 2018-03-05 08:53:09
tags:
	- OpenGL
	- Computer graphics
---

&emsp;&emsp;在上一篇文章中我们了解了 Assimp 这个库，但是只知道这个库只是作为一个加载的作用，并不清楚如何使用，这这一篇便是介绍  Assimp 的数据结构以及我们自己创建的两个为方便使用它而创建的类。

<!--more-->

&emsp;&emsp;首先看一下 Assimp 的结构图：

![assimp_structure](https://ws2.sinaimg.cn/large/006tKfTcly1fp1v7m6m24j30m80crdh0.jpg)

&emsp;&emsp;Assimp 读取模型文件后，会将其加载到一套通用的数据结构中，如上图所示，首先是一个 Scene 对象，对象中包括了多个场景对象，各个对象以树状结构展开，而 Scene 中储存了根节点 mRootNode。mMeshes 和 mMatertials 则是存储了模型的 Mesh（网格：代表一个可绘制的实体）与 Material（材质）数据，每一个 Children（子节点） 包括了它的孩子节点集合和网格（Mesh）索引集合，注意 Child Node 中的 Mesh 集合中存储的是 Scene 对象里的 Mesh 数组的索引，也就是说具体数据是在 Scene 中存储的，这里只有索引。

&emsp;&emsp;除了 Scene 对象之外，便是代表具体网格数据的 Mesh 对象了，Mesh 对象存储了网格的顶点位置（mVertices）、法向量（mNormals）、纹理坐标（mTetureCoords）、面(mFaces)和物体的材质索引（mMaterialIndex），材质数据则是在在 Scene 对象的 mMaterials[] 中存储。

&emsp;&emsp;在了解了 Assimp 的通用数据结构后，我们就可以从 Scene 对象中接收数据了。首先构造一个我们自己的 Mesh 类，用来直接进行 Mesh 的绘制工作。在此之前仍需要定义好两个东西，Vertex 与 Texture 。

```c++
struct Vertex {
    glm::vec3 Position;
    glm::vec3 Normal;
    glm::vec2 TextCoords; //U V
    glm::vec3 Tangent;
    glm::vec3 Bitangent;
};

struct Texture {
    unsigned int id;
    std::string type;
    std::string path;
};

class Mesh {
public:
    /*网格数据*/
    std::vector<Vertex> vertices;
    std::vector<unsigned int> indices;
    std::vector<Texture> textures;
    /*方法*/
    Mesh(std::vector<Vertex> vertices , std::vector<unsigned int> indices , std::vector<Texture> textures);
    void Draw(Shader shader);
private:
    unsigned int VAO;
	unsigned int VBO , EBO;
    void setupMesh();
};
```

&emsp;&emsp;我们定义了公开的包括顶点 Vertex（顶点中定义了位置，法线等等），索引以及纹理，同时留下了构造方法以及 Draw 绘制接口 。而 Private 则是定义了我们在构造或者绘制过程中需要做的一些事。以下来看具体实现：

```c++
Mesh::Mesh(std::vector<Vertex> vertices, std::vector<unsigned int> indices, std::vector<Texture> textures) {
    this->vertices = vertices;
    this->indices = indices;
    this->textures = textures;

    setupMesh();
}

void Mesh::setupMesh() {
    glGenVertexArrays(1, &VAO);
    glGenBuffers(1, &VBO);
    glGenBuffers(1, &EBO);

    glBindVertexArray(VAO);

    //VBO
    glBindBuffer(GL_ARRAY_BUFFER, VBO);
    glBufferData(GL_ARRAY_BUFFER, vertices.size() * sizeof(Vertex), &vertices[0], GL_STATIC_DRAW);

    //EBO
    glBindBuffer(GL_ELEMENT_ARRAY_BUFFER, EBO);
    glBufferData(GL_ELEMENT_ARRAY_BUFFER, indices.size() * sizeof(unsigned int), &indices[0], GL_STATIC_DRAW);

    glEnableVertexAttribArray(0);
    glVertexAttribPointer(0, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *) 0);

    glEnableVertexAttribArray(1);
    glVertexAttribPointer(1, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *)offsetof(Vertex,Normal));

    glEnableVertexAttribArray(2);
    glVertexAttribPointer(2, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void *)offsetof(Vertex,TextCoords));

    glEnableVertexAttribArray(3);
    glVertexAttribPointer(3, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)offsetof(Vertex, Tangent));

    glEnableVertexAttribArray(4);
    glVertexAttribPointer(4, 3, GL_FLOAT, GL_FALSE, sizeof(Vertex), (void*)offsetof(Vertex, Bitangent));

    glBindVertexArray(0);
}

void Mesh::Draw(Shader shader) {
    unsigned int diffuseNr  = 1;
    unsigned int specularNr = 1;
    unsigned int normalNr   = 1;
    unsigned int heightNr   = 1;
    for(unsigned int i = 0; i < textures.size(); i++)
    {
        glActiveTexture(GL_TEXTURE0 + i); // active proper texture unit before binding
        // retrieve texture number (the N in diffuse_textureN)
        std::string number;
        std::string name = textures[i].type;
        if(name == "texture_diffuse")
            number = std::to_string(diffuseNr++);
        else if(name == "texture_specular")
            number = std::to_string(specularNr++); // transfer unsigned int to stream
        else if(name == "texture_normal")
            number = std::to_string(normalNr++); // transfer unsigned int to stream
        else if(name == "texture_height")
            number = std::to_string(heightNr++); // transfer unsigned int to stream

        // now set the sampler to the correct texture unit
        glUniform1i(glGetUniformLocation(shader.ID, (name + number).c_str()), i);
        // and finally bind the texture
        glBindTexture(GL_TEXTURE_2D, textures[i].id);
    }

    // draw mesh
    glBindVertexArray(VAO);
    glDrawElements(GL_TRIANGLES, indices.size(), GL_UNSIGNED_INT, 0);
    glBindVertexArray(0);

    // always good practice to set everything back to defaults once configured.
    glActiveTexture(GL_TEXTURE0);
}
```

&emsp;&emsp;实现的内容没有多少难度，Mesh 的构造中就是复制以及我们之前学过的绑定顶点对象以及填充数据等，  Draw 方法传入一个 Shader（着色器对象），并读取纹理贴图后为 Shader 对象的属性赋值以及绘制等。

&emsp;&emsp;暂时明白了 Mesh 的实现，接下来看看将 Scene 解析并将所有 Mesh 绘制的 Model 类：

```c++
class Model {
public:
    Model(char *path , bool gamma = false);

    void Darw(Shader shader);

    std::vector<Mesh> meshes;

private:

    bool gammaCorrection;

    std::string directory;

    std::vector<Texture> textures_loaded;

    void loadModel(std::string path);

    void processNode(aiNode *node, const aiScene *scene);

    Mesh processMesh(aiMesh *mesh, const aiScene *scene);

    std::vector<Texture> loadMaterialTextures(aiMaterial *mat, aiTextureType type, std::string typeName);
};
```

&emsp;&emsp;Mesh 中已经完成了对网格的操作，而在 Model 中我们的重点仅仅是将 Scene 的属性按照数据结构解析并赋给 Mesh 而已，我们留下了构造方法已经绘制函数，绘制函数很简单，就是遍历 meshes 数组，挨个调用 mesh 对象的 draw 方法就可以了。

```c++
void Model::Darw(Shader shader) {
    for (auto &meshe : meshes) {
        meshe.Draw(shader);
    }
}
```

&emsp;&emsp;而构造方法则是层层调用解析函数，最外层是 loadModel 方法，在 loadModel 里使用 Assimp 读取 obj 文件以及贴图文件的文件夹地址，然后调用解析节点方法（processNode）：

```c++
Model::Model(char *path, bool gamma) : gammaCorrection(gamma) {
    loadModel(path);
}

void Model::loadModel(std::string path) {
    Assimp::Importer importer;
    const aiScene *scene = importer.ReadFile(path, aiProcess_Triangulate | aiProcess_FlipUVs | aiProcess_CalcTangentSpace); //翻转UV坐标

    if (!scene || scene->mFlags & AI_SCENE_FLAGS_INCOMPLETE || !scene->mRootNode) {
        std::cout << "ERROR::ASSIMP::" << importer.GetErrorString() << std::endl;
        return;
    }

    directory = path.substr(0, path.find_last_of('/'));

    processNode(scene->mRootNode, scene);
}
```

&emsp;&emsp;我们之前说了，每个节点中存放的是 Mesh 在 Scene 对象里的数组的索引和子节点，所以 processNode 的实现则是依据索引解析 Mesh 对象并将其保存到 Model 的 meshes 数组里，同时递归子节点。

```c++
void Model::processNode(aiNode *node, const aiScene *scene) {

    for (unsigned int i = 0; i < node->mNumMeshes; ++i) {
        aiMesh *mesh = scene->mMeshes[node->mMeshes[i]];
        meshes.push_back(processMesh(mesh, scene));
    }

    for (unsigned int i = 0; i < node->mNumChildren; ++i) {
        processNode(node->mChildren[i], scene);
    }

}
```

&emsp;&emsp;解析 Mesh 对象则是使用了 processMesh 方法，里边的内容也不难考虑，即将 assimp 的 Mesh 里存放的数据挨个转移到我们自定义的 Mesh 类对象里，包括顶点数据 Vertex 对象以及材质 Texture 对象，而 assimp 的Mesh 类只有材质的索引属性，所以需要我们调用读取材质的方法（loadMaterialTextures）：

```c++
Mesh Model::processMesh(aiMesh *mesh, const aiScene *scene) {
    std::vector<Vertex> vertices;
    std::vector<unsigned int> indices;
    std::vector<Texture> textures;

    //Vertex
    for (unsigned int i = 0; i < mesh->mNumVertices; ++i) {

        Vertex vertex;
        vertex.Position = glm::vec3(mesh->mVertices[i].x, mesh->mVertices[i].y, mesh->mVertices[i].z);
        vertex.Normal = glm::vec3(mesh->mNormals[i].x, mesh->mNormals[i].y, mesh->mNormals[i].z);
        if (mesh->mTextureCoords[0]) {
            vertex.TextCoords = glm::vec2(mesh->mTextureCoords[0][i].x, mesh->mTextureCoords[0][i].y);
        } else {
            vertex.TextCoords = glm::vec2(0.0f, 0.0f);
        }
        if(mesh->mTangents){
            vertex.Tangent = glm::vec3(mesh->mTangents[i].x, mesh->mTangents[i].y, mesh->mTangents[i].z);
        }
        if(mesh->mBitangents){
            vertex.Bitangent = glm::vec3(mesh->mBitangents[i].x, mesh->mBitangents[i].y, mesh->mBitangents[i].z);
        }

        vertices.push_back(vertex);
    }

    //Indices
    for (unsigned int i = 0; i < mesh->mNumFaces; ++i) {
        aiFace face = mesh->mFaces[i];
        for (unsigned int j = 0; j < face.mNumIndices; ++j) {
            indices.push_back(face.mIndices[j]);
        }
    }

    //Textures
    aiMaterial *material = scene->mMaterials[mesh->mMaterialIndex];
    std::vector<Texture> diffuseMaps =
            loadMaterialTextures(material, aiTextureType_DIFFUSE, "texture_diffuse");
    textures.insert(textures.end(), diffuseMaps.begin(), diffuseMaps.end());

    std::vector<Texture> specularMaps =
            loadMaterialTextures(material, aiTextureType_SPECULAR, "texture_specular");
    textures.insert(textures.end(), specularMaps.begin(), specularMaps.end());

    std::vector<Texture> normalMaps =
            loadMaterialTextures(material, aiTextureType_NORMALS ,"texture_normal");
    textures.insert(textures.end(), normalMaps.begin(), normalMaps.end());

    std::vector<Texture> heightMaps =
            loadMaterialTextures(material, aiTextureType_HEIGHT, "texture_height");
    textures.insert(textures.end(), heightMaps.begin(), heightMaps.end());

    return Mesh(vertices, indices, textures);

}
```

&emsp;&emsp;除此之外还有从 scene 对象里读取材质以及从文件里读取贴图的两个方法：

```c++
std::vector<Texture> Model::loadMaterialTextures(aiMaterial *mat, aiTextureType type, std::string typeName) {
    std::vector<Texture> textures;
    for (unsigned int i = 0; i < mat->GetTextureCount(type); i++) {
        aiString str;
        mat->GetTexture(type, i, &str);
        // check if texture was loaded before and if so, continue to next iteration: skip loading a new texture
        bool skip = false;
        for (auto &j : textures_loaded) {
            if (std::strcmp(j.path.data(), str.C_Str()) == 0) {
                textures.push_back(j);
                skip = true; // a texture with the same filepath has already been loaded, continue to next one. (optimization)
                break;
            }
        }
        if (!skip) {   // if texture hasn't been loaded already, load it
            Texture texture;
            texture.id = TextureFromFile(str.C_Str(), this->directory);
            texture.type = typeName;
            texture.path = str.C_Str();
            textures.push_back(texture);
            textures_loaded.push_back(texture);  // store it as texture loaded for entire model, to ensure we won't unnecesery load duplicate textures.
        }
    }
    return textures;
}


unsigned int TextureFromFile(const char *path, const std::string &directory, bool gamma) {
    std::string filename = std::string(path);
    filename = directory + '/' + filename;

    unsigned int textureID;
    glGenTextures(1, &textureID);

    int width, height, nrComponents;
    unsigned char *data = stbi_load(filename.c_str(), &width, &height, &nrComponents, 0);
    if (data) {
        GLenum format;
        if (nrComponents == 1)
            format = GL_RED;
        else if (nrComponents == 3)
            format = GL_RGB;
        else if (nrComponents == 4)
            format = GL_RGBA;

        glBindTexture(GL_TEXTURE_2D, textureID);
        glTexImage2D(GL_TEXTURE_2D, 0, format, width, height, 0, format, GL_UNSIGNED_BYTE, data);
        glGenerateMipmap(GL_TEXTURE_2D);

        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_REPEAT);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_REPEAT);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR_MIPMAP_LINEAR);
        glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);

        stbi_image_free(data);
    } else {
        std::cout << "Texture failed to load at path: " << path << std::endl;
        stbi_image_free(data);
    }

    return textureID;
}
```

