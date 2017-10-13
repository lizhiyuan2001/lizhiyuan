## 李志远
***
* Floyd 判最小环 
* int Floyd_MinCircle()
*{
*    int Mincircle = Mod;
*    int i,j,k;
*    for(k=1;k<=n;k++)
*    {
*        for(i=1;i<=n;i++)
*        {
*            for(j=1;j<=n;j++)
*            {
*                if(dis[i][j] != Mod && mp[j][k] != Mod && mp[k][i] != Mod && dis[i][j] + mp[j][k] + mp[k][i] < Mincircle)
*                   Mincircle = dis[i][j] + mp[j][k] + mp[k][i];
*            }
*        }
*        //正常Floyd
*        for(i=1;i<=n;i++)
*        {
*            for(j=1;j<=n;j++)
*            {
*                if(dis[i][k] != Mod && dis[k][j] != Mod && dis[i][k] + dis[k][j] < dis[i][j])
*                {
*                    dis[i][j] = dis[i][k] + dis[k][j];
*                    pre[i][j] = pre[k][j];
*                }
*            }
*        }
*    }
*    return Mincircle;
*}
***
*dp合唱队形
*#include<iostream>						//这个题求 严格 上升子序列 
*#define x 200
*using namespace std;
*
*int dpl[x],dpr[x],a[x];
*int n;
*
*int main()
*{
*	cin>>n;
*	for(int i=1;i<=n;++i)
*		cin>>a[i];
*	dpl[1]=1;
*	for(int i=2;i<=n;++i)
*	{
*		int max=0,w;			//dp
*		for(w=1;w<=i;++w)		//从1到w  
*			if(a[w]<a[i] && dpl[w]>max)
*			//如果a[w]<a[i] 说明有更优的数 即比上一次的数小的数 如果dpl[w]又比max大 说明选这个数 子序列的最大值又小 长度又长 选	 
*				max=dpl[w];	//选！ 
*		dpl[i]=max+1;			//把最左侧的加上 
*	}
*	dpr[n]=1;				//从最右侧反向求最长上升子序列 最右侧设为 1
*	for(int i=n-1;i>=1;--i)
*	{
*		int max=0,w;
*		for(w=i+1;w<=n;++w)
*			if(a[w]<a[i] && dpr[w]>max)
*				max=dpr[w];
*		dpr[i]=max+1;			//把最右侧的加上 
*	 } 
*	int max=-1,sum;				//max设个最小值 
*	for(int i=1;i<=n;++i)			//从1到n 枚举每个点的dpl dpr 相加 求出1到n中 dpl+dpr的最大处 
*	{
*		sum=dpl[i]+dpr[i];
*		if(sum>max) 
*			max=sum;
*	}
*	max--;					//dpl 和 dpr 都加了选中的i点 所以要减去1 
*	cout<<n-max;				//max求的是选中在合唱队里的人，题目求出列人数 所以用n减去剩余人数 即为出列人数 
*}
*
