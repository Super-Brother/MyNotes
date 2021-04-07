AS Run 出来的 Apk，之所以无法安装，是因为其携带了 `FLAG_TEST_ONLY` 这个 Flag，它会阻止我们使用正常的方式安装。想要安装，可以通过 `adb install -t` 来解决。

虽然这个 Flag 初始于 API Level 4，但是它在 AS 3.0 中，才被默认加入。想要去掉可以通过增加 `android.injected.testOnly=false` 来实现。

