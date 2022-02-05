/*
Powered by PbXBear 2021
*/
//基础设置
#define _CRT_SECURE_NO_WARNINGS
#define NDEBUG
#ifndef _DEBUG
#pragma optimize("2", on)
#endif
//库
#include <cassert>
#include <climits>
#include <clocale>
#include <cmath>
#include <cstdio>
#include <cstring>
#include <ctime>
#include <windows.h>
#include <algorithm>
#include <atomic>
#include <fstream>
#include <future>
#include <iostream>
#include <iomanip>
#include <limits>
#include <locale>
#include <memory>
#include <random>
#include <sstream>
#include <string>
#include <thread>
#include <vector>
//一些参数
#define MAXTIME 3
#define MULTYVAL 8
//执棋方
#define BLACK -1
#define NEU 0
#define WHITE 1
//状态列表
#define REGRET -4
#define HELP -3
#define EXIT -2
#define END -1
#define MENU 0
#define GAME 1
#define HUMAN 2
#define ERR -INT_MAX
//难度列表
#define NIL 0
#define EASY 1
#define NORMAL 2
#define HARD 3
//声音
#define C(t) Beep(262, t)
#define D(t) Beep(294, t)
#define E(t) Beep(330, t)
#define F(t) Beep(349, t)
#define G(t) Beep(392, t)
#define A(t) Beep(440, t)
#define B(t) Beep(494, t)
#define C1(t) Beep(523, t)
#define D1(t) Beep(587, t)
#define E1(t) Beep(659, t)
#define F1(t) Beep(698, t)
#define G1(t) Beep(784, t)
//调试预处理
#ifdef NDEBUG
#define trace(_EXPRESSION) ((void)0)
#define cnt(num) ((void)0)
#else
#define trace(_EXPRESSION) cout << fixed << setprecision(2) << _EXPRESSION
#define cnt(num) ++num
#endif
//命名空间和类型名
using namespace std;
//全局变量
int board[9][9];                               //棋盘
wchar_t base[9][9];                            //输出用的棋盘底
int status = END;                              //目前程序状态
int side = NEU;                                //电脑方执棋
int diff = NIL;                                //所选难度
vector<int> last;                              //记录上一手位置
int rnd = 0;                                   //回合数
bool con = false;                              //是否为人人对战中白方下棋前保存
bool visit[9][9];                              //dfs判断是否有气时记录是否搜索过
const int dx[8] = {-1, 1, 0, 0, -1, -1, 1, 1}; //dx指某个点周围的8个点的x坐标变化值
const int dy[8] = {0, 0, -1, 1, -1, 1, -1, 1}; //dy指某个点周围的8个点的y坐标变化值
const int angle[4] = {0, 8, 72, 80};           //四个角的坐标
const double gs = (sqrt(5) - 1) / 2;           //参数
int rndlist[81] =                              //用于打乱后生成随机数表
    {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10,
     11, 12, 13, 14, 15, 16, 17, 18, 19, 20,
     21, 22, 23, 24, 25, 26, 27, 28, 29, 30,
     31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
     41, 42, 43, 44, 45, 46, 47, 48, 49, 50,
     51, 52, 53, 54, 55, 56, 57, 58, 59, 60,
     61, 62, 63, 64, 65, 66, 67, 68, 69, 70,
     71, 72, 73, 74, 75, 76, 77, 78, 79, 80};
struct point //存储每个点的坐标和估值
{
    int pos;
    double val;
};
//辅助函数和宏
#define distance(x, y) sqrt(abs(x / 9 - y / 9) + abs(x % 9 - y % 9)) //计算两点距离
#define coefficient(r) log(1 + gs * (r + 1))                         //系数
#define border(x, y) (x >= 0 && y >= 0 && x < 9 && y < 9)            //判断坐标是否在界内
inline bool cmp(const point &a, const point &b) noexcept             //用于排序
{
    return a.val > b.val;
}
inline bool is_num(char ch) noexcept //判断是否为数字
{
    return ch >= '0' && ch <= '9';
}
//函数声明
//基本运行
void start();                //开始游戏的准备
void game();                 //人机
void human();                //人人
void print_menu() noexcept;  //打印主菜单
void make_board() noexcept;  //制作棋盘背景
void print_board() noexcept; //打印棋盘
void menu_input();           //主菜单界面的输入
int game_input();            //游戏界面的输入
//功能
void print_rule() noexcept; //帮助文档
bool save() noexcept;       //存档
bool reload() noexcept;     //读档
bool retract() noexcept;    //悔棋
void playback() noexcept;   //回放
//裁判
bool air(int x, int y);                   //判断是否有气
bool available(int x, int y, int myside); //判断是否可落子
bool judge(int thisside) noexcept;        //判断这一方是否胜利
//算法
int evaPoint(int pos, int myside);                                          //用于贪心的估值
int seperate(const vector<int> &pos, int myside, const vector<int> &piece); //打散方法
int choose_easy();                                                          //随机落子
int choose_normal();                                                        //两步估值贪心
int choose_hard();                                                          //改进的MCTS
//MCTS节点
class node //MCTS博弈树节点
{
public:
    int board[9][9] = {};             //棋盘
    node *parent = nullptr;           //父节点
    vector<node *> child;             //子节点
    vector<point> valid;              //所有可走点，用于扩展
    int index = 0;                    //valid的下标
    int total = 0;                    //总模拟次数
    int role = 0;                     //下一手下棋方
    int pos = -1;                     //这一手走的坐标
    int layer = 0;                    //向下搜索层数
    double val = 0;                   //该点估值
    double win = 0;                   //获胜分数
    double score = 0;                 //UCB分数
    inline bool find_valid() noexcept //寻找可下点
    {
        static point p;
        for (int i = 0; i < 9; ++i)
            for (int j = 0; j < 9; ++j)
                if (available(i, j, role))
                {
                    p.pos = i * 9 + j;
                    p.val = evaPoint(i * 9 + j, role);
                    valid.emplace_back(p);
                }
        sort(valid.begin(), valid.end(), cmp); //按照估值排序
        int maxvld = (100 - rnd) / 3;
        if (valid.size() > maxvld)
            valid.erase(valid.begin() + maxvld, valid.end());
        return !valid.empty();
    }
    inline node *select() //选择节点
    {
        if (valid.empty()) //叶节点，直接返回
            return this;
        if (!index || (index < valid.size() && total > gradual())) //使用渐进展开
        {
            int pos = valid[index].pos;
            node *n = new node; //扩展和初始化
            memcpy(n->board, board, sizeof(board));
            n->parent = this;
            n->role = -role;
            n->pos = pos;
            n->layer = layer + 1;
            n->board[pos / 9][pos % 9] = role;
            n->val = MULTYVAL * valid[index++].val;
            n->find_valid();
            child.emplace_back(n);
            return n;
        }
        double maxscore = -INFINITY;
        node *bestchild = nullptr; //完全扩展，从子结点中选择一个继续向下选择
        for (auto &it : child)
        {
            it->score = it->UCB();
            if (maxscore < it->score)
            {
                maxscore = it->score;
                bestchild = it;
            }
        }
        assert(bestchild);
        return bestchild->select();
    }
    inline double simulate() //模拟行棋
    {
        if (valid.empty())
        {
            double val = evaend(true);
            if (role == 1)
                return val * log(val + 1) / 3 + 1.2;
            return val * log(1 - val) / 3 - 1.2;
        }
        shuffle(rndlist, rndlist + 81, default_random_engine(clock())); //打乱随机数表
        static int board_cpy[9][9] = {};
        int thisside = role;
        bool end = false;                        //判断是否结束游戏
        memcpy(board_cpy, board, sizeof(board)); //存储副本
        int t = 20;
        for (; t > 0; --t) //最大模拟次数，达到后直接停止用估值判断
        {
            int pos = -1;
            for (auto i : rndlist) //遍历随机数表找可下点
                if (available(i / 9, i % 9, thisside))
                {
                    pos = i; //找到，跳出
                    break;
                }
            if (pos == -1) //未找到，游戏结束，thisside一方输
                break;
            board[pos / 9][pos % 9] = thisside;
            thisside = -thisside;
        }
        double val = evaend(end); //估值
        memcpy(board, board_cpy, sizeof(board));
        if (end) //游戏结束
        {
            if (thisside == 1)
                return val * log(val + 1) / 3 + 1 + t / 100.0;
            return val * log(1 - val) / 3 - 1 - t / 100.0;
        }
        val /= 5; //未结束，估值方法略有不同
        if (val >= 0)
            return val * log(val + 1) / 3;
        return val * log(1 - val) / 3;
    }
    static inline void backtrace(node *n, const double res) noexcept //回传
    {
        n->win += res;    //胜利分数增加
        ++(n->total);     //模拟次数增加
        if (!(n->parent)) //根节点，停止
            return;
        backtrace(n->parent, res);
    }
    static inline void clear(node *n) noexcept //删除树
    {
        if (n->child.empty()) //叶节点，直接删除
        {
            delete n;
            return;
        }
        for (auto &it : n->child) //非叶节点，递归删除每个子结点
            clear(it);
        delete n; //之后删除该节点
    }

private:
    friend bool air(int x, int y);
    friend bool available(int x, int y, int myside);
    inline double UCB() const noexcept //计算UCB值
    {
        return 12 * sqrt(log(parent->total) / total) + role * side * win / total + gs * val / coefficient(rnd + layer);
    }
    inline double gradual() const noexcept //渐进展开用
    {
        return pow(81 - rnd - layer, 2) * 0.01 * gs * pow(8, valid[index - 1].val - valid[index].val);
    }
    inline int evaend(bool end) //模拟后估值
    {
        static bool ava[2][9][9]; //存储哪些点可下
        memset(ava, 0, sizeof(ava));
        int val = 0;
        for (int i = 0; i < 9; ++i)
            for (int j = 0; j < 9; ++j)
                if (!board[i][j])
                {
                    if (available(i, j, side)) //本方可下
                    {
                        ++val;
                        ava[0][i][j] = true;
                    }
                    if (available(i, j, -side)) //对方可下
                    {
                        --val;
                        ava[1][i][j] = true;
                    }
                }
        if (end) //模拟对局游戏结束，估值只到这一步
            return val;
        val *= 4;
        for (int x = 0; x < 9; ++x) //对眼位和虎口进一步估值
            for (int y = 0; y < 9; ++y)
                if (!board[x][y])
                {
                    int me = 0, opp = 0;
                    for (int i = 0; i < 4; ++i)
                    {
                        int nx = x + dx[i], ny = y + dy[i];
                        if (!border(nx, ny))
                        {
                            ++me, ++opp;
                            continue;
                        }
                        if (board[nx][ny] == side)
                            ++me;
                        else if (board[nx][ny] == -side)
                            ++opp;
                        else
                        {
                            if (me == 3)
                            {
                                if (!ava[0][nx][ny])
                                    --me;
                                if (!ava[1][nx][ny] && !ava[1][x][y])
                                    ++me;
                            }
                            else if (opp == 3)
                            {
                                if (!ava[1][nx][ny])
                                    --opp;
                                if (!ava[0][nx][ny] && !ava[0][x][y])
                                    ++opp;
                            }
                        }
                    }
                    if (me == 4)
                        ++val;
                    else if (opp == 4)
                        --val;
                    else if (me == 3 && !opp)
                        ++val;
                    else if (opp == 3 && !me)
                        --val;
                }

        return val;
    }
    inline double evaPoint(int pos, int myside) //对点估值
    {
        if (pos == -1) //输入-1说明无子可下
            return -INFINITY;
        double val = 0;
        vector<int> piece; //存储自己已有子的点，用于计算距离
        const int x = pos / 9, y = pos % 9;
        board[x][y] = myside; //模拟自己落子
        for (int i = 0; i < 9; ++i)
            for (int j = 0; j < 9; ++j)
            {
                if (board[i][j])
                    continue;
                if (!available(i, j, -myside)) //对方不可下
                    val += 6.4;
                if (!available(i, j, myside)) //己方不可下
                    val -= 6.4;
            }
        board[x][y] = -myside; //模拟对方落子
        for (int i = 0; i < 9; ++i)
            for (int j = 0; j < 9; ++j)
            {
                if (board[i][j])
                {
                    if (board[i][j] == myside) //存储本方棋子
                        piece.emplace_back(i * 9 + j);
                    continue;
                }
                if (!available(i, j, -myside))
                    --val;
                if (!available(i, j, myside))
                    ++val;
            }
        board[x][y] = NEU;
        const double r = coefficient(rnd + layer);
        val += (1 - gs) * distance(pos, 40) / r;
        if (piece.empty())
            return val;
        int surme = 0, suropp = 0;
        for (int i = 0; i < 4; ++i) //遍历周围
        {
            const int nx = x + dx[i], ny = y + dy[i];
            if (!border(nx, ny))
            {
                ++surme, ++suropp;
                continue;
            }
            else if (board[nx][ny] == myside)
                ++surme;
            else if (board[nx][ny] == -myside)
                ++suropp;
        }
        if (surme == 3 && !suropp) //堵自己虎口
            val -= 1.4;
        else if (surme == 4) //堵自己眼
            val -= 2.2;
        else if (suropp == 3 && !surme) //堵对方虎口
            val += 2.2;
        double mindis = INFINITY;
        for (auto it : piece) //计算与本方棋子最小距离
        {
            double dis = distance(it, pos);
            mindis = min(mindis, dis);
        }
        val += gs * mindis / r; //与本方棋子距离越大，附加估值越大
        return val;
    }
};
//函数原型
inline void start() //开始游戏
{
    rnd = 0;
    last.clear();
    bool flag = false;               //判断输入是否合法
    memset(board, 0, sizeof(board)); //新游戏重置棋盘
    system("cls");
    string choose;                             //用于读入玩家选择
    printf("Choose game mode 选择游戏模式\n"); //选择人机或者人人
    printf("1: with Computer 人机\n2: with Human 人人\n");
    do
    {
        getline(cin, choose);
        C1(500);
        switch (choose[0])
        {
        case '1':
        case 'c':
        case 'C': //人机
            flag = true;
            status = GAME; //切换状态至GAME
            break;
        case '2':
        case 'h':
        case 'H': //人人
            flag = true;
            status = HUMAN;
            break;
        default: //未知输入
            printf("Invalid input! 非法输入\n");
            break;
        }
    } while (!flag);
    if (status == GAME) //人机
    {
        flag = false;
        printf("\nChoose difficulty 选择难度\n"); //选择难度f
        printf("1: Easy\n2: Normal\n3: Hard\n");
        do
        {
            getline(cin, choose);
            C1(500);
            switch (choose[0])
            {
            case '1':
            case 'e': //简单
            case 'E':
                flag = true;
                diff = EASY;
                break;
            case '2':
            case 'n': //一般
            case 'N':
                flag = true;
                diff = NORMAL;
                break;
            case '3':
            case 'h': //困难
            case 'H':
                flag = true;
                diff = HARD;
                break;
            default: //非法输入
                printf("Invalid input! 非法输入\n");
                break;
            }
        } while (!flag);
        printf("\nChoose Black or White 选择黑白方\n"); //选择人类玩家执棋
        printf("1: Black\n2: White\n");
        flag = false;
        do //判断电脑执棋方
        {
            getline(cin, choose);
            C1(500);
            switch (choose[0])
            {
            case '1':
            case 'b': //人执黑
            case 'B':
                flag = true;
                side = WHITE;
                break;
            case '2':
            case 'w': //人执白
            case 'W':
                flag = true;
                side = BLACK;
                break;
            default: //非法输入
                printf("Invalid input! 非法输入\n");
                break;
            }
        } while (!flag);
        int i = angle[rand() % 4];
        if (side == BLACK) //电脑执黑，则第一手下在角
        {
            board[i / 9][i % 9] = BLACK;
            last.emplace_back(i);
            print_board();       //打印棋盘
            printf("Round 2\n"); //打印回合数
            rnd = 2;
            printf("Response: %d,%d\n", i % 9 + 1, i / 9 + 1); //电脑执黑则输出第一手位置
        }
        else
        {
            print_board();               //打印棋盘
            printf("Round %d\n", ++rnd); //打印回合数
        }
    }
    else //人人
    {
        print_board();
        printf("Round %d\n", ++rnd); //打印回合数
    }
}
inline void game() //人机对战主程序
{
    int pos_human = -1, pos_ai = -1, invalid = 3;
    do
    {
        if (side == WHITE)
            printf("Turn: Black (You)\n");
        else
            printf("Turn: White (You)\n");
        do
        {
            pos_human = game_input(); //人类玩家输入
            trace(pos_human << endl);
            if (pos_human == END) //输入的内容为退出
            {
                C1(400), G(400), E(400), C(500);
                status = END;
                return;
            }
            else if (pos_human == HELP)
            {
                side = -side;
                int help = choose_hard(); //用hard难度的AI返回位置
                printf("Tip: %d %d\n", help % 9 + 1, help / 9 + 1);
                G(400), A(400), B(400), C1(500);
                side = -side;
                continue;
            }
            else if (pos_human == REGRET)
            {
                if (retract())
                    rnd -= 2;
                print_board();
                printf("Round %d\n", rnd);
                if (side == WHITE)
                    printf("Turn: Black (You)\n");
                else
                    printf("Turn: White (You)\n");
                C(400), D(400), E(400), C(400);
                continue;
            }
            if (available(pos_human / 9, pos_human % 9, -side)) //落点合法，跳出
                break;
            else //落点违规
            {
                printf("Invalid placing (remain %d)\n", --invalid);
                if (!invalid)
                    goto lose;
            }
        } while (true);
        board[pos_human / 9][pos_human % 9] = -side; //记录人类玩家落子
        last.emplace_back(pos_human);
        print_board();
        printf("Round %d\n", ++rnd); //打印回合数
        if (side == WHITE)
            printf("Turn: White (Computer)\n");
        else
            printf("Turn: White (Computer)\n");
        printf("Waiting for the response\n");
        pos_ai = -1;
        switch (diff) //电脑搜索，根据难度选用不同算法
        {
        case EASY:
            pos_ai = choose_easy();
            break;
        case NORMAL:
            pos_ai = choose_normal();
            break;
        case HARD:
            pos_ai = choose_hard();
            break;
        }
        trace(pos_ai << endl);
        if (pos_ai == -1) //电脑无处落子，人类玩家获胜
        {
            print_board();
            printf("You Win\n");
            C(400), E(400), G(400), C1(500);
            status = END;
            break;
        }
        board[pos_ai / 9][pos_ai % 9] = side; //记录电脑落子
        last.emplace_back(pos_ai);
        print_board();                                              //输出此时棋盘情况
        printf("Round %d\n", ++rnd);                                //打印回合数
        printf("Reponse: %d,%d\n", pos_ai % 9 + 1, pos_ai / 9 + 1); //继续游戏，输出电脑反应
        if (judge(side))                                            //判断人类玩家是否有子可下，不可则电脑获胜
        {
        lose:
            printf("You Lose\n");
            F(400), E(400), D(400), C(500);
            status = END;
            break;
        }
        G1(300);
    } while (status != END); //status==END时说明游戏停止
    string endcmd;
    do
    {
        printf("1: Playback\nElse: Back to Main Menu\n");
        getline(cin, endcmd);
        C1(500);
        if (endcmd.empty())
        {
            C1(400), G(400), E(400), C(500);
            break;
        }
        if (endcmd[0] == '1' || endcmd[0] == 'P' || endcmd[0] == 'p')
            playback();
        else
        {
            C1(400), G(400), E(400), C(500);
            break;
        }
    } while (true);
}
inline void human() //人人对战主程序
{
    int black = -1, white = -1, invb = 3, invw = 3;
    if (con) //特判白棋是否恢复存档而回到游戏
    {
        con = false;
        goto intogame;
    }
    do
    {
        printf("Turn: Black\n");
        do
        {
            side = BLACK;
            black = game_input();
            trace(black << endl);
            if (black == END) //输入的内容为退出
            {
                C1(400), G(400), E(400), C(500);
                status = END;
                return;
            }
            else if (black == HELP) //黑方需要提示
            {
                int help = choose_hard(); //用hard难度的AI返回位置
                printf("Tip: %d %d\n", help % 9 + 1, help / 9 + 1);
                G(400), A(400), B(400), C1(500);
                continue;
            }
            else if (black == REGRET)
            {
                if (retract())
                    rnd -= 2;
                print_board();
                printf("Round %d\n", rnd);
                printf("Turn: Black\n");
                C(400), D(400), E(400), C(400);
                continue;
            }
            if (available(black / 9, black % 9, BLACK)) //落点合法，跳出
                break;
            else //落点违规
            {
                printf("Invalid placing (remain %d)\n", --invb);
                if (!invb)
                    goto blose;
            }
        } while (true);
        board[black / 9][black % 9] = BLACK; //记录黑方玩家落子
        last.emplace_back(black);
        print_board();               //输出此时棋盘情况
        printf("Round %d\n", ++rnd); //打印回合数
        if (judge(BLACK))            //判断黑方是否胜利
        {
        wlose:
            printf("Black Win\n");
            status = END;
            break;
        }
        printf("Response: %d,%d\n", black % 9 + 1, black / 9 + 1);
    intogame:
        printf("Turn: White\n");
        do
        {
            side = WHITE;
            white = game_input();
            trace(white << endl);
            if (white == END) //输入的内容为退出
            {
                C1(400), G(400), E(400), C(500);
                status = END;
                return;
            }
            else if (white == HELP) //白方需要提示
            {
                int help = choose_hard(); //用hard难度的AI返回位置
                printf("tip: %d %d\n", help % 9 + 1, help / 9 + 1);
                G(400), A(400), B(400), C1(500);
                continue;
            }
            else if (white == REGRET)
            {
                if (retract())
                    rnd -= 2;
                print_board();
                printf("Round %d\n", rnd);
                printf("Turn: White\n");
                C(400), D(400), E(400), C(400);
                continue;
            }
            if (available(white / 9, white % 9, WHITE)) //落点合法，跳出
                break;
            else //落点违规
            {
                printf("Invalid placing (remain %d)\n", --invw);
                if (!invw)
                    goto wlose;
            }
        } while (true);
        board[white / 9][white % 9] = WHITE; //记录白方落子
        last.emplace_back(white);
        print_board();               //输出此时棋盘情况
        printf("Round %d\n", ++rnd); //打印回合数
        if (judge(WHITE))            //判断白方是否胜利
        {
        blose:
            printf("White Win\n");
            status = END;
            break;
        }
        printf("Response: %d,%d\n", white % 9 + 1, white / 9 + 1);
    } while (status != END); //status==END时说明游戏停止
    C(400), E(400), G(400), C1(500);
    string endcmd;
    do
    {
        printf("1: Playback\nElse: Back to Main Menu\n");
        getline(cin, endcmd);
        C1(500);
        if (endcmd.empty())
        {
            C1(400), G(400), E(400), C(500);
            break;
        }
        if (endcmd[0] == '1' || endcmd[0] == 'P' || endcmd[0] == 'p')
            playback();
        else
        {
            C1(400), G(400), E(400), C(500);
            break;
        }
    } while (true);
}
inline void print_menu() noexcept //输出主菜单
{
    status = MENU;
    system("cls");
    printf("NoGo\n\n");
    printf("1: New Game 新游戏\n");
    printf("2: Continue 继续\n");
    printf("3: Help 帮助文档\n");
    printf("4: Exit 退出\n\n");
    printf("by PbXBear 熊辰浩 2000017715\n\n");
}
inline void make_board() noexcept //制作输出用的棋盘
{
    system("chcp 65001");          //设置文件编码格式
    setlocale(LC_CTYPE, ".65001"); //设置本地信息 以上两步均是打印棋盘制表符准备操作
    system("cls");
    base[0][0] = L'┌';
    base[0][8] = L'┐';
    base[8][0] = L'└';
    base[8][8] = L'┘';
    for (int i = 1; i < 8; ++i)
    {
        base[0][i] = L'┬';
        base[i][0] = L'├';
        base[8][i] = L'┴';
        base[i][8] = L'┤';
    }
    for (int i = 1; i < 8; ++i)
        for (int j = 1; j < 8; ++j)
            base[i][j] = L'┼';
}
inline void print_board() noexcept //输出棋盘
{
    system("cls");
    for (int i = 1; i <= 9; ++i)
        printf(" %d", i); //输出列号
    for (int i = 0; i < 9; ++i)
    {
        printf("\n%d", i + 1); //输出行号
        for (int j = 0; j < 9; ++j)
        {
            switch (board[i][j])
            {
            case BLACK:
                printf("%lc", L'○');
                break;
            case WHITE:
                printf("%lc", L'●');
                break;
            case 0:
                printf("%lc", base[i][j]);
                break;
            }
            if (j < 8)
                printf("%lc", L'─');
        }
    }
    if (status == GAME)
    {
        printf("\nMode: ");
        switch (diff)
        {
        case EASY:
            printf("Easy\n");
            break;
        case NORMAL:
            printf("Normal\n");
            break;
        case HARD:
            printf("Hard\n");
            break;
        }
    }
    else
        printf("\nMode: Human 人人对战\n");
    printf("1: Save 存档\n");
    printf("2: Retract 悔棋\n");
    printf("3: Help 提示\n");
    printf("4: Exit and Save 退出\n\n");
}
inline void menu_input() //主菜单界面输入
{
    static string input;
    do
    {
        getline(cin, input);
        C1(500);
        auto first = find_if(input.begin(), input.end(), [&](char ch) { return ch != ' ' && ch != '\n'; });
        if (first == input.end())
            continue;
        switch (*first)
        {
        case '1':
        case 'n':
        case 'N':
            start(); //开始新一局
            return;
        case '2':
        case 'c':
        case 'C':
            if (!reload())
                printf("No archive\n");
            else
            {
                print_board();
                if (status == HUMAN && side == WHITE) //人人对战继续，判断是继续由黑还是白执棋
                    con = true;
                printf("Round: %d\n", rnd);
                return;
            }
            break;
        case '3':
        case 'h':
        case 'H':
            print_rule();
            G1(500);
            print_menu();
            break;
        case '4':
        case 'e':
        case 'E':
            status = EXIT; //退出程序
            return;
        default:
            printf("Invalid command\n"); //无效操作
            break;
        }
    } while (true);
}
inline int game_input() //游戏过程中界面输入
{
    static string input;
    do
    {
        getline(cin, input);
        C1(500);
        auto first = find_if(input.begin(), input.end(), [&](char ch) { return ch != ' ' && ch != '\n'; });
        if (first == input.end())
            continue;
        if (is_num(*first))
        {
            int pos = *first - '1';
            auto it = find_if(first + 1, input.end(), is_num); //查找数字
            if (it != input.end())
            {
                pos += (*it - '1') * 9;
                return pos;
            }
        }
        switch (*first)
        {
        case '1':
        case 's':
        case 'S': //存档
            if (save())
                printf("Saved successfully\n");
            else
                printf("Error\n");
            break;
        case '2':
        case 'r':
        case 'R': //悔棋
            return REGRET;
        case '3':
        case 'h':
        case 'H': //要求提示
            return HELP;
        case '4':
        case 'e':
        case 'E':
            save();     //保存
            return END; //退出
        default:
            printf("Invalid command\n"); //未知输入
            break;
        }
    } while (true);
    return ERR;
}
inline void print_rule() noexcept //帮助文档
{
    system("cls");
    printf("不围棋规则：\n");
    printf("游戏使用9×9棋盘，黑方先手执棋。\n");
    printf("一个（或一块）棋子周围的空位数称为“气”，无气的棋子将被提走。\n");
    printf("游戏过程中不得提走对方的子，也不得自杀。\n");
    printf("该游戏没有平局，若一方无合法落子位置，则判负。\n\n");
    printf("操作方法：\n");
    printf("每个需要输入命令的地方，均可输入其序号或者文本交互。\n\n");
    system("pause");
}
inline bool save() noexcept //存档
{
    ofstream fout("record.txt", ios::out);
    if (!fout.is_open())
        return false;
    fout << status << ' ' << side << ' ' << rnd << ' ' << diff << endl;
    for (auto &row : board) //向存档内写入棋盘
    {
        for (auto p : row)
            fout << p << ' ';
        fout << endl;
    }
    fout.close();
    C1(400), B(400), A(400), G(400);
    return true;
}
inline bool reload() noexcept //读档
{
    ifstream fin("record.txt", ios::in);
    if (!fin.is_open())
        return false;
    fin >> status >> side >> rnd >> diff;
    for (auto &row : board) //向存档内写入棋盘
        for (auto &p : row)
            fin >> p;
    fin.close();
    G(400), A(400), B(400), C1(400);
    return true;
}
inline bool retract() noexcept //悔棋
{
    if (last.size() <= 2)
        return false;
    board[last.back() / 9][last.back() % 9] = 0; //撤销上面两步
    last.pop_back();
    board[last.back() / 9][last.back() % 9] = 0; //撤销上面两步
    last.pop_back();
    return true;
}
inline void playback() noexcept //回放
{
    memset(board, 0, sizeof(board));
    print_board();
    printf("Round 0\n");
    for (int i = 0; i < rnd; ++i)
    {
        int pos = last[i];
        if (i % 2)
            board[pos / 9][pos % 9] = WHITE;
        else
            board[pos / 9][pos % 9] = BLACK;
        print_board();
        printf("Round %d\n", i + 1);
        if (i % 2)
            printf("White ");
        else
            printf("Black ");
        printf("Response: %d,%d\n", pos % 9 + 1, pos / 9 + 1);
        Sleep(600);
    }
    if (rnd % 2)
        printf("Black win\n");
    else
        printf("White win\n");
}
inline bool air(int x, int y) //判断是否有气
{
    visit[x][y] = true; //记录该点已搜索
    bool flag = false;
    for (int i = 0; i < 4; ++i)
    {
        int nearx = x + dx[i], neary = y + dy[i]; //分别向四周搜索
        if (nearx >= 0 && neary >= 0 && nearx < 9 && neary < 9)
        {
            if (board[nearx][neary] == 0) //有空格，说明有气
                flag = true;
            else if (board[nearx][neary] == board[x][y] && !visit[nearx][neary]) //找周围棋子是否有气
                flag += air(nearx, neary);
        }
    }
    return flag;
}
inline bool available(int x, int y, int myside) //判断是否可下
{
    if (!border(x, y)) //在界外
        return false;
    if (board[x][y]) //该点已经有子
        return false;
    board[x][y] = myside; //无子，假设该点落子
    memset(visit, 0, sizeof(visit));
    if (!air(x, y)) //无气，不可放
    {
        board[x][y] = NEU;
        return false;
    }
    for (int i = 0; i < 4; ++i)
    {
        int nearx = x + dx[i], neary = y + dy[i];
        if (border(nearx, neary) && board[nearx][neary] && !visit[nearx][neary] && !air(nearx, neary))
        { //周围点有子将无气，不可落子
            board[x][y] = NEU;
            return false;
        }
    }
    board[x][y] = NEU; //恢复原状
    return true;
}
inline bool judge(int thisside) noexcept //判断这一方是否获胜
{
    int ava = 0; //记录对方可下子的点位数
    for (int i = 0; i < 9; ++i)
        for (int j = 0; j < 9; ++j)
            if (available(i, j, -thisside))
                ++ava;
    return !ava; //对方无子可下
}
inline int seperate(const vector<int> &pos, int myside, const vector<int> &piece) //打散规则落子
{
    if (piece.empty()) //场上没有子，随机落子
    {
        int i = angle[rand() % 4];
        while (board[i / 9][i % 9])
            i = angle[rand() % 4];
        return i;
    }
    static vector<int> slc;
    slc.clear();
    double maxdis = 0;
    for (auto itp : pos)
    {
        double dis = 0;
        for (auto ite : piece)
        {
            double thisdis = distance(itp, ite);
            dis = min(dis, thisdis); //计算到己方场上棋子距离之和
        }
        if (maxdis < dis) //找最大距离可下点
        {
            maxdis = dis;
            slc.clear();
            slc.emplace_back(itp);
        }
        else if (maxdis == dis)
            slc.emplace_back(itp);
    }
    assert(!slc.empty());
    if (slc.size() == 1) //选取最大距离点
        return slc[0];
    return slc[rand() % slc.size()]; //多个相同距离，随机选取
}
inline int evaPoint(int pos, int myside) //用于贪心的估值
{
    if (pos == -1) //无子可落，返回极小值
        return -INT_MAX;
    board[pos / 9][pos % 9] = myside; //模拟为己方落点
    int val = 0;
    for (int i = 0; i < 9; ++i)
        for (int j = 0; j < 9; ++j)
        {
            if (board[i][j])
                continue;
            if (!available(i, j, -myside)) //对方不可下，估值增加
                val += 10;
            if (!available(i, j, myside)) //己方不可下，估值减少
                val -= 10;
        }
    board[pos / 9][pos % 9] = -myside; //模拟为对方落子，对方价值高的点可以抢占
    for (int i = 0; i < 9; ++i)
        for (int j = 0; j < 9; ++j)
        {
            if (board[i][j])
                continue;
            if (!available(i, j, -myside)) //导致对方不可下，则对方下该点概率减小
                val -= 3;
            if (!available(i, j, myside)) //导致对己方不可下，则对方下该点概率增大
                val += 3;
        }
    board[pos / 9][pos % 9] = NEU;
    val *= 10;
    int x = pos / 9, y = pos % 9, allside = 0; //记录xzy坐标和该点周围边界
    for (int i = 0; i < 4; ++i)
        if (border(x + dx[i], y + dy[i])) //判断其周围点在界内
            ++allside;
    //图案匹配法估值
    //pattern a
    if (allside == 2)
        return 10 + val;
    else if (allside == 3)
    {
        for (int i = 0; i < 4; ++i)
        {
            int nx = x + dx[i], ny = y + dy[i];
            if (!border(nx, ny))
                continue;
            int a = 0, b = 0;
            for (int j = 0; j < 4; ++j)
            {
                if (border(nx + dx[j], ny + dy[j]))
                {
                    ++a;
                    ++b;
                    continue;
                }
                if (board[nx + dx[j]][ny + dy[j]] == myside)
                    ++a;
                else if (board[nx + dx[j]][ny + dy[j]] == -myside)
                    ++b;
            }
            if (a == 3 && !b)
                val += 12;
            else if (!a && b == 3)
                val += 12;
            else if (!a && b == 2)
                val += 6;
        }
    }
    else if (allside == 4)
    {
        int nearp[8];
        for (int i = 0; i < 8; ++i)
            nearp[i] = board[x + dx[i]][y + dy[i]];
        //pattern b
        for (int i = 4; i < 8; ++i)
            if (nearp[i] == myside)
            {
                int s = 0;
                for (int j = 0; j < 4; ++j)
                    if (border(x + dx[i] + dx[j], y + dy[i] + dy[j]))
                        ++s;
                if (s == 2)
                    val += 9;
            }
        //pattern c&f
        if (nearp[0] == myside)
        {
            if (nearp[6] == myside && nearp[4] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (!border(x - 1 + dx[i], y - 1 + dy[i]) || board[x - 1 + dx[i]][y - 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
            else if (nearp[7] == myside && nearp[5] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (!border(x - 1 + dx[i], y + 1 + dy[i]) && board[x - 1 + dx[i]][y + 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
        }
        else if (nearp[1] == myside)
        {
            if (nearp[4] == myside && nearp[6] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (border(x + 1 + dx[i], y - 1 + dy[i]) && board[x + 1 + dx[i]][y - 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
            else if (nearp[5] == myside && nearp[7] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (!border(x + 1 + dx[i], y + 1 + dy[i]) && board[x + 1 + dx[i]][y + 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
        }
        else if (nearp[2] == myside)
        {
            if (nearp[5] == myside && nearp[4] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (!border(x - 1 + dx[i], y - 1 + dy[i]) && board[x - 1 + dx[i]][y - 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
            else if (nearp[7] == myside && nearp[6] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (!border(x + 1 + dx[i], y - 1 + dy[i]) && board[x + 1 + dx[i]][y - 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
        }
        else if (nearp[3] == myside)
        {
            if (nearp[4] == myside && nearp[5] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (!border(x - 1 + dx[i], y + 1 + dy[i]) && board[x - 1 + dx[i]][y + 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
            else if (nearp[6] == myside && nearp[7] == -myside)
            {
                int a = 0;
                for (int i = 0; i < 4; ++i)
                    if (!border(x + 1 + dx[i], y + 1 + dy[i]) && board[x + 1 + dx[i]][y + 1 + dy[i]] == myside)
                        ++a;
                if (a == 3)
                    return 9 + val;
            }
        }
        //pattern h&j&k
        int sur = 0;
        for (int i = 0; i < 4; ++i)
        {
            if (nearp[i] == -myside)
                ++sur;
            else if (nearp[i] == myside)
            {
                sur = 0;
                break;
            }
        }
        if (sur == 2)
        {
            int rme = 0;
            for (int i = 4; i < 8; ++i)
                if (nearp[i] == myside)
                    ++rme;
            if (rme != 2)
                val += 7;
        }
        else if (sur == 3)
            val += 16;
        // pattern g&i
        for (int i = 0; i < 4; ++i)
        {
            if (nearp[i])
                continue;
            int nx = x + dx[i], ny = y + dy[i];
            int a = 0, b = 0;
            for (int j = 0; j < 4; ++j)
            {
                if (border(nx + dx[j], ny + dy[j]))
                {
                    ++a;
                    ++b;
                    continue;
                }
                if (board[nx + dx[j]][ny + dy[j]] == myside)
                    ++a;
                else if (board[nx + dx[j]][ny + dy[j]] == -myside)
                    ++b;
            }
            if (a == 3 && !b)
                val += 13;
            else if (!a && b == 3)
                val += 13;
            else if (!a && b == 2)
                val += 5;
        }
    }
    return val;
}
inline int choose_easy() //随机落子
{
    shuffle(rndlist, rndlist + 81, default_random_engine(clock()));
    for (auto i : rndlist) //遍历随机数表找可下点
        if (available(i / 9, i % 9, side))
            return i;
    return -1;
}
inline int choose_normal() //两步估值贪心
{
    int maxval = -INT_MAX;
    static vector<int> valid, piece; //记录所有最大估值点和已有棋子
    piece.clear(), valid.clear();
    for (int i = 0; i < 9; ++i) //遍历棋盘
        for (int j = 0; j < 9; ++j)
        {
            if (board[i][j] == side)
                piece.emplace_back(i * 9 + j);
            else if (available(i, j, side)) //判断是否可落子
            {
                int val = evaPoint(i * 9 + j, side);
                if (val > maxval) //选取可以令对方估值最小的走法
                {
                    maxval = val;
                    valid.clear();
                    valid.emplace_back(i * 9 + j);
                }
                else if (val == maxval)
                    valid.emplace_back(i * 9 + j); //增加同样估值的点
            }
        }
    if (valid.empty()) //没有可落子的点
        return -1;
    if (valid.size() == 1) //返回唯一值
        return valid[0];
    int minval = INT_MAX;
    static vector<int> best;
    best.clear();
    for (auto p : valid)
    {
        board[p / 9][p % 9] = side;
        int val = -INT_MAX;         //记录对方最大估值
        for (int i = 0; i < 9; ++i) //遍历棋盘
            for (int j = 0; j < 9; ++j)
                if (available(i, j, -side)) //判断是否可落子
                {
                    int itval = evaPoint(i * 9 + j, -side); //估值
                    val = max(itval, val);
                }
        board[p / 9][p % 9] = NEU;
        if (val < minval) //选取可以令对方估值最小的走法
        {
            minval = val;
            best.clear();
            best.emplace_back(p);
        }
        else if (val == minval)
            best.emplace_back(p);
    }
    if (best.size() == 1)
        return best[0];
    return seperate(best, side, piece);
}
inline int choose_hard() //改进的MCTS
{
    const clock_t bgntime = clock();              //设定起始时间
    node *root = new node;                        //建立根节点
    memcpy(root->board, board, 81 * sizeof(int)); //根节点复制棋盘
    root->role = side;                            //根节点执棋方
    if (!root->find_valid())                      //寻找可下点
        return -1;
    while (clock() - bgntime < MAXTIME * CLOCKS_PER_SEC)
    {
        node *explore = root->select();                   //选择+扩展
        explore->backtrace(explore, explore->simulate()); //模拟+回传
    }
    double maxval = -INFINITY;
    int pos = -1;
    const double r = coefficient(rnd + 1);
    for (auto it : root->child)
    {
        const double val = it->win / it->total + (1 + 2 * gs) * it->val / r; //用估值修正的胜率
        trace(it->pos % 9 + 1 << ' ' << it->pos / 9 + 1 << ' ');
        trace(it->win << ' ');
        trace(it->total << ' ');
        trace(it->val << ' ');
        trace(val << endl);
        if (maxval < val)
        {
            maxval = val;
            pos = it->pos;
        }
    }
    assert(pos != -1);
    root->clear(root);
    return pos;
}
int main()
{
    srand(clock());        //设置随机数种子
    make_board();          //制作棋盘
    while (status != EXIT) //根据程序状态判断操作
        switch (status)
        {
        case MENU: //在菜单，要求一个输入操作
            menu_input();
            break;
        case GAME: //在游戏，进入游戏程序
            game();
            break;
        case HUMAN: //在人人游戏，进入人人游戏程序
            human();
            break;
        case END: //结束了游戏，进入主菜单
            print_menu();
            break;
        case EXIT: //退出程序
            C(400), E(400), G(400), C1(400), G(400), E(400), C(600);
            return 0;
        }
    return 0;
}
