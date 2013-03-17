---
layout: post
title:  echo输出换行
category: bash
---

		qiuji_000@king-pc /tmp
		$ echo "aa\nbb"
		aa\nbb

		qiuji_000@king-pc /tmp
		$ echo "aa"\n"bb"
		aanbb

		qiuji_000@king-pc /tmp
		$ echo "aa"$'\n'"bb"
		aa
		bb



##参考
1. <http://bbs.chinaunix.net/thread-1727336-1-1.html>
