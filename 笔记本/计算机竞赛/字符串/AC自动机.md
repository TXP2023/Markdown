# AC自动机 (多模式串匹配自动机, Aho–Corasick)

其为执行多模式串匹配自动机

[P5357 【模板】AC 自动机](https://www.luogu.com.cn/problem/P5357)

**AC自动机最重要的是失配指针$fail$, 理解了这个就理解了整个算法**

AC自动机要存储一下信息
```cpp
// AC 自动机数组
using ll = ;

int64_t trie_index[MAX_LENGTH][26]; // 子节点编号
int64_t fail[MAX_LENGTH];           // 失配指针
int64_t trie_cnt_arr[MAX_LENGTH];   // 节点匹配次数
int64_t end_id[MAX_LENGTH];         // 终止节点对应的模式串ID（首次出现）
int64_t strPos[MAX_LENGTH];         // 每个模式串首次出现的ID
int64_t ans[MAX_LENGTH];            // 每个模式串的匹配次数
int64_t inDegree[MAX_LENGTH];       // 入度
```

AC自动机的核心思想是使用模式串构建一个 $Trie$ 并在节点之间连接 $fail$ 指针， 让文本串可以同时匹配多个模式串

节点$u$的 $fail$ 指针指向最深的节点 $v$ 使得 $u$ 所代表的前缀有一个后缀为 $v$ 所代表的前缀

例如，现有2个模式串

$mods:$ ``ab``;  ``b``;

来看节点 $2$ 的 $fail$ 指针， 根据节点$2$的 $fail$ 指针指向最深的节点 $v$ 使得 $2$ 所代表的前缀有一个后缀为 $v$ 所代表的前缀

那么这个 $v$ 就是节点 $3$  因为 $2$ 所代表的前缀 ``ab`` 有一个后缀 ``b`` 为 $3$ 所代表的前缀

根节点的直接子节点的 $fail$ 指针就直接指向根节点

例图

![树的深度
示意图](/library/AC自动机.md.fail指针示意图.png)

构建 $fail$ 指针的核心代码如下

```cpp

// 构建失配指针（BFS）
void get_fail() {
    std::queue<int64_t> que;

    // 根节点的子节点失配指针指向根节点
    for (int64_t i = 0; i < 26; i++) {
        if (trie_index[ROOT][i]) {
            fail[trie_index[ROOT][i]] = ROOT;
            que.push(trie_index[ROOT][i]);
        }
    }

    // BFS 构建失配指针
    while (!que.empty()) {
        int64_t u = que.front();
        que.pop();

        for (int64_t i = 0; i < 26; i++) {
            if (trie_index[u][i]) {

                // 子节点的失配指针指向父节点失配指针的对应字符子节点
                fail[trie_index[u][i]] = trie_index[fail[u]][i];
                // // 增加入度（被多少失配指针指向）
                inDegree[trie_index[fail[u]][i]]++;
                // 子节点入队，继续 BFS
                que.push(trie_index[u][i]);
            }
            else {
                // 如果没有该字符子节点，则直接跳到失配节点对应的子节点
                trie_index[u][i] = trie_index[fail[u]][i];
            }
        }
    }
}
```

代码全览
```cpp
// AC 自动机数组
#define MAX_LENGTH

int64_t trie_index[MAX_LENGTH][26]; // 子节点编号
int64_t fail[MAX_LENGTH];           // 失配指针
int64_t trie_cnt_arr[MAX_LENGTH];   // 节点匹配次数
int64_t end_id[MAX_LENGTH];         // 终止节点对应的模式串ID（首次出现）
int64_t strPos[MAX_LENGTH];         // 每个模式串首次出现的ID
int64_t ans[MAX_LENGTH];            // 每个模式串的匹配次数
int64_t inDegree[MAX_LENGTH];       // 入度

int64_t trie_cnt = 0; // 节点计数（0是根节点）
int64_t n;            // 模式串数量
std::string text;      // 文本串

// 插入模式串（迭代版本）
void insertMod(const std::string& s, int64_t str_id) {
    int64_t u = ROOT;
    for (char ch : s) {
        int64_t c = ch - 'a';
        if (!trie_index[u][c]) trie_index[u][c] = ++trie_cnt;
        u = trie_index[u][c];
    }
    if (!end_id[u]) end_id[u] = str_id; // 记录首次出现的模式串ID
    strPos[str_id] = end_id[u];         // 所有相同的模式串指向首次出现的ID
}

// 构建失配指针（BFS）
void get_fail() {
    std::queue<int64_t> que;

    // 根节点的子节点失配指针指向根节点
    for (int64_t i = 0; i < 26; i++) {
        if (trie_index[ROOT][i]) {
            fail[trie_index[ROOT][i]] = ROOT;
            que.push(trie_index[ROOT][i]);
        }
    }

    // BFS 构建失配指针
    while (!que.empty()) {
        int64_t u = que.front();
        que.pop();

        for (int64_t i = 0; i < 26; i++) {
            if (trie_index[u][i]) {

                // 子节点的失配指针指向父节点失配指针的对应字符子节点
                fail[trie_index[u][i]] = trie_index[fail[u]][i];
                // // 增加入度（被多少失配指针指向）
                inDegree[trie_index[fail[u]][i]]++;
                //// 子节点入队，继续 BFS
                que.push(trie_index[u][i]);
            }
            else {
                // 如果没有该字符子节点，则直接跳到失配节点对应的子节点
                trie_index[u][i] = trie_index[fail[u]][i];
            }
        }
    }
}

// 将文本串匹配进 AC 自动机
void insert_text() {
    int64_t u = ROOT;
    for (char c : text) {
        u = trie_index[u][c - 'a'];
        trie_cnt_arr[u]++;
    }
}

// 统计所有模式串的匹配次数（拓扑排序 + 失配指针回溯）
void getAnswer() {
    std::queue<int64_t> que;
    for (int64_t i = 0; i <= trie_cnt; i++) {
        //// 入度为 0 的节点入队
        if (!inDegree[i]) {
            que.push(i);
        }
    }

    while (!que.empty()) {
        int64_t u = que.front();
        que.pop();
        // // 如果该节点是某个模式串的结尾，则记录匹配次数
        ans[end_id[u]] = trie_cnt_arr[u]; // 如果是模式串结尾节点，记录匹配次数
        int64_t v = fail[u];
        trie_cnt_arr[v] += trie_cnt_arr[u];
        // 失配节点入度减 1，若入度为 0，则入队
        if (!(--inDegree[v])) {
            que.push(v);
        }
    }
}

int main() {

    fread(&n);
    for (int64_t i = 1; i <= n; i++) {
        std::string mod;
        std::cin >> mod;
        insertMod(mod, i);
    }

    std::cin >> text;

    get_fail();
    insert_text();
    getAnswer();

    for (int64_t i = 1; i <= n; i++) {
        printf("%lld\n", ans[strPos[i]]);
    }
    return 0;
}
```