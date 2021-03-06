# Paint System
The Paint System of Qt is consist of `QPainter, QPaintEngine and QPaintDevice`. QPainter is used to perform drawing operations; QPaintDevice is an abstraction of a two-dimensional space, this two-dimensional space allows QPainter to draw on it, which is the space where QPainter works; QPaintEngine provides a brush (QPainter) to draw on different devices Unified interface. The QPaintEngine class is used between QPainter and QPaintDevice and is usually transparent to developers. Unless you need to customize a device, you don't need to care about the QPaintEngine class. We can understand QPainter as a paintbrush; understand QPaintDevice as a place where paintbrushes are used, such as paper, screen, etc. For paper and screen, we must use different paintbrushes to draw. In order to use one paintbrush uniformly, we designed QPaintEngine Class, this class allows different papers and screens to use one type of brush.


## QPainter
```cpp
QPainter painter(qPaintDevice);
```
`qPaintDevice` is indicated to the subclass of QPaintDevice, such as QWidget, QImage, QPicture and QPrinter. And if it's graphic components, the `paintEvent()` of the QWidget is usually redefined. When the Paint Device needs to be repainted, such as when the component appear, overwritten and reappear. You could use `repaint()` or `update()` function to trigger the `paintEvent()` to handle that event.<br>
<br>
The members of the QPainter class: `QBrush`, `QPen` and `QFont`.
### QBrush
QBrush defines the filling mode of QPainter, which has some properties such as style, color, gradient, and texture.
<hr/>

#### Style 
`style()` defines the filling style, using the `Qt::BrushStyle` for enumeration, the default value is `Qt::NoBrush`, that is, no filling is performed.<br><br>
![image](./figures/QBrushStyle.png)
<hr/>

#### Color
`color()` defines the color of the fill mode. This color can be a Qt predefined color constant, which is [`Qt::GlobalColor`](https://doc.qt.io/qt-5/qt.html#GlobalColor-enum), or it can be any QColor object.
```cpp
#include <QColorDialog>
QColor color = QColorDialog::getColor(Qt::black,this,tr("Color Selector"),QColorDialog::ShowAlphaChannel);
// Set the deflaut color to the black
// QColorDialog::ShowAlphaChannel allow the user to select the alpha component of a color.
```
Ones could use the Qt standard dialog box to get select the color and stores it in the QColor object.<br>
[`QColorDialog::ColorDialogOption`](https://doc.qt.io/qt-5/qcolordialog.html#ColorDialogOption-enum)
<hr/>

#### Gradient
`gradient()` defines a gradient fill. This property is only valid when the style is one of `Qt::LinearGradientPattern`, `Qt::RadialGradientPattern` or `Qt::ConicalGradientPattern`. Gradients can be represented by QGradient objects. Qt provides three gradients: QLinearGradient, QConicalGradient and QRadialGradient, they are all subclasses of QGradient. Gradient colors need to be specified using two attributes: stop-point and color. 
```cpp
void QGradient::setColorAt( qreal postion, const QColor &color)
```
You can use the `QGradient::setColorAt()` function to set a single stop-point and color. The stop-point is represented by a value between 0 and 1, and 0 means Start-point, 1 means End-point. And ones could also indicate the color in the assigned position.<br>
* QLinearGradient 
```cpp
QPainter painter(this);
QLinearGradient linearGradient(0, 0, 100, 100); // gradient from (0,0) to (100,100)
linearGradient.setColorAt(0, Qt::white);
linearGradient.setColorAt(0.5, Qt::green);
linearGradient.setColorAt(0, Qt::black);
painter.setBrush(QBrush(linearGradient));
painter.drawRect(10, 10, 50, 50); //draw a rectangle from (10,10) to (50,50) with linearGradient
```
Then could create a rectangle from (10,10) to (50,50) that with linearGradient that white color in start-point, green in mid-point and black in the end. From the code above, we could know that the four parameter of the `QLinearGradient` is the 2D position of start point and end point. Ones could rewrite the code like :
```cpp
QLinearGradient linearGradient(QPoint(0,0), QPoint(100,100));
// or using QPointF to store the coordinate postion in float data and display the gradient more accurately
```
* QConicalGradient
```cpp
QPainter painter(this);
const int R = 100; // declare the radius of the circle
QConicalGradient conicalGradient(0, 0, 0);
conicalGradient.setColorAt(0, Qt::red);
conicalGradient.setColorAt(60/360, Qt::yellow);
conicalGradient.setColorAt(120/360, Qt::green);
conicalGradient.setColorAt(180/360, Qt::cyan);
conicalGradient.setColorAt(240/360, Qt::blue);
conicalGradient.setColorAt(300/360, Qt::magenta);
conicalGradient.setColorAt(1, Qt::red);

painter.translate(R, R);
painter.setPen(Qt::NoPen);
painter.setBrush(QBrush brush(conicalGradient));
painter.drawEllipse(QPoint(0, 0), R, R);
```
Then that could create a color wheel, without edge lines. From the code above, we could find that the first and second parameters of `QConicalGradient` is the postition of the center point, so that could be replace by `QPoint`, and the third parameter is initial angle of Cartesian Coordination. Besides, `QPainter::translate(x, y)` function means to set the origin of the coordinate system to the point (x, y). In the above code, the position (100,100) is set to be the origin of the coordinate.
* QRadialGradient
```cpp
QPainter painter(this);
QRadialGradient radialGradient(QPoint(200,200),200,QPoint(250,250),50);
radialGradient.setColorAt(0, Qt::white);
radialGradient.setColorAt(0.5,Qt::red);
radialGradient.setColorAt(1,Qt::yellow);
radialGradient.setSpread(QGradient::RepeatSpread); //spread the gradient in repeat mode
painter.setBrush(QBrush(radialGradient));
painter.drawRect(0, 0, 400, 400);
```
To realize the parameter of the QRadialGradient function, we could observe the constructor of it :
```cpp
QRadialGradient(const QPoint &centerPoint, qreal centerRadius, const QPoint &focalPoint, qreal focalRadius);
QRadialGradient(qreal cx, qreal cy, qreal centerRadius, qreal fx, qreal fy, qreal focalRadius);
```
( _cx_ , _cy_ ) is the position of the center, and ( _fx_ , _fy_ ) is the position of the focal point. Besides, using `QGradient::setSpread()` function could set the ways how does the gradient color spread in the area outside the gradient area. `QGradient::PadSpread` is the default and the color outside the gradient area is the color of end point; `QGradient::RepeatSpread` will repeat the gradient outside the area and `QGradient::ReflectSpread` will reflect, in opposite way of, the gradient outside the area.<br>
<br>
Note that `QGradient::Spread` only has an effect on linear gradients and radial gradients, because these two types of gradients have boundaries, while the conical gradients have a gradient range of 0 to 360 degrees, so there is no gradient boundary.<br><br>
![image](./figures/QBrushGradient.png)
<hr/>

#### Texture
```cpp
QBrush.setTexture(QPixmap pixmap);
``` 
Set the brush style be the QPixmap, and in this condition, the brush only has single color. Without giving the `Qt::TexturePattern` as you calling `setTexture`, QBrush will spontaneously use the `style()` to be the `Qt::TexturePattern`.
```cpp
QBrush.setTextureImage(QImage image);
``` 
Could also set the brush be a image.


### QPen
Used to draw the edges of geometric figures, composed of parameters such as color, width, line style. QPen contains different properties such as brush, width, style, capStyle and joinStyle.
<hr/>

#### Brush
`brush()` is used to fill the lines drawn by the brush, it decides the color or image of the pen. The detailed introductions could see the color of QBrush.

#### Width
The width of the pen `width()` or `widthF()` (float data) defines the width of the pen. Note that there is no line with a width of 0. Suppose you set the width to 0, QPainter will still draw a line, and the width of this line is 1 pixel. In other words, the brush width is usually at least 1 pixel.
<hr/>

#### Style
`style()` defines the style of the line. <br><br>
![image](./figures/QPenStyle.png) <br>
Ones can use `setDashPattern()` function to define the style of the pen :
```cpp
QPen pen;
QVector<qreal> dashes;
qreal space = 4;

dashes << 1 << space << 3 << space << 9 << space << 27 << space << 9 << space;
pen.setDashPattern(dashes);
```
<hr/>

#### CapStyle
`capStyle()` defines the end of the line drawn using QPainter. <br><br>
![image](./figures/QPenCapStyle01.png) <br>
The difference between them is that `Qt::SquareCap` is a square cap that contains the last point and is covered by half the line width; `Qt::FlatCap` does not contain the last point; `Qt::RoundCap` contains the last point Round end : <br><br>
![image](./figures/QPenCapStyle02.png)
<hr/>

#### JoinStyle
`joinStyle()` defines how the two lines are connected.<br><br>
![image](./figures/QPenJoinStyle02.png)<br><br>
<hr/>

#### Construct
Using the constructor to reset the style of the pen.
```cpp
QPainter painter(this);
QPen pen(Qt::black, 5, Qt::SolidLine, Qt::RoundCap, Qt::RoundJoin);
painter.setPen(pen);
```
Or using set function to indicate the style respectively :
```cpp
QPainter painter(this);
Qpen pen; //create a default pen

pen.setBrush(Qt::black);
pen.setWidth(5);
pen.setStyle(Qt::SolidLine);
pen.setCapStyle(Qt::RoundCap);
pen.setJoinStyle(Qt::RoundJoin);

painter.setPen(pen);
```
The advantage of using the constructor is that the code is shorter, but the meaning of the parameters is not clear; using the set function is just the opposite. Besides, the default of QPen is `Qt::black`, 0 pixel, `Qt::squareCap` and `Qt::BevelJoin`.


### QFont
The `drawText()` function in the QPainter class can draw text on the drawing device, and also could use `setFont()` function to set the font for drawing.
```cpp
QPainter painter(this);
painter.drawText(100,100,"texts"); //draw a string "texts" in the position (100,100)
```
In the another constructor of the `drawText()` function, we could control the position of the text in a rectangle.
```cpp
void QPainter::drawText(const QRect &rectangle, int flags, const QString &text);
```
Its first parameter specifies the rectangle where the text is drawn; the second parameter specifies the alignment of the text in the rectangle, which is defined by the [`Qt::AlignmentFlag`](https://doc.qt.io/qt-5/qt.html#AlignmentFlag-enum) enumeration variable. Different alignments can also use the `|` operator at the same time, here you can also use some other flags defined by `Qt::TextFlag`, such as automatic line wrapping. The third parameter is the text to be drawn.
```cpp
QPainter painter(this);
QRect rect(10,10,100,100); // define a rectangle between (10,10) and (100,100)

painter.drawRect(rect)
painter.setPen(QColor(Qt::black));
painter.drawText(rect, Qt::AlignHCenter, "texts");
```
<hr/>

#### QFont
In order to draw beautiful text, you can use the `QFont` class to set the text font.
```cpp
QFont font("Microsoft YaHei UI", 15, QFont::Bold, true);
font.setOverline(true);
font.setUnderline(true); // set the top and buttom line for the texts
font.setCapitalization(Qt::smallCaps);
font.setLetterSpacing(Qt::AbsoluteSpacing,10);

painter.setFont(font);
painter.setPen(Qt::black);
painter.drawText(150,100,"texts");
```
The QFont font object is created here, and the constructor used is :
```cpp
QFont::Qfont(const QString &family, int pointSize = -1, int weight = -1, bool italic = false)
```
The first parameter sets the family attribute of the font. The font family used here is "Microsoft YaHei UI". You can use the QFontDatabase class to get all the supported fonts; the second parameter is the point size, the default size is 15; the third parameter is the weight attribute, Bold is used here; whether to use italics for the last property setting.<br>
<br>
Besides, several other functions are used to format the font. `setCapitalization()` set the space between characters according to the enum of [`QFont::Capitalization`](https://doc.qt.io/qt-5/qfont.html#Capitalization-enum).
```cpp
void QFont::setLetterSpacing(QFont::SpacingType type, qreal spacing)
```
Sets the letter spacing for the font to spacing and the type of spacing to type.Letter spacing changes the default spacing between individual letters in the font. The spacing between the letters can be made smaller as well as larger either in percentage of the character width or in pixels, depending on the selected spacing type. Finally, call the `setFont()` function to use the font, and use another overloaded form of the `drawText()` function to draw the text at the point (150,100).
<hr/>

#### QFontMetrics
After setting the font, you can use the `fontMetrics()` method to obtain the geometric information of the font, such as _ascent_ (the distance from the highest point of the character to the baseline of the character), _descent_ (the distance from the lowest point of the character to the bottom of the character), _leading_ ( The space value between two lines) _height_ (the height of the font when printing, equivalent to _ascent_ + _descent_ + 1, 1 pixel is the height of the bottom line of the character) and _linespacing_ (_height_ + _leading_).<br><br>
![image](./figures/QFontMetrics.PNG)
### Member Functions
Can only draw graphics in `QWidget::paintEvent`
```cpp
void paintEvent(QPaintEvent *event){
    QPainter painter(this);
    painter.drawLine(QPoint(10,10), QPoint(100,100));
}
```
Different types of drawing functions, see [`QPainter::members`](https://doc.qt.io/qt-5/qpainter-members.html) to obtain detailed parameters of each functions.



## QPaintDevice 
Qt provides four classes to process image data: `QImage`, `QPixmap`, `QBitmap` and `QPicture`, they are also commonly used drawing devices. Among them, QImage is mainly used for I/O processing. It optimizes I/O processing operations, and can also be used to directly access and manipulate pixels; QPixmap is mainly used to optimized the displayment  images on the screen; QBitmap is a subclass of QPixmap, which is a convenient class used to process images with a color depth of 1, that is, only black and white can be displayed; QPicture is used to record and replay QPainter commands.
### QPixmap
`QPixmap` inherits QPaintDevice. You can create QPainter and draw on it. You can also directly specify the pattern to load the graphics files supported by Qt, such as BMP, GIF, JPG, JPEG, PNG. And use QPainter's `drawPixmap( )` function drawn on other drawing devices. You can set `QPixmap` on QLabel and QPushButton to display images. QPixmap is designed and optimized for the image displayed on the screen. It depends on the native graphics engine of the platform, so the display of some effects (such as anti-aliasing) may have inconsistent results on different platforms.
```cpp
QPainter painter(this);
QPixmap pix;
pix.load("filePath"); // absolute or relative path
painter.drawPixmap(0, 0, 100, 80, pix); 
```
The `drawPixmap()` function draws a picture in a given rectangle, where the upper left corner of the rectangle is (0, 0) point, width 100, height 80. If this is not the same size as the picture, the picture will be stretched by default.
<hr/>

#### Size
We can use the `scaled()` function in the `QPixmap` class to zoom in and out of the picture :
```cpp
QPainter painter(this);
QPixmap pix;
pix.load("filePath"); // absolute or relative path

qreal width = pix.width();
qreal height = pix.height();
pix = pix.scaled(width*2, height*2, Qt::KeepAspectRatio); 
// Expand the width and height of the picture twice, and keep the ratio of width to height unchanged within a given rectangle
painter.drawPix(50, 50, pix);
```
![image](./figures/AspectRatio.PNG)<br>
`Qt::IgnoreAspectRatio` does not keep the aspect ratio of the picture, `Qt::KeepAspectRatio` keeps the aspect ratio in the given rectangle and `Qt::KeepAspectRatioByExpanding`  also keeps the aspect ratio, but may exceed the given rectangle.
<hr/>

#### QBitmap
`QBitmap` is a subclass of `QPixmap`, which provides monochrome images, which can be used to make cursors (QCursor) or brushes (QBrush) objects.
### QImage
`QImage` is mainly designed for image I/O, image access and pixel modification. And `QImage` uses Qt's own drawing engine, which can provide the same image rendering effect on different platforms, and can directly access the specified pixels through methods such as `setPixpel()`, `pixel()`. [Pixel Manipulation](https://doc.qt.io/qt-5/qimage.html#pixel-manipulation)
```cpp
// 32 bit
QImage = image(3, 3, QImage::Format_RGB32);
QRge vales;

value = qRgb(189, 149, 39); // 0xffbd9527
image.setPixel(1, 1, value);

value = qRgb(122, 163, 39); // 0xff7aa327
image.setPixel(0, 1, value);
image.setPixel(1, 0, value);

value = qRgb(237, 187, 51); // 0xffedba31
image.setPixel(2, 1, value);
```
Interface prototype for drawing `QImage` in QPainter :
```cpp
void QPainter::drawImage(int x, int y, const QImage &image, int sx = 0, int sy = 0, int sw = -1, int sh = -1, Qt::ImageConversionFlags flags = Qt::AutoColor )
```
Where _x_ and _y_ are the drawing positions, _sx_ and _sy_ are the coordinates of the upper left corner of the image, _sw_ and _sh_ are the size of the specified image, if both are 0 or negative, the entire image is displayed.
```cpp
QPainter painter(this);
QImage image;
image.load("filePath"); // absolute or relative path
painter.drawImage(0, 0, image);
```
When the picture is large, we can first load the picture through `QImage`, then scale the picture to the required size, and finally convert it to `QPixmap` for display.
```CPP
QPainter painter(this);
QImage image;
image.load("filaPath");
QPixmap pix = QPixmap::fromImage(image.scaled(size(), Qt::KeepAspectRatio));
painter.drawPixmap(0, 0, pix);
```
### QPicture 
`QPicture` is a drawing device used to record and replay Qpainter's drawing instructions. Use `begin()` function to draw on QPicture, use `end()` to end drawing and use `save()` to save to the file.
```cpp
QPainter painter;
QPicture picture;
painter.begin(&picture); 
painter.drawRect(0, 0, 100, 100);
painter.end();
picture.save("draw_record.pic");
```
If you need to replay the drawing commands, create a new `QPicture` object, use `load()` to reload the saved file, and then draw QPicture on the specified drawing device.
```cpp
QPainter painter;
QPicture picture;
picture.load("draw_record.pic");  
painter.begin(this);
painter.drawPicture(0, 0, picture); 
painter.end(); 
```
