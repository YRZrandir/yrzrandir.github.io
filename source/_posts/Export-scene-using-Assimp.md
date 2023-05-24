---
title: Export scene using Assimp
date: 2023-05-24 22:32:48
tags:
---

# 使用Assimp导出网格为任意格式
生成网格的顶点和面后，使用Assimp库导出为任意格式。
注意建立Assimp的aiScene对象时，除了网格数据，还有其他必须的辅助数据也要初始化，才能正常导出。
```c
void WriteMesh(
    const std::vector<Point_3>& vertices,   //顶点数据
    const std::vector<Triangle>& faces,     //顶点索引
    const std::string& path                 //导出路径
)
{
    Assimp::Exporter exporter;
    auto scene = std::make_unique<aiScene>();

    //每个aiScene必须有mRootNode
    scene->mRootNode = new aiNode();
    //建立mRootNode的mMeshes数组,表示位于这个节点下的网格编号. 这里只有0号网格.
    scene->mRootNode->mNumMeshes = 1;
    scene->mRootNode->mMeshes = new unsigned[1];
    scene->mRootNode->mMeshes[0] = 0;

    //必须有一个材质，这里设为空材质.
    scene->mNumMaterials = 1;
    scene->mMaterials = new aiMaterial*[]{ new aiMaterial() };

    //必须初始化mMetadata.
    scene->mMetaData = new aiMetadata();

    //场景的网格数据
    scene->mNumMeshes = 1;
    scene->mMeshes = new aiMesh*[1]{ new aiMesh() };
    aiMesh* m = scene->mMeshes[0];
    m->mNumFaces = faces.size();
    m->mNumVertices = vertices.size();
    m->mVertices = new aiVector3D[m->mNumVertices];
    m->mFaces = new aiFace[m->mNumFaces];
    m->mPrimitiveTypes = aiPrimitiveType_TRIANGLE;
    for(int i = 0; i < m->mNumVertices; i++)
    {
        m->mVertices[i] = aiVector3D(vertices[i].x(), vertices[i].y(), vertices[i].z());
    }
    for(int i = 0; i < m->mNumFaces; i++)
    {
        m->mFaces[i].mNumIndices = 3;
        m->mFaces[i].mIndices = new unsigned int[3];
        m->mFaces[i].mIndices[0] = faces[i][0];
        m->mFaces[i].mIndices[1] = faces[i][1];
        m->mFaces[i].mIndices[2] = faces[i][2];
    }

    //根据后缀区分ply和stl格式按照二进制还是文本格式存储。"plyb"和"stlb"代表导出二进制格式文件。
    //这里把ply和stl都设为了导出二进制格式。
    std::string postfix = path.substr(path.rfind('.') + 1);
    if(postfix == std::string("ply"))
    {
        postfix = "plyb";
    }
    else if (postfix == std::string("stl"))
    {
        postfix = "stlb";
    }
    exporter.Export(scene.get(), postfix, path);
    //注意不需要释放前面new动态分配的内存，aiScene在析构时会自动释放。
}
```