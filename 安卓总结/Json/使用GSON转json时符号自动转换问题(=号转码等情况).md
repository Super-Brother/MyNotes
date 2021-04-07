解决方案:

```java
Gson gson = new GsonBuilder().disableHtmlEscaping().create();
```

构造gson的时候只要调用`disableHtmlEscaping()`这个方法就好了。