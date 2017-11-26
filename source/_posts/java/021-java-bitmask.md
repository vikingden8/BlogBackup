---
title: Bitmask in Java
date: 2017-11-26 21:14:02
tags:
categories: "Java学习笔记"
---

转载自：[Java位运算在程序设计中的使用：位掩码（BitMask）](http://xxgblog.com/2013/09/15/java-bitmask/)

在Java中，位运算符有很多，例如与(&)、非(~)、或(|)、异或(^)、移位(<<和>>)等。这些运算符在日常编码中很少会用到。

在下面的一个例子中，会用到位掩码（BitMask），其中包含大量的位运算。不只是在Java中，其他编写语言中也是可以使用的。

例如，在一个系统中，用户一般有查询(Select)、新增(Insert)、修改(Update)、删除(Delete)四种权限，四种权限有多种组合方式，也就是有16中不同的
权限状态（2的4次方）。

### Permission

一般情况下会想到用四个boolean类型变量来保存：

```
public class Permission {
	
	// 是否允许查询
	private boolean allowSelect;
	
	// 是否允许新增
	private boolean allowInsert;
	
	// 是否允许删除
	private boolean allowDelete;
	
	// 是否允许更新
	private boolean allowUpdate;

	// 省略Getter和Setter
}
```

上面用四个boolean类型变量来保存每种权限状态。

<!--more-->

### NewPermission

下面是另外一种方式，使用位掩码的话，用一个二进制数即可，每一位来表示一种权限，0表示无权限，1表示有权限。

```
public class NewPermission {
	// 是否允许查询，二进制第1位，0表示否，1表示是
	public static final int ALLOW_SELECT = 1 << 0; // 0001
	
	// 是否允许新增，二进制第2位，0表示否，1表示是
	public static final int ALLOW_INSERT = 1 << 1; // 0010
	
	// 是否允许修改，二进制第3位，0表示否，1表示是
	public static final int ALLOW_UPDATE = 1 << 2; // 0100
	
	// 是否允许删除，二进制第4位，0表示否，1表示是
	public static final int ALLOW_DELETE = 1 << 3; // 1000
	
	// 存储目前的权限状态
	private int flag;

	/**
	 *  重新设置权限
	 */
	public void setPermission(int permission) {
		flag = permission;
	}

	/**
	 *  添加一项或多项权限
	 */
	public void enable(int permission) {
		flag |= permission;
	}
	
	/**
	 *  删除一项或多项权限
	 */
	public void disable(int permission) {
		flag &= ~permission;
	}
	
	/**
	 *  是否拥某些权限
	 */
	public boolean isAllow(int permission) {
		return (flag & permission) == permission;
	}
	
	/**
	 *  是否禁用了某些权限
	 */
	public boolean isNotAllow(int permission) {
		return (flag & permission) == 0;
	}
	
	/**
	 *  是否仅仅拥有某些权限
	 */
	public boolean isOnlyAllow(int permission) {
		return flag == permission;
	}
}
```

以上代码中，用四个常量表示了每个二进制位代码的权限项。

例如：

ALLOW_SELECT = 1 << 0 转成二进制就是0001，二进制第一位表示Select权限。

ALLOW_INSERT = 1 << 1 转成二进制就是0010，二进制第二位表示Insert权限。

private int flag存储了各种权限的启用和停用状态，相当于代替了Permission中的四个boolean类型的变量。

用flag的四个二进制位来表示四种权限的状态，每一位的0和1代表一项权限的启用和停用，如flag为1（0001）只允许查询（即等于ALLOW_SELECT），flag为
15(1111)，四项权限都允许等。

使用位掩码的方式，只需要用一个大于或等于0且小于16的整数即可表示所有的16种权限的状态。

此外，还有很多设置权限和判断权限的方法，需要用到位运算，例如：

```
public void enable(int permission) {
	flag |= permission; // 相当于flag = flag | permission;
}
```

调用这个方法可以在现有的权限基础上添加一项或多项权限,添加一项Update权限：

```
permission.enable(NewPermission.ALLOW_UPDATE);
```

假设现有权限只有Select，也就是flag是0001。执行以上代码，flag = 0001 | 0100，也就是0101，便拥有了Select和Update两项权限。

添加Insert、Update、Delete三项权限：

```
permission.enable(NewPermission.ALLOW_INSERT 
    | NewPermission.ALLOW_UPDATE | NewPermission.ALLOW_DELETE);
```

NewPermission.ALLOW_INSERT | NewPermission.ALLOW_UPDATE | NewPermission.ALLOW_DELETE运算结果是1110。假设现有权限只有Select，
也就是flag是0001。flag = 0001 | 1110，也就是1111，便拥有了这四项权限，相当于添加了三项权限。

上面的设置如果使用最初的Permission类的话，就需要下面三行代码：

```
permission.setAllowInsert(true);
permission.setAllowUpdate(true);
permission.setAllowDelete(true);
```

### 建议阅读

[BitMask 使用参考](https://www.liaohuqiu.net/cn/posts/bitmasp-and-lipo/)

