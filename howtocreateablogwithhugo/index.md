# Create a blog with Hugo on Github


## write ahead

my blog code would help you create your own blog https://github.com/mobilephone724/hugoBlog/

## create a blog on local

[Quick Start | Hugo (gohugo.io)](https://gohugo.io/getting-started/quick-start/)

### install hugo

[Install Hugo | Hugo (gohugo.io)](https://gohugo.io/getting-started/installing)

It seems installation on windows is a little harder. The following actions are based on **linux**

### create a new site

```bash
hugo new site quickstart
```

The above will create a new Hugo site in a folder named `quickstart`.

### choose a theme

I have tried many themes, and in the end, `LoveIt` is what I love best. So I use it to create my blogs.

Install it as follow

[Theme Documentation - Basics - LoveIt (hugoloveit.com)](https://hugoloveit.com/theme-documentation-basics/)

and configure ` config.toml` as what you want. If you can't understand it completely right now, just copy all in `quickstart/themes/LoveIt/exampleSite` to `quickstart`. 

some notifications:
1.	The line `themesDir = "../.."` must be removed or hugo can't get right the right path of the theme
2.	you can remove `about`  directory in `content` because something in it prevents me from publishing my blog on `github` and I don't need the function with `about`. So I just removed it

### create your first post

I don't like the way on [Quick Start | Hugo (gohugo.io)](https://gohugo.io/getting-started/quick-start/) to create a post, for inserting pictures can be undesirable. The best way for me now it as follows

```bash
cd content/posts
mkdir my_first_post
cd my_first_post
mkdir pic 		# create a directory to store pictures.
touch index.md 	# a markdown file in which you write a blog
```

The advantage is that the `path` of pictures is like `pic/picture.png`. It is very similar to the way to write in `typora`

notice that you can use multi-level directories like

```bash
ubuntu@VM-4-13-ubuntu:~/quickstart/content/posts$ tree .
.
├── APUE
│   └── chapter3
│       ├── index.md
│       └── pic
│           ├── file.png
│           ├── Screenshot-2021-05-04-19-58-12.png
│           ├── Screenshot-2021-05-04-20-40-06.png
│           └── Screenshot-2021-05-05-09-34-21.png
```

then write something to `index.md`(If you don't know what to write, copy a post in my blogs)

and now see your blog on website

```
hugo server #in directory quickstart
```

open website at `http://localhost:1313/` to see your blog

### create a github account

It may be quite easy for you do so. Suppose your github user name is `hugoAuthor`

### create a repo

The repo's name must be like `hugoAuthor.github.io` and visibility must be `public`

### push your blog to github page

in `quickstart` directory

```bash
hugo
```

This command will create a directory named `public` in which is what all we need to show our blog.

What we'll do following is push  content in `public` to the repo you just created.

```
cd public
git init
git add .
git commit -m "first commit"
git remote add origin https://github.com/hugoAuthor/hugoAuthor.github.io.git
git push origin master
```

Open the website of your repo then wait until you see `active` on `Environments`

![image-20220117224010950](pic/image-20220117224010950.png "environments")

Now you can see your blog on  `hugoAuthor.github.io`


