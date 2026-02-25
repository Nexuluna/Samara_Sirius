# Samara_Sirius
code_cpp
#include <bits/stdc++.h>

using namespace std;

struct DSU {
    vector<int> p;
    DSU(int n) {
        p.resize(n);
        for (int i = 0; i < n; ++i) p[i] = i;
    }
    int find(int i) {
        if (p[i] == i) return i;
        return p[i] = find(p[i]);
    }
    void unite(int i, int j, const vector<string>& words) {
        int r1 = find(i);
        int r2 = find(j);
        if (r1 != r2) {
            if (words[r1] < words[r2]) p[r2] = r1;
            else p[r1] = r2;
        }
    }
};

string normalize(const string& s) {
    string r = "";
    for (char c : s) {
        if (isalpha((unsigned char)c)) r += (char)tolower((unsigned char)c);
        else if (c == '\'') r += c;
    }
    return r;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL);
    int K;
    if (!(cin >> K)) return 0;
    string line;
    vector<string> raw;
    set<string> uset;
    while (getline(cin, line)) {
        stringstream ss(line);
        string w;
        while (ss >> w) {
            string c = normalize(w);
            if (!c.empty()) {
                raw.push_back(c);
                if (c.length() > 1) uset.insert(c);
            }
        }
    }
    if (raw.empty()) return 0;
    vector<string> uwords(uset.begin(), uset.end());
    unordered_map<string, int> w2id;
    for (int i = 0; i < (int)uwords.size(); ++i) w2id[uwords[i]] = i;
    DSU dsu(uwords.size());
    unordered_map<string, vector<int>> masks;
    for (int i = 0; i < (int)uwords.size(); ++i) {
        string w = uwords[i];
        int len = w.length();
        for (int j = 0; j < len; ++j) {
            char old = w[j];
            w[j] = '*';
            masks[to_string(len) + "_" + w].push_back(i);
            w[j] = old;
        }
    }
    for (auto const& [key, ids] : masks) {
        for (size_t i = 1; i < ids.size(); ++i) {
            dsu.unite(ids[0], ids[i], uwords);
        }
    }
    for (int i = 0; i < (int)uwords.size(); ++i) {
        string w = uwords[i];
        string we = w + "e";
        string ws = w + "s";
        if (w2id.count(we)) dsu.unite(i, w2id[we], uwords);
        if (w2id.count(ws)) dsu.unite(i, w2id[ws], uwords);
    }
    vector<int> ids;
    for (const string& w : raw) {
        if (w.length() > 1) ids.push_back(w2id[w]);
        else ids.push_back(-1);
    }
    unordered_map<int, int> counts;
    for (int i = 0; i < (int)ids.size(); ++i) {
        if (ids[i] == -1) continue;
        int root = dsu.find(ids[i]);
        bool ok = false;
        int start = max(0, i - K);
        int end = min((int)ids.size() - 1, i + K);
        for (int j = start; j <= end; ++j) {
            if (i == j || ids[j] == -1) continue;
            if (dsu.find(ids[j]) == root) {
                ok = true;
                break;
            }
        }
        if (ok) counts[root]++;
    }
    struct Res { string s; int c; };
    vector<Res> out;
    for (auto const& [rid, c] : counts) out.push_back({uwords[rid], c});
    sort(out.begin(), out.end(), [](const Res& a, const Res& b) {
        if (a.c != b.c) return a.c > b.c;
        return a.s < b.s;
    });
    for (const auto& r : out) cout << r.s << ": " << r.c << "\n";
    return 0;
}
