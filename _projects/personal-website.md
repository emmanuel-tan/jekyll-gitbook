---
title: Personal website
author: Emmanuel Tan
date: 2025-05-27
layout: post
---

Time taken to complete: 48 hours

I go through the steps I followed, my experience through the process, and what I learned and would do differently. 

Reasons for making this: 
- I think it's nice to have a place to put up things you have worked on. I have often been inspired by looking at researcher's pages and reading their projects and sometimes there are even nice blog posts that share their thoughts. 
- Frankly, this is also for visibility.
- I wanted to see how hard it would be and how long this would take me.

How I set about making this: 
- A few weeks back, I actually went way deeper into this and made a website using Vite and React and Tailwind whatnot. It was fun but I wasn't personally happy with the results and though I could have kept going back to reiterate, I decided to call it quits and just make something simpler with a simple jekyll theme so I could actually spend time on the projects I was more interested in. 
- The basic steps I had in mine were as so: 
    1) Identify a jekyll theme 
    2) Set it up to run locally
    3) Set it up to be hosted on netlify
    4) Set up Github actions to build and deploy to netlify (not required but I wanted to dabble)
    5) Populate with my information and content 
    6) Link it to my domain name
    <!-- 7) Optional: set up Google analytics -->

Since you're reading this, it means I succeeded. So let's dive in! 

### 1) Identify a jekyll theme 
This comes down to personal preference, I didn't want to get boggled up so I selected something that looked simple and familiar and landed on what you are currently looking at, a Gitbook theme. I know it's typically used for documentation, but the beauty of themes is I'd be able to switch if I like. It can be found on [jekyllthemes website](http://jekyllthemes.org/themes/gitbook/) or on its own [Github repo](https://github.com/sighingnow/jekyll-gitbook). 

Side note: after I had set up things locally and with netlify and Github actions, did I discover Hugo. As I'm not from a front-end background, I wasn't very well versed in all these different options. I may consider switching to Hugo (just because it seems faster) if a specific theme catches my eye and the switching efforts aren't too high. But I was determined not to get side-tracked. 

### 2) Set it up to run locally
This entire process (following steps included) are pretty standard regardless of the theme selected. Two things needed are 1) Jekyll needs to be installed and 2) You need to build from a theme.

The [website](https://jekyllrb.com/docs/installation/) walks you through installing Ruby and Jekyll nicely. This isn't my area of expertise so I won't comment too much, and the process went smoothly for me. I believe `bundler` is required as well, which I installed with `gem intall bundler`. 


After Jekyll is installed, I needed to fork the repo of the theme I wanted and then download it to my computer.
ie 
```
git clone https://github.com/sighingnow/jekyll-gitbook.git
```
except the link is replaced with your forked repo. 
[Github theme repo](https://github.com/sighingnow/jekyll-gitbook)

Now, inside the directory of the repo, the theme should contain a file that contains the dependencies needed. To install them I ran `bundle install`. 

The last step was to build and serve the site to be accessed locally. 

Running `bundle exec jekyll build` just builds it but doesn't serve it. The built static files go into the _site folder. 

Running `bundle exec jekyll serve` builds it and also starts up the server, which very nicely allows me to look at whether my markdown is showing up how I like. But since at this point I hadn't put in any content yet, it's just leftover template stuff from the theme. 

### 3) Set it up to be hosted on Netlify
I'm sure you could use any hosting software, but I went with Netlify just because I had used it before. To set things up I simply
1) Created a new project and give it a name
2) Linked it to my Github repo
3) Configured the build settings
    - publish directory should be set to "_site"
    - build command is `bundle exec jekyll build`

And that's about it! Everything built successfully and the site was up live. 

### 4) Set up Github actions to build and deploy to netlify (not required but I wanted to dabble)
Now, this step is optional, but I recently finished a course on Github which covered Github actions and so I wanted to try a bit of CI/CD with this very simple project. 
The main goal of using Actions here is to have Github build the site by itself when a new commit is made to the main branch, and it should also display each step of the build process so I can catch if something goes wrong. Technically, Netlify and Jekyll alone is fine, and Github Actions is probably only really useful if I am doing some fancy pre-processing or complex build process. But, even though I'm not, I can still DevOps-ify this process for learning purposes.

To set up Github actions, I created a directory to store the config file for my Action (.github/workflows/deploy.yml). 

The `deploy.yml` file starts with the name of the Action and the trigger. 

```
name: Build and Deploy Jekyll Site to Netlify

on:
  push:
    branches: [master]
```
Quite straightforward, although I ran into some hiccup with the Action not triggering at first. This was because this theme I'm using is a little older and the main branch of the repo was titled "master" and my initial trigger was set to run when a new commit was pushed to "main." A simple error but one that was enough to prevent the whole workflow from running. 

Next, the file specifies the steps to running the only job this Action has to do. 

```
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        
      - name: Set up Ruby
        uses: ruby/setup-ruby@v1
        with:
          ruby-version: 3.2
          bundler-cache: true
          
      - name: Install Dependencies
        run: | 
          gem install bundler
          bundle install
          
      - name: Build Jekyll Site
        run: bundle exec jekyll build
```
Simple enough, we checkout the code, set up ruby so that it has all the libraries needed, install the dependencies using gem and `bundle install`, and then build the site using `bundle exec jekyll build`. The steps aren't really any different from how they were set up to run locally, but they're just laid out sequentially with a nice name to it for Github to show us later on. 

The final step is to deploy the build site to Netlify. I used the [Action on the Github Actions Marketplace](https://github.com/marketplace/actions/netlify-actions) that does exactly what we need (there is a [netlify-cli](https://github.com/marketplace/actions/netlify-cli) Action that should work, although it seemed a little clunkier as you're just feeding it CLI commands). To use this Action, we do need to put our Netlify Authentication Token and the site ID into the repo as secret so that it can access them. (If you are trying to replicate this, please DO NOT just put your site ID and authentication token into the yml file). 

```
      - name: Deploy to Netlify
        uses: nwtgck/actions-netlify@v3.0
        with:
          publish-dir: './_site'
          production-branch: master
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
          enable-pull-request-comment: false
          enable-commit-comment: true
          overwrites-pull-request-comment: true
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}

```

I saved the deploy.yml file and pushed it to the repo and did a few test commits to see if the workflow would run and the site remains hosted and it reflects updates. Now I have a full (but very simple) CI/CD workflow!

### 5) Populate with my information and content 
Now at this point, I didn't actually want to leave the site hosted online so I took it down. To keep my repo a bit cleaner with fewer commits, I would edit the content locally and serve it locally and only commit a new repo when a page was complete. 

To be honest, this step probably took me the longest because I was also doing this writeup of how I set everything up, but the rest of the site is and will probably remain rather bare other than basic info. Like I mentioned, I just want a place to write about small projects like this.

I did however struggle a little bit with the file structure (which is on me as I skipped on reading the Jekyll documentation). 

### 6) Link it to my domain name
If you've already got a domain name, Netlify lets you link it. I didn't already, and they also let you buy a domain name through them. Frankly, their price was cheaper than elsewhere, so I went with Netlify's price. I am definitely not an expert on this, but the process was smooth and I don't anticipate any issues unless I intend to switch from Netlify at some point. 
Of course, there is always the option to let this run for free on Netlify. I just thought it was nice to have a personal domain and they offered a decent price for it. 

<!-- ### 7) Optional: set up Google analytics and SEO basics -->

<!-- ## Wrapping up
There you go!  -->
