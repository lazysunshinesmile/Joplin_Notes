YUV图片格式

YUV图片格式

YUV 有两种排序方式
（1）planar(铺平)
Y     Y     Y     Y
U     U     U     U
V     V     V     V
（2）packed
YUV YUV    YUV YUV
YUV YUV    YUV YUV
YUV YUV    YUV YUV

采样比率有区分(如果图片宽高是:720*480)：
（1）
4：4：4
Y - UV
总的存储：720*480*3，Y=720*480，U=720*480，V=720*480

（2）
4：2：2
YY - UV  (-> Y-U, Y-V    纯粹是为了方便记忆)
总的存储：720*480*2，Y=720*480，U=720*480/2，V=720*480/2

（3）
4：2：0
YYYY - UV
总的存储：720*480/4 * 6 = 780*420 * 3/2;  Y=720*480，U=720*480/4，V=720*480/4

存储方式：
（1）YUVYUV （属于YUV422）packed方式

start + 0        Y00        U00        Y01        V00        Y02        U01        Y03        V01

start + 8        Y10        U10        Y11        V10        Y12        U11        Y13        V11

（2）UYVY（属于YUV422）packed方式

start + 0        U00        Y00        V00        Y01        U01        Y02        V01        Y03

start + 8        U10        Y10        V10        Y11        U11        Y12        V11        Y13

（3）YUV422P（属于YUV422）plane方式
start + 0        Y00        Y01     Y02        Y03
start + 4        Y10        Y11     Y12        Y13
start + 8        U00        U01
start + 10       U10        U11
start + 12       V00        V01
start + 14       V10        V11

（4）YV12,YU12（属于YUV420）plane方式

start + 0        Y00        Y01     Y02        Y03
start + 4        Y10        Y11     Y12        Y13
start + 8        U00        U01
start + 10       V00        V01

（5）NV12,NV21（属于YUV420）two-plane方式    (Y正常平铺方式，UV采用packed的方式)

start + 0        Y00        Y01     Y02        Y03
start + 4        Y10        Y11     Y12        Y13
start + 8        U00        V00     U10        V01

YV12与I420的区别：
YV12 ： 亮度（行×列） ＋ U（行×列/4) + V（行×列/4）

I420 ： 亮度（行×列） ＋ V（行×列/4) + U（行×列/4）

I420(YU12):    YYYYYYYY UU VV    =>YUV420P
YV12:          YYYYYYYY VV UU    =>YUV420P
NV12:          YYYYYYYY UVUV     =>YUV420SP
NV21:          YYYYYYYY VUVU     =>YUV420SP