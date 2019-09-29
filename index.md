## 李志远  George2001
***
*infinite flight
*account 
*facebook

*1522561925@qq.com
*google

*lzhiyuan2001@gmail.com
*yahoo

*Roit games
*US
*lizhiyuan2001
*EU
*lizhiyuan2001

*apple
*1522561925@qq.com

*APPLE US
*dregt44@icloud.com
*STEAM  BACKUP  R14183
*
** TesT

##################################################################
kk‘s code
//小康康的教学程序！
#include<iostream>
#include<cstdio>
#include<ctime>
#include<cmath>
#include<algorithm>
#include<cstring>
#include<cstdlib>

using namespace std;

void pxjx();
void xzpx();
void kspx();
void mppx();

void zfcjx();
void dgjx();
void STLjx();
void iojx();
void oj();
void ydm();


void vectorjx();
void stringjx();
void mapjx();


int main()
{
	int No;
	int password=0;
	int time = 0;
	cout << "欢迎来到小康康的教学程序！" << endl;
	cout << "请输入4位课程密码" << endl;
	cin >> password;
	while (password != 2599)
	{
		cout << "密码错误！" << endl;
		time++;
		if (time == 3)
		{
			cout << "伤心了。。。。。3次都没输对。。。。。难受！！！课程到此结束！！！！" << endl;
			return 0;
		}
		cout << "您还有" << 3 - time << "次机会" << endl;
	}
	cout << "恭喜你输入正确！！！" << endl;
	cout << "请输入相应的教学代码" << endl;
	cout << "1.排序教学" << endl;
	cout << "2.字符串教学" << endl;
	cout << "3.递归教学" << endl;
	cout << "4.STL容器教学" << endl;
	cout << "5.输入输出优化教学" << endl;
	cout << "6.源代码输出" << endl;
	cout << "7.各大oj典型题目讲解" << endl;
	cout << "0.退出" << endl;
	cin >> No;
	
	while (No != 0)
	{
		switch (No)
		{
			case 1:
				pxjx();
				break;
			case 2:
				zfcjx();
				break;
			case 3:
				dgjx();
				break;
			case 4:
				STLjx();
				break;
			case 5:
				iojx();
				break;
			case 6:
				ydm();
				break;
			case 7:
				oj();
				break;
			case 0:
				break;
		}
		cout << "恭喜你输入正确！！！" << endl;
		cout << "请输入相应的教学代码" << endl;
		cout << "1.排序教学" << endl;
		cout << "2.字符串教学" << endl;
		cout << "3.递归教学" << endl;
		cout << "4.STL容器教学" << endl;
		cout << "5.输入输出优化教学" << endl;
		cout << "6.源代码输出" << endl;
		cout << "0.退出" << endl;
		cin >> No;
	}
	cout << "请输入任意键继续" << endl;
	getchar();
	return 1;
	
}


void pxjx()
{
	int px;
	cout << "欢迎来到排序相关教学" << endl;
	cout << "请输入代码:" << endl;
	cout << "1.选择排序" << endl;
	cout << "2.冒泡排序" << endl;
	cout << "3.插入排序" << endl;
	cout << "4.快速排序" << endl;
	cout << "5.堆排序" << endl;
	cout << "0.返回上一层" << endl;
	cin >> px;
	while (px != 0)
	{
		switch (px)
		{
		case 1:

			break;
		case 2:

			break;
		case 3:

			break;
		case 4:

			break;
		case 5:

			break;
		case 0:
			break;
		}
	}

}
void xzpx()
{
	int len;
	int xz[100];
	int minn;
	cout << "请输入数字数量" << endl;
	cin >> len;
	cout << "请输入原数据" << endl;
	for (int i = 1; i <= len; ++i)
		cin >> xz[i];
	//首先  一层循环  从1-n开始遍历
	for (int i = 1; i <= len; ++i)
	{
		minn = i;
		/*
		设置一个变量储存从i+1 到 len 中 最小的元素的下标
		开始循环，找到比现有元素更小的就更新minn，一直储存最小的小标
		遍历完成以后 minn中储存的就是里面最小的小标
		最后交换数组当前i地址的元素与minn地址的元素  即将最小的元素放到数组开头
		○○○○○○●○				开始扫描 黑球为当前最小 选中  与第一位交换
		●○○○○○○○
		●○○○○○○○				继续扫描 
		●○○●○○○○				又找到第二小的数  选中 与第二位交换
		●●○○○○○○				交换
		●●○○○○○●
		●●●○○○○○
		●●●○○○○○
		●●●●○○○○				。。。。。。。。。
		
		*/
		for (int j = i + 1; j <= len; ++j)		//开始遍历	
			if (xz[j] < xz[minn])				//开始判断，找到更小的就更新minn
				minn = j;
		swap(xz[i], xz[minn]);					//交换   swap(a,b);就是交换a,b中的数;也可以是字符串或单个字符
	}
	for (int i = 1; i <= len; ++i)
		cout << xz[i] << " ";
}

void mppx()
{
	int len;
	int xz[100];
	int minn;
	cout << "请输入数字数量" << endl;
	cin >> len;
	cout << "请输入原数据" << endl;
	for (int i = 1; i <= len; ++i)
		cin >> xz[i];
	/*冒泡排序  顾名思义  就是冒泡
	啥是冒泡呢  举个形象的例子  
	●○○○○○				假设从第一位开始	开始临位比较  发现比第二位大  交换第一第二位
	○●○○○○				交换成功				
	○○●○○○				第二第三位开始比较  如果大于 开始交换
	○○○●○○				
	○○○○●○
	○○○○○●				最后  不再有符合条件的数，结束第一大的排序
	●○○○○●				开始第二位
	○●○○○●
	○○●○○●
	○○○●○●
	○○○○●●				第二大排序成功	
	。。。。。。。。
	
	如图  正方形为当前所需要比较的数 然后进行比较 然后往后运行
	左右比较  往下运行 正方形 往下移 一直比较左右两个数  一直比较。。。。。。
	*/
	for (int i = 1; i < len; ++i)
	{
		for (int j = i + 1; j <= len; ++j)
		{
			if (xz[j] < xz[i])					//临位判断
				swap(xz[i], xz[j]);				//交换
		}
	}
	for (int i = 1; i <= len; ++i)
		cout << xz[i] << " ";
}

void zfcjx()
{

}
void dgjx()
{

}
void STLjx()
{

}
void iojx()
{

}
void oj()
{

}
void ydm()
{

}

void kpjx()
{

}
void vectorjx()
{

}
void stringjx()
{

}
void mapjx()
{

}
