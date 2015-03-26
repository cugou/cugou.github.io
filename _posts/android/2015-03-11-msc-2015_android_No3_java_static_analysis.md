---
layout: post
title: MSC-2015移动安全挑战赛 第三题 java静态代码分析
category: android
tags: reverse
keywords: 
description: 
---

# java代码分析
 静态分析实在是坑爹，这个跳来跳去，还有混淆和干扰的代码实在是调试来的快一些。

```java
//Main.java
public void onCreate(android.os.Bundle p5)
{
    dn.b(dn.a());	//这个代码调用testdex2jarcrash，针对dex2jar的反编译
    super.onCreate(p5);
    this.setContentView(2130903040);
    this.b = ((android.widget.Button) this.findViewById(2131034115));
    this.a = ((android.widget.TextView) this.findViewById(2131034116));
	//注册按钮事件处理类
    this.b.setOnClickListener(new d(this, ((android.widget.EditText) this.findViewById(2131034114))));
    return;
}
```

```java
//d.java
public d(crackme.a3.Main p1, android.widget.EditText p2)
{
    this.b = p1;
    this.a = p2;
    return;
}

public void onClick(android.view.View p5)
{
    dn.b(dn.a());	//每个关键函数都有这个东东，呵呵
    this.b.b.setEnabled(0);
    try {
        this.b.a.setText("");
        try {
			//由于Main的c变量根本没有赋值，所以这里必然产生异常
            this.b.c.setText(2130968577);
        } catch (a v0) {
            new a().a(this.a.getText().toString(), crackme.a3.Main.b(this.b));
        }
        return;
    } catch (a v0) {
        throw new RuntimeException();
    }
}
```

```java
//a.java
public void a(String p6, android.os.Handler p7)
{
    dn.b(dn.a());	//每个都有
	//注册了一个计时器类来处理数据，2s后启动
	//跟踪用户的输入，这里应该是参数 p6
    new java.util.Timer().schedule(new b(this, p7, p6), 2000);
    return;
}
```

```java
//b.java
b(a p1, android.os.Handler p2, String p3)
{
    this.d = p1;
    this.b = p2;
    this.c = p3;	//跟踪用户输入
    return;
}

//分析用户输入的关键函数。
public void run()
{
    dn.b(dn.a());
	//这里存在反调试
    if ((android.os.Build$VERSION.SDK_INT < 10) || (!android.os.Debug.isDebuggerConnected())) {
        try {
			//调用 e 类的 a 方法对用户的字符串进行处理。这里v5_0类型应该是String，反编译工具有些问题。分析见下面的 e.java。
            byte[] v5_0 = new e().a(this.c);
        } catch (android.os.Handler v0) {
			//Handler一般用来更新UI线程的界面，是阻塞单线程模式，流水处理发来的消息。
			//这里用来显示用户输入判断后的结果。分析见下面的 e.java。
            this.b.sendEmptyMessage(3);
        }
        try {
            if (!v5_0.equals("sos")) {
                android.os.Handler v0_8 = new java.util.zip.CRC32();
                v0_8.update(v5_0.getBytes());
                v0_8.getValue();
                v5_0.hashCode();
                android.os.Handler v0_10 = java.security.MessageDigest.getInstance("sha1");
                try {
                    int v1_2 = javax.crypto.Cipher.getInstance("AES");
                } catch (android.os.Handler v0_12) {
                    v0_12.printStackTrace();
                }
                if ((b.a) || (v1_2 != 0)) {
                    try {
                        v1_2.init(2, new javax.crypto.spec.SecretKeySpec(android.util.Base64.decode("GXiQHT1CZ2elMzwpvvAoPA==".getBytes(), 0), "AES"));
                    } catch (byte[] v2) {
                    }
                    try {
                        byte[] v2_8 = v1_2.doFinal(android.util.Base64.decode("hjdsUjIT5je69WXIZP7Kzw==".getBytes("UTf-8"), 0));
                    } catch (int v1) {
                    } catch (int v1) {
                    }
                    int v6_7 = new String(v2_8);
                    int v1_6 = new byte[1];
                    v1_6[0] = 127;
                    v0_10.update(v1_6);
                    v0_10.update(v5_0.getBytes());
                    int v1_9 = new byte[1];
                    v1_9[0] = 1;
                    v0_10.update(v1_9);
                    int v1_11 = new String(android.util.Base64.encode(v0_10.digest(), 0));
                    android.os.Handler v0_17;
                    if (!v5_0.equals(v6_7)) {
                        v0_17 = v1_11;
                    } else {
                        if (!java.util.Arrays.equals(v1_11.getBytes(), "2398lj2n".getBytes())) {
                            v0_17 = "234";
                        } else {
                            this.b.sendEmptyMessage(0);
                        }
                    }
					//上面的CRC32、sha1、AES实际上都是干扰项。这个if里面的代码才是真正的判断条件。
                    if (!v0_17.equals("lsdf==")) {
                        int v1_15 = v5_0.toCharArray();
                        android.os.Handler v0_23 = v5_0.substring(0, 2).hashCode();
                        if (v0_23 <= 3904) {
                            if ((v0_23 == 3618) && ((v1_15[0] + v1_15[1]) == 168)) {
								//上面两个if是判断“密码”的前两个字符，满足hashCode()==3618和ASCII之和==168。
								//知道java的String.hashCode算法就可以列方程组求解，不知道也可以暴力一下。字符的取值范围见下面的 e.java，这里面是密码表。
                                do {
									//v5_9应该是String类型，反编译器的问题。
									//这个语句是取 e class 关于 f class 的 Annotation中的a()字符串 + a class 关于 f class 的 Annotion的a()字符串。使用010Editor，查看 e 和 a 的class_def_item中的Annotation
                                    byte[] v5_9 = new StringBuilder().append(((f) e.getAnnotation(f)).a()).append(((f) a.getAnnotation(f)).a()).toString().getBytes();
                                    if ((v1_15.length - 2) == v5_9.length) {
                                        android.os.Handler v0_39 = 0;
                                        while (v0_39 < v5_9.length) {
											//循环实际上是判断“密码”的后面的字符是不是等于v5_9。
                                            if (v1_15[(v0_39 + 2)] == v5_9[v0_39]) {
                                                v0_39++;
                                            } else {
                                                android.os.Handler v0_40 = 0;
                                            }
                                            if (v0_40 != null) {
                                                this.b.sendEmptyMessage(0);	//显示正确结果
                                            }
                                        }
                                        v0_40 = 1;
                                    }
                                } while(v2_8 == null);
                            }
							//这个代码就比较坑爹了，似乎刚显示正确的结果，这里又马上显示错误信息。不知道是不是反编译器错误导致。
                            this.b.sendEmptyMessage(1);
                        } else {
                            this.b.sendEmptyMessage(4);
                        }
                    } else {	//干扰项，sha1(string)不可能那么短
                        this.b.sendEmptyMessage(0);
                    }
                } else {
                    throw new AssertionError();
                }
            } else {
                this.b.sendEmptyMessage(2);
            }
        } catch (android.os.Handler v0) {
            this.b.sendEmptyMessage(1);
        }
    } else {
        this.b.sendEmptyMessage(1);
    }
    return;
}
```

```java
//e.java
//e.a方法将用户输入的空格间隔的摩斯码转换成字符串，所以要求用户的输入实际上是结果字符串对应的摩斯码，空格分割。
static e()
{
    e.a = new java.util.HashMap();
    e.a("a", ". _");
    e.a("b", "_ . . .");    //这里是调用 e 类的静态方法 a。很容易错误的理解为 e.a 是 HashMap 的构造函数。这样key和value就反了，而且还有一个去空格的步骤
    e.a("c", "_ . _ .");
    e.a("d", "_ . .");
    e.a("e", ".");
    e.a("f", ". . _ .");
    e.a("g", "_ _ .");
    e.a("h", ". . . .");
    e.a("i", ". .");
    e.a("j", ". _ _ _");
    e.a("k", "_ . _");
    e.a("l", ". _ . .");
    e.a("m", "_ _");
    e.a("n", "_ .");
    e.a("o", "_ _ _");
    e.a("p", ". _ _ .");
    e.a("q", "_ _ . _");
    e.a("r", ". _ .");
    e.a("s", ". . .");
    e.a("t", "_");
    e.a("u", ". . _");
    e.a("v", ". . . _");
    e.a("w", ". _ _");
    e.a("x", "_ . . _");
    e.a("y", "_ . _ _");
    e.a("z", "_ _ . .");
    e.a("2", ". _ _ _ _");
    e.a("1", ". . _ _ _");
    e.a("3", ". . . _ _");
    e.a("4", ". . . . _");
    e.a("0", ". . . . .");
    e.a("6", "_ . . . .");
    e.a("9", "_ _ . . .");
    e.a("8", "_ _ _ . .");
    e.a("7", "_ _ _ _ .");
    e.a("5", "_ _ _ _ _");
    return;
}

public e()
{
    return;
}

static void a(String p4, String p5)
{
    dn.b(dn.a());
    e.a.put(p5.replaceAll(" ", ""), p4);    //. -去空格，key = 莫斯码 value = ascii
    return;
}

public String a(String p8)
{
    IllegalArgumentException v0_4;
    dn.b(dn.a());
    if (!p8.equals("...___...")) {
        StringBuilder v2_1 = new StringBuilder();
        String[] v3 = p8.split("\\s+");     // \s表示空白字符     莫斯码之间用空白字符间隔
        int v4 = v3.length;
        int v1 = 0;
        while (v1 < v4) {
            IllegalArgumentException v0_7 = ((String) e.a.get(v3[v1]));     //莫斯码转换成字母
            if (v0_7 == null) {
                throw new IllegalArgumentException();
            } else {
                v2_1.append(v0_7);
                v1++;
            }
        }
        v0_4 = v2_1.toString();
    } else {
        v0_4 = "sos";
    }
    return v0_4;
}
```

```java
//c.java	Hander的message处理函数
public void handleMessage(android.os.Message p5)
{
    dn.b(dn.a());
    this.a.b.setEnabled(1);
    switch (p5.what) {
        case 0:
            this.a.a.setTextColor(-16776961);
            try {
                this.a.a.setText((103 / p5.what));	//除0总会产生异常
            } catch (android.widget.TextView v0) {
				//这个字符串差string.xml可知“坐标正确，准备启航!”。
				//所以在 a.run 中只有SendEmptyMessage(0)才是正确的。
                this.a.a.setText(2130968586);
            }
            break;
        case 1:
            this.a.a.setTextColor(-65536);
            switch (crackme.a3.Main.a(this.a).nextInt(3)) {
                case 0:
                    this.a.a.setText(2130968583);
                    break;
                case 1:
                    this.a.a.setText(2130968585);
                    break;
                case 2:
                    this.a.a.setText(2130968584);
                    break;
                default:
            }
            break;
        case 2:
            try {
                this.a.a.setTextColor(-65536);
            } catch (android.widget.TextView v0) {
                this.a.a.setTextColor(-7829368);
            }
            this.a.a.setText(2130968580);
            break;
        case 3:
            android.widget.TextView v0_9;
            this.a.a.setTextColor(-65536);
            if (!crackme.a3.Main.a(this.a).nextBoolean()) {
                v0_9 = 2130968581;
            } else {
                v0_9 = 2130968582;
            }
            this.a.a.setText(v0_9);
            break;
    }
    return;
}
```
```java
new StringBuilder().append(((f) e.getAnnotation(f)).a()).append(((f) a.getAnnotation(f)).a()).toString().getBytes()
```
这句代码调用类的getAnnotation方法获取类的Annotation信息。这个信息存储在类的class_def_item的Annotation相关字段中。下面是010Editor的截图：

![图](/public/img/e.getAnnotation_f_.a.png)
![图](/public/img/a.getAnnotation_f_.a.png)