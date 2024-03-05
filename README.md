# [HW1.P4] Dragon's Song: Awakening Tutorial
<!-- 文字敘述如何實作 -->
Keywords: `skip list`, `prefix sum`.

In the provided code, we utilize $N$ linked-lists to record the attacks by each classmate, and maintaining the association between rank and classmates through `Classmate.rank` and `rank_table`.

Below are implementations for each incident:
1. Swap the rankings of the two classmates and add a new record to the appointed classmate. Note that only two people change their rankings in this incident. $O(1)$.
2. We count the occurrences of this incident so far, denoted as `n_up`. To check a person's power, the `update()` function quickly calculates their current power. It's essential to invoke `update()` prior to any ranking change. $O(1)$.
* Use binary search to quickly find the answer. $O(\log N)$.
* For the basic problem, you can just maintain the summation of the last $M$ attacks for each classmate. $O(1)$
For the bonus problem, you should store the prefix sum in each node and apply skip list to quickly access the nodes. $O(\log N)$.

#### Other approaches
Alternatively, employing advanced data structures like vectors could reduce the time complexity of incident 4 to $O(1)$.
Another strategy involves pre-reading all input and allocating precisely sized arrays for each classmate, also achieving a time complexity of $O(1)$ for incident 4.

## sample code
<!-- 我到時候會整理出一份 code，註解函數、幾個重要變數在幹嘛 -->
```c
#include<stdio.h>
#include<stdlib.h>
#include<assert.h>

#define MAXN 1000006
typedef long long ll;
int N, M;
int n_up = 0;

typedef struct Node {
  struct Node *pre, *nxt, *below;
  ll value;
  int ind, level;
} Node;

Node *new_node(ll v, int ind, int level) {
  Node *node = (Node *) malloc(sizeof(Node));
  node->value = v;
  node->ind = ind;
  node->level = level;
  node->nxt = NULL;
  node->pre = NULL;
  node->below = NULL;
  return node;
}

typedef struct Classmate {
  ll p;
  Node *record_head; // head: top
  Node *cursor;
  int n_record;
  int rank; // classmate -> rank
  int last_up;
} Classmate;

Classmate player[MAXN];
int rank_table[MAXN]; // rank -> classmate

Node *buildup(Node *node) {
  Node *tmp = new_node(node->value, node->ind, node->level+1);
  tmp->below = node;
  return tmp;
}

void append_back(int i, ll v) {
  Node *path[40] = {}; // 2 * lg(5e5)
  int level[40] = {};
  Node *tmp = player[i].record_head;
  Node *head = player[i].record_head;
  for (int i = 1; 1; i++) {
    assert(i<40);
    while (tmp->nxt) tmp = tmp->nxt;
    path[i] = tmp;
    level[tmp->level] = i;
    if (!tmp->level) break;
    else tmp = tmp->below;
  }
  int new_ind = tmp->ind+1;
  int new_level = 0;
  v += tmp->value;
  Node *below = new_node(v, new_ind, new_level);
  tmp->nxt = below;
  while (rand()&1) {
    tmp = path[level[++new_level]];
    if (!tmp) {
      player[i].record_head = new_node(head->value, head->ind, head->level+1);
      player[i].record_head->below = head;
      head = player[i].record_head;
      tmp = head;
    }
    tmp->nxt = new_node(v, new_ind, new_level);
    tmp->nxt->below = below;
    below = tmp->nxt;
  }
  player[i].n_record++;
}

ll query(int i, int ind) {
  Node *tmp = player[i].record_head;
  while (tmp->ind < ind) {
    if (tmp->nxt && tmp->nxt->ind <= ind) tmp = tmp->nxt;
    else tmp = tmp->below;
  }
  return tmp->value;
}

void player_init(int i, int p, int rank) {
  player[i].p = p;
  player[i].cursor = player[i].record_head = new_node(0, 0, 0);
  player[i].n_record = 0;
  player[i].rank = rank;
  player[i].last_up = 0;
}
ll update(int i) {
  player[i].p += (ll)(N - player[i].rank) * (n_up - player[i].last_up);
  player[i].last_up = n_up;
  return player[i].p;
}
void top_up(int i, ll power) {
  player[i].p += power;
  append_back(i, power);
}

void swap_rank(int a, int b) {
  int tmp = player[a].rank;
  player[a].rank = player[b].rank;
  player[b].rank = tmp;
  rank_table[player[a].rank] = a;
  rank_table[player[b].rank] = b;
}

int binary_search(ll q) {
  if (q > update(rank_table[1])) return -1;
  int l = 1, r = N+1;
  while (r-l > 1) {
    int m = (l+r)/2;
    ll power = update(rank_table[m]);
    if (power < q) r = m;
    else l = m;
  }
  return l;
}

void incident() {
  int type;
  scanf("%d", &type);
  if (type == 1) {
    int a;
    scanf("%d", &a);
    int rank = player[a].rank;
    if (rank == 1) return;
    int pre_rank = rank-1;
    int pre = rank_table[pre_rank];
    ll diff = update(pre) - update(a);
    top_up(a, diff);
    swap_rank(a, pre);
  } else if (type == 2) {
    n_up++;
  } else if (type == 3) {
    ll q;
    scanf("%lld", &q);
    int rank = binary_search(q);
    if (rank == -1) printf("0 0\n");
    else printf("%d %d\n", rank, rank_table[rank]);
  } else if (type == 4) {
    int b, m;
    scanf("%d", &b);
    scanf("%d", &m);
    int r = player[b].n_record;
    int l = (r-m) > 0? (r-m):0;
    printf("%lld\n", query(b, r) - query(b, l));
  }
}

int main() {
  srand(48763);
  int T;
  scanf("%d%d%d", &N, &T, &M);
  for (int i = 1; i <= N; i++) {
    int p;
    scanf("%d", &p);
    player_init(i, p, i);
    rank_table[i] = i;
  }
  while (T--) {
    incident();
  }
  printf("\n");
  for (int i = 1; i <= N; i++) {
    printf("%d", player[i].n_record);
    Node *it = player[i].record_head;
    while (it->level) it = it->below;
    ll tmp = 0;
    it = it->nxt;
    while (it != NULL) {
      printf(" %lld", it->value - tmp);
      tmp = it->value;
      it = it->nxt;
    }
    printf("\n");
  }
  return 0;
}
```

<!-- 如果可以的話加上： -->
## common mistake
<!-- 寫幾個常見錯誤 -->
## coding tips
<!-- 一些簡化程式複雜程度的技巧 -->