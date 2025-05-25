---
title: A*寻路算法 C++ 实现
date: 2024-04-07T17:05:15+08:00
categories: 
- c++
- algorithm
tags:
- algorithm
- AStar
---

这几天在试着用c++和opengl写一个[连连看小游戏](https://github.com/voidvvv/LinkA)。发现坑还挺多。其中一个就是判断两个块之间是否可以连通，经过分析之后我使用了AStar寻路算法来完成判断。为了保证复用还使用了模板。但是抽象这一块费了很大的劲，因为需要让代码能够跟我游戏中定义的一些类来相配合。
先说一下简单的思路：
按照AStar寻路算法，就是从起点开始，依次判断周围连同点的cost大小，若联通点没有被经过或者之前经过的cost大于当前计算的cost，则使用当前的路径，否则使用之前的路径，依次判断，直到找到终点.

<!-- more -->

```c++
// 首先先定义三个类，作为我们算法的基本类型，
// 第一个，连通类，保存有连通两端的节点，以及当前连通所需要的cost
template <typename N>
class Connection
{
public:
    virtual float getCost() = 0;
    virtual N *getFromNode() = 0;
    virtual N *getToNode() = 0;
};
// 第二个，我们的地图，这里将地图抽象出来几个方法，1. 查询指定节点的所有连通，2. 查询指定节点的索引，3. 当前地图节点的数量（size）
template <typename N>
class Graph
{
public:
    virtual std::vector<Connection<N> *> getConnections(N *fromNode) = 0;
    virtual std::vector<Connection<N> *> getConnections(N *fromNode, N* matchNode) = 0;
    virtual int getIndex(N *node) = 0;
    virtual int size() = 0;
};
// 第三个，就是我们算法中需要用到的节点记录类，用来记录某个节点以及其联通的额匹配对，以及节点的访问状态

enum Node_Category
{
    UNVISITED,
    OPEN,
    CLOSE
};

template <typename N>
class NodeRecord
{
public:
    N *node;

    /** The incoming connection to the node */
    Connection<N> *connection;

    /** The actual cost from the start node. */
    float costSoFar = 0;

    /** The node category: {@link #UNVISITED}, {@link #OPEN} or {@link #CLOSED}. */
    Node_Category category = Node_Category::UNVISITED;

    /** ID of the current search. */
    unsigned int searchId = 0;
};
```
然后，我们定义我们的寻路算法类
```c++
template <typename N>
class PathFinder
{
public:
    virtual bool searchNodePath(N *startNode, N *endNode,
                                float (*_Heuristic)(N *, N *),
                                bool (*_shouldStop)(N *, N *),
                                std::vector<N *> &outPath) = 0;
    // virtual bool searchNodePath(N *startNode, N *endNode, Heuristic<N> &, std::vector<N *> &outPath) = 0;

    virtual void generateNodePath(N *startNode, std::vector<N *> &outPath) = 0;
};
```
这个算法类有两个抽象方法，一个是查询路径，将返回结果放入一个vector中，另一个是生成节点路径，是为了在查询路径成功后，将路劲记录系下来的方法.
下面是这个类的实现类:
```c++

#include "AStar.h"
#include <memory>
#include <algorithm>

template <typename N>
class AStarPathFinder : public PathFinder<N>
{
private:
    unsigned int searchId = 0;
    std::vector<NodeRecord<N> *> records;
    std::shared_ptr<Graph<N>> graph;
    NodeRecord<N> *current;

public:
    NodeRecord<N> *getNodeRecord(N *node)
    {
        Graph<N> *gp = graph.get();
        int index = gp->getIndex(node);
        NodeRecord<N> *nodeRecord = records[index];
        if (nodeRecord == NULL)
        {
            nodeRecord = new NodeRecord<N>();
            nodeRecord->category = Node_Category::UNVISITED;
            nodeRecord->searchId = this->searchId;
            nodeRecord->connection = NULL;
            records[index] = nodeRecord;
        }
        if (nodeRecord->searchId != this->searchId)
        {
            nodeRecord->searchId = this->searchId;
            nodeRecord->connection = NULL;
            nodeRecord->category = Node_Category::UNVISITED;
        }
        nodeRecord->node = node;
        return nodeRecord;
    }

    void init(Graph<N> *_graph)
    {
        current = NULL;
        records = std::vector<NodeRecord<N> *>(_graph->size());
        this->graph.reset(_graph);
        // this->graph = std::shared_ptr<Graph<N>>(_graph);
    }
    void generateNodePath(N *startNode, std::vector<N *> &outPath) override
    {
        // todo
        while (current && current->connection)
        {
            outPath.push_back(current->node);
            // current->connection->getFromNode
            Graph<N> *gp = graph.get();
            current = records[gp->getIndex(current->connection->getFromNode())];
        }
        outPath.push_back(startNode);
    }

    void visitChild(N *end, float (*_Heuristic)(N *, N *), std::vector<NodeRecord<N> *> &openList)
    {
        Graph<N> *graphPtr = this->graph.get();

        std::vector<Connection<N> *> connections = graphPtr->getConnections(current->node,end);

        for (int x = 0; x < connections.size(); x++)
        {
            Connection<N> *con = connections[x];
            N *toNode = con->getToNode();
            float nodeCost = current->costSoFar + con->getCost();

            NodeRecord<N> *nrNode = getNodeRecord(toNode);

            if (toNode == end)
            {
                nrNode->costSoFar = nodeCost;
                nrNode->connection = con;

                nrNode->category = Node_Category::OPEN;
                openList.push_back(nrNode);
                return;
            }

            if (nrNode->category == Node_Category::CLOSE && nrNode->costSoFar <= nodeCost)
            {
                continue;
            }
            else if (nrNode->category == Node_Category::OPEN)
            {
                if (nrNode->costSoFar <= nodeCost)
                {
                    continue;
                }
                openList.erase(std::find(openList.begin(), openList.end(), nrNode));
            }

            nrNode->costSoFar = nodeCost;
            nrNode->connection = con;

            nrNode->category = Node_Category::OPEN;
            openList.push_back(nrNode);
        }
    }

    bool searchNodePath(N *startNode, N *endNode,
                        float (*_Heuristic)(N *, N *),
                        bool (*_shouldStop)(N *, N *),
                        std::vector<N *> &outPath) override
    {
        this->current = NULL;
        std::vector<NodeRecord<N> *> openList;
        this->searchId++;
        NodeRecord<N> *nr = getNodeRecord(startNode);
        nr->category = Node_Category::OPEN;
        nr->connection = NULL;
        openList.push_back(nr);

        while (openList.size() > 0)
        {
            this->current = openList.back();
            openList.pop_back();
            this->current->category = Node_Category::CLOSE;

            if (_shouldStop(this->current->node, endNode))
            {
                generateNodePath(startNode, outPath);
                return true;
            }
            visitChild(endNode, _Heuristic, openList);
        }
        return false;
    }
};
```

其中最主要的算法逻辑是参考了[libgdx-AI框架](https://github.com/libgdx/gdx-ai)中的AStar寻路算法.
之前有一段时间特别着迷的读这个框架源码。感觉收益颇丰.

