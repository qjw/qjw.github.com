---
layout: post
title: linux安装Jekyll
category: other
---

和Windows相比，Linux下安装Jekyll真心方便

	#!/bin/bash	
	aptitude install jekyll
	aptitude install rails
	aptitude install ruby1.9.1 ruby1.9.1-dev
	gem install rdiscount
	jekyll --server
