<center><font face="黑体" size=13>QT学习</font></center>



# 使用QT完成老师布置的项目，积累的一些关于QT的操作，做一个备份

**学习的心得，在任务布置下来的时候，感觉优点难。应为只学过c++，但老师要求只用c++做GUI程序。考虑到MFC的复杂性，所以选用QT做为开发工具，就涉及到学习QT，下面讲讲自己学习的感受：**

* 我认为先把基础语法，和重要的控件，事件和槽过一遍，不需要学得多精细，只需要知道这个东西是干嘛的，有什么样应用场景可以用到。
* 然后拿一个小项目跟着练手。最好是一个视频的项目，有讲解。或者在GitHub上找一些项目（注意找最近几年的，不然时间早的可能和现在的开发环境不兼容，而且代码的注释比较多的），目的是让我们对开发一个小项目有一个整体的认识。
* 最后就是结合自己知道的一些控件与应用的场景，还有搜索引擎进行开发，想实现某个功能就在网上进行搜索如何实现。这样以项目的方式来学习的感觉印象比较深刻，也可以怎加自信心（一个个功能的实现）

## 1. QT连接SQL server数据库

### （1）首先要配置数据库操作

参

考SQL server的配置：[Qt连接SQL server数据库_Bruce-XIAO的博客-CSDN博客_qt连接sqlserver数据库](https://blog.csdn.net/ccsuxwz/article/details/72875376)

```c++
//使用到的头文件
#include<QSqlQuery>
#include<QSqlError>
```



### （2) 数据库连接的具体代码

```c++
  需要在pro文件当中引用QT       +=  sql

    QSqlDatabase db = QSqlDatabase::addDatabase("QODBC");   //数据库驱动类型为SQL Server
    qDebug()<<"ODBC driver?"<<db.isValid();
    QString dsn = QString::fromLocal8Bit("QTDSN");      //数据源名称
    db.setHostName("localhost");                        //选择本地主机，127.0.1.1
    db.setDatabaseName(dsn);                            //设置数据源名称
    db.setUserName("sasa");                               //登录用户
    db.setPassword("sasasa");                              //密码
    if(!db.open())                                      //打开数据库
     {
        qDebug()<<db.lastError().text();
        QMessageBox::critical(0, QObject::tr("Database error"), db.lastError().text());
        return;                                   //打开失败
     }
    else
     {
       qDebug()<<"database open success!";
     }
```

### （3）对数据库的增删查改

```c++
//例1，对数据库进行插入操作
 QString str=QString("insert into 用户表(账号,密码,用户名,电话,性别,头像)"
                            " values ('%1','%2','%3','%4','%5','%6')").arg(count).arg(pwd)
                .arg(name).arg(call).arg(sex).arg(imgpath);
        QSqlQuery query;
        if(query.exec(str))//执行成功
        {
           QMessageBox::about(this,"提示","注册成功！");
        }
        else//执行失败
        {
            //获取错误信息并提示错误信息
            qDebug()<<query.lastError().text();
        }
```

### （4）QT获取数据库的报错信息

```c++
qDebug()<<query.lastError().text();
```

### （5）获取数据库的内容进行验证

```c++
 //数据库查询该账号的密码
        QSqlQuery query;
        QString count1=ui->lineEdit->text();
        QString str=QString("select 密码 from 用户表 where 账号='%1'").arg(count1);
        query.exec(str);
       //如果账号存在，就看密码是否匹配
        if(!query.next())//账号不存在
        {
            QMessageBox::warning(this,"提示","该账户不存在");
            return;
        }
        else
        {
            QString tempwd=query.value(0).toString();//数据库中的密码
            QString inputpwd=ui->lineEdit_pwd->text();//输入的密码
            if(tempwd.compare(inputpwd)!=0)//如果不相等
            {
            QMessageBox::warning(this,"提示","密码错误");
            return;
            }
        }
```



## 2. Lambda格式来做信号和槽的连接

```c++
//例子1，使用lambda格式来监听窗体中&QPushBUtton的点击事件，并做相应的事件响应
connect(ui->pBtnEnroll,&QPushButton::clicked,this,[=](){
    QString count=ui->linEdtCount->text();
    QString pwd=ui->linEdtPwd->text();
}
```



## 3. QT对字符串的操作

### （1）获取特定字符的位置，抓取想要的字符串。

```c++
QString tumunr="shkdhdk.shskh"
int x=timunr.indexOf(".");//查看"."的位置
QString temp=timunr.mid(0,x);//截取获取题目代号
```

#### （2）数据库获取的字符内容转换成int类型

```c++
sql=QString("select 贡献值 from 用户表 where 账号='%1'").arg(glbaccount);
query.exec(sql);
query.next();//指向第一条数据
int contribut=query.value(0).toInt();
```



## 4. QT对时间的操作

### (1)获取现在的时间

有时需要用时间来对文件进行命令，防止文件的重复。或者获取当前时间来使用的场景。

```c++
#include<QDateTime>//头文件的引用
 QDateTime curDateTime=QDateTime::currentDateTime();
 QString time=curDateTime.toString("yyMMddhhmmss");//获取时间年取后两位，如果要是四位数的年份的化的用yyyy
```

## 5. QT不同窗体之间的切换与信息传递

### （1）使用全局变量来传递窗体之间的信息参数

在一个窗体的顶部设置全局变量

```c++
//注意全局变量设置的位置
QString picname="";//定义一个全局变量，存文字解答的图片路径，供图片展示窗体使用
QString vidopathname="";//定义一个全局变量，存视频解答的图片路径，供频频播放窗体使用
MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
    , ui(new Ui::MainWindow)
{
    ui->setupUi(this);
}
```

在另一个窗体引用这一个全局变量

```c++
extern QString vidopathname;//应用主窗体的视频路径
```



## 6. QT播放视频

```c++
QT       +=  multimedia multimediawidgets//需要在pro文件中添加
#include<QVideoWidget>//包含的头文件
    QVideoWidget* widget=new QVideoWidget(this);   //视频播放控件
    QMediaPlayer* player=new QMediaPlayer();   //播放器
    widget->resize(ui->label->size());
    widget->setGeometry(50,100,widget->width(),widget->height());//设置位置
    player->setVideoOutput(widget);
    QString vidopath=vidopathname;//视频的地址
    player->setMedia(QUrl::fromLocalFile(vidopath));
    player->play();
    connect(ui->pushButton,&QPushButton::clicked,this,[=](){//暂停按钮
        player->pause();
    });
    connect(ui->pushButton_2,&QPushButton::clicked,this,[=](){//播放按钮
        player->play();
    });
```



## 7. QT展现图片,鼠标对图片的放大

### （1）用窗体的背景图来来展现我们想展现的图片

```c++
void PictureShow::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    QPixmap pix;
    pix.load(picname);//picname为要展现的图片的路径
    painter.drawPixmap(x,y,w,h,pix);
}
```

### （2）鼠标滑轮对图片的放大缩小

```c++
void PictureShow::wheelEvent(QWheelEvent *event)
{
    if (event->delta()>0) {
        w=w*0.8;
        h=h*0.8;
        this->update();//上滑,缩小
         } else { //下滑,放大
        w=w*1.2;
        h=h*1.2;
        this->update();
         }
         event->accept();
}
```

### （2）鼠标左击拖动图片移动图片

```c++
void PictureShow::mouseMoveEvent(QMouseEvent *event)
{
    Qt::MouseButtons btns=event->buttons();
    if(Qt::LeftButton==(btns &Qt::LeftButton))//鼠标左键按下移动事件
    {
        x=event->x()-temp.x();
        y=event->y()-temp.y();
        this->update();
    }
}
```



## 8. QT设置背景图

### （1）给一个窗体设置一个背景图片

使用场景：为了软件的美化

```c++
#include<QPainter>//使用到的头文件
```



```c++
//需要子啊对应的.h文件中重写void paintEvent(QPaintEvent *)函数
void enrollDialog::paintEvent(QPaintEvent *)//绘制背景图
{
    QPainter painter(this);
    QPixmap pix;
    pix.load(":/res/注册背景.jpeg");
    painter.drawPixmap(0,0,this->width(),this->height(),pix);
    //其中前两个参数为背景图开始的位置（左上角），3，4参数为背景图的大小，pix为背景图
}
```

### （2）给一个窗体设置多个背景图

```c++
void MainWindow::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    QPixmap pix;
    pix.load(":/res/主页背景5.jpeg");
    painter.drawPixmap(245,0,this->width()-245,this->height(),pix);
    pix.load(":/res/主页背景8.jpeg");
    painter.drawPixmap(0,0,245,this->height(),pix);
}
```



## 9. QT 播放音乐

### （1）播放音频文件

有时候需要做游戏的音效，播放背景音乐时

```c++
QSound *banksound=new QSound(":/souds/sound/coin2.wav");//播放移动的背景音乐
banksound->play();
QTimer::singleShot(3000,this,[=](){//3秒后关闭
       banksound->stop();
 });
```



## 10. QT根据文字播放播放对应的声音

有时后想要语音播放文字时使用

```c++
QT       += texttospeech//需要在pro文件中添加
```

### （1）根据文字内容语音播放

```c++
#include<QTextToSpeech>//文字转语音
QTextToSpeech *ttp=new QTextToSpeech(this);
QString str=QString("你输入的斜线数为%1，是大于二的奇数，最少移动的金币数目为%2，可获得%3种不同正方形,请看演   示").arg(n).arg(coinumts).arg("1");
ttp->say(str);
QTimer::singleShot(13000,this,[=](){//设置9秒，等语言播放完在移动
                delete ttp;//需要销毁才能在界面中重新绘制，不然会死机
            });
```



## 11. QT实现动画

### （1）mycoin是继承QPushButton的，给他设置一个移动变换的动画

```c++
#include<QPropertyAnimation>//使用头文件
void mycoin::movesite(int newx, int newy)//传入X，Y坐标并进行移动
{
    //创建动态对象
    QPropertyAnimation *animation=new QPropertyAnimation(this,"geometry");
    //创建动画时间间隔
    animation->setDuration(2000);
    //起始位置
    animation->setStartValue(QRect(this->x(),this->y(),this->width(),this->height()));
    //结束位置
    animation->setEndValue(QRect(newx,newy,this->width(),this->height()));
    //设置弹跳曲线
    animation->setEasingCurve(QEasingCurve::OutBounce);
    //执行动画
    animation->start();
}
```



## 12.  QT对文件的操作

### （1）获取文件的后缀名

有时我们想对文件复制并重命名的时候，需要获取文件的后缀。

```c++
QFileInfo fileInfo = QFileInfo(filename);//filename是文件的全名包括后缀名
QString houzhui=fileInfo.suffix();//使用suffix获取后缀名
```

### （2）对文件复制，并给复制的文件重命名，移动到相对位置

把文件重命名并复制到我们想要移动的地方

```c++
//拼凑图片的路径，账号+时间+原图片后缀
QString imgpath=QString("./userimg/%1%2.%3").arg(ui->linEdtCount->text()).arg(time).arg(houzhui);
//相对text为账号，time为获取的时间，houzhui为文件的后缀
QFile::copy(filename,imgpath);//把filename文件复制到imgpath（含命名）的相对路径之下
```

### （3）打开文件选择框选择自己想要的文件

使用场景：用户添加图片，用户添加视频的地方

**选择图片**

```c++
#include<QFileDialog>//所使用的头文件
filename=QFileDialog::getOpenFileName(this,tr("选择图像"),"",tr("Images (*.png *.bmp *.jpg)"));
//tr("选择图像")文件选择框的标题，tr("Images (*.png *.bmp *.jpg)")文件的类型
         if(filename.isEmpty())
               return;
           else
           {
              QImage img;
               if(!(img.load(filename))) //加载图像
              {
                  QMessageBox::information(this, tr("打开图像失败"),tr("打开图像失败!"));
                  return;
              }
           }
```

**选择视频**

```c++
QString filename = QFileDialog::getOpenFileName(this,tr("选择视频文件"),".",
                                                        tr("视频格式(*.mov *.avi *.mp4 *.flv *.mkv)"));
        if(filename.isEmpty())
            return;
```



## 13. QT资源文件位置的使用

QT对资源文件存放有两种方式，不同方式的读取速度不同。但在程序中要动态的添加和删除资源文件时还是觉得使用想对路劲比较方便。详细资源文件的操作请参考：[Qt图片资源存放的两种方式_阿衰0110的博客-CSDN博客_qt保存图片](https://blog.csdn.net/m0_50669075/article/details/119381763?spm=1001.2014.3001.5506)

## 14. QT正确的显示中文字符，不是乱码

有些时候QT展现出来的中文字符是乱码，想要其正常显示

```c++
//在头文件中使用
#pragma execution_character_set("utf-8")
```

## 15. QT窗体图标，控件图标的显示方法

### （1）窗体图标与名称

```c++
this->setWindowTitle(QString("注册"));//设置title
this->setWindowFlags(Qt::CustomizeWindowHint | Qt::WindowCloseButtonHint);//去掉问好
this->setWindowIcon(QPixmap(":/res/page.png"));//设置图标
```

### （2）利用lable控件来显示图片，作为图标

在一个控件前面使用一个lable控件，用lable来显示图标

```c++

QPixmap pix.load(":/res/账号.png");//账号图标
pix=pix.scaled(ui->label_9->width(),ui->label_9->height(),Qt::IgnoreAspectRatio, Qt::SmoothTransformation);
//设置图标的大小和呈现的方式
ui->label_9->setPixmap(pix);//设置图标
```



### （3）Qpushbutton可以显示图标

```c++
    QPixmap pix;
    pix.load(":/res/确认.png");//确认注册的图标
    pix = pix.scaled(120,160, Qt::IgnoreAspectRatio, Qt::SmoothTransformation);
    ui->pBtnEnroll->setIcon(QIcon(pix));//给注册添加图标
    ui->pBtnEnroll->setIconSize(QSize(ui->pBtnEnroll->width(), ui->pBtnEnroll->height()));
    ui->pBtnEnroll->setFlat(true);//边框是否突起
    ui->pBtnEnroll->setStyleSheet("border: 0px"); //消除边框
```

### (4)给radiobutton添加图标

```c++
 pix.load(":/res/男店员.png");//男性别图标
 pix=pix.scaled(ui->rdBtnMan->width(),ui->rdBtnMan->height(),Qt::IgnoreAspectRatio,                  Qt::SmoothTransformation);
 ui->rdBtnMan->setIcon(pix);// ui->rdBtnMan为radiobutton
 ui->rdBtnMan->setIconSize(QSize(80,120));
```

## 16. QT的QPushButton

### （1）设置图标

```c++
    QPixmap pix;
    pix.load(":/res/确认.png");//确认注册的图标
    pix = pix.scaled(120,160, Qt::IgnoreAspectRatio, Qt::SmoothTransformation);
    ui->pBtnEnroll->setIcon(QIcon(pix));//给注册添加图标
    ui->pBtnEnroll->setIconSize(QSize(ui->pBtnEnroll->width(), ui->pBtnEnroll->height()));
    ui->pBtnEnroll->setFlat(true);
    ui->pBtnEnroll->setStyleSheet("border: 0px"); //消除边框
```

### （2）更改填充的颜色

利用颜色我们可以提示用户可不可点击。

```c++
 ui->pushButton_3->setStyleSheet("QPushButton{background-color:orange;}");//更改颜色
```

### （3）鼠标按下事件，图标发生变化

```c++
void MyPushBotton::mousePressEvent(QMouseEvent *e)
{
    if(this->pressImgPath!="")//传入的按下图片不为空
    {
        QPixmap pix;
        bool ret =pix.load(this->pressImgPath);
        if(!ret)
        {
            qDebug()<<"图片加载失败";
            return;
        }
        this->setFixedSize(pix.width(),pix.height());//图片的原始大小显示出来
       // this->setFixedSize(200,200);
        this->setStyleSheet("QPushButton{border:0px;}");

        this->setIcon(pix);//不设置图标不会显示
        this->setIconSize(QSize(pix.width(),pix.height()));
    }
    //让父类执行其他的操作
    return QPushButton::mousePressEvent(e);
}
```



## 17. QT的listWidget

利用listWidget我们可以建立一个列表来显示，比如音乐列表，题目列表等。

### （1）监听双击事件

```c++
connect(ui->listWidget,&QListWidget::currentTextChanged,this,[=](){
    //触发内容
}
```

### （2）获取当前选项的内容

```c++
 QString timunr=ui->listWidget->currentItem()->text();//获取当前选项的内容
```

### （3）添加背景图

```c++
#include<QPalette>//头文件
QPalette palette1;//给listwidget加上背景图
palette1.setBrush(QPalette::Base, QBrush(QPixmap(":/res/主页背景6.jpeg")));
ui->listWidget->setPalette(palette1);
```



## 18. Qt的widget容器

widget是在一个窗体中容纳其他控件的容器，可以实现整体隐藏，设置背景图

### （1）给widget设置背景图

```c++
#include<QPalette>//给widget设置背景图
ui->widget->setAutoFillBackground(true);//给widget加上背景图
QPalette palette=ui->widget->palette();
palette.setBrush(QPalette::Window,QBrush(QPixmap(":/res/主页背景10.jpeg").scaled(ui->widget-     >size(),Qt::IgnoreAspectRatio,Qt::SmoothTransformation))); // 使用平滑的缩放方式
ui->widget->setPalette(palette);
```

## 19. QT的textbrowser

利用textbrowser我们可以做一些浏览文字的效果

### （1）设置背景图片

```c++
#include<QPalette>//头文件   
QPalette palette1;//给listwidget加上背景图
palette1.setBrush(QPalette::Base, QBrush(QPixmap(":/res/主页背景6.jpeg")));
ui->listWidget->setPalette(palette1);
palette1.setBrush(QPalette::Base, QBrush(QPixmap(":/res/主页背景11.jpeg")));//给textbrowser添加背景图
ui->textBrowser->setPalette(palette1);
```

## 20. QT遍历窗体找到自己想要的控件（并进行删除）等

```c++
int coincount=this->children().count();
        for(int i=0;i<coincount;i++)//访问所有控件
        {
            //判断是否为自己定义的控件
            if(this->children().at(i)->inherits("mycoin"))
            {
                mycoin *bt=(mycoin *)(this->children().at(i));
                if((bt->posx==2)&&(bt->posy==2))
                {
                    QSound *banksound=new QSound(":/souds/sound/ConFlipSound.wav");
                    banksound->play();
                    bt->movesite(bt->graphposx,bt->graphposy+50);
                    CreatMoveCoin(bt->graphposx,bt->graphposy);//把原先位置的金币的颜色变成灰色
                }
            }
        }
```

