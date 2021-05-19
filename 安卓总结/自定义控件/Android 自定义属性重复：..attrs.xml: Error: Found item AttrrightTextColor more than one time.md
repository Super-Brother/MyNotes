自定义属性时，对于不同的 属性域，采用了相同的属性名，会引发错误：
`../attrs.xml: Error: Found item Attr/rightTextColor more than one time`

```
<declare-styleable name="s1">
	<attr name="rightTextColor"  format="color"/>
</declare-styleable>

<declare-styleable name="s2">
	<attr name="rightTextColor"  format="color"/>
</declare-styleable>
```

解决：
将属性定义在外部，内部只声明引用。

```
<attr name="rightTextColor" format="color" />

<declare-styleable name="s1">
	<attr name="rightTextColor"/>
</declare-styleable>

<declare-styleable name="s2">
	<attr name="rightTextColor"/>
</declare-styleable>
```

