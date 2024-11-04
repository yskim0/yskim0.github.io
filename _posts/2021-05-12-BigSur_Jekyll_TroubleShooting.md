---
title: Mac OS BigSur 11.2.3 "bundle exec jekyll serve" Error
categories:
 - Troubleshooting
tags: 
 - jekyll
---

How to Solve "Mac OS BigSur 11.2.3 `bundle exec jekyll serve` Error"

---

# Problem

## 1. Problem Recognition

![image](https://user-images.githubusercontent.com/48315997/117841944-6dc0a380-b2b8-11eb-8ff4-43268452b53e.png)


평소와 같이 github 블로그 글을 쓰고 `bundle exec jekyll serve` 를 터미널에 입력했는데 아래와 같은 에러가 뜸

> Could not find commonmarker-0.17.13 in any of the sources.  
Run `bundle install` to install missing gems.



## 2. To solve #1., `bundle install` or install commonmarker directly... But :(

I didn't screenshot these commands' error message... but if you get error message like 

![image](https://user-images.githubusercontent.com/48315997/117842511-f8090780-b2b8-11eb-9237-f48769da9548.png)

> (Gem::FilePermissionError)  
    You don't have write permissions for the /Library/Ruby/Gems/2.X.0 directory.

Then, it must be installed **rbenv ruby-build**.


# Solution
## 1. brew install rbenv ruby-build
If you installed sucessfully, then go to the `number 3`.

But if you got another problem like me...

![image](https://user-images.githubusercontent.com/48315997/117843531-dbb99a80-b2b9-11eb-831b-12e75ffd770b.png)

> Error: Your CLT does not support macOS 11.   
It is either outdated or was modified.   
Please update your CLT or delete it if no updates are available.   
Update them from Software Update in System Preferences or run:  
...

It is probably the effect of the **Big Sur update**.

So I solved this problem by ...


![image](https://user-images.githubusercontent.com/48315997/117843903-34893300-b2ba-11eb-95f5-7ab80b0c42f2.png)

> sudo rm -rf /Library/Developer/CommandLineTools  
sudo xcode-select --install


ref)
- [https://apple.stackexchange.com/questions/401899/-homebrew-your-clt-does-not-support-macos-11-0](https://apple.stackexchange.com/questions/401899/-homebrew-your-clt-does-not-support-macos-11-0)
- [https://flaviocopes.com/how-to-fix-clt-support-macos-11/](https://flaviocopes.com/how-to-fix-clt-support-macos-11/)
    - 여기에있는 사진들이 정상적으로 나오면 OK인듯.


## 2. Follow [this](https://popcorn16.tistory.com/56) blog's steps(~source ~/.zshrc)

## 3. bundle install

Finally, I installed commonmarker successfully.

## 4. bundle exec jekyll serve

Finish!

---
-	어느 단계인지 기억은 안나는데 Gemfile에 
`gem 'commonmarker'`
를 추가하기는 했음. (이게 필수인지는… 순서가 뒤죽박죽이라 모르겠음, 근데 bundle instasll로 다 되지 않았을까 싶음. Bundle exec Jekyll serve 를 했을 때 1번과 같은 에러가 뜬다면 그 때 Gemfile에 써도 늦지 않을 듯.)
