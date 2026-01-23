---
title: PyCharm unittest Error:no tests were found
id: 13
date: 2023-04-10 18:19:13
auther: FREEDOM
cover: https://codeblog.net/upload/2021/03/image-1fb47f469f8844429199bcafff1d994d.png
excerpt: error like thissolved just add &quot;if name == 'main'&quot;
permalink: /archives/pycharmunittesterrornotestswerefound
categories:
 - code
tags: 
 - python
 - pycharm
---

error like this:

![image.png](https://codeblog.net/upload/2021/03/image-1fb47f469f8844429199bcafff1d994d.png)

![image.png](https://codeblog.net/upload/2021/03/image-b0651a4a32964776bf8445fa0e7954ed.png)

solved: just add "if __name__ == '__main__'"

![image.png](https://codeblog.net/upload/2021/03/image-7e82ed2a0133434aa640bfd7ef687eb1.png)