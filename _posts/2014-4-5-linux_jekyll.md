---
layout: post
title: linux安装Jekyll
category: www
---

和Windows相比，Linux下安装Jekyll真心方便

	#!/bin/bash	
	aptitude install jekyll
	aptitude install rails
	aptitude install ruby1.9.1 ruby1.9.1-dev
	gem install rdiscount
	jekyll server


gem install rdiscount有可能失败，这是因为被墙的缘故，重试，或者手动下载离线安装解决问题
