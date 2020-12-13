




## 句柄

理解为程序id 

### 获取所有的句柄

获取活动的窗口的句柄
```
import win32gui
 
hwnd_title = dict()
 
 
def get_all_hwnd(hwnd, mouse):
	if win32gui.IsWindow(hwnd) and win32gui.IsWindowEnabled(hwnd) and win32gui.IsWindowVisible(hwnd):
		hwnd_title.update({hwnd: win32gui.GetWindowText(hwnd)})
 
 
if __name__ in "__main__":
    win32gui.EnumWindows(get_all_hwnd, 0)
    for h, t in hwnd_title.items():
	    if t is not "":
		    print(h, t)

```

### 根据句柄获取句柄标题和类名
```
import win32gui
 
title = win32gui.GetWindowText(jbid)   #jbid为句柄id
#获取标题
clsname = win32gui.GetClassName(jbid)  
#获取类名
```
### 根据句柄获取窗口位置
实际电脑屏幕大小*0.8=程序获得的尺寸
```
import win32gui
left, top, right, bottom = win32gui.GetWindowRect(jbid)
#分别为左、上、右、下的窗口位置
```

### 鼠标点击

将SendMessage换成PostMessage
```
def doClick(cx,cy):
        long_position = win32api.MAKELONG(cx, cy)#模拟鼠标指针 传送到指定坐标
        win32api.SendMessage(hwnd, win32con.WM_LBUTTONDOWN, win32con.MK_LBUTTON, long_position)#模拟鼠标按下
        win32api.SendMessage(hwnd, win32con.WM_LBUTTONUP, win32con.MK_LBUTTON, long_position)#模拟鼠标弹起

```




### 参考链接
https://blog.csdn.net/claire017/article/details/104521681
https://www.cnblogs.com/zhangquan-yw/p/10239132.html
https://blog.csdn.net/qq_16234613/article/details/79155632
https://www.cnblogs.com/zoro-robin/p/5591185.html
https://blog.csdn.net/qq_40134903/article/details/88297476
https://blog.csdn.net/u010906468/article/details/106108906
https://www.cnblogs.com/us-wjz/articles/11479005.html
https://segmentfault.com/q/1010000016704104/a-1020000019622101






