---
layout: post
title: warpPerspective实现
date: 2019-05-12
categories: blog
---

```
#include <highgui.h>
#include <opencv2/opencv.hpp>
using namespace cv;

Point2f srcTri[4], dstTri[4];
int clickTimes = 0;  //在图像上单击次数
Mat image;
Mat imageWarp;

void mywarpPerspective(Mat src, Mat &dst, Mat T) {
//此处注意计算模型的坐标系与Mat的不同
//图像以左上点为（0,0），向左为x轴，向下为y轴，所以前期搜索到的特征点 存的格式是（图像x，图像y）---（rows，cols）
//而Mat矩阵的是向下为x轴，向左为y轴，所以存的方向为（图像y，图像x）----（cols，rows）----（width，height）
//这个是计算的时候容易弄混的
//创建原图的四个顶点的3*4矩阵（此处我的顺序为左上，右上，左下，右下）
    Mat tmp(3, 4, CV_64FC1, 1);
    tmp.at < double >(0, 0) = 0;
    tmp.at < double >(1, 0) = 0;
    tmp.at < double >(0, 1) = src.cols;
    tmp.at < double >(1, 1) = 0;
    tmp.at < double >(0, 2) = 0;
    tmp.at < double >(1, 2) = src.rows;
    tmp.at < double >(0, 3) = src.cols;
    tmp.at < double >(1, 3) = src.rows;

//获得原图四个顶点变换后的坐标，计算变换后的图像尺寸
    Mat corner = T * tmp;      //corner=(x,y)=(cols,rows)
    int width = 0, height = 0;
    double maxw = corner.at< double >(0, 0) / corner.at< double >(2, 0);
    double minw = corner.at< double >(0, 0) / corner.at< double >(2, 0);
    double maxh = corner.at< double >(1, 0) / corner.at< double >(2, 0);
    double minh = corner.at< double >(1, 0) / corner.at< double >(2, 0);
    for (int i = 1; i < 4; i++) {
        maxw = max(maxw, corner.at< double >(0, i) / corner.at< double >(2, i));
        minw = min(minw, corner.at< double >(0, i) / corner.at< double >(2, i));
        maxh = max(maxh, corner.at< double >(1, i) / corner.at< double >(2, i));
        minh = min(minh, corner.at< double >(1, i) / corner.at< double >(2, i));
    }
    std::cout << minw << " " << maxw << " " << minh << " " << maxh << std::endl;

//创建向前映射矩阵 map_x, map_y
//size(height,width)
    dst.create(int(maxh - minh), int(maxw - minw), src.type());
    for (int i = 0; i < dst.rows; i++) {        
        for (int j = 0; j < dst.cols; j++) {
            if (dst.at<Vec3b>(i, j)[0] == 0 && dst.at<Vec3b>(i, j)[1] == 0 && dst.at<Vec3b>(i, j)[2] == 0) {
                continue;
            }
            std::cout << "not:true::" << i << " " << j << " " <<  dst.at<Vec3b>(i, j) << std::endl;
        }
    }

    Mat map_x(dst.size(), CV_32FC1);
    Mat map_y(dst.size(), CV_32FC1);
    Mat proj(3, 1, CV_32FC1,1);
    Mat point(3, 1, CV_32FC1,1);
    T.convertTo(T, CV_32FC1);  
//本句是为了令T与point同类型（同类型才可以相乘，否则报错，也可以使用T.convertTo(T, point.type() );）
    Mat Tinv=T.inv();

    auto f = [](float u, float v, int a00, int a01, int a10, int a11) {
        return u * v * a00 + u * (1 - v) * a01 + (1 - u) * v * a10 + (1 - u) * (1 - v) * a11;
    };
    for (int i = 0; i < dst.rows; i++) {        
        for (int j = 0; j < dst.cols; j++) {
            point.at<float>(0) = j + minw;
            point.at<float>(1) = i + minh;
            proj = Tinv * point;
            float x = proj.at<float>(0) / proj.at<float>(2) + 0.5 * (src.cols / dst.cols);
            float y = proj.at<float>(1) / proj.at<float>(2) + 0.5 * (src.rows / dst.rows);
            int xx = x;
            int yy = y;

            if (xx < 0 || yy < 0 || xx + 1 >= src.cols || yy + 1 >= src.rows) {
                continue;
            }

            //img.at<Vec3b>(r, c)[0];
            //dst.at<Vec3b>(i, j) = src.at<Vec3b>(yy, xx);
            for (int k = 0; k < 3; k++) {
                dst.at<Vec3b>(i, j)[k] = f(yy + 1 - y, xx + 1 - x, 
                    src.at<Vec3b>(yy, xx)[k],
                    src.at<Vec3b>(yy, xx + 1)[k],
                    src.at<Vec3b>(yy + 1, xx)[k],
                    src.at<Vec3b>(yy + 1, xx + 1)[k]);
            }

            map_x.at<float>(i, j) = proj.at<float>(0) / proj.at<float>(2);
            map_y.at<float>(i, j) = proj.at<float>(1) / proj.at<float>(2);
        }
    }
    
    Mat new_dst;
    new_dst.create(int(maxh - minh), int(maxw - minw), src.type());
    remap(src, new_dst, map_x, map_y, CV_INTER_LINEAR);

    for (int i = 0; i < dst.rows; i++) {        
        for (int j = 0; j < dst.cols; j++) {
            if (dst.at<Vec3b>(i, j)[0] == 0 && dst.at<Vec3b>(i, j)[1] == 0 && dst.at<Vec3b>(i, j)[2] == 0) {
                continue;
            }
            if (dst.at<Vec3b>(i, j) != new_dst.at<Vec3b>(i, j)) {
                std::cout << i << " " << j << " " << dst.at<Vec3b>(i, j) << " " << new_dst.at<Vec3b>(i, j) << std::endl;
            }
        }
    }
}

int main(int argc, char *argv[]) {
    freopen("out.txt", "w", stdout);

    image = imread(argv[1]);
    //imshow("Source Image",image);

    srcTri[0].x = 371;
    srcTri[0].y = 0;
    srcTri[1].x = 20;
    srcTri[1].y = 165;
    srcTri[2].x = 613;
    srcTri[2].y = 62;
    srcTri[3].x = 287;
    srcTri[3].y = 356;

    dstTri[0].x = 0;
    dstTri[0].y = 0;
    dstTri[2].x = image.rows - 1;
    dstTri[2].y = 0;
    dstTri[1].x = 0;
    dstTri[1].y = image.cols - 161;  
    dstTri[3].x = image.rows - 1;
    dstTri[3].y = image.cols - 161;

    Mat transform = Mat::zeros(3, 3, CV_32FC1); //透视变换矩阵
    transform = getPerspectiveTransform(srcTri, dstTri);  //获取透视变换矩阵       
    mywarpPerspective(image, imageWarp, transform);  //透视变换
    //imshow("After WarpPerspecttive", imageWarp);
    imwrite("4.png", imageWarp);

    //waitKey();
    return 0;
}
```

```
g++ test.cpp -o test `pkg-config --cflags --libs opencv` --std=c++11
./test 2.jpg
```
