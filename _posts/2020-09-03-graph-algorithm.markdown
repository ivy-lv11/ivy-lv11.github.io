---
layout: post
title:  Leetcode-SSSP
date:   2020-09-03 07:45:35 +0300
image:  05.jpg
tags:   Home
---
#### 1. [网络延迟时间](https://leetcode-cn.com/problems/network-delay-time/)
思路：单源点最短路径  
**Dijkstra算法**  
Dijkstra算法只适合边权重均为正数的情况
```
const int INF=1e9;
class Solution {
    vector<int> dist;
    vector<vector<int>> link;
    vector<int> result;
    vector<bool> certain;
public:
    int networkDelayTime(vector<vector<int>>& times, int N, int K) {
        link.resize(N);
        result.resize(N);
        certain.resize(N);
        for(int i=0;i<N;i++)
        {
            for(int j=0;j<N;j++)
                link[i].push_back(-1);
        }
        for(auto v:times)
        {
            link[v[0]-1][v[1]-1] = v[2];
        }

        for(int i=0;i<N;i++)
        {    dist.push_back(INF);
             certain.push_back(false);
        }
        dist[K-1] = 0;
        for(int i=0;i<N;i++)
        {
            //get min
            vector<int>::iterator it = min_element(dist.begin(),dist.end());
            int u = it-dist.begin();
            result[u] = dist[u];
            for(int i=0;i<link[u].size();i++)
            {
                if(link[u][i]!=-1)
                {
                    int v = i;
                    if(certain[v]==false && dist[v]>result[u]+link[u][v])
                    {
                        dist[v] = result[u]+link[u][v];
                    }
                }
            }
            dist[u] = INF;
            certain[u] = true;     
        }
        vector<int>::iterator m = max_element(result.begin(),result.end());
        if(*m == INF)
            return -1;
        return *m;
    }
};

```
  
**Bellman-Ford算法**  
Bellman-ford算法适合权重为负数的情况，还可以检查是否有负环 
```
const int INF = 1e9;
class Solution {
    vector<int> dist;//记录距离
    vector<vector<int>> link;//邻接矩阵
public:
    int networkDelayTime(vector<vector<int>>& times, int N, int K) {
        //初始化所有距离到最大值，并初始化邻接矩阵
        link.resize(N);
        for(int i=0;i<N;i++)
        {
            dist.push_back(INF);
            for(int j=0;j<N;j++)
                link[i].push_back(-1);
        }
        for(auto v:times)
        {
            link[v[0]-1][v[1]-1] = v[2];
        }
        //源点距离设为1
        dist[K-1] = 0;
        //在图上重复N-1次，每一次对所有边relax
        for(int c = 0;c<N;c++)
        {
           for(int u=0;u<N;u++)
           {
                for(int v = 0;v<N;v++)
                {
                    if(link[u][v]!=-1)
                    {
                        if(dist[v]>dist[u]+link[u][v])
                            dist[v] = dist[u] + link[u][v];
                    }
                }
            }    
        }
        //检查是否有负环
        for(int u = 0;u<N;u++)
        {
            for(int v = 0;v<N;v++)
            {
                if(link[u][v]!=-1)
                {
                    if(dist[v]>dist[u]+link[u][v])
                        cerr<<"detect negative cycle\n";
                }
            }
        }
    }
};
```
**加入拓扑排序**
Bellman-Ford算法对很多边做了无效的relax，所以改进点在于将relax变得更有序。所以先做一遍拓扑排序，然后按照拓扑排序的顺序来relax。但注意：**这种改进只能用于DAG！**
```
const int INF = 1e9;
class Solution {
    vector<int> dist;//记录距离
    vector<vector<int>> link;//邻接矩阵
    deque<int> sort;
    vector<int> visit;
public:
    int networkDelayTime(vector<vector<int>>& times, int N, int K) {
        link.resize(N);
        for(int i=0;i<N;i++)
        {
            dist.push_back(INF);
            for(int j=0;j<N;j++)
                link[i].push_back(-1);
        }
        for(auto v:times)
        {
            link[v[0]-1][v[1]-1] = v[2];
        }
        //topo sort
        topo_sort();
        //Bellman-ford
        dist[K-1] = 0;
        for(int i=0;i<sort.size();i++)
        {
            int u = sort[i];
            for(int v=0;v<N;v++)
            {
                if(link[u][v]!=-1)
                {
                    if(dist[v]>dist[u]+link[u][v])
                        dist[v] = dist[u]+link[u][v];
                }
            }
        }
        vector<int>::iterator it = max_element(dist.begin(),dist.end());
        if(*it==INF)
            return -1;
        return *it;  
    }
    void topo_sort()
    {
        for(int i=0;i<dist.size();i++)
            visit.push_back(0);
        for(int i=0;i<visit.size();i++)
        {
            if(visit[i]==0)
                dfs(i);
        }
    }
    void dfs(int pos)
    {
        if(visit[pos]==1)
            return;
        visit[pos] = 1;
        for(int i=0;i<visit.size();i++)
        {
            if(link[pos][i]!=-1)
            {
                dfs(i);
            }
        }
        sort.push_front(pos);
    }
};
```

例题：
#### 1.[概率最大的路径](https://leetcode-cn.com/problems/path-with-maximum-probability/)

#### 2. [阈值距离内邻居最少的城市](https://leetcode-cn.com/problems/find-the-city-with-the-smallest-number-of-neighbors-at-a-threshold-distance/)
思路：Dijkstra的模板！！
```
const int INF = 1e9;
typedef pair<int,int> PII;
class Solution {
    vector<vector<int>> link;
public:
    int findTheCity(int n, vector<vector<int>>& edges, int distanceThreshold) {
        vector<int> re;
        link.resize(n);
        for(int i=0;i<n;i++)
        {
            for(int j=0;j<n;j++)
                link[i].push_back(-1);
        }
        for(auto v:edges)
        {
            link[v[0]][v[1]] = v[2];
            link[v[1]][v[0]] = v[2];
        }
        for(int i=0;i<n;i++)
        {
            vector<int> temp= dijkstra(i);
            int t = 0;
            for(auto v:temp)
            {
                if(v<=distanceThreshold)
                    t++;
            }
            re.push_back(t);
        }

        int min = 0;
        for(int i=0;i<n;i++)
        {
            if(re[min]>=re[i])
                min = i;
        }
        return min;
        
    }
    vector<int> dijkstra(int start)
    {
        vector<bool> visit;
        vector<int> dist;
        vector<int> label;
        int n = link.size();
        for(int i=0;i<n;i++)
        {
            dist.push_back(INF);
            label.push_back(INF);
            visit.push_back(false);
        }
        dist[start] = 0;
        for(int i=0;i<n;i++)
        {
            vector<int>::iterator it = min_element(dist.begin(),dist.end());
            int u = it-dist.begin();
            label[u] = dist[u];
            visit[u] = true;
            for(int v = 0;v<n;v++)
            {
                if(link[u][v]!=-1)
                {
                    if(visit[v]==false && dist[v]>label[u]+link[u][v])
                        dist[v]= label[u]+link[u][v];
                }
            }
            dist[u] = INF;

        }
        return label;
    }
};
```
