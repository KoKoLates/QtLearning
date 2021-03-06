# Double Buffering
Without double buffering painting method, when using the mouse to drag out a geometric shapes and then found a lot of shadows. Try dragging the mouse quickly and slowly to draw the geometric shapes respectively. As a result, you will find that the faster the dragging speed, the less shadowing. In fact, in the process of dragging the mouse, the screen has been refreshed many times. It can also be understood that the `paintEvent()` function has been executed many times, and the geometric shapes will be drawn every time as it's executed.

### Add private member in PaintWidget class
Add an template widget. if you are drawing, that is, as the mouse button has not been released, then drawing on the template widget. Only when the mouse button is released, so that could paint on the real `image` widget.<br>
<br>
Adding the private members in the `paintwidget.h` header file:
```cpp
QImage tempImage;
bool scribbling;
```

### Initialize the constructor
Initialize the variable of constructor in `paintwidget.cpp`:
```cpp
scribbling = false;
```

### Fix paintEvent function
```cpp
void PaintWidget::paintEvent(QPaintEvent *){
    QPainter painter(this);
    if(scribbling == true ) painter.drawImage(0,0,tempImage);
    // if it's drawing, the mouse is moving with clicked, then paint on tempImage
    else painter.drawImage(0,0,image);
    // if the mouse is release, then store in the image
}
```

### Fix the QMouseEvent
Changing the contents of the mousePressEvent and mouseReleaseEvent:
```cpp
void PaintWidget::mousePressEvent(QMouseEvent *event){
    if(event->buttons() == Qt::LeftButton){
        lastPoint = event->pos();
        scribbling = true; //marking
    }
}

void PaintWidget::mouseReleaseEvent(QMouseEvent *event){
    scribbling = false; //cancel the mark
    if(type != Pen) paint(image);
}
```
When the left mouse button is pressed, then marking the drawing, and when the button is released, we cancel the mark.

### Double Buffers
When drawing, all contents would drawn on a drawing device, and then painting entire image to the part for display. Using double-buffers can avoid flickering during display. Starting from Qt 4.0, all painting module of QWidget components are automatically possess double buffering. So that it's generally not necessary to use double buffering code int the `paintEvent()` function to avoid flickering.<br>
<br>
The reason why double buffering cannot be used on the brush or eraser is because the traces drawn will not be retained, and the image will only be formed when the mouse is released at the end, so there will only be one point at the end.
