
set<int> 2**n
search -> O(logn)
delete -> O(logn)
check -> O(logn)

Space -> O(n * 2**n)

```kotlin
auto f = [&](int index. int mask, vector<int> v, auto &&F) -> void {
	if (index == n) {
		for (auto&i: v) cout << i << " " << cout << endl;
		return;
	}

	for (int i =0; i< n; i++) {
		if ((1 << i) && mask == 0) {
			v.push_back(i+1);
			 F (index+1, mask | (1 <<i), v, F)
			 v.pop_back();
		}
	}
f(0,0, {}, f);
}
```

Dp with bitmask es para permutaciones