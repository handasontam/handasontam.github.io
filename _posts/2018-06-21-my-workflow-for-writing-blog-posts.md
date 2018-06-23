---
layout: article
title: My workflow for writing blog posts
---

## Steps to write a post
1. Switch to the **source** branch and write a new post with markdown format and save it in the **_posts** directory

   ```shell
   $ git checkout source
   ```

2. Test it locally

   ```shell
   $ jekyll build
   $ jekyll serve
   ```

3. Push to github

   ```shell
   $ rake
   ```

   As github page only support a few [whitelisted jekyll plugins](https://pages.github.com/versions/) due to security reasons. Many
useful plugins like [jekyll-scholad](https://github.com/inukshuk/jekyll-scholar), [jekyll-katex](https://github.com/linjer/jekyll-katex) cannot be used.

   A workaround is to build the site locally using ```jekyll build``` and push the resulting html (everything inside the **\_site** folder) to the master branch.

   To Automate the whole process, I modified the Rakefile from [David Ensinger](http://davidensinger.com/2013/07/automating-jekyll-deployment-to-github-pages-with-rake/). You can see the modified script [here](https://github.com/handasontam/handasontam.github.io/blob/source/Rakefile). This command will run ```jekyll build``` on the **source** branch and copy the evrythin in the **_site** directory to the **master** branch and push everything to github

## Some Demo
### Math equation using KaTeX!

$$
E(w_i,c_k,z) = -\sum_{j=1}^z w_{i,j} c_{k,j} + \Big[z\log a -\sum_{j=1}^z (w_{i,j}^2 +  c_{k,j}^2)\Big].
$$

### Code Snippet!

```python
def set_seed(x):
  """Set seed for both NumPy and TensorFlow.

  Parameters
  ----------
  x : int, float
    seed
  """
  np.random.seed(x)
  tf.set_random_seed(x)
```

```javascript
var s = "JavaScript syntax highlighting";
alert(s);
```
## Reference

[Jekyll](https://jekyllrb.com/).

[Markdown Cheat Sheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).
