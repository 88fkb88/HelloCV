# Opencv学习

说在前面的话：本文中提到的大部分代码都被笔者以“Chapters.zip”同步上传至Github。

### 任务目标：

1. 前言
2. 基础图象与视频读取相关操作
   - 图像读取与显示
   - 图像属性
   - 视频读取
   - 视频处理
3. 基础图像处理与⼏何变换
   - 色彩空间
   - 算子与滤波算法
   - 形态学操作
   - 图像的裁剪与缩放
   - 图形绘制与文字绘制
   - 仿射变换、投射变换
   - 颜色提取
   - 图形提取
4. 感悟
5. 鸣谢

### 一、前言

笔者经过漫长的对Linux的学习，最终遇到了最后的重头戏：Opencv。前面学习的东西都是在为Opencv的使用铺路，现在终于要开始接触这个强大的工具了。而笔者对opencv的学习主要是在c++背景下进行的。

### 二、基础图象与视频读取相关操作

##### 首先，在编写opencv程序之前，要优先找到用于Linux的头文件：

``` c++
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
```

这三个头文件都是opencv加上的，并不在我们经常编程的时候用的范围里面，但是编写opencv的时候就知道这几个头文件有多强大了。他们里面包含很多函数与指令，会帮我们做好很多事。

此外，就如同在写c++程序的时候，为了避免` std::cout ` 的麻烦，通常会在程序开头处加入` using namespace std; ` 。在opencv里面也是这样，有许多代码前面也要加入` cv:: ` 的，于是我们也可以在开头写上：

``` c++
using namespace cv;
```

##### 图像处理与显示

``` c++
string path="";//存放路径
Mat img；
img = imread(path);          //读取图像
img = imread(path, IMREAD_GRAYSCALE); // 灰度图像
img = imread(path, IMREAD_COLOR);//彩色图像

imshow("Image", img);//显示图像
waitKey(0);//等待，单位为毫秒，0代表无穷大

imwrite(path, img);//保存图像
```

##### 图像属性

```c++
img.cols   //列
img.raws   //行
img.channels()    //通道数
```

另外，opencv具有独特的BGR色彩模式，与常听说的RGB不同。顾名思义，BGR是指的蓝-绿-红，所以在编写程序的时候一定要注意颜色代码与RGB不同。

opencv也具有单个像素访问的功能，通常使用` img.at<Vec3b>(y,x) ` 来实现。

这个代码中的xy就是指的要访问像素的实际坐标，可以进行修改像素值、查找像素数据等操作。

##### 视频读取

如果想要在自己的虚拟机里面访问电脑摄像头，可以考虑在VMware的虚拟机栏目找到“可移动设备”然后将自己的摄像头连接至虚拟机即可。

``` c++
//访问网络摄像头
int main() {
	VideoCapture cap(0);//通常直接填0，如果有多个摄像头填摄像头编号
   	Mat img;

	while (true) {//使用循环逐帧读取视频
		cap.read(img);
		imshow("Image", img);
		waitKey(1);
	}
}
```



``` c++
//读取视频
int main() {
	string path = "这里放路径.mp4";
	VideoCapture cap(path);
	Mat img;

	while (true) {
		cap.read(img);
		imshow("Image", img);
		waitKey(20);//这个是每一帧之间间隔，可以用来控制视频播放速度
	}
}
```

##### 实时处理

在学习如何处理视频之前，应该先以图片为例学习处理命令：

``` c++
int main() {
	string path = "这里放路径.png";
	Mat img = imread(path);
	Mat imgGray, imgBlur, imgCanny, imgDil, imgErode;

	cvtColor(img, imgGray, COLOR_BGR2GRAY);//灰度处理
	GaussianBlur(imgGray, imgBlur, Size(7, 7), 5, 0);//模糊化处理
	Canny(imgBlur, imgCanny, 25,75);//提取边缘

	Mat kernel = getStructuringElement(MORPH_RECT, Size(3, 3));
	dilate(imgCanny, imgDil, kernel);//加粗边缘
	erode(imgDil, imgErode, kernel);//使边缘变细

	imshow("Image", img);
	imshow("Image Gray", imgGray);
	imshow("Image Blur", imgBlur);
	imshow("Image Canny", imgCanny);
	imshow("Image Dilation", imgDil);
	imshow("Image Erode", imgErode);
	waitKey(0);
}
```

处理后的图片会发生如下变化：

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-26%20101301.png?raw=true)

然后我们就可以接触到视频的处理了，笔者这里使用笔者自己录屏的一小段视频用作示例，具体视频笔者会上传至笔者的GitHub：

``` c++
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include <iostream>		//头文件
using namespace cv;
using namespace std;	//为了省事

int main() {
	string path = "video/videoPS.mp4";//路径
	VideoCapture cap(path);
	Mat img ;	
	int time0=time(0);	//这里用来计时，确定什么时候进行什么处理
    //下面这里是为了导出视频获取原视频格式
	double fps = cap.get(CAP_PROP_FPS);
	Size frameSize(cap.get(CAP_PROP_FRAME_WIDTH), cap.get(CAP_PROP_FRAME_HEIGHT));	
	VideoWriter writer("output.avi", VideoWriter::fourcc('M', 'J', 'P', 'G'), fps, frameSize);	
    //
	while(1){
		cap.read(img);//读取当前帧
		if(img.empty()){//视频播放完了结束程序
			break;
		}
		Mat imgPS=img.clone();
		int op=(int)time(0)-(int)time0;//确定现在时间
		if(op>=0&&op<4){//前4秒不处理
			putText(imgPS,"Nothing!",Point(100,100),FONT_HERSHEY_COMPLEX,3,Scalar(0,69,255),2);
		}
		else if(op>=4&&op<=8){//4-8秒灰度处理
			cvtColor (img,imgPS,COLOR_BGR2GRAY);
			putText(imgPS,"Gray!",Point(100,100),FONT_HERSHEY_COMPLEX,3,Scalar(0,69,255),2);
			cvtColor(imgPS, imgPS, COLOR_GRAY2BGR);//将单通道转化为三通道便于导出
		}
		else if(op>8&&op<=12){//8-12秒模糊处理
			GaussianBlur(imgPS,imgPS,Size(15,15),5,5);
			putText(imgPS,"Blur!",Point(100,100),FONT_HERSHEY_COMPLEX,3,Scalar(0,69,255),2);
		}
		else{//其余时间边缘化处理
			Canny(img,imgPS,25,75);
			putText(imgPS,"Canny!",Point(100,100),FONT_HERSHEY_COMPLEX,3,Scalar(255,255,255),5);
			cvtColor(imgPS, imgPS, COLOR_GRAY2BGR);//将单通道转化为三通道便于导出
		}
		writer.write(imgPS);//导出
		imshow("VideoPS",imgPS);
		waitKey(20);//20ms一帧
	}
    return 0;
}

```

### 三、基础图像处理与⼏何变换

##### 色彩空间

在opencv中，存在不同的色彩空间，比如BGR、灰度、HSV。BGR和灰度我们在前面都已经见过了，现在我们来看看HSV。

> - **HSV**：HSV颜色空间表示色调（Hue）、饱和度（Saturation）和亮度（Value）。

也就是说，我们通过把图像转换为HSV，可以使每种颜色之间区别更加分明，便于处理颜色。

``` c++
cvtColor(img, hsv, COLOR_BGR2HSV);
```

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-26%20120302.png?raw=true)

##### 各类算⼦与滤波算法

算子与滤波算法听起来似乎很高端，但是实际上就是之前笔者提到过的模糊处理与边缘化。

主流的模糊处理与边缘化：

``` c++
GaussianBlur(imgGray, imgBlur, Size(7, 7), 5, 0);//高斯滤波
Canny(imgBlur, imgCanny, 25,75);//坎尼边缘检测
```

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-26%20125357.png?raw=true)

此外，还有很多不同的其他算法也具有这些功能：
```c++
// 均值滤波
blur(img, blur_avg, Size(5, 5));

// 中值滤波 (对椒盐噪声效果好)
medianBlur(img, blur_median, 5);
    
// 双边滤波 (保持边缘)
bilateralFilter(img, blur_bilateral, 9, 75, 75);
```

```c++
// Sobel 算子
Sobel(img, sobel_x, CV_16S, 1, 0, 3);
Sobel(img, sobel_y, CV_16S, 0, 1, 3);
convertScaleAbs(sobel_x, sobel_x);
convertScaleAbs(sobel_y, sobel_y);
addWeighted(sobel_x, 0.5, sobel_y, 0.5, 0, sobel_combined);
    
// Scharr 算子 (对弱边缘更敏感)
Scharr(img, scharr_x, CV_16S, 1, 0);
Scharr(img, scharr_y, CV_16S, 0, 1);
convertScaleAbs(scharr_x, scharr_x);
convertScaleAbs(scharr_y, scharr_y);
addWeighted(scharr_x, 0.5, scharr_y, 0.5, 0, scharr_combined);
    
// Laplacian 算子
Laplacian(img, laplacian, CV_16S, 3);
convertScaleAbs(laplacian, laplacian);
    
// Canny 边缘检测 (最常用)
Canny(img, canny_edges, 50, 150);
```

##### 形态学操作

这里的形态学操作主要是指对黑白图像（例如执行Canny操作之后的图片）进行的一些操作，比如缩小或扩大白色区域等。

``` c++
// 腐蚀 - 缩小白色区域
erode(binary, eroded, kernel);

// 膨胀 - 扩大白色区域  
dilate(binary, dilated, kernel);

// 开运算 - 先腐蚀后膨胀 (去噪点)
morphologyEx(binary, opened, MORPH_OPEN, kernel);

// 闭运算 - 先膨胀后腐蚀 (填充空洞)
morphologyEx(binary, closed, MORPH_CLOSE, kernel);

// 形态学梯度 - 膨胀减腐蚀 (边缘检测)
morphologyEx(binary, gradient, MORPH_GRADIENT, kernel);
```

这里的“噪点”主要指的是意外出现在图像中的一些小点，通过这些指令可以让程序识别出他们并处理掉。

##### 图像的裁剪与缩放

``` c++
resize(img,imgResize,Size(),0.5,0.5);//缩放调整大小，0.5表示长/宽缩短为原来的一半
Rect roi(100,100,300,300);//框选目标裁剪位置
imgCrop=img(roi);//裁剪
```

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-26%20132521.png?raw=true)

（tips ：这里的矩形框的位置可以通过目测调整，也可以在画图等软件中通过鼠标悬停来获取该位置的坐标。）

##### 图形绘制和⽂字绘制

```c++
circle(img/*这里是要画的图像*/, Point(256, 256)/*这是圆心的坐标*/, 155/*圆的半径*/, Scalar(0, 69, 255)/*颜色BGR*/,2/*边缘粗细*/);//画圆
rectangle(img, Point(130, 226)/*左上角*/, Point(382, 286)/*右下角*/, Scalar(255, 255, 255), FILLED/*这里表示填满*/);//画矩形
line(img, Point(130, 296), Point(382, 296), Scalar(255, 255, 255), 2);//画一条线

putText(img, "I Love 718!!!!", Point(137, 262)/*坐标*/, FONT_HERSHEY_DUPLEX/*字体*/, 0.75/*字号*/, Scalar(0, 69, 255),2/*线条粗细*/);//输入汉字
```

##### 仿射变换、透视变换

仿射变换指的是线性变换+平移：

``` c++
Mat affineMatrix = getAffineTransform(srcPoints, dstPoints);
Mat affineImage;
warpAffine(image, affineImage, affineMatrix, image.size());
```

透视变换指的是更一般的变换，比方说平时在使用手机扫描文档的时候手机自动把宽窄不一、有一定倾斜的图片扶正：

``` c++
matrix = getPerspectiveTransform(src, dst);
warpPerspective(img, imgWarp, matrix, Point(w, h));
```

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-26%20133817.png?raw=true)

##### 颜色提取

```c++
Mat imgHSV,mask;
int hmin=0,smin=110,vmin=153;
int hmax=19,smax=240,vmax=255;

int main() {
    string path = "images/lambo.png";
    Mat img = imread(path);
    cvtColor(img,imgHSV,COLOR_BGR2HSV);

    namedWindow("Trackbars",(641,200));
    createTrackbar("Hue Min","Trackbars",&hmin,179);
    createTrackbar("Hue Max","Trackbars",&hmax,179);
    createTrackbar("Sat Min","Trackbars",&smin,255);
    createTrackbar("Sat Max","Trackbars",&smax,255);
    createTrackbar("Val Min","Trackbars",&vmin,255);
    createTrackbar("Val Max","Trackbars",&vmax,255);

    while(1){
        Scalar lower(hmin,smin,vmin);
        Scalar upper(hmax,smax,vmax);
        inRange(imgHSV,lower,upper,mask);

        imshow("Image", img);
        imshow("Image HSV", imgHSV);
        imshow("Image MUSK", mask);
        waitKey(1);
    }
}
```

以上代码实现了一个拥有调节功能的图片提取颜色的小程序。用法是通过在滑块调整使musk窗口内容不断变化直到剩下我们希望被提取的那一种颜色。这时候就可以记录下我们得到的数据，反向传输给inRange来生成对应的被提取的颜色就好了。

##### 图形提取

图形提取并没有独特的函数或命令实现，但是我们可以借助获取一块区域的顶点数来大致推测：

```c++
#include <opencv2/imgcodecs.hpp>
#include <opencv2/highgui.hpp>
#include <opencv2/imgproc.hpp>
#include <iostream>//头文件 
using namespace cv;
using namespace std;
void getContours(Mat imgDil,Mat img){

    vector<vector<Point>> contours;
    vector<Vec4i> hierarchy;
    findContours(imgDil,contours,hierarchy,RETR_EXTERNAL,CHAIN_APPROX_SIMPLE);
    for(int i=0;i<contours.size();i++){
        int area=contourArea(contours[i]);
        vector<vector<Point>> conPoly(contours.size());
        vector<Rect> boundRect(contours.size());

        string objectType;

        if(area>900){//判断是噪点还是图形 
            float peri=arcLength(contours[i],true);
            approxPolyDP(contours[i],conPoly[i],0.02*peri,true);
            boundRect[i]=boundingRect(conPoly[i]);
            
            int objCor=(int)conPoly[i].size();//用于记录顶点数目，如果定点特别多就是圆  

            if(objCor==3){
                objectType="Tri";
            }
            else if(objCor==4){
                float aspRatio=(float)boundRect[i].width/(float)boundRect[i].height;
                if(aspRatio>0.95 && aspRatio<1.05){//用于判断是不是正方形，如果长宽比近似等于1，那么就是正方形  
                    objectType="Square";
                }
                else{
                    objectType="Rect";//否则是长方形 
                }
                
            }
            else{
                objectType="Circle";
            }
            rectangle(img,boundRect[i].tl(),boundRect[i].br(),Scalar(0,255,0),5);
            putText(img,objectType,{boundRect[i].x,boundRect[i].y-5},FONT_HERSHEY_PLAIN,1,Scalar(0,0,255),1);
        }
    }
}

int main() {	
	string path = "images/shape2.jpeg";
	Mat img = imread(path);
	Mat imgGray,imgBlur,imgCanny,imgDil,imgErode;
	
	cvtColor (img,imgGray,COLOR_BGR2GRAY);
	GaussianBlur(imgGray,imgBlur,Size(7,7),5,0);
	Canny(imgBlur,imgCanny,10,50);
	Mat kernel=getStructuringElement(MORPH_RECT,Size(3,3));
	dilate(imgCanny,imgDil,kernel);
	
	getContours(imgDil,img);
	
	imshow("Image", img);
	
	waitKey(0);
	return 0;
}
```

通过这个完整的程序，我们就可以得到一个简单的图像形状识别程序了：

![笔者将该图片上传至GitHub，如果不能正常显示，有可能是因为笔者操作失误。图片内容为操作过程截图。](https://github.com/88fkb88/Photos/blob/main/%E5%B1%8F%E5%B9%95%E6%88%AA%E5%9B%BE%202025-10-26%20140607.png?raw=true)

### 四、感悟

在这次opencv的学习中，问题层出不穷，包括终端能找到路径但是vscode找不到、头文件丢失、Linux不自带图片编辑器、突然断网等等。不过好在在笔者的不懈努力之下，这些难题都被一一攻破了。

### 五、鸣谢

~~(tips:笔者其实并不知道应不应该有这部分，但是笔者决定加上)~~

在此次学习过程中笔者主要要感谢群中学长学姐拓展了笔者的视野。

其次要感谢deepseek以及网上各路大神的帖子帮助了我的学习。















