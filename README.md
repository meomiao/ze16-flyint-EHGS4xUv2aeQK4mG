

# 前言




  模型较大的时候，出现卡顿，那么使用LOD（细节层次）进行层次细节调整，可以让原本卡顿的模型变得不卡顿。  本就是LOD介绍。



 

# Demo




  ![请添加图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241112101919401-629312304.gif)



 

# LOD




## 概述




  LOD也称为层次细节模型，是一种实时三维计算机图形技术，旨在通过根据物体在场景中的位置和重要性动态调整其渲染的详细程度，从而提高渲染效率和性能。  视点离物体近时，能观察到的模型细节丰富；视点远离模型时，观察到的细节逐渐模糊。系统绘图程序根据一定的判断条件，选择相应的细节进行显示，从而避免了因绘制那些意义相对不大的细节而造成的时间浪费，同时有效地协调了画面连续性与模型分辨率的关系。




## Osg::LOG节点




  在OSG中，LOD技术通过osg::LOD节点来实现。osg::LOD节点是一个特殊的场景节点，它可以包含多个子节点，每个子节点代表一个不同详细程度的模型。根据视点与物体的距离或屏幕上的像素大小，osg::LOD节点会选择相应的子节点进行渲染。




## 子节点的添加




  通过addChild方法，可以将不同详细程度的模型作为子节点添加到osg::LOD节点中。每个子节点都需要指定一个距离范围（或像素大小范围），在这个范围内，该子节点会被渲染。




## 距离和像素大小模式




  osg::LOD节点支持两种切换模式距离模式和像素大小模式。




* 距离模式：根据视点到物体包围盒中心的距离来选择子节点；
* 像素大小模式：根据物体在屏幕上的像素大小来选择子节点。




## 中心模式的设置




  对于距离模式，osg::LOD节点还支持两种中心模式：包围盒中心模式和自定义中心模式。包围盒中心模式使用物体的包围盒中心作为计算距离的点；自定义中心模式则允许用户指定一个自定义的中心点。  LOD技术的优点和应用




* 提高渲染效率：通过动态调整模型的详细程度，LOD技术可以显著减少需要渲染的多边形数量，从而提高渲染速度。
* 优化内存使用：虽然OSG中的osg::LOD节点会一次性载入所有模型进入内存，但它只是有选择地进行绘制，这仍然有助于优化内存使用，因为不需要为每个模型都分配独立的内存空间。此外，OSG还提供了osg::PagedLOD节点，它支持动态分页加载，可以根据需要来加载模型文件，进一步优化内存使用。
* 提升视觉效果：LOD技术可以在保证视觉效果的前提下，通过简化模型来减少渲染负担，从而允许开发者在场景中放置更多的物体或实现更复杂的视觉效果。
* 广泛的应用场景：LOD技术适用于各种需要高效渲染的三维场景，如城市规划、地形渲染、游戏开发等。在这些场景中，物体的数量和详细程度往往非常高，使用LOD技术可以显著提高渲染性能和用户体验。




## LOD技术的局限性




  尽管LOD技术具有诸多优点，但它也存在一些局限性。例如，在切换不同详细程度的模型时，可能会出现视觉上的跳跃现象，特别是当两个模型之间的详细程度差异较大时。此外，设计和管理不同详细程度的模型也需要一定的时间和资源投入。




## 其他




  OSG中的LOD技术是一种高效且灵活的三维渲染技术，它通过动态调整模型的详细程度来优化渲染性能和内存使用。在开发大规模三维场景时，LOD技术是一个不可或缺的工具。



 

# LOD实现




  模型就不多说了，关键就是osg::Lod结点，如何添加的问题：





```
            // 添加模式，0~100范围内使用线模型
            pLod->addChild(pGeode, 0, 100);

```



  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241112101919337-997154394.png)





```
           // 添加模型，非设置的范围内的都是这个
//            pLod->addChild(pGeode); // 不能使用，预想中是没设置的都使用这个，实际上这个函数实际无用，反正都不显示
           pLod->addChild(pGeode, 100, 1000);

```



  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241112101919777-571723634.png)



 

# Demo关键源码




## :[MeoMiao 萌喵加速](https://biqumo.org)绘制部分





```
// 绘图
{
#if 1
    // 第一个模型
    for(int partIndex = 0; partIndex < kMode.listPart.size(); partIndex++)
    {
        // 创建一个用户保存几何信息的对象
        osg::ref_ptr pGeometry = new osg::Geometry;
        // 创建四个顶点的数组
        osg::ref_ptr pVec3Array = new osg::Vec3Array;
        // 添加四个顶点
        pGeometry->setVertexArray(pVec3Array.get());

        // 创建四种颜色的数据
        osg::ref_ptr pVec4Array = new osg::Vec4Array;
        // 添加四种颜色
        pGeometry->setColorArray(pVec4Array.get());
        // 绑定颜色
        pGeometry->setColorBinding(osg::Geometry::BIND_PER_VERTEX);

        double r, g, b;
        r = 1.0f;
        g = 1.0f;
        b = 0.0f;
        for(int elementShellIndex = 0; elementShellIndex < kMode.listPart.at(partIndex).listElementShell.size(); elementShellIndex++)
        {
            //                               x     y     z
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).z));
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).z));
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).z));
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).z));


            //                               r    g    b    a(a设置无效，估计需要其他属性配合)
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));


        }
        // 注意：此处若不绑定画笔，则表示使用之前绑定的画笔

        // 为唯一的法线创建一个数组    法线: normal
        osg::ref_ptr pVec3ArrayNormal = new osg::Vec3Array;
        pGeometry->setNormalArray(pVec3ArrayNormal.get());
        pGeometry->setNormalBinding(osg::Geometry::BIND_OVERALL);
        pVec3ArrayNormal->push_back(osg::Vec3(0.0, -1.0, 0.0));

        // 由保存的数据绘制四个顶点的多边形
        pGeometry->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::QUADS, 0, kMode.listPart.at(partIndex).listElementShell.size() * 4));
//            pGeometry->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::QUADS, 0, 4));

        // 向Geode类添加几何体(Drawable)
        osg::ref_ptr pGeode = new osg::Geode;
        pGeode->addDrawable(pGeometry.get());
#if 1
        // 线宽模式
        {
            osg::ref_ptr pStateSet = pGeometry->getOrCreateStateSet();
            osg::ref_ptr pPolygonMode = new osg::PolygonMode(osg::PolygonMode::FRONT_AND_BACK, osg::PolygonMode::LINE);
            pStateSet->setAttribute(pPolygonMode);
        }
#endif
        // 添加模式，0~100范围内使用线模型
        pLod->addChild(pGeode, 0, 100);
        // 只是用一个部件
        break;
    }
#endif
#if 1
    // 第一个模型
    for(int partIndex = 0; partIndex < kMode.listPart.size(); partIndex++)
    {
        // 创建一个用户保存几何信息的对象
        osg::ref_ptr pGeometry = new osg::Geometry;
        // 创建四个顶点的数组
        osg::ref_ptr pVec3Array = new osg::Vec3Array;
        // 添加四个顶点
        pGeometry->setVertexArray(pVec3Array.get());

        // 创建四种颜色的数据
        osg::ref_ptr pVec4Array = new osg::Vec4Array;
        // 添加四种颜色
        pGeometry->setColorArray(pVec4Array.get());
        // 绑定颜色
        pGeometry->setColorBinding(osg::Geometry::BIND_PER_VERTEX);

        double r, g, b;
        r = 1.0f;
        g = 1.0f;
        b = 0.0f;
        for(int elementShellIndex = 0; elementShellIndex < kMode.listPart.at(partIndex).listElementShell.size(); elementShellIndex++)
        {
            //                               x     y     z
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n1).z));
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n2).z));
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n3).z));
            pVec3Array->push_back(osg::Vec3(kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).x,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).y,
                                            kMode.hashNid2Node.value(kMode.listPart.at(partIndex).listElementShell.at(elementShellIndex).n4).z));


            //                               r    g    b    a(a设置无效，估计需要其他属性配合)
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));
            pVec4Array->push_back(osg::Vec4(r, g, b, 1.0));


        }
        // 注意：此处若不绑定画笔，则表示使用之前绑定的画笔

        // 为唯一的法线创建一个数组    法线: normal
        osg::ref_ptr pVec3ArrayNormal = new osg::Vec3Array;
        pGeometry->setNormalArray(pVec3ArrayNormal.get());
        pGeometry->setNormalBinding(osg::Geometry::BIND_OVERALL);
        pVec3ArrayNormal->push_back(osg::Vec3(0.0, -1.0, 0.0));

        // 由保存的数据绘制四个顶点的多边形
        pGeometry->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::QUADS, 0, kMode.listPart.at(partIndex).listElementShell.size() * 4));
//            pGeometry->addPrimitiveSet(new osg::DrawArrays(osg::PrimitiveSet::QUADS, 0, 4));

        // 向Geode类添加几何体(Drawable)
        osg::ref_ptr pGeode = new osg::Geode;
        pGeode->addDrawable(pGeometry.get());
        // 添加模型，非设置的范围内的都是这个
//            pLod->addChild(pGeode); // 不能使用，预想中是没设置的都使用这个，实际上这个函数实际无用，反正都不显示
        pLod->addChild(pGeode, 100, 1000);
        // 只是用一个部件
        break;
    }
#endif
    pGroup->addChild(pLod);
}

```


 

# 工程模板v1\.34\.0




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241112101919356-918062943.png)



 

# 入坑




## 入坑：复制osg::ref\_ptr相关类失败




### 问题




  想深度复制一个模型，做lod，复制编译不过  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241112101919405-1778236180.png)




### 原理




  加了osg::ref\_ptr，不能常规方法复制，也尝试使用get()之后在\*取其类实体（非指针），也报错。  下面是浅拷贝的示例：  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241112101919776-892088057.png)




  无法直接深拷贝，osg::ref\_ptrosg::Geometry 本身并不提供直接的深拷贝功能，因为 osg::ref\_ptr 只是一个智能指针，它管理对象的生命周期，但并不关心对象的具体内容或如何复制这些内容。深拷贝通常涉及到对象的逐字段复制，这通常需要在对象类内部实现，或者通过外部函数来实现。  在 OpenSceneGraph（OSG）中，osg::Geometry 类没有内置的深拷贝方法，因此需要自己实现深拷贝逻辑。这通常包括复制几何体的所有属性，如顶点数组、颜色数组、法线数组、纹理坐标数组、图元集等。




## 解决




  直接复制前面一段代码，区别得地方调整下，当作新模型加入。




## 入坑二：lod添加节点不设置范围的不显示




### 问题




  lod添加节点不设置范围的不显示




### 原理




  这个函数看起来就是没用，改成都设置范围




### 解决




  ![在这里插入图片描述](https://img2024.cnblogs.com/blog/1971530/202411/1971530-20241112101919613-1070503118.png)



