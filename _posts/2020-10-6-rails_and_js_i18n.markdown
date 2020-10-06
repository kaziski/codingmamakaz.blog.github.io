---
layout: post
title:      "Internationalization with Rails and i18n-js"
date:       2019-05-10 23:49:43 +0000
permalink:  happy_trails
---

According to Wikipedia, internationalization (shorthand as i18n as there are exactly eighteen characters between the first “i” and the last “n”) is the process of designing a software application so that it can be adapted to various languages and regions without engineering changes.

I was working on an app built with React on Rails. Most of the pages were written in React, but some pages were written in and rendered from the Rails views. This meant that I needed to have i18n on the backend and also send translated strings to the React frontend. 
I looked for articles and tutorials, but I couldn’t find one that was solving the same problem I had. I found a lot of tutorials on i18n-ing Rails app, and some on Rails and Javascript, but not on Rails and React. </br>
After spending some time researching, I decided to try [rails-i18n](https://github.com/svenfuchs/rails-i18n) and [i18n-js](https://github.com/fnando/i18n-js) gems.

`rails-i18n` provides an easy-to-use and extensible framework for translating your app. `i18n-js` is a small library to generate the Rails i18n translations on the JavaScript side. It provides middleware that auto-generates translated Javascript files from the yml files on the Rails side. 
These seemed like a good way to solve the problem I had.</br>
I would like to share how I did it. 

First, add gems to your Gemfile.
```
gem "i18n-js"
gem "rails-i18n"
```

Don't forget to install gems by running
`bundle install`

If you are using webpacker, run
`npm install i18n-js`</br>
If you are using asset pipeline, check out [this ReadMe](https://github.com/fnando/i18n-js).

Define available locales in `config/application.rb`. In my case, I'm adding support for English and Japanese. <br />
`config.i18n.available_locales = [:ja, :en]`

You can check all the locales available in the Rails console <br />
```
I18n::JS.filtered_translations.keys
=> [:en, :ja]
```

Now add English strings in `en.yml`
```
en:
  hello: "Hello world"
```
You can check to see if it's working in the rails console like so  
```
I18n.t 'hello'
=> "Hello world"
```
Now let's add Japanese translation in `ja.yml`
```
ja:
  hello: "こんにちは。世界。"
```
This is how we can access the translation in a slim file.
`h1 = t('hello')`

Now we got it working on the Rails side, let's move on to getting the translation on the React side.

Add a middleware by adding this line `config.middleware.use I18n::JS::Middleware` to `config/application.rb`<br />
The middleware generates translation files on the frontend.

Run `rails generate i18n:js:config`, which will generate `config/i18n-js.yml`

This is my configuration added to  `config/i18n-js.yml`
```
translations:
- file: 'app/javascript/bundles/i18n/en.js'
  prefix: "import I18n from 'i18n-js';\n"
  pretty_print: true
  only: 'en.*'
- file: 'app/javascript/bundles/i18n/ja.js'
  prefix: "import I18n from 'i18n-js';\n"
  pretty_print: true
  only: 'ja.*'
```
`file` is specifying the path of my choice for js translation file.<br />
`prefix` is optional, but without it, I was getting "I18n is not defined" error, and I couldn’t get it to work. It will add the line at the beginning of the resultant translation file. <br />
`pretty_print` is optional as well, but I definitely recommend putting this. It adds whitespace and indentation in your output file, which makes it so much easier to read the file.

To export all translation files and every time you have new translation in the .yml file, run `rake i18n:js:export`.</br>
This will generate translation files to your chosen paths.

Add this to a react file `import i18n from 'i18n-js'`

Also remember to import your generated translation files.
```
import en from '../../i18n/en';
import ja from '../../i18n/ja';
```
To set default locale and locale on the backend, I added this in `views/layouts/application.slim`

```
- javascript_tag do
      I18n.locale = I18n.locale
      I18n.defaultLocale = I18n.default_locale
```      
To set them on the React side, you can add this inside the render (this set them to be in Japanese) 

```
I18n.defaultLocale = "ja";
I18n.locale = "ja";
```
**ja needs to be a string like `"ja"`**

      
Finally, add we can access your Rails translations from React like so. 
` <h2>{I18n.t('hello')}</h2>`   
![English hello world](https://res.cloudinary.com/codingmamakaz/image/upload/v1601997665/Screen_Shot_2020-10-06_at_11.18.31_AM_seh2hd.png)
![Japanese hello world](https://res.cloudinary.com/codingmamakaz/image/upload/v1601997665/Screen_Shot_2020-10-06_at_11.14.36_AM_izs69h.png)

The left side is English and the right side is "hello world" in Japanese. As you can see, some of the translation's strings might be longer, which may cause you a lot of headaches dealing with css 😱

I hope this post helps someone. </br>
Happy i18n-ing!
