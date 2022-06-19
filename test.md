<details>
<summary>Code</summary>
<pre><code class="language-cpp">int found(int a[], int left, int right, int x) {
    while (left < right) {
        int mid = (right + left) >> 1;
        if (a[mid] < x) left = mid + 1;
        else
            right = mid;
    }
    return left;
}
</code></pre>
</details>
s
------------
<details><summary>代码名称</summary>
~~~java
System.out.println("hello");
~~~
</details>