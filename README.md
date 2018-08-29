See [https://handasontam.github.io/2018/06/21/my-workflow-for-writing-blog-posts.html]
Source can be found under the **source** branch.
## Dependencies
   https://jekyllrb.com/docs/installation/
   ```shell
   $ bundle install
   ```

## Steps to write a post
1. Switch to the **source** branch and write a new post with markdown format and save it in the **_posts** directory

   ```shell
   $ git checkout source
   $ vim _posts/2018-06-23-a-new-post.md
   # write a new blog post
   ```

2. Test it locally

   ```shell
   $ bundle exec jekyll build
   $ bundle exec jekyll serve
   ```

3. Push to github

   ```shell
   $ rake
   ```

   As github page only support a few [whitelisted jekyll plugins](https://pages.github.com/versions/) due to security reasons. Many
useful plugins like [jekyll-scholad](https://github.com/inukshuk/jekyll-scholar), [jekyll-katex](https://github.com/linjer/jekyll-katex) cannot be used.

   A workaround is to build the site locally using ```jekyll build``` and push the resulting html (everything inside the **\_site** folder) to the master branch.

   To Automate the whole process, I modified the Rakefile from [David Ensinger](http://davidensinger.com/2013/07/automating-jekyll-deployment-to-github-pages-with-rake/). You can see the modified script [here](https://github.com/handasontam/handasontam.github.io/blob/source/Rakefile). This command will run ```jekyll build``` on the **source** branch and copy everythin in the **_site** directory to the **master** branch and push everything to github


## Reference

[Jekyll](https://jekyllrb.com/).

[Markdown Cheat Sheet](https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet).

[TeXt Theme Docs](https://tianqi.name/jekyll-TeXt-theme/docs/en/quick-start).
