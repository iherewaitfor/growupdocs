# Protobuf

## 一、Protobuf是什么
Protobuf是一种平台无关、语言无关、可扩展且轻便高效的序列化数据结构的协议，可以用于网络通信和数据存储。

## 二、Protobuf优势劣势
![avatar](../img/protobuf/good.png)

## 三、Protobuf使用
!> protobuf源码地址： https://github.com/protocolbuffers/protobuf.git  

### 使用流程：
	1. 编译源码，生成protoc.exe文件。
	2. 通过protoc.exe文件将.proto格式文件制作生成pb文件。
	3. 链接protobuf的lib库。
	4. 将pb文件加入工程。
    5. 使用pb文件里面的对象进行读写操作。

### 1. 使用cmake生成工程项目
![avatar](../img/protobuf/cmake.png)

编译后，在输出目录有个文件protoc.exe

### 2. 使用protoc.exe生成pb文件
```c
// SRC_DIR   .proto文件存放目录
// --cpp_out  指示编译器生成C++代码,DST_DIR为生成文件存放目录
// test.proto 待编译的协议文件
protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/test.proto
```
![avatar](../img/protobuf/protoc.png)
使用终端到protoc.exe目录，执行以上命令后，在存放目录会生成 xxxx.pb.cc 和 xxxx.pb.h文件  
>proto代码编译  
语法：  
protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR --python_out=DST_DIR --go_out=DST_DIR --ruby_out=DST_DIR --javanano_out=DST_DIR --objc_out=DST_DIR --csharp_out=DST_DIR path/to/file.proto  
参数说明：  
--proto_path 当proto文件中使用import时指定的导入文件的位置  
--cpp_out c++输出目录  
--java_out java输出目录  
--python_out  
--go_out  
--ruby_out ruby输出目录  
--objc_out oc输出目录  
--csharp_out c#输出目录  
path/to/file.proto为要编译的proto文件  

这里示例的proto文件  
```c
syntax = "proto3";

message SearchRequest{
	string query=1;
	int32 page_number=2;
	int32 result_per_page=3;
}
```

### 3. 链接protobuf的lib库
在使用protobuf的项目中链接protobuf的库文件
```c
#pragma comment(lib, "../release/libprotoc.lib")
#pragma comment(lib, "../release/libprotobuf.lib")
#pragma comment(lib, "../release/libprotobuf-lite.lib")
```

### 4. 将pb文件导入工程
```c
#include "test.pb.h"
```

### 5. pb文件使用读写操作
写入数据及保存文件
```c
    SearchRequest sr;
	sr.set_query("test value");
	sr.set_page_number(1314.0001f);
	sr.set_result_per_page(520);
	fstream output("myfile", ios::out | ios::binary);
	sr.SerializeToOstream(&output);
```
从文件中读取数据
```c
    fstream input("myfile", ios::in | ios::binary);
	SearchRequest sr;
	sr.ParseFromIstream(&input);
	std::string query = sr.query();
	float number = sr.page_number();
	float page = sr.result_per_page();
```

### 6. svga文件读取数据
```c
	com::opensource::svga::MovieEntity me;
	if (!me.ParseFromArray(fileData.c_str(), fileData.size()))
	{
		return false; 
	}

	d->m_version = QString::fromStdString(me.version());
	d->m_width = me.params().viewboxwidth();
	d->m_height = me.params().viewboxheight();
	d->m_fps = me.params().fps();
	d->m_frames = me.params().frames();

    // 图片
	::google::protobuf::Map<::std::string, ::std::string>::const_iterator it = me.images().begin();
	for (; it != me.images().end(); ++it)
	{
		QPixmap pix;
		pix.loadFromData((uchar*)it->second.c_str(), it->second.size());
		d->m_mapPixmap[QString::fromStdString(it->first)] = pix;
	}

    // 动作
	for (int i = 0; i<me.sprites_size(); ++i)
	{
		SvgaSprite spriteMsg;
		::com::opensource::svga::SpriteEntity se = me.sprites(i);
		spriteMsg.imageName = QString::fromStdString(se.imagekey()); 
		for (int f = 0; f<se.frames_size(); ++f)
		{
			::com::opensource::svga::FrameEntity fe = se.frames(f);
			if (!fe.has_layout())
			{
				spriteMsg.frames.push_back(SvgaFrame());
				continue;
			}

			SvgaFrame frameData(true);
			frameData.alpha = fe.alpha();
			frameData.layout = QRect(fe.layout().x(), fe.layout().y(),
				fe.layout().width(),fe.layout().height());
			frameData.mat.setMatrix(fe.transform().a(), fe.transform().b(), fe.transform().c(), fe.transform().d(),
				fe.transform().tx(), fe.transform().ty());
			frameData.clipPath = QString::fromStdString(fe.clippath());
			spriteMsg.frames.push_back(frameData);
		}
		d->m_listSvgaMsg.push_back(spriteMsg);
	}

```







 

