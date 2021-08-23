### java 绘图常用知识点总结

### 1、指定字体

+ 将指定的字体库放置到项目的资源文件夹下
+ 以文件流的方式加载文件
+ 缓存字体，避免重复加载。

```
// 获取指定的字体
private static java.awt.Font getSelfDefinedFont(String fontName) {
	InputStream inputStream=This.class.getResourceAsStream(MessageFormat.format("/font/{0}.TTF",fontName)); 
	java.awt.Font font = null;
	try {
		if (map.containsKey(fontName)) {
			return map.get(fontName);
		}
		font = java.awt.Font.createFont(java.awt.Font.TRUETYPE_FONT, inputStream);
		map.put(fontName, font);
	} catch (Exception e) {
		log.error("font load failed {}", e.getMessage());
		return null;
	}
	return font;
}
```

```
font.deriveFont(java.awt.Font.PLAIN, fontSize);
设置字号
```



### 2、获取指定文本的宽高

用指定的字体、字号、文本生成图片

```
 // 获取font的样式应用在str上的整个矩形
Rectangle2D r =
		font.getStringBounds(
				content,
				new FontRenderContext(
						AffineTransform.getScaleInstance(1, 1), false, false));
// 获取单个字符的高度
int unitHeight = (int) Math.floor(r.getHeight());
// 获取整个str用了font样式的宽度这里用四舍五入后+1保证宽度绝对能容纳这个字符串作为图片的宽度
int width = (int) Math.round(r.getWidth()) + 1;
// 把单个字符的高度+3保证高度绝对能容纳字符串作为图片的高度
int height = unitHeight + 3;
// 创建图片
BufferedImage image = new BufferedImage(width, height, BufferedImage.TYPE_INT_BGR);
Graphics g = image.getGraphics();
g.setColor(Color.WHITE);
// 先用白色填充整张图片,也就是背景
g.fillRect(0, 0, width, height);
// 在换成黑色
g.setColor(Color.black);
// 设置画笔字体
g.setFont(font);
// 画出字符串
g.drawString(content, 0, font.getSize());
g.dispose();

// 返回数据流
ByteArrayOutputStream out = new ByteArrayOutputStream();
image = imageTransparencyhandle(image);
ImageIO.write(image, "png", out);
byte[] b = out.toByteArray();
```



### 3、图片透明化处理

```
private BufferedImage imageTransparencyhandle(BufferedImage image) {
// 高度和宽度
int height = image.getHeight();
int width = image.getWidth();

// 生产背景透明和内容透明的图片
ImageIcon imageIcon = new ImageIcon(image);
BufferedImage bufferedImage =
		new BufferedImage(width, height, BufferedImage.TYPE_4BYTE_ABGR);
Graphics2D g2D = (Graphics2D) bufferedImage.getGraphics(); // 获取画笔
g2D.drawImage(imageIcon.getImage(), 0, 0, null); // 绘制Image的图片

int alpha = 0; // 图片透明度
// 外层遍历是Y轴的像素
for (int y = bufferedImage.getMinY(); y < bufferedImage.getHeight(); y++) {
	// 内层遍历是X轴的像素
	for (int x = bufferedImage.getMinX(); x < bufferedImage.getWidth(); x++) {
		int rgb = bufferedImage.getRGB(x, y);
		// 对当前颜色判断是否在指定区间内
		if (colorInRange(rgb)) {
			alpha = 0;
		} else {
			// 设置为不透明
			alpha = 255;
		}
		// #AARRGGBB 最前两位为透明度
		rgb = (alpha << 24) | (rgb & 0x00ffffff);
		bufferedImage.setRGB(x, y, rgb);
	}
}
return bufferedImage;
}
// 判断是背景还是内容
public static boolean colorInRange(int color) {
int red = (color & 0xff0000) >> 16; // 获取color(RGB)中R位
int green = (color & 0x00ff00) >> 8; // 获取color(RGB)中G位
int blue = (color & 0x0000ff); // 获取color(RGB)中B位
// 通过RGB三分量来判断当前颜色是否在指定的颜色区间内
if (red >= color_range && green >= color_range && blue >= color_range) {
	return true;
}
;
return false;
}

// 色差范围0~255
public static int color_range = 210;
```



