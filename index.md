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
