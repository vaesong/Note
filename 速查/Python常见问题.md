# TensorFlow存储图片问题

Erorr:‘tensorflow.python.framework.ops.EagerTensor‘ object has no attribute **‘__array_interface_‘**

```Python
    X = ( image_data[0] * 0.5 + 0.5 ) * 255 #归一化之后的图片数组，如果你的数组是正确的，就不需要归一化。
    print(X)#输出x,观察自己的数组类型，我的是tf.Tensor类型，所以张量转换类型很重要
    img = tf.cast(X, dtype=tf.uint8)# 这一步转换张量数据类型很重要
    img = tf.image.encode_png(img)# 编码回图片
    savepath = "E:\\..\\..\\"#写自己保存图片的地址
    savepicture_name = savepath + str(ii + 1) + '.jpg'  #字符串拼接保存图像的地址，ii指循环，如果要保存多张图片则可以加上for循环语句使用，保存一张就去掉ii. for 循环语句的位置要自己考虑一下。
    with tf.io.gfile.GFile(savepicture_name, 'wb') as file:# 保存
        file.write(img.numpy())
        
    # 法2   或者直接归一化然后处理
    img = (img[0] * 127.5 + 127.5).numpy().astype(np.uint8)
    img = PIL.Image.fromarray(img)
    img.save("images/" + str(i) + "_photo" + ".jpg")
```

**注释：**输出的X的[数组](https://so.csdn.net/so/search?q=数组&spm=1001.2101.3001.7020)类型，是张量类型，所以保存时要转换：下图红色笔已经标出来

![](https://img-blog.csdnimg.cn/20200827224121399.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NzI2ODc3Nw==,size_16,color_FFFFFF,t_70#pic_center)
