# 使用DX渲染SVGA

## 一、SVGA是什么
SVGA 是一种跨平台的开源动画格式，同时兼容 iOS / Android / Web。SVGA 除了使用简单，性能卓越，同时让动画开发分工明确，各自专注各自的领域，大大减少动画交互的沟通成本，提升开发效率。  
介绍页：http://svga.io/intro.html   
格式说明：https://github.com/svga/SVGA-Format


## 二、DX环境创建
### 1. 初始化Direct3D
```c
// 初始化 Direct3D 
	if((IDirect3D9 *pD3D = Direct3DCreate9(D3D_SDK_VERSION)) == NULL)
		return E_FAIL;
```
### 2. 创建3D设备
```c
	D3DDISPLAYMODE d3ddm;
	pD3D->GetAdapterDisplayMode(D3DADAPTER_DEFAULT, &d3ddm);

	d3dpp.BackBufferWidth  = DesiredWidth;
	d3dpp.BackBufferHeight = DesiredHeight;
	d3dpp.BackBufferFormat = D3DFMT_A8R8G8B8; //d3ddm.Format;
	d3dpp.BackBufferCount  = 1;
	d3dpp.SwapEffect       = D3DSWAPEFFECT_DISCARD;
	d3dpp.Windowed         = TRUE;
	d3dpp.EnableAutoDepthStencil = TRUE;
	d3dpp.AutoDepthStencilFormat = D3DFMT_D24S8;
	d3dpp.FullScreen_RefreshRateInHz = D3DPRESENT_RATE_DEFAULT;
	d3dpp.PresentationInterval       = D3DPRESENT_INTERVAL_IMMEDIATE;
	// 检测顶点运算
	D3DCAPS9 caps;
	pD3D->GetDeviceCaps(D3DADAPTER_DEFAULT, D3DDEVTYPE_HAL, &caps);
	DWORD Flags = 0;
	if (caps.DevCaps & D3DDEVCAPS_HWTRANSFORMANDLIGHT)
	{
		Flags = D3DCREATE_HARDWARE_VERTEXPROCESSING | D3DCREATE_PUREDEVICE | D3DCREATE_NOWINDOWCHANGES;
	}
	else
	{
		Flags = D3DCREATE_SOFTWARE_VERTEXPROCESSING | D3DCREATE_NOWINDOWCHANGES;
	}
	// 多线程
	if(MultiThreaded == TRUE)
		Flags |= D3DCREATE_MULTITHREADED;
	// 创建3D设备 
	if(FAILED(hr = pD3D->CreateDevice(
		D3DADAPTER_DEFAULT, 
		D3DDEVTYPE_HAL, hWnd, Flags, 
		&d3dpp, &pD3DDevice)))
		return hr;
```
### 3. 创建ID3DXSprite对象
```c
	D3DXCreateSprite(pD3DDevice, &m_pSprite);
```

## 三、解析SVGA文件看protobuf文档

## 四、渲染
	1) 每次Begin后都会重置为默认的Render状态，所以在Begin后，再去重新设置。
	2) 发现透明位置合成时，会使用最后一层的透明值，目前改成使用alpha最大值。  
   
```c
	if(SUCCEEDED(m_pSprite->Begin(D3DXSPRITE_ALPHABLEND))) 
	{
		// 透明处理
		m_pD3DDevice->SetRenderState(D3DRS_ALPHABLENDENABLE, TRUE);
		m_pD3DDevice->SetRenderState(D3DRS_SRCBLEND, D3DBLEND_SRCALPHA);
		m_pD3DDevice->SetRenderState(D3DRS_DESTBLEND, D3DBLEND_INVSRCALPHA);
		// Alpha通道处理，此处功能为使用Alpha最大值
		m_pD3DDevice->SetRenderState(D3DRS_SRCBLENDALPHA, D3DBLEND_SRCALPHA);
		m_pD3DDevice->SetRenderState(D3DRS_DESTBLENDALPHA, D3DBLEND_DESTALPHA);
		m_pD3DDevice->SetRenderState(D3DRS_BLENDOPALPHA, D3DBLENDOP_MAX);
		m_pD3DDevice->SetRenderState(D3DRS_SEPARATEALPHABLENDENABLE, TRUE); 

		std::list<SvgaSprite> &svgaSpriteList = m_svgaResource.getSvgaSprites();
		for (std::list<SvgaSprite>::iterator it = svgaSpriteList.begin(); it != svgaSpriteList.end(); ++it)
		{ 
			SvgaSprite& svgaMsg = *it; 
			if (frameIndex >= svgaMsg.frames.size())
			{
				continue;
			}
			SvgaFrame& svgaFrame = svgaMsg.frames[frameIndex]; 
			if (!svgaFrame.bInit)
			{
				continue;
			}

			// get image 
			CTexture2D* pTexture = GetImageTexture(svgaMsg.imageName, svgaFrame.clipPath);
			if (pTexture && pTexture->GetTexture())
			{
				RECT rc;
				rc.left = svgaFrame.layout.x();
				rc.top = svgaFrame.layout.y();
				rc.right = svgaFrame.layout.right();
				rc.bottom = svgaFrame.layout.bottom();

				// 设置旋转矩阵
				D3DXMATRIX mat(
					svgaFrame.mat.m11(), svgaFrame.mat.m12(), 0, 0,
					svgaFrame.mat.m21(), svgaFrame.mat.m22(), 0, 0,
					0, 0, 1, 0,
					svgaFrame.mat.dx(), svgaFrame.mat.dy(), 0, 1
					); 
				m_pSprite->SetTransform(&mat); 
				// 设置透明度
				D3DXCOLOR color(255.0f,255.0f,255.0f,svgaFrame.alpha);
				m_pSprite->Draw(pTexture->GetTexture(), &rc, NULL, NULL, color);
			}
		}
		m_pSprite->End();
	}
```

## 五、 裁剪图片
	目前使用Dx来做裁剪比较复杂，所以这里使用CPU去做裁剪，然后再给回DX使用。
```c
QImage SvgaResource::drawImageClipPath(QImage& srcImg, QString& clipPath)
{
	QImage image(srcImg.width(), srcImg.height(), QImage::Format_ARGB32);
	image.fill(Qt::transparent);
	QPainter imagePainter(&image);
	imagePainter.setRenderHint(QPainter::SmoothPixmapTransform, true); 

	if (!clipPath.isEmpty())
	{
		QPainterPath painterPath;
		StringToCliPath(clipPath,painterPath);
		imagePainter.setClipping(true);
		imagePainter.setClipPath(painterPath);
		imagePainter.drawImage(0,0,srcImg);
		imagePainter.setClipping(false);
	}
	else
	{
		imagePainter.drawImage(0,0,srcImg);
	}

	return image;
}

char SvgaResource::popListChar(QStringList& list)
{
	if (list.isEmpty())
	{
		return '0';
	}

	QString str = list.at(0);
	if (str.isEmpty())
	{
		list.pop_front();
		return '0';
	}

	char tmp = str.at(0).toAscii();
	if (str.size()>1)
	{
		list.replace(0,str.right(str.length()-1));
	}else{
		list.pop_front();
	}
	return tmp;
}

float SvgaResource::popListNumber(QStringList& list)
{
	if (list.isEmpty())
	{
		return 0;
	}

	float val = ::atof(list.at(0).toStdString().c_str());
	list.pop_front();
	return val;
}

void SvgaResource::StringToCliPath(QString& strClipath, QPainterPath& painterPath)
{
	QString strTemp = strClipath.replace(",", " ");
	QStringList nodeList = strTemp.split(" ");

	float posX = 0;
	float posY = 0;
	float posX1 = 0;
	float posY1 = 0;
	float posX2 = 0;
	float posY2 = 0;
	while (!nodeList.isEmpty())
	{
		switch (popListChar(nodeList)) 
		{
		case 'M':
			posX = popListNumber(nodeList);
			posY = popListNumber(nodeList);
			painterPath.moveTo(posX, posY);
			break;
		case 'm':
			posX += popListNumber(nodeList);
			posY += popListNumber(nodeList);
			painterPath.moveTo(posX, posY);
			break;
		case 'L':
			posX = popListNumber(nodeList);
			posY = popListNumber(nodeList);
			painterPath.lineTo(posX, posY);
			break;
		case 'l':
			posX += popListNumber(nodeList);
			posY += popListNumber(nodeList);
			painterPath.lineTo(posX, posY);
			break;
		case 'H':
			posX = popListNumber(nodeList);
			painterPath.lineTo(posX, posY);
			break;
		case 'h':
			posX += popListNumber(nodeList);
			painterPath.lineTo(posX, posY);
			break;
		case 'V':
			posY = popListNumber(nodeList);
			painterPath.lineTo(posX, posY);
			break;
		case 'v':
			posY += popListNumber(nodeList);
			painterPath.lineTo(posX, posY);
			break;
		case 'C':
			posX1 = popListNumber(nodeList);
			posY1 = popListNumber(nodeList);
			posX2 = popListNumber(nodeList);
			posY2 = popListNumber(nodeList);
			posX = popListNumber(nodeList);
			posY = popListNumber(nodeList);
			painterPath.cubicTo(posX1, posY1, posX2, posY2, posX, posY);
			break;
		case 'c':
			posX1 = posX + popListNumber(nodeList);
			posY1 = posY + popListNumber(nodeList);
			posX2 = posX + popListNumber(nodeList);
			posY2 = posY + popListNumber(nodeList);
			posX += popListNumber(nodeList);
			posY += popListNumber(nodeList);
			painterPath.cubicTo(posX1, posY1, posX2, posY2, posX, posY);
			break;
		case 'S':
			if (posX1 && posY1 && posX2 && posY2) {
				posX1 = posX - posX2 + posX;
				posY1 = posY - posY2 + posY;
				posX2 = popListNumber(nodeList);
				posY2 = popListNumber(nodeList);
				posX = popListNumber(nodeList);
				posY = popListNumber(nodeList);
				painterPath.cubicTo(posX1, posY1, posX2, posY2, posX, posY);
			} else {
				posX1 = popListNumber(nodeList);
				posY1 = popListNumber(nodeList);
				posX = popListNumber(nodeList);
				posY = popListNumber(nodeList);
				painterPath.quadTo(posX1, posY1, posX, posY);
			}
			break;
		case 's':
			if (posX1 && posY1 && posX2 && posY2) {
				posX1 = posX - posX2 + posX;
				posY1 = posY - posY2 + posY;
				posX2 = posX + popListNumber(nodeList);
				posY2 = posY + popListNumber(nodeList);
				posX += popListNumber(nodeList);
				posY += popListNumber(nodeList);
				painterPath.cubicTo(posX1, posY1, posX2, posY2, posX, posY);
			} else {
				posX1 = posX + popListNumber(nodeList);
				posY1 = posY + popListNumber(nodeList);
				posX += popListNumber(nodeList);
				posY += popListNumber(nodeList);
				painterPath.quadTo(posX1, posY1, posX, posY);
			}
			break;
		case 'Q':
			posX1 = popListNumber(nodeList);
			posY1 = popListNumber(nodeList);
			posX = popListNumber(nodeList);
			posY = popListNumber(nodeList);
			painterPath.quadTo(posX1, posY1, posX, posY);
			break;
		case 'q':
			posX1 = posX + popListNumber(nodeList);
			posY1 = posY + popListNumber(nodeList);
			posX += popListNumber(nodeList);
			posY += popListNumber(nodeList);
			painterPath.quadTo(posX1, posY1, posX, posY);
			break;
		case 'A':
			break;
		case 'a':
			break;
		case 'Z':
		case 'z':
			painterPath.closeSubpath();
			break;
		default:
			break;
		}

	}
}
```

## 六. QImage/QPixmap转到纹理
```c
	// 从内存中的图片创建纹理
	void CTexture2D::LoadFromPixmap(IDirect3DDevice9* pD3DDevice, QPixmap& pixmap)
	{
		if (!pD3DDevice || !LPD3DXCreateTextureFromFileInMemoryExPtr)
		{
			return;
		}
		Release();

		QByteArray bytes;
		QBuffer buffer(&bytes);
		buffer.open(QIODevice::ReadWrite);
		pixmap.save(&buffer, "png", 100);
	

		LPD3DXCreateTextureFromFileInMemoryExPtr(pD3DDevice,(void*)buffer.data().data(), buffer.size(),
			pixmap.width(), pixmap.height(),
			D3DFMT_A8R8G8B8, 0, D3DFMT_UNKNOWN, D3DPOOL_MANAGED, D3DX_DEFAULT, D3DX_DEFAULT, 0, 
			&m_TextureInfo, NULL, &m_pTexture );

		m_Width = pixmap.width();
		m_Height = pixmap.height();
	}

	void CTexture2D::LoadFromImage(IDirect3DDevice9* pD3DDevice, QImage& image)
	{
		if (!pD3DDevice || !m_pTexture)
		{
			return;
		}
		if (image.width() != m_Width || image.height() != m_Height)
		{
			return;
		}
	
		D3DLOCKED_RECT lockRect;
		if (FAILED(m_pTexture->LockRect(0,&lockRect, NULL, 0)))
		{
			return  ;
		}
		unsigned char * pBitsSrc = image.bits();
		int srcCount = image.byteCount();
		unsigned char * pBitsDest = (unsigned char *)lockRect.pBits;

		memcpy_s(pBitsDest, m_Width*m_Height*4, pBitsSrc, srcCount);

		m_pTexture->UnlockRect(0);
	}
```



