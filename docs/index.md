## 引入
本题与[P2662牛场围栏](https://www.luogu.com.cn/problem/P2662)以及[P2371[国家集训队]墨墨的等式](https://www.luogu.com.cn/problem/P2371)类似，属于此二题的基础版。

关键在于思路，怎样转化为**最短路**。

## 核心思想与操作
- 简化参数：

	对于题目所给参数$x,\ y,\ z$，我们可以先考虑仅利用$y,\ z$这两个参数就能达到的楼层，再考虑从这个楼层开始不断地利用$x$向上能达到的楼层，最后计入答案即可。
    
    
- 统计优化
	
    如果穷举所有仅利用$y,\ z$这两个参数就能到达的楼层，再继续向上统计，这样不仅浪费时间，还会造成统计重复。例如对于满$floor_i \ = \ floor_j  \ + \ kx \ (k \ \in \ \mathbb{Z})$的$floor_i$与$floor_j$就会出现统计重复的情况。而观察$floor_i$与$floor_j$我们可以发现$floor_i \ \equiv \ floor_j \ \pmod{x}$，所以我们只需要分别找出$\bmod \ x \  = p \ (p \ \in \ \left[0,x\right))$这一类楼层中最小的仅利用$y,\ z$就能达到的楼层，再将这个楼层一直向上利用$x$直到达到$h$上限总共可能达到的楼层数计入答案即可。统计所需时间复杂度为$\mathcal{O}(x)$。每次计入的数值是多少建议大家自己手推一下，还是比较简单的。~~代码里有标注。~~
    
    
- DP与最短路的转化

	根据上面的分析我们可以得出DP的转移方程：
    
    我们设$dp[i]$的意义为$\bmod \ x \  = i \ (i \ \in \ \left[0,x\right))$这一类楼层中最小的楼层，则有这样两种转移方式：
    
    $dp[(i+y)\%x] = \min(dp[i] + y,dp[(i+y)\%x])$
    
    $dp[(i+z)\%x] = \min(dp[i] + z,dp[(i+z)\%x])$
    
    看到这两行式子，~~非常擅长图论的~~你一定会联想到最短路更新时的式子（尤其是在这种DP时空复杂度均较大的情况下）：
    
    $dist[to] = \min(dist[from] + cost, dist[to])$
    
    通过观察，我们发现只需要将$i$作为父亲结点$from$，将$(i+y\ or\ z)\%x$作为儿子结点$to$就可以成功地将DP转化为最短路啦！
    
    至于最短路究竟是使用SPFA还是Dijkstra堆优化就要看你们自己的选择了~~我才不会告诉你我选的是SPFA~~。
    
## $\mathbf{AC \ CODE}$
借鉴于[Mr_Leceue](https://www.luogu.com.cn/user/142518)
```cpp
#include<bits/stdc++.h>
using namespace std;
const int N = 1e5 + 10;
long long hig, dist[N], ans;
int x, y, z, cnt = 0, h[N], flag[N];
queue<int> q;
struct node
{
    int to, nxt, val;
}edge[N << 2];
void add(int st, int en, int ct)
{
    edge[++cnt].to = en;
    edge[cnt].nxt = h[st];
    edge[cnt].val = ct;
    h[st] = cnt;
}
void SPFA()
{
    memset(dist, 0x3f3f3f, sizeof(dist));//dist是long long类型
    memset(flag, 0, sizeof(flag));
    dist[1] = 1;
    q.push(1);
    flag[1] = 1;
    while(!q.empty())
    {
        int now = q.front();
        q.pop();
        flag[now] = 0;
        for(int i = h[now]; i; i = edge[i].nxt)
        {
            int nowto = edge[i].to;
            int nowval = edge[i].val;
            if(dist[now] + nowval < dist[nowto])
            {
                dist[nowto] = dist[now] + nowval;
                if(flag[nowto] == 0)
                {
                    q.push(nowto);
                    flag[nowto] = 1;
                }
            }
        }
    }
}
int main()
{
    cin >> hig >> x >> y >> z;
    if(x < y){swap(x, y);}
    if(y < z){swap(y, z);}
    if(x < y){swap(x, y);}//从小到大排个序，否则过不了#1
    for(int i = 0; i < x; i++)
    {
        add(i, (i + y) % x, y);
        add(i, (i + z) % x, z);
    }
    SPFA();
    for(int i = 0; i < x; i++)
    {
        if(dist[i] <= hig)
        {
            ans += (hig - dist[i]) / x + 1;//统计答案，加一是因为(hig-dist[i])/x所求的相当于是包含几段x，而计入的层数应该等于段数加一
        }
    }
    cout << ans << endl;
    return 0;
}

```
