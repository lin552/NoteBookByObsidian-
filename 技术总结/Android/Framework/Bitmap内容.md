---
创建时间: 2025-03-14T18:39:00
作者: wangxiaoming
tags:
  - Bitmap
---

#### Bitmap介绍

1. **Bitmap的物理定义**  
    Bitmap是Android系统中表示图像的内存对象，本质是一个**像素矩阵**，存储了每个像素的颜色信息（如ARGB值）。例如，一个1000×1000像素的图片，在内存中对应一个100万像素的Bitmap，每个像素占用4字节（ARGB_8888模式），总内存为3.8MB
    
2. ​**Bitmap的不可变性**  
    Bitmap对象一旦创建，其像素数据无法直接修改。若需编辑图片（如旋转、裁剪），需通过`Bitmap.createBitmap()`生成新对象，旧对象需调用`recycle()`主动释放内存
    
3. ​**色彩模式与内存优化**
    - ​**ARGB_8888**：全彩带透明度，每像素4字节（默认模式）
    - ​**RGB_565**：无透明度，每像素2字节（内存节省50%）
    - ​**ALPHA_8**：仅透明度通道，每像素1字节  
        开发者可通过`BitmapFactory.Options.inPreferredConfig`指定色彩模式

#### 性能优化策略

- ​**异步加载**：使用Glide/Picasso等库后台解码，避免主线程阻塞（代码示例见网页6）
- ​**内存复用**：通过`BitmapFactory.Options.inBitmap`复用已回收的Bitmap内存，减少GC频率
- ​**采样压缩**：设置`inSampleSize=2`可降低分辨率至1/4，内存占用减少75%

#### Bitmap使用
