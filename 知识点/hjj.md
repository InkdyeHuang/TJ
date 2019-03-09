1. 拿到最大的
if 序列长度为偶数
    先手必胜
else if 序列第一个数或最后一个数为最大的数
    先手必胜
else
    后手必胜
2. 拿到最大和
    sum(l, r)表示区间[l, r]的数字和
    dp(l, r)表示当前这个人从[l, r]区间能拿到的最大的和
    dp(l, r) = max(sum(l, r)-dp(l+1,r), sum(l, r)-dp(l, r-1))
    dp(l, r) = 0 if l > r
    if 2*dp(1, n) > sum(1, n)
        先手必胜
    else
        后手必胜
3. 

#include<iostream>
#include<stack>
using namespace std;
typedef int semaphore;
semaphore s1_mutex = 1;
stack<int> s1,s2;
void _push(int n)
{
    if(s1_mutex-- > 0)
    {
       s1.push(n);
       s1_mutex++;
    }

}
bool _empty()
{
    return s1.empty() && s2.empty();
}
int _front()
{
    if(_empty())
    {
        throw "queue is empty";
    }
    else if(s2.empty())
        return s2.top();
    else
    {
        if(s1_mutex-- > 0)
        {
            while(s1.size() > 0)
            {
                s2.push(s1.top());
                s1.pop();
            }
            s2.pop();
            s1_mutex++;
        }
    }
}
void _pop()
{
    if(_empty())
    {
        throw "queue is empty";
    }
    if(!s2.empty())
        s2.pop();
    else
    {
        if(s1_mutex-- > 0)
        {
            while(s1.size() > 0)
            {
                s2.push(s1.top());
                s1.pop();
            }
            s2.pop();
            s1_mutex++;
        }
    }
}
int main()
{
    return 0;
}
